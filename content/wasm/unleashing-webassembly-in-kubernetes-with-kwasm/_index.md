+++
title = 'Unleashing WebAssembly in Kubernetes with Kwasm'
date = 2024-06-10T16:00:00+00:00
draft = false
+++

I am going to show you something you've seen many times before. Yet, looks can be deceiving. You will likely come to a wrong conclusion. What I will show is not what it might look like.

Here it goes.

<!--more-->

{{< youtube oY9le4DDAOY >}}

> Do NOT run the command that follows. It is only a preview. We'll set it all up later.

```sh
cat app.yaml
```

The output is as follows (truncated for brevity).

```yaml
---
apiVersion: apps/v1                                                  
kind: Deployment
metadata:
  name: silly-demo
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: silly-demo
  template:
    metadata:
      labels:
        app.kubernetes.io/name: silly-demo
    spec:
      runtimeClassName: wasmtime-spin-v2
      containers:
      - name: main
        image: ttl.sh/kwasm-demo:v0.0.1
        command: ["/"]
---
apiVersion: v1
kind: Service
...
---
apiVersion: networking.k8s.io/v1
kind: Ingress
...
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
...
```

<img src="/logo/wasm.png" style="width:30%; float:right; padding: 10px">

This is the most typical and, arguably, one of the simplest sets of Kubernetes manifests one can imagine. There is a `Deployment` which will, ultimately, spin up Pods containing a single container called `main`. There is a `Service` that enables communication to the Pods, an `Ingress` that enables incoming traffic from outside the cluster, and a `HorizonalPodAutoscaler` that scales the replicas, the Pods. The only "special" thing about that explanation is that a part of it is not true. That Deployment will not spin up containers but, rather, it will run [WASM](https://webassembly.org) binaries.

That solves the biggest issue we're facing with WASM. That makes WASM useful. That elevates it to a completely new level. It changes my reaction to WASM from "Interesting, yet I don't know what to do with it" to "Heck yeah!"

**WASM is supperior to containers**. WASM programs run at nearly native speed, they are smaller in size than their container counterparts, they start significantly faster, they are more secure, and so on and so forth.

Yet, WASM did not see a wide adoption. Why is that? If WASM is superior to containers, why didn't we switch to it already? The answer is simple. It's not about containers. The question is not whether WASM is better or worse than containers. The questions is whether WASM has the ecosystem needed to provide everything we need to run apps. **It's about the ecosystem**.

<img src="/logo/cncf.png" style="width:40%; float:right; padding: 10px">

We run containers in Kubernetes and that means that the whole Kubernetes ecosystem is at our disposal. We can use Kubernetes scheduler to distribute containers across a fleet of nodes. We can use Prometheus, Jaeger, Loki, and Grafana to observe what's going on with those containers running in a Kubernetes cluster. We can leverage Cilium for networking, Argo CD or Flux for GitOps, Kyverno or OPA Gatekeeper for policies, and so on and so forth. Most of the projects in the [CNCF](https://cncf.io) ecosystem are designed to be Kubernetes-native meaning that we can pick and choose any of them and they will all work seamesly with each other. CNCF alone contains around two hundred projects, and that's only a fraction of what we have at our disposal. There's an endless stream of projects and serviecs that are not in CNCF. That's what makes Kubernetes so amazing. It is **the biggest ecosystem** anyone ever saw and it is **the standard**. Kubernetes runs anywhere and is offered by everyone.

![](/logo/fermyon.png?height=5vw&classes=inline)
![](/logo/aws.png?height=5vw&classes=inline)
![](/logo/google-cloud.png?height=8vw&classes=inline)
![](/logo/azure.png?height=5vw&classes=inline)
![](/logo/civo.svg?height=5vw&classes=inline)
![](/logo/digital-ocean.png?height=8vw&classes=inline)

WASM's ecosystem, on the other hand, feels **insignificant**. Anyone wanting to deploy, observe, and manage applications as WASM has to deal with almost non-existent ecosystem. We can use [Fermyon Cloud](https://fermyon.com/cloud) to run WASM but how can that be compared with AWS, Google Cloud, Azure, CIVO, Digital Ocean, and almost any other cloud provider that offers endless number of services designed to run containers. We can run WASM on a single server using Fermyon Spin or even Docker itself, but that cannot be compared with the power Kubernetes brings to the table.

Even though WASM is, arguably, a superior way to run applications, containers, mostly through Kubernetes, have the ecosystem that WASM doesn't. Hence, there are only two paths forward.

WASM can **create its own ecosystem**.

That would be fools errand. It seems very unlikely WASM's ecosystem alone will be able to reach even ten percent of the scope of Kubernetes, meaning that the second path is the only reasonable one.

That second path is to **enable Kubernetes to run WASM** in the same way as it runs containers today. If we could do that, we would leverage all the power and extensibility Kubernetes gives us, yet with lighter, faster, and more secure runtime than containers.

That is certainly possible today.

Anyone can install WASM runtime on the nodes, configure [containerd](https://containerd.io) with Spin or WasmEdge shims, and do whatever else needs to be done. However, that is annoyingly complicated, brittle, and error prone. I don't want to do any of that. Instead, I would like to have a Kubernetes operator that will do whatever needs to be done to enable Kubernetes to treat WASM applications in the same way as it is treating containers.

<img src="/logo/kwasm.svg" style="width:30%; float:right; padding: 10px">

That's what [Kwasm](https://kwasm.sh) give us.

Kwasm is a Kubernetes operator that adds WebAssembly support to Kubernetes nodes. It is, probably, the simplest way to do whatever needs to be done in your Kubernetes clusters so that they can run WASM in (almost) the same way they are running containers.

Today, we'll explore three important aspects of running WASM in Kubernetes.

We'll see how we can enable Kubernetes clusters to run WASM. Then we'll explore how to package WASM into container images. Finally, we'll see what do we need to change, if anything, in our Kubernetes manifests so that we're running applications as WebAssembly instead of containers.

Let's go.

## Setup

```sh
git clone https://github.com/vfarcic/kwasm-demo

cd kwasm-demo
```

> Make sure that Docker is up-and-running. We'll use it to create a KinD cluster.

> Watch https://youtu.be/WiFLtcBvGMU if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell

kind create cluster --config kind.yaml

kubectl apply \
    --filename https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

sleep 15 # Wait until NGINX Ingress resources are created

kubectl wait --namespace ingress-nginx \
    --for=condition=ready pod \
    --selector=app.kubernetes.io/component=controller \
    --timeout=90s

kubectl create namespace a-team
```

## Enable WASM in Kubernetes with Kwasm Operator

Kwasm makes installation of whatever is needed to run WASM in Kubenetes a trivial operation that consists of three steps.

First, we need to install the Kwasm operator.

```sh
helm upgrade --install kwasm-operator kwasm-operator \
    --repo http://kwasm.sh/kwasm-operator \
    --namespace kwasm --create-namespace --wait
```

That operator does not do anything. It just waits for us to tell it which nodes should be enabled to run WASM applications. We do that by annotating nodes with `kwasm-node` annotation. That gives us the flexibility to choose nodes from a specific node pool, or individual nodes or, as I'm going to do today, `--all` nodes of a cluster.

```sh
kubectl annotate node kwasm.sh/kwasm-node=true --all
```

The Kwasm operator is watching for that annotation and, once it detects that a node got it, it spins up a Job on that node. The Job, in turn, run a one-shot priviledged Pod that installed WebAssembly runtime on the annotated node and did whatever else needs to be done, except for one thing.

We still need to do one more thing. We need to apply a new RuntimeClass which will tell Kubernetes to use the runtime we just installed. Kwasm supports [WasmEdge](https://wasmedge.org) and [Fermyon Spin](https://fermyon.com/spin). We'll go with the latter.

Here's the manifest.

```sh
cat spin.yaml
```

The output is as follows.

```yaml
---
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: wasmtime-spin-v2
handler: spin
```

There's not much here to comment on. There's a name which we'll use in a moment and a handler that, as the name suggests, reference which runtime should handle workloads.

Let's apply it.

```sh
kubectl apply --filename spin.yaml
```

That's it. The cluster is now ready to work with WebAssembly applications just as if they are containers.

Let's move on an see how to package WASM applications.

## Package WASM Applications as Container (OCI) Images with Fermyon Spin

Let's take a look at the code of a silly demo we'll run in the cluster.

```sh
cat main.go
```

The output is as follows.

```go
package main

import (
        "fmt"
        "log"
        "net/http"
        "os"

        spinhttp "github.com/fermyon/spin/sdk/go/v2/http"
)

func init() {
        spinhttp.Handle(func(w http.ResponseWriter, r *http.Request) {
                w.Header().Set("Content-Type", "text/plain")
                fmt.Fprintln(w, "This is a silly demo running as WASM!")
                logger := log.New(os.Stderr, "", log.LstdFlags)
                logger.Println("This is a silly demo running as WASM!")
        })
}

func main() {}
```

It should not matter whether you're used to write Go code or something else. There's obviously not much going on there, except that we're using Spin `http` library to handle requests and spitting out a response message (`This is a silly demo running as WASM!`).

Now, to compile a Spin app into WASM, we need a TOML config.

```sh
cat spin.toml
```

The output is as follows.

```toml
spin_manifest_version = 2

[application]
name = "silly-demo"
version = "0.1.0"
authors = ["Viktor Farcic <viktor@farcic.com>"]
description = ""

[[trigger.http]]
route = "/..."
component = "silly-demo"

[component.silly-demo]
source = "main.wasm"
allowed_outbound_hosts = []
[component.silly-demo.build]
command = "tinygo build -target=wasi -gc=leaking -no-debug -o main.wasm main.go"
watch = ["**/*.go", "go.mod"]
```

This is where we specify `application` metadata, how to `trigger.http` requests, and how to `build` it. Think of that file as Spin's equivalent to Multi-Stage Dockerfile.

Next, we can execute `spin build` to build a WASM binary,...

```sh
spin build
```

...and convert it into a container image and push it to a registry.

```sh
spin registry push ttl.sh/kwasm-demo:v0.0.1
```

The last part is important. We packaged WASM into a container image, yet we are not going to run it as containers.

I feel it's important to make the distinction between container or [OCI](https://opencontainers.org) images and containers themselves. Container images are a packaging mechanism. They are, in a way, a replacement for zip and tar formats. It's just a package. What makes it special is that it is a standard package. Everyone and everything expects software to be packaged as container images. That, however, does not mean that those images will run as containers. Just as there is no certainty who or what will extract a ZIP file, there is no certainty who or what will extract a container image. It's just a package and it's up to a runtime to decide what to do with it. A container runtime might convert an image into containers while a WASM runtime might extract WASM binary from it.

The previous command not only created a container image but it also pushed it to the [ttl.sh](https://ttl.sh). All that's left is to see how we can run that container image that contains a WASM application in a Kubernetes cluster.

## Running WASM Through "Standard" Kubernetes Resources

Let's take another look at application manifests.

```sh
cat app.yaml
```

The output is as follows (truncated for brevity).

```yaml
---
apiVersion: apps/v1                                                  
kind: Deployment
metadata:
  name: silly-demo
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: silly-demo
  template:
    metadata:
      labels:
        app.kubernetes.io/name: silly-demo
    spec:
      runtimeClassName: wasmtime-spin-v2
      containers:
      - name: main
        image: ttl.sh/kwasm-demo:v0.0.1
        command: ["/"]
---
apiVersion: v1
kind: Service
...
---
apiVersion: networking.k8s.io/v1
kind: Ingress
...
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
...
```

Those are Kubernetes manifests I used in the past with an application that is running as containers. There is a `Deployment`, a `Service`, an `Ingress`, and a `HorizontalPodAutoscaler`. There's nothing special about those, except for a single line.

The `spec` entry inside the `template` that creates the Pods has `runtimeClassName` set to `wasmtime-spin-v2`. It instruct Kubernetes to use a runtime class other than the default one. That's it. That's the only change I did to my manifests. The rest of the Deployment is the same. Even though it'll run as WASM instead of containers, we can still have a Service that enables internal communication to it, an Ingress that enables external communication, HorizontalPodAutoscaler that will scale it up and down, or almost anything else you'd normally do in Kubernetes. It works as it's supposed to work, but with a different runtime.

I could, for example, push that manifest to a Git repo and let Flux or Argo CD synchronize it. I won't do that, mostly to avoid expanding the scope of this article too much. Instead, we'll apply it...

```sh
kubectl --namespace a-team apply --filename app.yaml
```

...and retrieve `all` the resources, including `ingresses`.

```sh
kubectl --namespace a-team get all,ingresses
```

The output is as follows.

```
NAME                            READY STATUS  RESTARTS AGE
pod/silly-demo-564cc7645c-mqn4w 1/1   Running 0        9s
pod/silly-demo-564cc7645c-mz4lr 1/1   Running 0        24s

NAME               TYPE      CLUSTER-IP   EXTERNAL-IP PORT(S) AGE
service/silly-demo ClusterIP 10.96.94.173 <none>      80/TCP  24s

NAME                       READY UP-TO-DATE AVAILABLE AGE
deployment.apps/silly-demo 2/2   2          2         24s

NAME                                  DESIRED CURRENT READY AGE
replicaset.apps/silly-demo-564cc7645c 2       2       2     24s

NAME                                           REFERENCE             TARGETS                      MINPODS MAXPODS REPLICAS AGE
horizontalpodautoscaler.autoscaling/silly-demo Deployment/silly-demo <unknown>/80%, <unknown>/80% 2       10      1        24s

NAME                                 CLASS HOSTS                       ADDRESS PORTS AGE
ingress.networking.k8s.io/silly-demo nginx silly-demo.127.0.0.1.nip.io         80    24s
```

All the resources were created and if you saw only that output you would never have guessed that those Pods are not running containers but WASM.

As a proof that everything works, we can send a request to the Ingress address...

```sh
curl "http://silly-demo.127.0.0.1.nip.io"
```

...and observe a silly output.

```
This is a silly demo running as WASM!
```

That's it. That's all there is to it, so let's talk about pros and cons.

## WASM, Kwasm Pros and Cons

There are quite a few question we might need to answer. Should you use WASM? If you should, which runtime should you choose? Where should you run it? I could probably come up dozens of questions and doubts. I won't go through all of them, partly because I answered some of them in [WASM vs Docker Containers vs Kubernetes vs Serverless: The Battle for Cloud Native Supremacy](https://youtu.be/uZ8xI26sno8).

The short version is that WASM is a valid **replacement for containers**, but it has **no ecosystem** that would make it useful. **Kubernetes is the ecosystem** and if we can run WASM in Kubernetes clusters in a way that is compatible with everything else we do in those clusters, WASM changes from "Not good enough" to "**Heck, yeah!**". WASM in Kubernetes is the type of solution we were waiting for. It's what was required for WASM to thrive.

We could, for example, use Knative to run serverless applications as WASM. That would increase security, startup time, and performance, while, at the same time, reducing the size. Or we could run it as "standard" Kubernetes Deployments or StatefulSets or as anything else. We can use whichever method we have to eventually run Pods and those Pods can be managing WASM applications instead of containers. That's amazing!

The ability to simply change the runtime class in Pods is the **game changer**, and we had that for a while now. What was missing is provider's adoption of WASM. At the time of this writing, only Azure AKS supports WASM out of the box. For everyone else, we need to go through the pain of setting everything up ourselves.

That's where Kwasm jump in. **Kwasm simplifies WASM setup in Kubernetes clusters**. With it, we can run WASM in almost any Kubernetes cluster. It works with Azure AKS, Google GKE, AWS EKS, Digital Ocean, CIVO, KinD, Minikube, and MicroK8s. That's the list of distributions that were tested and it should work almost anywhere else as well. The only ones that are known not to work with Kwasm are Oracle OKE and OpenShift, and GKE is limited to Ubuntu while it works in EKS if it's based on Ubuntu or Amazon Linux.

There's not much bad I can say about Kwasm.

It's a hack, of sorts, and that's what it should be. It is not a project that is supposed to exist for long. WASM will soon be available in majority of Kubernetes distributions out of the box, just as it is today in Azure AKS. When that happens, there will be no reason for the existence of Kwasm. But, until that happens, **Kwasm is amazing**. It simplifies a tedious process. It is a setup engine that installs and configures whatever is needed to run WASM on some or all nodes of a cluster.

The documentation is almost non-existent, and that's fine since there's not much to it.

It uses privileged containers which, under other circumstances should be illegal, but that's the only way it could access nodes to install the needed binaries. If we'd do the setup ourselves, we'd need to run priviledged pods as well or to SSH into nodes.

There are some limitations specific to WASM, but that's not something we can blame Kwasm. As a matter of fact, that's not WASM's fault either since most of those limitations are intentional for security purposes.

Depending on which WASM runtime class we choose, we might not be able to mix WASM and containers in the same Pod. Hence, sidecar containers attached to WASM might not work, thus making solutions like Istio and Linkerd useless. That's probably the biggest downside of WASM in Kubernetes, which should not be a big deal since we should be moving away from side-cars anyways. Besides, that's not a problem of Kwasm either, but of WASM runtime classes.

All in all, Kwasm is an amazing little project that enables us to **setup WASM in Kubernetes** without going crazy. It allows us to have a glimpse into what will probably become a capability baked into managed Kubernetes clusters.

## Destroy

```sh
kind delete cluster

exit
```

+++
title = 'Mastering Kubernetes Debugging: Leveraging eBPF with Inspektor Gadget'
date = 2024-05-20T16:00:00+00:00
draft = false
+++

Kubernetes ecosystem is one of the most, if not the most extensive we've ever seen. There are tools for everything, including observability. We can collect metrics, logs, and traces for almost anything, we can query them, and we can see them in dashboards. There are hundreds of solutions for that alone, yet, sometimes, I miss simplicity of tools I would normally use in Linux. Sometimes, I crave for **simple commands** similar to those I would use when trying to figure out what's going on in a single server.

...and a few others.
<!--more-->

{{< youtube 6cwb3xNcqqI >}}

Sometimes I miss something similar to, let's say, `netstat`...

```sh
netstat
```

The output is as follows.

```
Active Internet connections
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)    
tcp4       0      0  10.20.1.111.54139      mad41s08-in-f10..https ESTABLISHED
tcp4       0      0  10.20.1.111.54135      mad07s10-in-f3.1.https ESTABLISHED
tcp4       0      0  10.20.1.111.54134      mad07s09-in-f3.1.https ESTABLISHED
...
```

...that provides statistics about all active connections.

Or `ps`....

```sh
ps aux | grep VSCode
```

The output is as follows.

```
vfarcic 54770 0.0  0.0 410755984 5360   ?? S< Thu08PM 0:00.03 /Applications/...
vfarcic  1403 0.0  0.0 443745824 1824   ?? S  15Mar24 0:01.61 /Applications...
vfarcic 83185 0.0  0.0 410072768  912 s070 R+ 10:15PM 0:00.00 grep --color=...
```

...that outputs all active processes.

![](/logo/prometheus.png?height=15vw&classes=inline)
![](/logo/grafana.png?height=15vw&classes=inline)
![](/logo/loki.png?height=15vw&classes=inline)

In other words, sometimes I miss the ability to inspect what's going on in a cluster without going into tools like Prometheus, Grafana, Loki, and others. I miss the simplicity of micro-CLI commands.

Now, to be clear, I can do a lot with `kubectl` alone. I can list all the Pods or any other type of resources. I can see events, I can see logs, and many other things. However, they are at a higher level than, for example, `ps` that lists processes.

That is understandable since `kubectl` knows what Kubernetes API knows, and that's great, but if I would like to dive deeper and see what's happening at lower levels, I would need to enter into one of the nodes and start digging in. That's a bad idea for many reasons. To begin with, there can be many nodes in a cluster and I have no intention going through each of the separately. As a matter of fact, I often do not even have SSH access to individual nodes. Why would I?

So I'm in a difficult situation. I might need to see what's happening a lower level, at the **kernel level**, but I do not want to or even cannot access nodes of a cluster, at least not directly.

What I would need is to inject **Kubernetes-aware yet lower level processes** into my cluster with the goal for them to give me the information that I normally cannot get through Kubernetes API.

<img src="/logo/ebpf.png" style="width:40%; float:right; padding: 10px">

The only sensible direction to get there would be eBPF that allows us to safely "inject" processes into Kernel. eBPF should be able to see **which processes are running**, **what's moving through the network**, **which files are used**, and so on.

Now, to be clear, we already have tools that leverage eBPF for that purpose, but they are often exposing information through a Web UI.

That's great, most of the time but, as I already mentioned, sometimes I would like the simplicity of micro-commands typically available in Linux.

I want quick and dirty way to do in Kubernetes what I can do on Linux, yet I want that something to be Kubernetes-aware.

To make my quest even more difficult, I'd like such tools to be available in one place instead of me having to "hunt them down" one by one.

I found such a tool and I must say that I'm embarressed that I was not aware of it until recently. Let's take a look at it.

## Setup

```sh
git clone https://github.com/vfarcic/inspektor-gadget-demo

cd inspektor-gadget-demo
```

> Watch https://youtu.be/WiFLtcBvGMU if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell
```

> Make sure that Docker is up-and-running. We'll use it to create a Kind cluster. Feel free to skip creating the `kind` cluster and installing NGINX Ingress if you already have a cluster with an Ingress controller.

```sh
kind create cluster --config kind.yaml

kubectl apply \
    --filename https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

kubectl --namespace ingress-nginx wait \
    --for condition=Available deployment ingress-nginx-controller

kubectl create namespace a-team
```

> Replace `127.0.0.1` with the IP of the Ingress service if you chose to use your own cluster with Ingress.

```sh
export INGRESS_HOST=127.0.0.1
```

> Replace `nginx` with the IP of the Ingress service if you chose to use your own cluster with Ingress.

```sh
export INGRESS_CLASS=$(kubectl get ingressclasses \
    --output jsonpath="{.items[0].metadata.name}")

yq --inplace ".spec.ingressClassName = \"$INGRESS_CLASS\"" \
    silly-demo/ingress.yaml
    
yq --inplace \
    ".spec.rules[0].host = \"silly-demo.$INGRESS_HOST.nip.io\"" \
    silly-demo/ingress.yaml

yq --inplace ".spec.ingressClassName = \"$INGRESS_CLASS\"" \
    pinger/ingress.yaml
    
yq --inplace \
    ".spec.rules[0].host = \"pinger.$INGRESS_HOST.nip.io\"" \
    pinger/ingress.yaml

kubectl --namespace a-team apply --filename silly-demo/

kubectl --namespace a-team apply --filename pinger/

kubectl --namespace a-team apply --filename caw/
```

## Inspect Kubernetes with Inspektor Gadget

The tool in question is Inspektop Gadget which can be used as a kubectl addon. The first thing we should do is... deploy it to a Kubernetes cluster.

```sh
kubectl gadget deploy
```

```
Creating Namespace/gadget...
Creating ServiceAccount/gadget...
Creating ClusterRole/gadget-cluster-role...
Creating ClusterRoleBinding/gadget-cluster-role-binding...
Creating Role/gadget-role...
Creating RoleBinding/gadget-role-binding...
Creating DaemonSet/gadget...
Creating CustomResourceDefinition/traces.gadget.kinvolk.io...
Waiting for gadget pod(s) to be ready...
1/1 gadget pod(s) ready
Retrieving Gadget Catalog...
Inspektor Gadget successfully deployed
```

![](/logo/argo-cd.png?height=10vw&classes=inline)
![](/logo/flux.png?height=10vw&classes=inline)
![](/logo/artifact-hub.png?height=7vw&classes=inline)

It's nice that a tool can be deployed with a single command, when using a demo like this one. But, when it comes to a serious usage, I would expect it to be able to output manifests so that I could store them in Git and synchronize them with Argo CD, Flux, or whichever other GitOps tool you might have chosen by mistake. It could also be a Helm chart and, if you search for it the ArtifactHub, you'll find it. So, you might say "Problem solved. Helm charts work with GitOps tools." That would be true. However, that Helm chart is not mentioned anywhere in the docs. It's not in the installation instructions nor anywhere else so I can only assume that chart is a result of some whichcraft, or that the community does not care about it, or that they don't think anyone should use it. Otherwise, it would be mentioned at least once in the docs. Right?

Anyways... Let's move on and take a look at what we can do with the kubectl addon.

```sh
kubectl gadget --help
```

The output is as follows.

```
Collection of gadgets for Kubernetes developers

Usage:
  kubectl-gadget [command]

Available Commands:
  advise      Recommend system configurations based on collected information
  audit       Audit a subsystem
  completion  Generate the autocompletion script for the specified shell
  deploy      Deploy Inspektor Gadget on the cluster
  help        Help about any command
  profile     Profile different subsystems
  prometheus  Expose metrics using prometheus
  run         Run a gadget (experimental)
  script      Run a bpftrace-compatible scripts
  snapshot    Take a snapshot of a subsystem and print it
  sync        Synchronize gadget information with server
  top         Gather, sort and periodically report events according to a given criteria
  trace       Trace and print system events
  traceloop   Get strace-like logs of a container from the past
  undeploy    Undeploy Inspektor Gadget from cluster
  version     Show version
...
```

The commands are divided into categories like `advice`, `audit`, `profile`, and so on.

For example, let's see what kind of tracing we can do with it.

```sh
kubectl gadget trace --help
```

The output is as follows.

```
Trace and print system events

Usage:
  kubectl-gadget trace [command]

Available Commands:
  bind         Trace socket bindings
  capabilities Trace security capability checks
  dns          Trace DNS requests
  exec         Trace new processes
  fsslower     Trace open, read, write and fsync operations slower than a threshold
  mount        Trace mount and umount system calls
  network      Trace network streams
  oomkill      Trace when OOM killer is triggered and kills a process
  open         Trace open system calls
  signal       Trace signals received by processes
  sni          Trace Server Name Indication (SNI) from TLS requests
  tcp          Trace TCP connect, accept and close
  tcpconnect   Trace connect system calls
  tcpdrop      Trace TCP kernel-dropped packets/segments
  tcpretrans   Trace TCP retransmissions
...
```

We can see that `trace` alone contains quite a few things like socket `bind`ing, check of security `capabilities`, trace of `dns` requests, and so on and so forth.

I won't bore you by going through all of the gadgets we have at our disposal. Instead, we'll take a quick look at a few I selected.

For example, we can ask Inspektor Gadget to monitor network traffic in the a-team namespace and store the output in network.json file.

```sh
kubectl gadget advise network-policy monitor --namespace a-team \
    --output network.log &
```

From now on, and until we stop monitoring, Inspektor Gadget will be collecting information about network traffic going in and out of Services in that Namespace.

So, let's generate some traffic. Normally, I would monitor real traffic in production for a while. The more information we gather, the better. However, for the purpose of showing you how it works, a single request to the Pinger app will be enough. That app will forward the request to the silly-demo app so we'll have at least three services involved. One from Ingress and two related to those apps.

Here it goes...

```sh
curl "http://pinger.$INGRESS_HOST.nip.io?url=silly-demo:8080"
```

Next, I will stop monitoring running in background by killing the kubectl process.

```sh
pkill kubectl
```

So far, all it did was to record traffic into network.log file. We can take a look at it...

```sh
cat network.log | jq .
```

The output is as follows.

```json
...
{
  "runtime": {
    "runtimeName": "containerd",
    "containerId": "68301787ca470019457615748ed1ddecf8ee9633a760d3d8a8734070724482ed",
    "containerName": "pinger",
    "containerImageName": "ghcr.io/vfarcic/silly-demo:1.4.120",
    "containerImageDigest": "sha256:7c27b05049a7de002e2fe01170b9e8579ddbed6d6cf7a289e48ccb3460eedb21"
  },
  "k8s": {
    "node": "kind-control-plane",
    "namespace": "a-team",
    "podName": "pinger-7c7ffbf6cf-n8zgh",
    "podLabels": {
      "app.kubernetes.io/name": "pinger",
      "pod-template-hash": "7c7ffbf6cf"
    },
    "containerName": "pinger"
  },
  "timestamp": 1712243244303049411,
  "type": "normal",
  "mountnsid": 4026533165,
  "netnsid": 4026533702,
  "pid": 3784,
  "tid": 3784,
  "comm": "silly-demo",
  "uid": 0,
  "gid": 0,
  "pktType": "HOST",
  "proto": "TCP",
  "port": 8080,
  "podHostIP": "172.18.0.2",
  "podIP": "10.244.0.10",
  "podOwner": "pinger",
  "podLabels": {
    "app.kubernetes.io/name": "pinger",
    "pod-template-hash": "7c7ffbf6cf"
  },
  "dst": {
    "addr": "10.244.0.1",
    "version": 4,
    "kind": "raw"
  }
}
```

...only to conclude that it's boring as hell and probably useless by itself. It's just endless list of requests going in and out.

The only way someone could convince me to go through those would be by saying that my only alterntive is the new Ghostbusters movie. Have you seen that one? If you haven't, don't... unless you're willing to pay for a therapist to help you with trauma it will imprint on you.

The interesting part is that we can use that information as input to generate network policies.

```sh
kubectl gadget advise network-policy report --input network.json
```

The output is as follows.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  creationTimestamp: null
  name: pinger-network
  namespace: a-team
spec:
  ingress:
  - from:
    - ipBlock:
        cidr: 10.244.0.1/32
    ports:
    - port: 8080
      protocol: TCP
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: ingress-nginx
      podSelector:
        matchLabels:
          app.kubernetes.io/component: controller
          app.kubernetes.io/instance: ingress-nginx
          app.kubernetes.io/name: ingress-nginx
          app.kubernetes.io/part-of: ingress-nginx
          app.kubernetes.io/version: 1.10.0
    ports:
    - port: 8080
      protocol: TCP
  podSelector:
    matchLabels:
      app.kubernetes.io/name: pinger
  policyTypes:
  - Ingress
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  creationTimestamp: null
  name: silly-demo-network
  namespace: a-team
spec:
  ingress:
  - from:
    - ipBlock:
        cidr: 10.244.0.1/32
    ports:
    - port: 8080
      protocol: TCP
  podSelector:
    matchLabels:
      app.kubernetes.io/name: silly-demo
  policyTypes:
  - Ingress
  - Egress
```

According to the traffic generated in that Namespace, we might want to generate those two network policies. The might not be perfect and we might want to modify them slightly. Still, no matter whether we use them as suggested by Inspektor Gadget or make some changes, having the ability to generate policies based on real traffic is amazing, especially since defining policies is tedios and, let's face it, boring.

From now on, I would store those policies in Git or apply them directly if I would like to be fired from the company I work at.

Don't do that. Don't apply anything directly. Use GitOps.

Another interesting feature is snapshot process which, in a way, is equivalent to the ps command, except that it looks for processes inside containers wrapped in Pods.

```sh
kubectl gadget snapshot process --namespace a-team
```

The output is as follows.

```
K8S.NODE           K8S.NAMESPACE K8S.POD                     K8S.CONTAINER COMM       PID   UID GID
kind-control-plane a-team        caw-58f7f578cd-kwbpj        caw           sh         11479 0   0  
kind-control-plane a-team        caw-58f7f578cd-kwbpj        caw           sleep      16033 0   0  
kind-control-plane a-team        pinger-7c7ffbf6cf-52gvw     pinger        silly-demo 11386 0   0  
kind-control-plane a-team        pinger-7c7ffbf6cf-mj8tp     pinger        silly-demo 11433 0   0  
kind-control-plane a-team        silly-demo-5d64dc9b7f-q42bl silly-demo    silly-demo 11338 0   0  
kind-control-plane a-team        silly-demo-5d64dc9b7f-v7bwl silly-demo    silly-demo 11292 0   0  
```

All the processes running, in this case, inside containers of the `a-team` Namespace, including `sleep` inside the container suspiciously called `caw`. That certainly does not look like a process that should be running and if this would not be a demo, I'd ban `caw` from my cluster.

We can also execute top command which, just as in Linux, shows which processes use the most resources, except that, this time, we'll limit those to eBPF.

```sh
kubectl gadget top ebpf
```

The output is as follows.

```
K8S.NODE           PROGID TYPE       NAME       PID   COMM         RUNTIME RUNCOU… MAPMEMORY MAPCOUNT
kind-control-plane 458    Tracing    ig_top_eb… 11690 gadgettra… 360.428µs 1755         328B 1
kind-control-plane 3      Kprobe     kprobe_mm…                   10.876µs 29           680B 2
kind-control-plane 253    CGroupDev…                                   1µs 1              0B 0
kind-control-plane 2      CGroupSKB  egress                             0s 0        16.94MiB 3
kind-control-plane 5      CGroupDev…                                    0s 0              0B 0
kind-control-plane 6      CGroupDev…                                    0s 0              0B 0
kind-control-plane 8      Kprobe     kprobe__o…                         0s 0        16.07MiB 1
```

There we go. Those are all eBPF apps running in the cluster.

> Stop watching `top` in the second terminal by pressing `ctrl+c`

What else...

We can also take a look at top usage of files, so let me spin up a container that will download files. To do that, I can clone a repo that is big. That is big enough? Linux! Let's clone the whole source code of `linux` inside a container. That will certainly result in decently heavy usage of the file system inside the cluster.

```sh
kubectl --namespace a-team run never-do-this --image alpine \
    -- /bin/sh -c \
    "apk add -U git && git clone https://github.com/torvalds/linux"
```

Now, while the container is cloning the whole Linux inside the cluster, we can take a look at `top file` inside the `a-team` Namespace.

```sh
kubectl gadget top file --namespace a-team
```

The output is as follows.

```
K8S.NODE       K8S.NAMESPACE  K8S.POD        K8S.CONTAINER  PID     COMM    READS   WRITES RBYTES WBYTES T FILE  
kind-co…-plane a-team         never-do-this  never-do-this  21262   git     0       5937   0B     23.18… R tmp_p…
```

There we go. We can see which process is writing to the file system, how many bytes are being written, and so on and so forth.

> Stop watching `top` in the second terminal by pressing `ctrl+c`

Let's do one more. Let's... trace TCP requests.

```sh
echo "http://pinger.$INGRESS_HOST.nip.io?url=silly-demo:8080"
```

> Use the URL from the output to generate load using [DDosify](https://ddosify.com/) or any other similar tool.

To do that, we'll first need to generate some traffic with, let's say DDosify which I can use to generate load testing. This is completely unrelated to the subject at hand and I'm using it only as a convenient way to generate some traffic. You might want to check it out though. Check it out.

Anyways... I'll create a test with `10000` iterations, choose to generate traffic from all continents, and run it. Imagine that's real traffic coming into one of the apps in the cluster.

Now, if I would like to trace all those requests, I can simply execute yet another Inspektor Gadget command.

```sh
kubectl gadget trace tcp --namespace a-team
```

The output is as follows.

```
K8S.NODE     K8S.NAMESP… K8S.POD     K8S.CONTAI… T PID     COMM  IP SRC                    DST                   
kind-c…plane a-team      silly…v7bwl silly-demo  A 11292   sill… 6  r/::ffff:10.244.0.8:80 r/::ffff:10.244.0.1:35
kind-c…plane a-team      silly…v7bwl silly-demo  A 11292   sill… 6  r/::ffff:10.244.0.8:80 r/::ffff:10.244.0.1:35
kind-c…plane a-team      silly…v7bwl silly-demo  X 11292   sill… 6  r/::ffff:10.244.0.8:80 r/::ffff:10.244.0.1:35
kind-c…plane a-team      silly…v7bwl silly-demo  X 11292   sill… 6  r/::ffff:10.244.0.8:80 r/::ffff:10.244.0.1:35
kind-c…plane a-team      silly…q42bl silly-demo  A 11338   sill… 6  r/::ffff:10.244.0.9:80 r/::ffff:10.244.0.1:33
kind-c…plane a-team      silly…q42bl silly-demo  A 11338   sill… 6  r/::ffff:10.244.0.9:80 r/::ffff:10.244.0.1:33
kind-c…plane a-team      silly…q42bl silly-demo  X 11338   sill… 6  r/::ffff:10.244.0.9:80 r/::ffff:10.244.0.1:33
kind-c…plane a-team      silly…q42bl silly-demo  X 11338   sill… 6  r/::ffff:10.244.0.9:80 r/::ffff:10.244.0.1:33
kind-c…plane a-team      pinge…52gvw pinger      A 11386   sill… 6  r/::ffff:10.244.0.10:8 r/::ffff:10.244.0.1:60
kind-c…plane a-team      pinge…52gvw pinger      A 11386   sill… 6  r/::ffff:10.244.0.10:8 r/::ffff:10.244.0.1:60
kind-c…plane a-team      pinge…52gvw pinger      X 11386   sill… 6  r/::ffff:10.244.0.10:8 r/::ffff:10.244.0.1:60
kind-c…plane a-team      pinge…52gvw pinger      X 11386   sill… 6  r/::ffff:10.244.0.10:8 r/::ffff:10.244.0.1:60
kind-c…plane a-team      pinge…mj8tp pinger      A 11433   sill… 6  r/::ffff:10.244.0.11:8 r/::ffff:10.244.0.1:58
kind-c…plane a-team      pinge…mj8tp pinger      A 11433   sill… 6  r/::ffff:10.244.0.11:8 r/::ffff:10.244.0.1:58
kind-c…plane a-team      pinge…mj8tp pinger      X 11433   sill… 6  r/::ffff:10.244.0.11:8 r/::ffff:10.244.0.1:58
kind-c…plane a-team      pinge…mj8tp pinger      X 11433   sill… 6  r/::ffff:10.244.0.11:8 r/::ffff:10.244.0.1:58
```

There we go. That's the traffic coming into Services in the a-team Namespace.

> Stop watching `top` in the second terminal by pressing `ctrl+c`

By now, you probably got the Gist of what Inspektor Gadget does. It is a set of commands added to `kubectl`. Those commands allow us to observe almost anything happening in the cluster. Since we're talking about eBPF, those are observations happening inside the kernel level, hence they are low-level, yet aware of Kubernetes itself. Inspektor Gadget truly reminds me of the commands I was using when working with Linux directly. It's similar to what I was doing before Kubernetes. It's something I was missing.

I should also mention that all the information available through the gadgets can be exposed as Prometheus metrics so if you prefer having it there, there is that option as well.

<img src="/logo/kubescape.png" style="width:30%; float:right; padding: 10px">

Another important note is that you might already be using Inspektor Gadget without even knowing that you're using it. A few tools are using it internally, one of them being Kubescape. If you're using it, you might want to know that, behind the scenes, it uses Inspektor Gadget for some of its features.

That's it. If you're looking for commands equivalent to ps, top, netstat, and other similar, yet indispensable commands when managing Linux, but adapted to Kuberentes, then Inspektor Gadget might be just the right tool for you.

## Destroy

```sh
kind delete cluster

exit
```


+++
title = 'Debunking Myths and Simplifying Compositions with Crossplane v2'
date = 2025-03-31T16:00:00+00:00
draft = false
+++

"**Crossplane is too complicated!**" "**Crossplane is only for infrastructure!** I need something else for applications."

I hear those and other similar statement very often so I decided to clarify a few things and, hopefully, eliminate some missconceptions.

To be more precise, today I want to debunk one missconception about Crossplane and, at the same time, show how Crossplane addressed one semi-legitimate complaint in the recent v2 release.

So, debunk one missconception and show one improvement with both of those being intertwined.

<!--more-->

{{< youtube ZQEVPnS3eeo >}}

"What is the missconception?" you might ask.

People see Crossplane as the **next generation Kubernetes-native tool** that should be used to manage **only infrastructure** and look for a different solution when trying to accomplish a similar outcome for applications.

As for the legitimate complaint... It is the perception that it is **hard to author Crossplane Compositions**.

Today I'll prove that **neither is true**, especially not since the release of Crossplane v2.

If you are thinking whether to adopt Crossplane, this post will resolve some missunderstandings that might lead you to look elsewhere. On the other hand, if you are already using Crossplane, you'll learn about a new feature that you were probably not aware of.

## Setup

```sh
git clone https://github.com/vfarcic/crossplane-app

cd crossplane-app

git pull

git fetch

git switch v2
```

> Make sure that Docker is up-and-running. We'll use it to create a KinD cluster.

> Watch [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell
```

> Watch [The Future of Shells with Nushell! Shell + Data + Programming Language](https://youtu.be/zoX_S6d-XU4) if you are not familiar with Nushell. Alternatively, you can inspect the `dot.nu` script and transform the instructions in it to Bash or ZShell if you prefer not to use that Nushell script.

```sh
chmod +x dot.nu
```

> At the time of this writing, Crossplane v2 is in preview mode and not yet GA. The setup script will install the preview version. If you're running the commands at some later time when v2 is GA, change the `--preview` argument to `false` or remove it altogether since `false` is the default value.

```sh
./dot.nu setup --preview true

source .env
```

## Crossplane Is NOT (Only) About Infrastructure

Here's a definition of two applications, a backend and a frontend.

```sh
cat examples/full.yaml
```

The output is as follows.

```yaml
apiVersion: devopstoolkit.live/v1beta1
kind: App
metadata:
  name: silly-demo
spec:
  image: ghcr.io/vfarcic/silly-demo
  tag: v1.5.38
  port: 8080
  host: silly-demo.127.0.0.1.nip.io
  scaling:
    enabled: true
    min: 2
    max: 5
  crossplane:
    compositionSelector:
      matchLabels:
        type: backend
        location: local
---
apiVersion: devopstoolkit.live/v1beta1
kind: App
metadata:
  name: silly-demo-frontend
spec:
  image: ghcr.io/vfarcic/silly-demo-frontend
  tag: v1.5.38
  port: 3000
  host: silly-demo-frontend.127.0.0.1.nip.io
  frontend:
    backendUrl: http://silly-demo.127.0.0.1.nip.io
  crossplane:
    compositionSelector:
      matchLabels:
        type: frontend
        location: local
```

Now, those are not your typical templates like Helm where you would work with values files while trying to ignore the rest of the chart. Those are Kubernetes Custom Resources based on Custom Resource Definitions and controllers created through Crossplane Compositions.

In any case, those are as simple as it can get. Both are CRs `App` that define `image` and the `tag` we want to use, followed with the `port` and the `host` through which we should be able to access those applications. On top of those, the Backend has `scaling` `enabled` while the frontend contains the information of the `backendUrl` so that it knows where to send requests.

We can apply those...

```sh
kubectl --namespace a-team apply --filename examples/full.yaml
```

...and list all the `apps` in that Namespace.

```sh
kubectl --namespace a-team get apps
```

The output is as follows.

```
NAME                  HOST                                   SYNCED   READY   COMPOSITION    AGE
silly-demo            silly-demo.127.0.0.1.nip.io            True     True    app-backend    4s
silly-demo-frontend   silly-demo-frontend.127.0.0.1.nip.io   True     False   app-frontend   4s
```

Today's subject, however, is not about explaining how Crossplane Compositions work. You can watch the [Crossplane Tutorial](https://youtube.com/playlist?list=PLyicRj904Z99i8U5JaNW5X3AyBvfQz-16) if you're not familiar with it. Instead, I want to show you a few new things released in v2 and debunk some myths about Crossplane.

Here's the first one.

```sh
kubectl tree --namespace a-team app silly-demo-frontend
```

The output is as follows.

```
NAMESPACE NAME                                           READY REASON    AGE
a-team    App/silly-demo-frontend                        True  Available 67s
a-team    ├─Deployment/silly-demo-frontend               -               67s
a-team    │ └─ReplicaSet/silly-demo-frontend-7b94d79bf7  -               67s
a-team    │   └─Pod/silly-demo-frontend-7b94d79bf7-56qxb True            67s
a-team    ├─Ingress/silly-demo-frontend                  -               67s
a-team    └─Service/silly-demo-frontend                  -               67s
a-team      └─EndpointSlice/silly-demo-frontend-n6w4w    -               67s
```

Those are all the resources composed by the `App` `silly-demo-frontend`, and there are two important observations we can make.

First of all, there is nothing related to infrastructure.

If you're still not sure that's the case, let's list all those related to the backend.

```sh
kubectl tree --namespace a-team app silly-demo
```

The output is as follows.

```
NAMESPACE NAME                                  READY REASON    AGE
a-team    App/silly-demo                        True  Available 35s
a-team    ├─Deployment/silly-demo               -               35s
a-team    │ └─ReplicaSet/silly-demo-5fd866ff58  -               35s
a-team    │   ├─Pod/silly-demo-5fd866ff58-9qgkr True            35s
a-team    │   └─Pod/silly-demo-5fd866ff58-ccq8f True            20s
a-team    ├─HorizontalPodAutoscaler/silly-demo  -               35s
a-team    ├─Ingress/silly-demo                  -               35s
a-team    └─Service/silly-demo                  -               35s
a-team      └─EndpointSlice/silly-demo-shbg7    -               35s
```

> `kubectl tree` is not a command bakedn into `kubectl`. It is an extension that was installed through Devbox during the setup.

There is still nothing related to infrastructure. It composed a `Deployment`, a `HorizontalPodAutoscaler`, an `Ingress`, and a `Service`.

That should stop conversations that claim that Crossplane is only about infrastructure. Crossplane Compositions, to be specific, create **CRDs and Controllers** that compose resources. Whether those resources, like in this case, represent an application or a infrastructure or a service or anything else, depends on the person who built that Composition. Similarly, whether those resources are applied in the same cluster, as in this case, or in a different cluster, is also up to the person who designed the composition. Crossplane Compositions do not care what resources are doing nor where they come from. There are many ways, as we'll see later, how resources can be composed, but they all ultimately compose resources no matter what they do or where they are.

Now, you might have known that already if you used Crossplane and you're not among those claiming that it is used only to compose infrastructure.

However, if that is the case, you might want to take a closer look at those resources and try to figure out what's missing.

I'll wait...

There are no, what we call, Managed Resources.

Until recently, we could compose only Managed Resources that come from Crossplane Providers. That could be resources like EC2 from the AWS provider, or GKE from the Google Cloud provider, or Repository from the GitHub provider, or a Dashboard from the Grafana provider, or anything else. If we wanted to compose Kubernetes resources like, for example, Deployments, we had to use the Managed Resource Object from the Kubernetes provider. Similarly, if we wanted to include a Helm chart, we had to use the Release, which is a Managed Resource from the Helm provider.

None of those are in that output. It composed a bunch of Kubernetes resources without using the Kubernetes provider.

That's one of the new additional features introduced in Crossplane v2. From now on, we can compose resources directly. Whether you should do that or not is something we'll discuss later. For now, the important note is that we can **compose anything**, no matter whether those are resources brought in through Crossplane Providers, or anything else. If you want to compose ACK resources directly, you can. If you want to include CNPG directly, you can.

So, if you were thinking that Crossplane Compositions are only for infra, now you know that's not the case. If you thought that you have to use Crossplane Providers, now you know that you don't.

That, by itself, can **reduce the complexity** when defining Compositions.

Let me explain.

## With and Without Crossplane Providers

Here's how I would have done a Composition we saw in action earlier a few years ago.

```sh
cat examples/object.yaml
```

The output is as follows (truncated for brevity).

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: app-frontend
  labels:
    type: frontend
    location: local
spec:
  compositeTypeRef:
    apiVersion: devopstoolkit.live/v1beta1
    kind: App
  mode: Pipeline
  pipeline:
    - step: patch-and-transform
      functionRef:
        name: crossplane-contrib-function-patch-and-transform
      input:
        apiVersion: pt.fn.crossplane.io/v1beta1
        kind: Resources
        ...
        resources:
          - name: deployment
            base:
              apiVersion: kubernetes.crossplane.io/v1alpha2
              kind: Object
              metadata:
                name: deployment
              spec:
                forProvider:
                  manifest:
                    apiVersion: apps/v1
                    kind: Deployment
                    spec:
                      template:
                        spec:
                          containers:
                            - name: main
                              livenessProbe:
                                httpGet:
                                  path: "/"
                              readinessProbe:
                                httpGet:
                                  path: "/"
                              resources:
                                limits:
                                  cpu: "250m"
                                  memory: "256Mi"
                                requests:
                                  cpu: "125m"
                                  memory: "128Mi"
                              env:
                                - name: BACKEND_URL
                providerConfigRef:
                  name: kubernetes-provider
            ...
          - name: service
            ...
          - name: ingress
            ...
```

That is, arguably, the worst way one would define a Composition.

There are two potential problems over there, one of those being something I could have fixed a while ago, while the other is fixable only recently.

Let's start with the mistake that can be corrected only now that v2 is out.

That Composition composes a number of resources. There is a `deployment`, a `service`, and an `ingress`. All those are manifests inside a Managed Resource `Object`. Now, that object is very useful for a number of reasons, one of them being able to apply any Kubernetes resource in any cluster, not only the cluster where Crossplane is running. We do that by specifying `providerConfigRef` which points to a config with credentials used to access a cluster.

However, in this specific case where I want to compose resources inside the same cluster, that is only noise. I spent time writing those seven lines that define the `Object`, and two more lines to point it to the `providerConfigRef` for no good reason apart from Crossplane's requirement to use only resources available in its providers.

That is not the case any more.

Now we can compose resources directly.

Here's an example.

```sh
cat package/frontend.yaml
```

The output is as follows (truncated for brevity).

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: app-frontend
  labels:
    type: frontend
    location: local
spec:
  compositeTypeRef:
    apiVersion: devopstoolkit.live/v1beta1
    kind: App
  mode: Pipeline
  pipeline:
    - step: patch-and-transform
      functionRef:
        name: crossplane-contrib-function-patch-and-transform
      input:
        apiVersion: pt.fn.crossplane.io/v1beta1
        kind: Resources
        patchSets:
          - name: name
            patches:
              - type: FromCompositeFieldPath
                fromFieldPath: metadata.name
                toFieldPath: metadata.name
              - type: FromCompositeFieldPath
                fromFieldPath: metadata.name
                toFieldPath: metadata.labels["app.kubernetes.io/name"]
        resources:
          - name: deployment
            base:
              apiVersion: apps/v1
              kind: Deployment
              spec:
                template:
                  spec:
                    containers:
                      - name: main
                        livenessProbe:
                          httpGet:
                            path: "/"
                          failureThreshold: 10
                        readinessProbe:
                          httpGet:
                            path: "/"
                          failureThreshold: 10
                        resources:
                          limits:
                            memory: 1024Mi
                          requests:
                            cpu: 500m
                            memory: 512Mi
                        env:
                          - name: BACKEND_URL
            readinessChecks:
              - type: MatchCondition
                matchCondition:
                  type: Available
                  status: "True"
            patches:
              - type: PatchSet
                patchSetName: name
              - type: FromCompositeFieldPath
                fromFieldPath: metadata.name
                toFieldPath: spec.selector.matchLabels["app.kubernetes.io/name"]
              - type: FromCompositeFieldPath
                fromFieldPath: metadata.name
                toFieldPath: spec.template.metadata.labels["app.kubernetes.io/name"]
              - type: CombineFromComposite
                combine:
                  variables:
                    - fromFieldPath: spec.image
                    - fromFieldPath: spec.tag
                  strategy: string
                  string:
                    fmt: "%s:%s"
                toFieldPath: spec.template.spec.containers[0].image
              - type: FromCompositeFieldPath
                fromFieldPath: spec.port
                toFieldPath: spec.template.spec.containers[0].livenessProbe.httpGet.port
              - type: FromCompositeFieldPath
                fromFieldPath: spec.port
                toFieldPath: spec.template.spec.containers[0].readinessProbe.httpGet.port
              - type: FromCompositeFieldPath
                fromFieldPath: spec.port
                toFieldPath: spec.template.spec.containers[0].ports[0].containerPort
              - type: FromCompositeFieldPath
                fromFieldPath: spec.frontend.backendUrl
                toFieldPath: spec.template.spec.containers[0].env[0].value
          - name: service
            base:
              apiVersion: v1
              kind: Service
              ...
          - name: ingress
            base:
              apiVersion: networking.k8s.io/v1
              kind: Ingress
              ...
```

On the first look, that Composition might look the same, but it's not. All those Objects are now gone and we now have `Deployment`, `Service`, and `Ingress` composed directly.

We reduced the number of lines and the complexity without making any functional changes to that Composition.

Not having to use Managed resources like *Object* but, instead, compose resources directly certainly **makes authoring Compositions easier**. There are very few things, it anything, that can be removed from that Composition to make it easier, except for those `patches`. They are horrible, and they lead me to the second problem.

It's YAML. It's "pure" YAML.

Now, I am not going to tell you that YAML is bad. It is, but I'm not going to rub it to your face because I'm a nice guy.

The problem with YAML is **verbosity and insufficient flexibility**.

The `patches` section is just silly, but necessary. For example, one of those is telling Crossplane that we want to patch by taking a value `FromCompositeFieldPath` `spec.port` and put it into `spec.template.spec.containers[0].livenessProbe.httpGet.port`. That's how, among other things, it takes input from a Composite Resource, like the one we saw at the very start, and puts it as the value of the resource it is composing.

Now, you might say: **"Wait a minute. Why doesn't Crossplane do something similar to kro that uses CEL expressions as values to make them much less verbose?"**

I have two answers to that question.

First, that is not YAML any more. It is a combination of YAML with CEL or some other type of an abomination. That does not necessarily mean that it is a bad approach but only that it is not YAML any more.

Now, if we don't want YAML, than we should ask ourselves which format we want. Do you prefer Go Templating like what's used with Helm? Or do you prefer Python? How about KCL? Well, you can have any of those and many others. Crossplane **does not care about the format** we use since it introduced Functions. We can assemble resources in any language or any format available as long as it is available as a Function.

That solves many problems, especially those related to "YAML abominations" which eventually all face the same issue of not being able to support "stuff" that is not available in YAML but is available in other languages like conditionals, loops, or anything else we might need.

## Any Language

Here's an example of a very similar Composition as the one we saw previously but, this time, written in KCL which happens to be my favorite language for that kind of tasks.

```sh
cat kcl/backend-resources.k
```

The output is as follows (truncated for brevity).

```kcl
oxr = option("params").oxr
ocds = option("params").ocds
_name = oxr.metadata.name
_spec = oxr.spec
...
_items = [
    {
        apiVersion = "apps/v1"
        kind = "Deployment"
        metadata = _metadata(_name, "deployment")
        spec = {
            selector.matchLabels = {
                "app.kubernetes.io/name" = _name
            }
            template = {
                metadata.labels = {
                    "app.kubernetes.io/name" = _name
                }
                spec = {
                    containers = [{
                        image = _spec.image + ":" + _spec.tag
                        ...
if _spec.scaling?.enabled:
    _items += [{
        apiVersion = "autoscaling/v2"
        kind = "HorizontalPodAutoscaler"
        ...
```

Instead of extending Crossplane Compositions YAML schema, now we can define resources in **any language** we want with all the benefits of the language we choose.

In this example, I'm declaring variable `_spec` that takes `spec` from Composite Resources, the manifests end users define, and using it to retrieve `image` and `tag` and combine those into the value of the `image` in a `Deployment`. Similarly, There is a conditional `if` statement that will include `HorizontalPodAutoscaler` among composed resources only if `scaling?.enabled` is set to *true*.

Now, don't be depressed if KCL looks offputting. I love it, but it might not be your cup of tee. If that's the case, let me remind you that you can use **any language** currently supported and if what you want it not among those, it's easy to create a function that will support it yourself and, hopefully, contribute it to the community.

## Wrap-Up

I hope that I debunked some missconceptions and shed some light on one of the Crossplane v2 features that should make authoring Compositions easier.

Let's summarize

Crossplane Compositions are **NOT only about infrastructure**. We can use them to compose anything, be it infrastructure, applications, or anything else.

Crossplane Compositions **do not force us, any more, to compose only Managed Resources** provided with Crossplane Providers. We can certainly use them, but we can also compose any other Kubernetes resources, be it those baked into Kubernetes itself, like Deployments, or third-party like CNPG or ACK, or anything else. If it can run in Kubernetes, it can be composed with Crossplane. Now, to be clear, anything could be composed before as well, but, this time, we do not need to wrap resources into Kubernetes Object or Helm Release or any other Managed Resource. The ability to compose anything existed since early days. What's new is that now we can compose any Kubernetes resource directly.

The ability to compose anything directly, means that authoring Compositions is easier than it was before. When taken into the account what Crossplane Compositions can do, I would go as far as to say that it is now as **easy** as it can get.

Finally, the flexibility to use **any language** currently supported is amazing. If YAML works for you, great, use it. If you prefer, let's say, Python, use it instead. If you'd like to combine Go templating and KCL, you can do that as well. Crossplane does not care which language you prefer. Instead, it enables you to use whatever you're familiar with.

To finish it off, I only mentioned one new feature in v2, the ability to compose resources directly without using Managed Resources. Would it be interesting to go through all the new features and the changes you might want to make to Crossplane and your Compositions? Please let me know in the comments.

## Destroy

```sh
./dot.nu destroy

git switch main

exit
```


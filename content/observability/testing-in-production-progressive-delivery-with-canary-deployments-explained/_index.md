
+++
title = 'Testing in Production! Progressive Delivery with Canary Deployments Explained!'
date = 2024-11-11T12:00:00+00:00
draft = false
+++

Today I will make an outrageous claim. Ready? Here it goes... The only testing that truly matters is **testing in production**. The only way to truly verify that a release is working as expected is to run it in production with "real" users and "real" workload. Testing a release before it reaches production is helpful and I am certainly not going to tell you to stop writing and running your unit tests, and functional tests, and integration tests, and whichever other type of testing you might normally do. What I am going to tell you is that you have to test your releases in production. Confirmation that "real" **users got what they expected** is the only thing that truly matters.

<!--more-->

{{< youtube sFzKrGSaIr8 >}}

## Intro

Here's the thing. We cannot truly know that a new release will be working correctly before we deploy it to production. We can certainly be more confident by running all sorts of tests, but we cannot be sure. As a result of that uncertainty, we have to test a release after it's been released to production.

Here comes the important note. Almost everyone is testing releases after they've been deployed to production, but not everyone is aware that's what they're doing. You see, every single company that has some sort of **observability is** effectively **testing in production**. But, before I back that claim, let me quickly explain what testing is.

In a nutshell, testing is all about **performing some actions and verifying** that **outcomes** of those actions are as expected. We click a link and we confirm that it led us to a specific page. We invoke a function with specific parameters and we test that the output of that function is as expected. We fill in some fields and click a submit button and we confirm that data was stored in a database. You get the point. Right? We perform some actions and we verify outcomes of those actions.

Now, let's get back to observability and testing in production.

When running something in production, that something is exposed to real users. They are interacting with our applications. They are performing some actions, and they are receiving some outcomes. If we don't do anything, our users are the only testers of applications running in production. They are even filing issues by opening support tickets.

We can do much better than that. We can keep "real" users perform actions, and do verifications ourselves. Instead of waiting for support tickets, we can be proactive and try to figure out that something is wrong ourselves. We can find out that something is wrong while only a small subset of our users is exposed to it.

## Setup

```sh
git clone https://github.com/vfarcic/argo-rollouts-demo

cd argo-rollouts-demo
```

> Please watch https://youtu.be/WiFLtcBvGMU if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell
```

> Please watch [The Future of Shells with Nushell! Shell + Data + Programming Language](https://youtu.be/zoX_S6d-XU4) if you are not familiar with Nushell. Alternatively, you can inspect the `setup.nu` script and transform the instructions in it to Bash or ZShell if you prefer not to use that Nushell script.

```sh
chmod +x setup.nu

./setup.nu
```

> Open two new terminal sessions and make sure that they are in the same directory as the first. All commands should be executed in the first terminal session unless specified otherwise.

> Execute the command that follows in all terminal sessions.

```sh
source .env
```

## Manual Testing Of Progressive Delivery (Canary Deployments)

I'll make an assumption that it is impossible to ensure that every single release to production is without bugs and based on features that our users actually want. If that's the case, if issues are unavoidable, the only thing we can do is **limit the blast radius**. Instead of exposing all our users to a new release, we can limit it to a subset of them and see how they react to it. If it seems to be working correctly for that subset of users, we can choose to increase the reach. If, let's say, we started by exposing that release to ten percent of the traffic, after we confirm that our users are having good experience with it, we can increase the exposure to twenty, then to thirty, and so on and so forth, all the way until hundred percent, at which point the new release is fully rolled out.

Alternatively, if at any point, we discover that the new release is not working as expected, we can roll it back. As a result, only a subset of users will be negatively affected. That's certainly not ideal, but it's still much better than if we let it blow into everyone's faces.

Such a gradual rollout of a release is called **Canary Deployments** which is a subset of a wider range of techniques called **progressive delivery**.

Progressive delivery is all about gradually or **progressively rolling out releases**. Besides canary deployments, we could employ rolling updates, blue-green, or any other similar strategy. As a matter of fact, you might already be using one of those strategies without even knowing that's what you're doing. For example, Kubernetes tends to rely on rolling updates. A change to a Deployment initiates rolling updates where Pods with the old release are gradually phased our while, at the same time, Pods with the new release are brought up. Similarly, most managed Kubernetes services tend to use rolling updates when upgrading nodes.

All those techniquest are based on a simple principle that we should release something progressively instead of "**big bag, here you go, it's all or nothing**" type of approach.

Here's how canary deployments look like in action.

```sh
cat kustomize/base/deployment.yaml
```

The output is as follows (truncated for brevity).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: silly-demo
  name: silly-demo
spec:
  replicas: 0
  ...
  template:
    ...
    spec:
      containers:
      - image: ghcr.io/vfarcic/silly-demo:1.4.126
        ...
```

That is a very simple Kubernetes `Deployment` that, under normal circumstances, would perform rolling updates of the release `1.4.126` of the `silly-demo` application. The only thing that makes that Deployment a bit "special" is that the number of `replicas` is set to `0`. If we would apply that Deployment alone, nothing would be running.

<img src="/logo/argo-rollouts.jpg" style="width:50%; float:right; padding: 10px">

I set it to zero replicas since it will not be me who decides how many replicas should run. That decision will be made by Argo Rollouts which, by the way, I'm using today only to demonstrate certain principles rather than to convince you that it is better than other similar tools like, for example, Flagger.

```sh
cat kustomize/overlays/simple/rollout.yaml
```

The output is as follows.

```yaml
---
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: silly-demo
spec:
  replicas: 5
  strategy:
    canary:
      steps:
        - setWeight: 20
        - pause: {}
        - setWeight: 40
        - pause: {duration: 10}
        - setWeight: 60
        - pause: {duration: 10}
        - setWeight: 80
        - pause: {duration: 10}
  revisionHistoryLimit: 2
  selector:
    matchLabels:
      app: silly-demo
  workloadRef:
    apiVersion: apps/v1
    kind: Deployment
    name: silly-demo
```

That is an Argo Rollouts `Rollout` definition. The `spec` starts with the numer of `replicas` set to `5`. As you'll see later, a release of our application might not always run as five replicas. Think of that number as the final number of replicas. Also, please note that, more often than not, we should not specify a static number of replicas but, rather, use HorizontalPodAutoscaler which, by the way, is supported by Argo Rollouts. Still, for the sake of simplicity, we'll use a static number of replicas today.

Next, we have the strategy set to `canary`. There are others but, in my opinion, that's, more often than not, the one we want.

Inside the *canary* strategy, we have an array of `steps`. In this scenario, we start by telling it to `setWeight` to `20`. That means that the approximate number of requests that will go to a new release will be twenty percent. As a result, only twenty percent of users accessing our application will see the new release, at least during the first step. As such, we can validate whether it's working correctly before we proceed to the next step. For now, I will assume that validation is done manually. You might be testing it yourself, or you might be watching some observability dashboards to see whether there are any anomalies, or you might be waiting for support issues from your users letting you know that they are very dissapointed. The only one of those options we should be doing is watching observability metrics since they show us the information we need to deduce whether it's working as expected. That could be error rates, latency, or anything else we might be observing today. There is a hidden motive for saying that observability is the key. I'll get to it later.

The second step instructs Argo Rollouts to `pause` progression indefinitely (`{}`). That should give us ample amount of time to test, observe, or collect support tickets related to the new release. The important thing is that, at this point, it will roll out to a limited number of users so that the blast radius is limited. Once we decide whether it's a good one or not, we can proceed to move forward, or roll back.

The rest of the steps keep increasing the `weight` to `40`, then `60`, and, finally, to `80` percent. No matter the steps, once the rollout is finished, it will reach hundred percent, so there's no need to set that explicitly.

In between those steps are `pause` instructions but, inlike the first one, those are not waiting for us indefinitely. Instead, they are pausing, in this case, for only *10* seconds. Since we don't have any automated validation, that means that we have ten seconds to validate before the rollout continues to the next *weight*. That's unreasonably short period to do anything yet it should be more than enough for our demo.

Further on we have the `selector` that tells Argo Rollouts how to find the app which, in this case, is through label `app` set to `silly-demo`.

Finally, there is `workloadRef`. In the past, we had to replace the typical `Deployment` for `Rollout`. In those cases, the `Rollout` would be managing ReplicaSets just as Deployment does. That was, in my opinion, a bad idea. It makes much more sense to keep our application definitions independent of Argo Rollouts instead of rewriting what we have. On top of that, if we wanted our application to use custom resources, we would need to look elsewhere. To be honest, that was one of the biggest grudges I had with Argo Rollouts.

Fortunately, that was fixed sometime in 2022 with the introduction of `workloadRef`. Now we can instruct Argo Rollouts to "control" any type of resource. In this case we're referencing the `Deployment` `silly-demo`.

That's it. Let's see it in action by applying the resources.

```sh
kubectl --namespace a-team apply \
    --kustomize kustomize/overlays/simple
```

We can see what's going on through the `argo rollout` plugin for `kubectl`.

> Execute the command that follows in the second terminal session

```sh
kubectl argo rollouts --namespace a-team get rollout silly-demo \
    --watch
```

The output of Argo Rollouts is as follows.

```
...
NAME                                    KIND        STATUS     AGE   INFO
⟳ silly-demo                            Rollout     ✔ Healthy  118s
└──# revision:1
   └──⧉ silly-demo-5c5547db68           ReplicaSet  ✔ Healthy  118s  stable
      ├──□ silly-demo-5c5547db68-2rn84  Pod         ✔ Running  118s  ready:2/2
      ├──□ silly-demo-5c5547db68-492kk  Pod         ✔ Running  118s  ready:2/2
      ├──□ silly-demo-5c5547db68-bm9vt  Pod         ✔ Running  118s  ready:2/2
      ├──□ silly-demo-5c5547db68-fkpp2  Pod         ✔ Running  118s  ready:2/2
      └──□ silly-demo-5c5547db68-zh8ck  Pod         ✔ Running  118s  ready:2/2 
```

Actually, there's no much to look at right now. When deploying the first release it does not make sense to do anything but roll it out as fast as possible. So, that's what it did. It deployed five replicas of the first `revision`.

But... If we modify the image,...

```sh
cd kustomize/overlays/simple

kustomize edit set image \
    ghcr.io/vfarcic/silly-demo=ghcr.io/vfarcic/silly-demo:1.4.127

cd ../../../
```

...and apply the manifests again, we should see a very different outcome.

```sh
kubectl --namespace a-team apply \
    --kustomize kustomize/overlays/simple
```

The output of Argo Rollouts is as follows (truncated for brevity.).

```
...
NAME                                    KIND        STATUS     AGE    INFO
⟳ silly-demo                            Rollout     ॥ Paused   2m15s
├──# revision:2
│  └──⧉ silly-demo-5d574b5f4f           ReplicaSet  ✔ Healthy    50s  canary
│     ├──□ silly-demo-5d574b5f4f-4f77d  Pod         ✔ Running    50s  ready:2/2
└──# revision:1
   └──⧉ silly-demo-5c5547db68           ReplicaSet  ✔ Healthy  2m15s  stable
      ├──□ silly-demo-5c5547db68-mk6p9  Pod         ✔ Running  2m15s  ready:2/2
      ├──□ silly-demo-5c5547db68-t457w  Pod         ✔ Running  2m15s  ready:2/2
...
```

There are two things we should note.

First, we can see that only one Pod is running as the `revision` `2`. Since there are five Pods in total and we specified that we want start by rolling out to twenty percent of users, we got one Pod or fifth of the total.

So, right now, approximately one fifth of the requests are being sent to the new release while all the others are still being served by the old release. We'll confirm that later. For now, you need to trust me on that one.

The second important note is that the `STATUS` is now set to `Paused`. We specified that we would like it to wait at this point indefinitely so that we can have all the time in the world to test the new release while affecting only a fraction of users.

Now, let's say that we finished testing, or observing, or collecting support issues, or whatever we might be doing. Also, let's say that we did not discover any major issue that would prevent us from rolling out the new release to an even bigger number of users.

To proceed with the rollout, we can simply `promote` `silly-demo`. That action, from the Argo Rollouts perspective, can be translated to "continue into the next step."

```sh
kubectl argo rollouts --namespace a-team promote silly-demo
```

The output of Argo Rollouts is as follows.

```
Name:            silly-demo
Namespace:       a-team
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
  Step:          3/8
  SetWeight:     40
  ActualWeight:  40
Images:          ghcr.io/vfarcic/silly-demo:1.4.126 (stable)
                 ghcr.io/vfarcic/silly-demo:1.4.127 (canary)
Replicas:
  Desired:       5
  Current:       5
  Updated:       2
  Ready:         5
  Available:     5

NAME                                    KIND        STATUS     AGE    INFO
⟳ silly-demo                            Rollout     ॥ Paused   6m6s
├──# revision:2
│  └──⧉ silly-demo-5d574b5f4f           ReplicaSet  ✔ Healthy  2m57s  canary
│     ├──□ silly-demo-5d574b5f4f-jq459  Pod         ✔ Running  2m56s  ready:2/2
│     └──□ silly-demo-5d574b5f4f-ffmxr  Pod         ✔ Running  10s    ready:2/2
└──# revision:1
   └──⧉ silly-demo-5c5547db68           ReplicaSet  ✔ Healthy  6m6s   stable
      ├──□ silly-demo-5c5547db68-492kk  Pod         ✔ Running  6m6s   ready:2/2
      ├──□ silly-demo-5c5547db68-fkpp2  Pod         ✔ Running  6m6s   ready:2/2
      └──□ silly-demo-5c5547db68-zh8ck  Pod         ✔ Running  6m6s   ready:2/2
```

We can see that the rollout continues to `40` percent, then it wait for ten seconds. After that, it continues to sixty percent, waits again, continues to eighty, waits again, and, finally, it rolls out to hundred percent.

```
Name:            silly-demo
Namespace:       a-team
Status:          ✔ Healthy
Strategy:        Canary
  Step:          8/8
  SetWeight:     100
  ActualWeight:  100
Images:          ghcr.io/vfarcic/silly-demo:1.4.127 (stable)
Replicas:
  Desired:       5
  Current:       5
  Updated:       5
  Ready:         5
  Available:     5

NAME                                    KIND        STATUS        AGE  INFO
⟳ silly-demo                            Rollout     ✔ Healthy     42m
├──# revision:2
│  └──⧉ silly-demo-5d574b5f4f           ReplicaSet  ✔ Healthy     39m  stable
│     ├──□ silly-demo-5d574b5f4f-jq459  Pod         ✔ Running     39m  ready:2/2
│     ├──□ silly-demo-5d574b5f4f-ffmxr  Pod         ✔ Running     36m  ready:2/2
│     ├──□ silly-demo-5d574b5f4f-xxfrp  Pod         ✔ Running     36m  ready:2/2
│     ├──□ silly-demo-5d574b5f4f-8tqtl  Pod         ✔ Running     36m  ready:2/2
│     └──□ silly-demo-5d574b5f4f-t2tdf  Pod         ✔ Running     35m  ready:2/2
└──# revision:1
   └──⧉ silly-demo-5c5547db68           ReplicaSet  • ScaledDown  42m
```

While the rollout of the new release was hapening, it was, at the same time, decreasing the number of replicas of the old release and, by doing that, maintaining the desired weights. Once it finished, we ended up with five replicas of the new release and zero replicas of the old. The old one has been `ScaledDown` and now all the traffic is going to the new one.

Let's do another release by changing the image tag one more time,...

```sh
cd kustomize/overlays/simple

kustomize edit set image \
    ghcr.io/vfarcic/silly-demo=ghcr.io/vfarcic/silly-demo:1.4.128

cd ../../../
```

...and re-applying the manifests.

```sh
kubectl --namespace a-team apply \
    --kustomize kustomize/overlays/simple
```

The output of Argo Rollouts is as follows.

```
Name:            silly-demo
Namespace:       a-team
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
  Step:          1/8
  SetWeight:     20
  ActualWeight:  20
Images:          ghcr.io/vfarcic/silly-demo:1.4.127 (stable)
                 ghcr.io/vfarcic/silly-demo:1.4.128 (canary)
Replicas:
  Desired:       5
  Current:       5
  Updated:       1
  Ready:         5
  Available:     5

NAME                                    KIND        STATUS        AGE   INFO
⟳ silly-demo                            Rollout     ॥ Paused      103m
├──# revision:3
│  └──⧉ silly-demo-6b8dbddd4b           ReplicaSet  ✔ Healthy     24m   canary
│     └──□ silly-demo-6b8dbddd4b-qqppw  Pod         ✔ Running     24m   ready:2/2
├──# revision:2
│  └──⧉ silly-demo-5d574b5f4f           ReplicaSet  ✔ Healthy     100m  stable
│     ├──□ silly-demo-5d574b5f4f-jq459  Pod         ✔ Running     100m  ready:2/2
│     ├──□ silly-demo-5d574b5f4f-ffmxr  Pod         ✔ Running     97m   ready:2/2
│     ├──□ silly-demo-5d574b5f4f-xxfrp  Pod         ✔ Running     97m   ready:2/2
│     └──□ silly-demo-5d574b5f4f-8tqtl  Pod         ✔ Running     96m   ready:2/2
└──# revision:1
   └──⧉ silly-demo-5c5547db68           ReplicaSet  • ScaledDown  103m
```

A few moments later it stopped after the first step that set the weight to twenty percent thus giving us one Pod of the new and reducing the number of the Pods of the old release to four.

Now, let's assume that we discovered a major issue and made a decision to roll back to the previous release.

All we have to do is execute the `abort` command.

```sh
kubectl argo rollouts --namespace a-team abort silly-demo
```

The output of Argo Rollouts is as follows.

```
Name:            silly-demo
Namespace:       a-team
Status:          ✖ Degraded
Message:         RolloutAborted: Rollout aborted update to revision 3
Strategy:        Canary
  Step:          0/8
  SetWeight:     0
  ActualWeight:  0
Images:          ghcr.io/vfarcic/silly-demo:1.4.127 (stable)
Replicas:
  Desired:       5
  Current:       5
  Updated:       0
  Ready:         5
  Available:     5

NAME                                    KIND        STATUS        AGE   INFO
⟳ silly-demo                            Rollout     ✖ Degraded    103m
├──# revision:3
│  └──⧉ silly-demo-6b8dbddd4b           ReplicaSet  • ScaledDown  24m   canary
├──# revision:2
│  └──⧉ silly-demo-5d574b5f4f           ReplicaSet  ✔ Healthy     100m  stable
│     ├──□ silly-demo-5d574b5f4f-jq459  Pod         ✔ Running     100m  ready:2/2
│     ├──□ silly-demo-5d574b5f4f-ffmxr  Pod         ✔ Running     97m   ready:2/2
│     ├──□ silly-demo-5d574b5f4f-xxfrp  Pod         ✔ Running     97m   ready:2/2
│     ├──□ silly-demo-5d574b5f4f-8tqtl  Pod         ✔ Running     97m   ready:2/2
│     └──□ silly-demo-5d574b5f4f-f8rhm  Pod         ✔ Running     13s   ready:2/2
└──# revision:1
   └──⧉ silly-demo-5c5547db68           ReplicaSet  • ScaledDown  103m
```

After a while, we can see that the `Status` is `Degraded`. We aborted the process so Argo Rollouts reverted all the changes. It scaled the old release back to five replicas and it scaled the new one to zero. As a result, all the traffic is now being served by the old release. We're, more or less, safe. Only a fraction of the users experienced issues.

Here comes an important note.

Do NOT do what I just did. We can do so much better than that.

Before I explain why I said that, let's delete the app so that we can start over.

```sh
kubectl --namespace a-team delete \
    --kustomize kustomize/overlays/simple
```

## Controlling the Traffic Through a Service Mesh (Istio)

Controlling the traffic by increasing and decreasing the number of Pods is silly. Imagine that we would like to start with ten percent of the traffic going to the new release. To do that, we'd need to have nine replicas of the old release and one of the new. While that might, somehow, work if we truly need ten replicas in total, more often than not, the number of replicas we need to run an application will not match the rollout increments.

We can solve that problem with Ingress controllers. Most of them can be configured to send specific percentage of requests to specific services. That's not a great idea either since not all applications might be exposed through Ingress. We might have one backend application that talks to another directly, without going through Ingress.

A better way to solve that issue is through a **Service Mesh**.

<img src="/logo/istio.png" style="width:25%; float:right; padding: 10px">

We'll use Istio today, but the logic should be the same for any other Service Mesh, and even for Ingress in case you do not want to use a Service Mesh.

Here's an example of Istio VirtualService.

```sh
cat kustomize/overlays/istio/virtualservice-01.yaml
```

The output is as follows.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: silly-demo-1
spec:
  gateways:
    - silly-demo-gateway
  hosts:
    - silly-demo.34.139.252.110.nip.io
  http:
    - name: primary
      route:
        - destination:
            host: silly-demo-stable
            port:
              number: 8080
          weight: 100
        - destination:
            host: silly-demo-canary
            port:
              number: 8080
          weight: 0
```

The only thing that matters in that definition is that all the traffic will be sent to the `host` `silly-demo-stable`. We can see that through the `weight` set to `100` percent. The second `destination` is `silly-demo-canary` with the `weight` set to `0`, meaning that no traffic will be sent to it.

You can probably guess where this is going.

```sh
cat kustomize/overlays/istio/rollout.yaml
``` 

The output is as follows.

```yaml
---
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: silly-demo
spec:
  replicas: 5
  strategy:
    canary:
      canaryService: silly-demo-canary
      stableService: silly-demo-stable
      trafficRouting:
        istio:
          virtualServices:
            - name: silly-demo-1
              routes:
                - primary
            - name: silly-demo-2
              routes:
                - secondary
      steps:
        - setWeight: 20
        - pause: {}
        - setWeight: 40
        - pause: {duration: 10}
        - setWeight: 60
        - pause: {duration: 10}
        - setWeight: 80
        - pause: {duration: 10}
  revisionHistoryLimit: 2
  selector:
    matchLabels:
      app: silly-demo
  workloadRef:
    apiVersion: apps/v1
    kind: Deployment
    name: silly-demo
```

Time time, we are instructing Argo Rollouts to use service `silly-demo-canary` for canary releases and `silly-demo-stable` for, as the name says, stable releases.

Further on, we're telling it to do `trafficRouting` using `istio` `virtualServices`.

The rest of the manifest is exactly the same as before. The only important difference is that, this time, it should not manipulate the number of Pods but, instead, use Istio to set weight on the virtual services.

Let's apply it.

```sh
kubectl --namespace a-team apply \
    --kustomize kustomize/overlays/istio
```

The output of Argo Rollouts is as follows.

```
Name:            silly-demo
Namespace:       a-team
Status:          ✔ Healthy
Strategy:        Canary
  Step:          8/8
  SetWeight:     100
  ActualWeight:  100
Images:          ghcr.io/vfarcic/silly-demo:1.4.126 (stable)
Replicas:
  Desired:       5
  Current:       5
  Updated:       5
  Ready:         5
  Available:     5

NAME                                    KIND        STATUS     AGE  INFO
⟳ silly-demo                            Rollout     ✔ Healthy  12s
└──# revision:1
   └──⧉ silly-demo-5c5547db68           ReplicaSet  ✔ Healthy  11s  stable
      ├──□ silly-demo-5c5547db68-4fjw2  Pod         ✔ Running  11s  ready:2/2
      ├──□ silly-demo-5c5547db68-7bfx8  Pod         ✔ Running  11s  ready:2/2
      ├──□ silly-demo-5c5547db68-9r6lx  Pod         ✔ Running  11s  ready:2/2
      ├──□ silly-demo-5c5547db68-hbjvl  Pod         ✔ Running  11s  ready:2/2
      └──□ silly-demo-5c5547db68-qcbc8  Pod         ✔ Running  11s  ready:2/2
```

This is the first release so there's not much to look at since first releases are always rolled out right away.

So, let's make a second release by changing the tag of the image,...

```sh
cd kustomize/overlays/istio

kustomize edit set image \
    ghcr.io/vfarcic/silly-demo=ghcr.io/vfarcic/silly-demo:1.4.127

cd ../../../
```

...and re-applying the manifests.

```sh
kubectl --namespace a-team apply \
    --kustomize kustomize/overlays/istio
```

The output of Argo Rollouts is as follows.

```
Name:            silly-demo
Namespace:       a-team
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
  Step:          1/8
  SetWeight:     20
  ActualWeight:  20
Images:          ghcr.io/vfarcic/silly-demo:1.4.126 (stable)
                 ghcr.io/vfarcic/silly-demo:1.4.127 (canary)
Replicas:
  Desired:       5
  Current:       6
  Updated:       1
  Ready:         6
  Available:     6

NAME                                    KIND        STATUS     AGE    INFO
⟳ silly-demo                            Rollout     ॥ Paused   2m24s
├──# revision:2
│  └──⧉ silly-demo-5d574b5f4f           ReplicaSet  ✔ Healthy  15s    canary
│     └──□ silly-demo-5d574b5f4f-cxwnl  Pod         ✔ Running  15s    ready:2/2
└──# revision:1
   └──⧉ silly-demo-5c5547db68           ReplicaSet  ✔ Healthy  2m23s  stable
      ├──□ silly-demo-5c5547db68-4fjw2  Pod         ✔ Running  2m23s  ready:2/2
      ├──□ silly-demo-5c5547db68-7bfx8  Pod         ✔ Running  2m23s  ready:2/2
      ├──□ silly-demo-5c5547db68-9r6lx  Pod         ✔ Running  2m23s  ready:2/2
      ├──□ silly-demo-5c5547db68-hbjvl  Pod         ✔ Running  2m23s  ready:2/2
      └──□ silly-demo-5c5547db68-qcbc8  Pod         ✔ Running  2m23s  ready:2/2
```

On the first look, the outcome is exactly the same. It rolled out to twenty percent of requests and it is now waiting for us to do whatever we need to do to validate it before promoting it to the next step.

The change, however, is in the virtual service we applied, so let's take a look at it.

```sh
kubectl --namespace a-team get virtualservice silly-demo-1 \
    --output yaml
```

The output is as follows (truncated for brevity).

```yaml
apiVersion: networking.istio.io/v1
kind: VirtualService
metadata:
  ...
  name: silly-demo-1
  ...
spec:
  ...
  http:
  - name: primary
    route:
    - destination:
        host: silly-demo-stable
        port:
          number: 8080
      weight: 80
    - destination:
        host: silly-demo-canary
        port:
          number: 8080
      weight: 20
```

We can see that Argo Rollouts changed the weight. The `silly-demo-stable` destination `weight` has been changed from *100* to `80` while the other one was increased to `20`. As a result, now it does not matter how many replicas we are running since that exact percentage of requests will be forwarded to one release or the other.

We can confirm that by, let's say, sending twenty requests to the application.

```sh
for i in {1..20}; do
    curl "http://silly-demo.$ISTIO_HOST"
done
```

The output is as follows.

```
This is a silly demo version 1.4.126
This is a silly demo version 1.4.126
This is a silly demo version 1.4.126
This is a silly demo version 1.4.126
This is a silly demo version 1.4.127
This is a silly demo version 1.4.126
This is a silly demo version 1.4.127
This is a silly demo version 1.4.126
This is a silly demo version 1.4.126
This is a silly demo version 1.4.127
This is a silly demo version 1.4.126
This is a silly demo version 1.4.126
This is a silly demo version 1.4.127
This is a silly demo version 1.4.126
This is a silly demo version 1.4.126
This is a silly demo version 1.4.126
This is a silly demo version 1.4.126
This is a silly demo version 1.4.126
This is a silly demo version 1.4.126
This is a silly demo version 1.4.126
```

We can see that, approximately, twenty percent of those twenty requests are coming from the release `127` while the rest keeps coming from the previous release `126`.

Please note that I said "approximately" instead of exactly since those releases might be receiving requests from other places so our final number might be skewed.

The rest follows the same pattern as before.

We can promote the canary to the next step,...

```sh
kubectl argo rollouts --namespace a-team promote silly-demo
```

...and the virtual services will be automatically updated with new weights.

To be on the safe side, let's send another round of requests.

```sh
for i in {1..20}; do
    curl "http://silly-demo.$ISTIO_HOST"
done
```

The output is as follows.

```
This is a silly demo version 1.4.126
This is a silly demo version 1.4.126
This is a silly demo version 1.4.126
This is a silly demo version 1.4.127
This is a silly demo version 1.4.126
This is a silly demo version 1.4.126
This is a silly demo version 1.4.126
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.126
This is a silly demo version 1.4.127
This is a silly demo version 1.4.126
This is a silly demo version 1.4.126
This is a silly demo version 1.4.126
This is a silly demo version 1.4.126
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.126
This is a silly demo version 1.4.127
```

We can see that, this time, the number of responses from the `1.4.127` release now increased.

If we wait long enough... and send another round of requests,...

```sh
for i in {1..20}; do
    curl "http://silly-demo.$ISTIO_HOST"
done
```

The output is as follows.

```
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
This is a silly demo version 1.4.127
```

We can see that all the requests are now coming from *1.4.127*.

Now, to be clear, we are currently using the easiest setup of Istio virtual services. There are many more sophisticated ways we could be deciding who sees the new and who sees the old release. We could be rolling it out to selected users, or users in specific location, or to users filtered through some other criteria. The only limitation are capabilities of the Service Mesh or Ingress we chose. Nevertheless, I feel that we made the point that Service Mesh or Ingress is a valuable addition to our progressive delivery setup.

While we made an improvement, that is still not what we should do. We can do better, much better than just adding a Service Mesh to the mix.

## Testing by Observing Metrics

You or someone in your company is almost certainly observing the state of production and reacting when bad things are spotted. Users are performing some actions and we are, through metrics, traces, and logs validating whether the system works as expected. That's, by definition, testing, even though we might not call it like that.

What matters, for today's subject, is that we should add metrics to the mix.

We'll start by executing `hey` to send traffic to our application. That won't give us "real user" behavior but, since this is a demo, we can use imagination and pretend that's the "real" traffic.

> Execute the command that follows in the third terminal session

```sh
hey -z 60m "http://silly-demo.$ISTIO_HOST"
```

<img src="/logo/prometheus.png" style="width:25%; float:right; padding: 10px">

If this would be a "real" production, we would be observing the state of our system in, let's say, Grafana. We would have dashboards that execute queries to, let's say, Prometheus, and "paint" the results as dashboards. We'll skip Grafana today and go straight into Prometheus.

```sh
echo "http://prometheus.$INGRESS_HOST"
```

> Open the URL from the output in a browser.

One way to deduce whether an application is behaving as expected could be to retrieve a sum of the rate of requests going into it. We can do that with the following query.

> Execute `sum(irate(istio_requests_total{reporter="source",destination_service=~"silly-demo-stable.a-team.svc.cluster.local"}[5m]))` query in the Prometheus Web UI.

[](prometheus-02.png)

That is the sum of the rate of requests generated over the last five minutes.

However, that alone is not enough. We should also be able to filter requests based on response code so that, let's say, we retrieve only those that are NOT in the five hundred range.

> Execute `sum(irate(istio_requests_total{reporter="source",destination_service=~"silly-demo-stable.a-team.svc.cluster.local",response_code!~"5.*"}[5m]))` query in the Prometheus Web UI

[](prometheus-03.png)

If we combine those two, we should be able to get the percentage of successful requests.

> Execute `sum(irate(istio_requests_total{reporter="source",destination_service=~"silly-demo-stable.a-team.svc.cluster.local",response_code!~"5.*"}[5m])) / sum(irate(istio_requests_total{reporter="source",destination_service=~"silly-demo-stable.a-team.svc.cluster.local"}[5m]))` query in the Prometheus Web UI.

[](prometheus-04.png)

Right now, the output is `1` meaning that hundred percent of requests are successful.

There are many other queries we should be executing to find out whether a release running in production is behaving as expected and that's probably what we should be doing during progressive delivery. That sounds like a much better method that just letting our users discover that there's something wrong and send us "angry" support tickets. That would be a proactive way to deduce whether to continue rolling forward or to roll back.

Still, we can do much better than that.

## Automating Progressive Delivery Decision Making

Once we find out what we are observing and which queries give us confidence that our applications are behaving as expected, we are likely to realize that's too much work. Who wants to sit on front of a laptop constantly executing queries or watching dashboards? A rollout can last minutes, hours, or, in some cases, even days. It would be ridiculous to work in shifts to deduce whether a release is working as expected while knowing all that time is spent on repetitive tasks. When something is repetitive, it can be automated.

We should be able to instruct Argo Rollouts to run the same queries we would normally run and use the results to decide whether to roll forward or to roll back.

We can do that by modifying the *Rollout* definition or, even better, separate analysis from rollouts given that different applications are likely going to use some if not all the same analysis.

Here's an example of a ClusterAnalysisTemplate.

```sh
cat cluster-analysis-template.yaml
```

The output is as follows.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ClusterAnalysisTemplate
metadata:
  name: success-rate
spec:
  args:
  - name: service-name
  - name: prometheus-addr
    value: http://kube-prometheus-stack-prometheus.monitoring
  - name: prometheus-port
    value: "9090"
  metrics:
  - name: success-rate
    successCondition: result[0] >= 0.95
    provider:
      prometheus:
        address: "{{args.prometheus-addr}}:{{args.prometheus-port}}"
        query: |
          sum(irate(
            istio_requests_total{reporter="source",destination_service="{{args.service-name}}",response_code!~"5.*"}[5m]
          )) /
          sum(irate(
            istio_requests_total{reporter="source",destination_service="{{args.service-name}}"}[5m]
          ))
```

The name is set to `success-rate`. Remember that since we'll have to reference it in our Rollout.

Inside the spec, we are defining a few arguments like the `service-name`. That one does not have a value since we'll set it when we reference that analysis in the Rollout. There is also `prometheus-addr` and `prometheus-port` that are used to point the analysis to our Prometheus instance.

The real action is happening in the `metrics`. We can have many but, for today, one should be enough to demonstrate how it all works.

We are setting the `successCondition` to be equal or greater than `0.95` or ninety five percent. That means that if the result of a query is within that threshold it will be considered a success and the rollout should be able to continue. Otherwise, if it's below that threshold it will be considered a failure and eligible for a rollback.

The rest should be self-explanatory. We're using `prometheus` in the `providers` (there are many others), and the `query` is almost the same as the one we executed earlier. The only difference is that instead of a hard-coded value for the `destination_service` we are using the `service-name` argument. That way, that same analysis template can be used for multiple applications.

Let's apply it,...

```sh
kubectl apply --filename cluster-analysis-template.yaml
```

...and take a look at a modified version of the `rollout`.

```sh
cat kustomize/overlays/istio-prometheus/rollout.yaml
```

The output is as follows (truncated for brevity).

```yaml
---
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: silly-demo
spec:
  replicas: 5
  strategy:
    canary:
      ...
      analysis:
        templates:
          - templateName: success-rate
            clusterScope: true
        startingStep: 2
        args:
          - name: service-name
            value: silly-demo-canary.a-team.svc.cluster.local
      steps:
        - setWeight: 20
        - pause: {duration: 300}
        - setWeight: 40
        - pause: {duration: 300}
        ...
```

This time, we added the `analysis` section that is referencing the `success-rate` template we just applied. The `startingStep` is set to `2` meaning that the analysis will start only after the rollout reaches the second step which is the `pause` right after the weight is set to `20` percent. As a result, the analysis will start only after twenty percent of users start seeing the new release.

Finally, we changed the `pause` `duration` to `300` seconds, or five minutes. Since the Prometheus query is set to take five minutes into the account, anything less than that could produce false sense of security. You'll probably want to set that value to a much higher number given that five minutes of interaction with the application might not be enough to convince you that it's working as expected.

Let's apply that rollout.

```sh
kubectl --namespace a-team apply \
    --kustomize kustomize/overlays/istio-prometheus
```

The output of Argo Rollouts is as follows (truncated for brevity).

```
NAME                                    KIND         STATUS        AGE    INFO
⟳ silly-demo                            Rollout      ॥ Paused      8m15s
├──# revision:3
│  ├──⧉ silly-demo-5c5547db68           ReplicaSet   ✔ Healthy     8m14s  canary
│  │  ├──□ silly-demo-5c5547db68-lqbdb  Pod          ✔ Running   5m69s    ready:2/2
│  │  └──□ silly-demo-5c5547db68-w5wff  Pod          ✔ Running   5m5s     ready:2/2
│  └──α silly-demo-5c5547db68-3         AnalysisRun  ✔ Successful  5s     ✔ 1
└──# revision:2
   └──⧉ silly-demo-5d574b5f4f           ReplicaSet   ✔ Healthy     8m     stable
      ├──□ silly-demo-5d574b5f4f-vl7ft  Pod          ✔ Running     7m59s  ready:2/2
      ├──□ silly-demo-5d574b5f4f-7997c  Pod          ✔ Running     7m43s  ready:2/2
      ├──□ silly-demo-5d574b5f4f-bwwx7  Pod          ✔ Running     7m30s  ready:2/2
      ├──□ silly-demo-5d574b5f4f-qnzck  Pod          ✔ Running     7m16s  ready:2/2
      └──□ silly-demo-5d574b5f4f-6mr6p  Pod          ✔ Running     7m3s   ready:2/2
```

If we wait for five minutes or more, the output is almost the same as what we saw earlier when we were running it without the analysis. The only tangible difference, judging from the output, is that it does not pause indefinitely waiting for out blessing to progress to the next step. Instead, the pause lasts for five minutes before progressing to the next step that increments the weight.

The key difference is in the `AnalysisRun` entry. It shows that it was `Successful`. So far, the application is passing all verifications and, if we give it enough time, it will, eventually, be rolled out completely.

Let's spice it up a bit by applying a new release but, this time, we'll do it with a twist.

So, let's change the tag of the image,...

```sh
cd kustomize/overlays/istio-prometheus

kustomize edit set image \
    ghcr.io/vfarcic/silly-demo=ghcr.io/vfarcic/silly-demo:1.4.129

cd ../../../
```

...and apply the manifests.

```sh
kubectl --namespace a-team apply \
    --kustomize kustomize/overlays/istio-prometheus
```

Next, we'll stop *hey* that is generating traffic that should normally be generated by "real" users.

> Press `ctrl+c` to stop `hey` in the third terminal session

Finally, we'll execute `hey` again, but, this time, with `fail=true`. The application we're using is configured to simulate failure if that query parameter is in a request.

> Execute the command that follows in the third terminal session

```sh
hey -z 60m "http://silly-demo.$ISTIO_HOST?fail=true"
```

If we pay attention to Argo Rollouts output, it starts in the same way as before, by increasing the reach of the new release to `20` percent and, after that, it starts running the analysis.

The output of Argo Rollouts is as follows (truncated for brevity).

```
Name:            silly-demo
Namespace:       a-team
Status:          ✖ Degraded
Message:         RolloutAborted: Rollout aborted update to revision 4: Metric "succes
s-rate" assessed Failed due to failed (1) > failureLimit (0)
Strategy:        Canary
  Step:          0/8
  SetWeight:     0
  ActualWeight:  0
Images:          ghcr.io/vfarcic/silly-demo:1.4.126 (stable)
Replicas:
  Desired:       5
  Current:       5
  Updated:       0
  Ready:         5
  Available:     5

NAME                                    KIND         STATUS        AGE    INFO
⟳ silly-demo                            Rollout      ✖ Degraded    18m
├──# revision:4
│  ├──⧉ silly-demo-64b5875f59           ReplicaSet   • ScaledDown  9m30s  canary,dela
y:passed
│  └──α silly-demo-64b5875f59-4         AnalysisRun  ✖ Failed      8m26s  ✖ 1
├──# revision:3
│  ├──⧉ silly-demo-5c5547db68           ReplicaSet   ✔ Healthy     18m    stable
│  │  ├──□ silly-demo-5c5547db68-lqbdb  Pod          ✔ Running     16m    ready:2/2
│  │  ├──□ silly-demo-5c5547db68-w5wff  Pod          ✔ Running     15m    ready:2/2
│  │  ├──□ silly-demo-5c5547db68-5bln2  Pod          ✔ Running    14m24s  ready:2/2
│  │  ├──□ silly-demo-5c5547db68-tb4vh  Pod          ✔ Running    13m21s  ready:2/2
│  │  └──□ silly-demo-5c5547db68-5scvp  Pod          ✔ Running    12m17s  ready:2/2
│  └──α silly-demo-5c5547db68-3         AnalysisRun  ✔ Successful  10m    ✔ 1
└──# revision:2
   └──⧉ silly-demo-5d574b5f4f           ReplicaSet   • ScaledDown  13m
```

After a while (five minutes or more), the *AnalysisRun* changed to `Failed`. The percentage of failed requests reached a number higher than five percent and, as we already saw, that's the threshold we defined. As a result, it scaled down the new release to zero replicas, and changed the weight of the virtual service to redirect all the traffic to the new release.

We succeeded.

![](/logo/argo-rollouts.jpg?height=15vw&classes=inline)
![](/logo/istio.png?height=15vw&classes=inline)
![](/logo/prometheus.png?height=15vw&classes=inline)

We are now testing in production by performing canary deployments with Argo Rollouts, Istio, and Prometheus metrics.

That is how fully automated testing in production is done.

Thank you for watching.
See you in the next one.
Cheers.

## Destroy

> Stop the processes in the second and the third terminal session by pressing `ctrl+c`.

```sh
chmod +x destroy.nu

./destroy.nu
```

> Execute the command that follows in all terminal sessions.

```sh
exit
```


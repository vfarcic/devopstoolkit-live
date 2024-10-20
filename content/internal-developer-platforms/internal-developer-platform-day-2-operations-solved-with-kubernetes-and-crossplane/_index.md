
+++
title = 'Internal Developer Platform Day 2 Operations Solved with Kubernetes and Crossplane'
date = 2023-10-20T16:00:00+00:00
draft = false
+++

I did it. No! We did it. Actually, that's not correct either. Someone else did it. Doesn't matter who did it. What matters is that one of, in my opinion, big problems has been solved. Developers can now not only create, update, and remove their applications and infrastructure, but they can also get to know **what's happening with the resources they are managing**.

<!--more-->

{{< youtube KLHNrLWmBfw >}}

So far, in most, cases, if we work very very very hard, we could enable developers to create something meaningful. Most of us do that by creating abstractions. Instead of forcing developers to spend years trying to understand intricacies of Kubernetes, we create Helm charts that enable developers to modify a simple YAML values file and apply charts to the cluster. If we are advanced, we create CRDs and controllers that result in even better abstractions and start resembling services that developers can consume. We progress further from there by creating services not only for applications but also for databases, clusters, or anything else. We, platform engineers, become service providers and developers start consuming those services.

Here's the problem though. All that tend to help only with day zero operations. Developers can create or update something in a very easy way but when it gets to operating and observing that something, they often end up being equally confused as they were before. Our services **do not show the information** they need so they need to dig deeper into lower-level resources to find out what's going on. That negates some of the main reasons we started offering them services. That's horrible. We're building something that looks useful when someone starts using it and it turns **useless** afterwards.

We need to change that and now we can do just that. Let me explain.

## Setup

```sh
git clone https://github.com/vfarcic/crossplane-sql

cd crossplane-sql

git pull

git fetch

git checkout status-transformer
```

> Make sure that Docker is up-and-running. We'll use it to create a KinD cluster.

> Watch [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell

chmod +x examples/setup.nu 

./examples/setup.nu

source .env

kubectl delete \
    --filename examples/provider-config-$HYPERSCALER.yaml

kubectl --namespace infra apply \
    --filename examples/$HYPERSCALER-secret.yaml
```

> Execute the command that follows only if you are using **AWS**

```sh
export MANAGED_RESOURCE=vpc.ec2.aws.upbound.io
```

> Execute the command that follows only if you are using **Google Cloud**

```sh
export MANAGED_RESOURCE=databaseinstance.sql.gcp.upbound.io
```

> Execute the command that follows only if you are using **Azure**

```sh
export MANAGED_RESOURCE=resourcegroup.azure.upbound.io
```

## The Problem In Kubernetes

Let me show what the problem is by showing you a relatively simple definition of an application that should be running in Kubernetes.

*Before we proceed, if you saw [Kubernetes Events Are Broken (If You Are Building a Developer Portal)](https://youtu.be/xAl3TAfFE_M), you might think that its deja vu. You might think you already heard me talking about this subject. That's partly true. This is a continuation. Back then, I had complaints. Today, I'll show the solution but, to get there, I might need to explain the problem again. I'll be brief. I promise.*

Here's the definition.

```sh
cat app/error.yaml
```

The output is as follows.

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: silly-demo
  labels:
    app.kubernetes.io/name: silly-demo
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: silly-demo
  template:
    metadata:
      labels:
        app.kubernetes.io/name: silly-demo
    spec:
      containers:
      - name: main
        image: "ghcr.io/vfarcic/silly-demo:this-tag-does-not-exist"
        ports:
        - containerPort: 8080
        resources:
          limits:
            cpu: 500m
            memory: 512Mi
          requests:
            cpu: 250m
            memory: 256Mi
        livenessProbe:
          httpGet:
            path: /
            port: 8080
        readinessProbe:
          httpGet:
            path: /
            port: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: silly-demo
  labels:
    app.kubernetes.io/name: silly-demo
spec:
  type: ClusterIP
  ports:
  - port: 8080
    targetPort: 8080
    protocol: TCP
    name: http
  selector:
    app.kubernetes.io/name: silly-demo
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: silly-demo
  labels:
    app.kubernetes.io/name: silly-demo
  annotations:
    ingress.kubernetes.io/ssl-redirect: "false"
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: ImplementationSpecific
        backend:
          service:
            name: silly-demo
            port:
              number: 8080
    host: silly-demo.127.0.0.1.nip.io
```

This is a very simple definition of an application which, with a little patience, we can teach developers how to write themselves.

We have a `Deployment` that defines our application. That's where we define which container `image` should run, which `ports` are exposed, how much `cpu` and `memory` it should consume, and so on and so forth. Then there is the `Service` which defines how other applications can communicate with that one and, finally, there is `Ingress` that specifies the `host` through which our application is exposed.

Even though those resource definitions might be a bit verbose, almost anyone can learn how to write them. I don't see a problem with that. However, there are at least two problems with it.

To begin with, our applications are often more complicated than that. There could be horizontal or vertical Pod autoscalers. There could be virtual services from a service mesh. There could be network policies. There could be... You get the point. It could be simple, or it could be complex to define and manage all those resources.

That problem can be solved in many different ways. We could create Helm charts so that people can focus only on things that matter and those "things" being defined in *values.yaml*. Alternatively, we could accomplish a similar result with CUE with Timoni, Carvel YTT, KCL, or through many other tools that all serve the same purpose. They all allow us to create templates, of some sort or another, and focus on customizations by changing parameters, values, or some other type of file.

That's not the problem we're solving today, especially since I believe that they are all wrong because we should be creating Custom Resource Definitions (CRDs) and controllers.

The second problem, the one we're exploring today, can be described as "**What the heck do we do after day zero operations?**" It's relatively easy to change a few manifests or a Helm *values.yaml* file or a custom resource or whatever else you might be using. What is much harder is to deduce the status of something running in or being managed by Kubernetes. It's much harder to understand "**what's the cause of an issue and how to fix it?**" Whatever we do during day zero is much easier than what we do afterwards.

Let me apply those resources as a way to demonstrate the issues I'm talking about.

```sh
kubectl --namespace infra apply --filename app/error.yaml
```

Now, imagine that you are not a person who neglected their family so that you can spend an infinite amount of time learning Kubernetes. Imagine that you were told to define a Kubernetes Deployment (and a few other simple things) or that you changed a few Helm values or something similar. Imagine that you are not a Kubernetes expert but that you just followed instructions from a platform engineer who decided to enable you to do it all by yourself. In this example, you defined a Deployment to the best of your abilities and applied it to the cluster. It's only natural to take a look at it to deduce whether it's working as expected, so let's do just that.

```sh
kubectl --namespace infra get deployments
```

The output is as follows.

```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
silly-demo   0/1     1            0           52s
```

Okay. That clearly shows that something is not ready. Since you're a fast learner, you figured out, or you were told to, `describe` something to get more information.

```sh
kubectl --namespace infra describe deployment silly-demo
```

The output is as follows (truncated for brevity).

```
...
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      False   MinimumReplicasUnavailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  <none>
NewReplicaSet:   silly-demo-7f8c98455b (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  70s   deployment-controller  Scaled up replica set silly-demo-7f8c98455b to 1
```

Most of the output is the same information we defined but you noticed that there are `Conditions` and `Events`. Based on that information, you can easily conclude that everything is okay. There's nothing to worry about except that the `Available` condition is set to `False` indicating that there is no minimum number of replicas (`MinimumReplicasUnavailable`).

What the heck! Only Kubernetes can claim that everything is okay but something might be wrong yet there is no indication of what that something might be.

The reality is that you would need to know Kubernetes much more than what a person who did not choose to get divorced because it's better to spend endless hours learning Kubernetes on top of doing the "real" job of writing NodeJS code than spending time with your wife. You would need to know that a Deployment creates a ReplicaSet which will also not give you any meaningful information because it created Pods which tried to create containers which, in this case, are not working.

Here's the proof.

```sh
kubectl --namespace infra get pods
```

The output is as follows.

```
NAME                          READY   STATUS             RESTARTS   AGE
silly-demo-7f8c98455b-djnfb   0/1     ImagePullBackOff   0          89s
```

That Pod, which was created by a ReplicaSet which was created by the Deployment you wrote could not pull the image (`ImagePullBackOff`).

We can confirm that by describing that specific Pod.

```sh
kubectl --namespace infra describe pod \
    --selector app.kubernetes.io/name=silly-demo
```

The output is as follows (truncated for brevity).

```
...
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       False 
  ContainersReady             False 
  PodScheduled                True 
Volumes:
  kube-api-access-7mhnt:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                 From               Message
  ----     ------     ----                ----               -------
  Normal   Scheduled  110s                default-scheduler  Successfully assigned infra/silly-demo-7f8c98455b-djnfb to kind-control-plane
  Normal   Pulling    15s (x4 over 109s)  kubelet            Pulling image "ghcr.io/vfarcic/silly-demo:this-tag-does-not-exist"
  Warning  Failed     15s (x4 over 109s)  kubelet            Failed to pull image "ghcr.io/vfarcic/silly-demo:this-tag-does-not-exist": rpc error: code = NotFound desc = failed to pull and unpack image "ghcr.io/vfarcic/silly-demo:this-tag-does-not-exist": failed to resolve reference "ghcr.io/vfarcic/silly-demo:this-tag-does-not-exist": ghcr.io/vfarcic/silly-demo:this-tag-does-not-exist: not found
  Warning  Failed     15s (x4 over 109s)  kubelet            Error: ErrImagePull
  Normal   BackOff    3s (x6 over 108s)   kubelet            Back-off pulling image "ghcr.io/vfarcic/silly-demo:this-tag-does-not-exist"
  Warning  Failed     3s (x6 over 108s)   kubelet            Error: ImagePullBackOff
```

We can see, in the `Conditions`, that the containers are not ready (`ContainersReady` set to `False`). Further down, we can see in the `Events` that it could not pull the image `silly-demo` with the tag `this-tag-does-not-exist`.

If you are a "Kubernetes ninja" it is easy to deduce all that, mostly because that was a simple scenario. Even those experienced with Kubernetes can easily get lost when faced with more complicated situations. A person who was given instructions to write a simple Deployment or who was told to modify Helm values or who was instructed to write a Custom Resource might **not be able to do anything** but cry.

What we need is **propagation of specific events, statuses, and logs** to the parent resource. Whomever applied that Deployment should be able to see that there is something wrong with the image. That person should be able to see what matters to them, and nothing else. A platform engineer that is in charge of internal services should have done that, and the only question is "How?"

Now, this is a good news bad news type of a situation. I do not have a solution for Kubernetes in general, but I do have a solution for those writing your own Custom Resource Definitions (CRDs) and controllers or, even better, those using Crossplane to do that. Let's take a look at the problem from a different angle.

## The Problem With Custom Resources

Here's yet another resource but, this time, based on a Custom Resource Definition I created through Crossplane Compositions.

```sh
cat examples/$HYPERSCALER-error.yaml
```

The output is as follows.

```yaml
apiVersion: devopstoolkitseries.com/v1alpha1
kind: SQLClaim
metadata:
  name: my-db
spec:
  id: my-db-20240925155601
  compositionSelector:
    matchLabels:
      provider: azure
      db: postgresql
  parameters:
    version: '11'
    size: small
    region: otherregion
    databases:
      - db-01
      - db-02
```

That's a very simple YAML definition based on a CRD I made with Crossplane. It enables developers to manage `postgresql` servers, `databases`, schemas, and quite a few other things in AWS, Google Cloud, and, as in this case, `azure`.

I won't go into details since I already explored Crossplane in quite a few videos in this channel. The only thing relevant for today's discussion is that developers can easily create, update, or delete PostgreSQL, but we are yet to discover whether they can do day 2 operations for which they might need to see the status, relevant events, and a few other things without being overwhelmed with low-level details. After all, one of the main reasons for creating such abstractions is to surface things that matter and hide those that don't.

*Before we proceed, let me stress out that today I am using Azure but the instructions are made for any of the "big three" hyperscalers. So, if you're following along, you should be able to get a similar result in AWS and Google Cloud as well.*

Now, let's apply that resource and...

```sh
kubectl --namespace infra apply \
    --filename examples/$HYPERSCALER-error.yaml
```

...see what we'll get.

```sh
kubectl --namespace infra get sqlclaims
```

The output is as follows.

```
NAME    SYNCED   READY   CONNECTION-SECRET   AGE
my-db   True     False                       17s
```

So far, everything seem to be working correctly. The database server and other resources are not yet ready but that's to be expected since it might take a while until everything is done.

*Before we proceed, let me stress that I am fully aware that not all developers like using *kubectl*. Some might prefer using a Web UI like Backstage or Port. Others might like sticking with plugins in VS Code. There is an infinite number of ways people might want to interact with control planes. That should not matter for today's story since the question is whether the information people need is available at the right level of abstractions. It should be relatively easy to display the same info anywhere, as long as the info exists at the right place.*

Okay. Since listing the resources seem to show that everything is working, but not necessarily ready, let's dive deeper and describe it.

```sh
kubectl --namespace infra describe sqlclaim my-db
```

The output is as follows (truncated for brevity).

```
...
Status:
  Conditions:
    Last Transition Time:  2024-09-25T14:08:10Z
    Reason:                ReconcileSuccess
    Status:                True
    Type:                  Synced
    Last Transition Time:  2024-09-25T14:08:10Z
    Message:               Claim is waiting for composite resource to become Ready
    Reason:                Waiting
    Status:                False
    Type:                  Ready
Events:
  Type    Reason                 Age                From                                                             Message
  ----    ------                 ----               ----                                                             -------
  Normal  BindCompositeResource  44s                offered/compositeresourcedefinition.apiextensions.crossplane.io  Successfully bound composite resource
  Normal  BindCompositeResource  43s (x8 over 44s)  offered/compositeresourcedefinition.apiextensions.crossplane.io  Composite resource is not yet ready
```

Everything seems to be working correctly, so far. It's `waiting for composite resource to become Ready` but, as I already mentioned, that is not necessarily a problem since it might take a while. Even if it is an issue, how would a developer know what is the `composite resource` and what is not `Ready` and why it's not ready? Based on the current information, the only conclusion a developer could make is that, so far, all is good and the only thing missing is a bit of patience.

Now, let's change roles and assume that we are a person who created that Composition. We are now a person who understands Kubernetes, Crossplane, Azure or whichever hyperscaler is used, and everything in between.

Such a person might execute the following command to see what's going on with the resources created from that claim.

```sh
crossplane beta trace sqlclaim my-db --namespace infra
```

The output is as follows.

```
NAME                                            SYNCED   READY   STATUS
SQLClaim/my-db (infra)                          True     False   Waiting: Claim is waiting for composite resource to become Ready
└─ SQL/my-db-fmfd2                              True     False   Creating: Unready resources: firewall-rule, resourcegroup, and server
   ├─ ResourceGroup/my-db-20240925155601        False    -       ReconcileError: ...viderConfig: ProviderConfig.azure.upbound.io "default" not found
   ├─ FirewallRule/my-db-20240925155601         False    -       ReconcileError: ...viderConfig: ProviderConfig.azure.upbound.io "default" not found
   ├─ Server/my-db-20240925155601               False    -       ReconcileError: ...viderConfig: ProviderConfig.azure.upbound.io "default" not found
   ├─ ProviderConfig/my-db-20240925155601-sql   -        -
   ├─ ProviderConfig/my-db-20240925155601-sql   -        -
   ├─ Database/my-db-20240925155601-db-01       False    False   ...
   ├─ Database/my-db-20240925155601-db-02       False    False   ...
   └─ ProviderConfig/my-db-20240925155601       -        -
```

Look at that? The `SQLClaim` created by the developer created the `SQL` composition which created a `ResourceGroup`, and a `FirewallRule`, and a `Server`, and some `ProviderConfig`, and `Database` resources. The "expert" understands what those are, how they work, why there are being created, and anything else related to running PostgreSQL server, databases, and other resources in Azure and beyond. That cannot be said for the developer. They created a claim that is an abstraction and it would NOT be reasonable to expect the same level of understanding of low-level resources created from it.

The "expert" can immediately see that there is a `ReconcileError`. Something's wrong with the `ProviderConfig`. It's nowhere to be found.

The "expert" would probably `describe` one of the resources created by that claim.

```sh
kubectl describe $MANAGED_RESOURCE my-db
```

The output is as follows (truncated for brevity).

```
...
API Version:  azure.upbound.io/v1beta1
Kind:         ResourceGroup
...
Status:
  At Provider:
  Conditions:
    Last Transition Time:  2024-09-25T14:08:10Z
    Message:               connect failed: cannot initialize the Terraform plugin SDK async external client: cannot get terraform setup: cannot get referenced ProviderConfig: ProviderConfig.azure.upbound.io "default" not found
    Reason:                ReconcileError
    Status:                False
    Type:                  Synced
Events:
  Type     Reason                   Age                 From                                                  Message
  ----     ------                   ----                ----                                                  -------
  Warning  CannotConnectToProvider  45s (x7 over 106s)  managed/azure.upbound.io/v1beta1, kind=resourcegroup  cannot initialize the Terraform plugin SDK async external client: cannot get terraform setup: cannot get referenced ProviderConfig: ProviderConfig.azure.upbound.io "default" not found
```

We can see that it `cannot get referenced ProviderConfig`. A person experienced with Crossplane would know immediately that the `default` provider is missing and that's the reason why `ResourceGroup` cannot be created.

A developer did not necessarily even know that there is a resource group.

We finished with the depressing part. Let's take a look at the solution.

## Status Propagation

*Before we proceed, let me stress that what you are about to see is a solution in Crossplane. If you are building CRDs and controllers or operators yourself, you should be able to replicate the same behavior. If you are using a third-party tool to do that, you might want to yell at the maintainers of the project you choose to implement something similar. In any case, think of what follows not only as a showcase of a solution in Crossplane but also as a path forward in other projects.*

Let's update the Composition to a newer version in which I implemented the solution.

```sh
yq --inplace \
    '.spec.package = "xpkg.upbound.io/devops-toolkit/dot-sql:v0.8.138"' \
    config.yaml

kubectl apply --filename config.yaml
```

Now we're changing the role again. From now on, we are a developer who created that claim.

Let's describe it, again.

```sh
kubectl --namespace infra describe sqlclaim my-db
```

The output is as follows (truncated for brevity).

```
...
Status:
  Conditions:
    Last Transition Time:  2024-09-25T14:08:10Z
    Reason:                ReconcileSuccess
    Status:                True
    Type:                  Synced
    Last Transition Time:  2024-09-25T14:08:10Z
    Message:               Claim is waiting for composite resource to become Ready
    Reason:                Waiting
    Status:                False
    Type:                  Ready
    Last Transition Time:  2024-09-25T14:10:55Z
    Message:               providerConfig is missing. Contact service owner.
    Reason:                FailedToConnect
    Status:                False
    Type:                  Developer
...
```

Look at that!

A new `Status` appeared automagically. Besides the massage that it `is waiting for composite resource to become Ready` that we saw before, now we have a new one. The type is `Developer` so that it's easy to see for whom it is. It says that it `FailedToConnect` because the `providerConfig is missing`. It instructs the developer to `Contact service owner`.

> If you do not see the *providerConfig is missing. Contact service owner* message, Crossplane probably did not yet to a new round of reconciliation. Wait for a few moments and *describe* the claim agian.

Typically, there are two types of issues when consuming services created by others, even when "others" are platform engineers in your company. In some cases, the service itself is not working correctly and only the service owner can fix it. Only the person or the team in charge of it can make it work. In some other cases, an error can be fixed by the service consumer, in this case the developer. That's similar to, let's say, consuming services from hyperscalers like AWS, Azure, and Google Cloud. If the whole zone goes down, they will let us know that they need to fix it. There's nothing we can do. On the other hand, if we do something wrong ourselves like, for example, specify a wrong region, they will not do anything. It's up to us to specify the correct one. More often than not, there is a clear division between responsabilities of the service provider and us.

This is a similar situation. There's nothing the developer can do to fix the issue so the only thing left is to contact whomever is in charge of it to fix it. We'll see the other case soon.

Bear in mind that the status we just saw does not come out of the box. It's not something baked into Crossplane but a conscious choice of the person who wrote the Composition, the service. The service owner.

For now, let's change the role again. Now we are the person in charge of it, the service owner. I just got an email from an angry developer saying that there is something wrong with the *providerConfig* and that we should fix it. There is also a footnote in that email stating that we should have a better alerting system and not wait for angry emails. They're right, but that would be a subject of a different post.

Since this is not a tutorial about Crossplane, I'll skip the explanation of what is needed and we'll just `apply` the fix.

```sh
kubectl apply \
    --filename examples/provider-config-$HYPERSCALER.yaml
```

The provider was created and the claim should, supposedly, maybe, hopefuly work. We'll see.

We reply back to the developer saying "It's fixed. You're good to go. Sorry for the inconvenience."

With that, we're switching the role back to the developer. What would that person do after receiving the email?

The logical course of action would be to verify whether the claim now works by describing it again, and fantasising how much nicer it would be if that information is in Backstage or Port. We should send an email to the platform engineering team to incorporate this into the portal and they might write a comment to this video asking Viktor to use that as the next subject.

```sh
kubectl --namespace infra describe sqlclaim my-db
```

The output is as follows (truncated for brevity).

```
...
Status:
  Conditions:
    Last Transition Time:  2024-09-25T14:08:10Z
    Reason:                ReconcileSuccess
    Status:                True
    Type:                  Synced
    Last Transition Time:  2024-09-25T14:08:10Z
    Message:               Claim is waiting for composite resource to become Ready
    Reason:                Waiting
    Status:                False
    Type:                  Ready
    Last Transition Time:  2024-09-25T14:12:28Z
    Message:               selected region otherregion is not available. Double check the `spec.parameters.region` value.
    Reason:                FailedToConnect
    Status:                False
    Type:                  Developer
...
```

> If you do not see the *selected region otherregion is not available...* message, Crossplane probably did not yet to a new round of reconciliation. Wait for a few moments and *describe* the claim again.

The previous message dissapeared or, to be more precise, it was replaced with a new one. Now it says that the `selected region otherregion is not available.`. That makes sense. Even if we are not proficient in Azure, it is highlighly unlikely that there is a region called *otherregion*. More importantly, this time, the message does not say that we should contact the service owner but instructs us to `Double check the spec.parameters.region value`. This is the case when it's not the fault of the service owner. The service or, in this case, the Composition works correctly. We made a mistake. There is no need to yell at anyone. We should own this one and just fix it by changing the region to a correct one.

Here's a modified version of the manifest.

```sh
cat examples/$HYPERSCALER.yaml
```

The output is as follows.

```yaml
apiVersion: devopstoolkitseries.com/v1alpha1
kind: SQLClaim
metadata:
  name: my-db
spec:
  id: my-db-20240925155601
  compositionSelector:
    matchLabels:
      provider: azure
      db: postgresql
  parameters:
    version: '11'
    size: small
    region: eastus
    databases:
      - db-01
      - db-02
```

The only difference is that, this time, we have `eastus` as the `region`. That sounds like a correct one and, at the same time, makes us want to have Backstage or Port integration even more since we could have a drop-down list of available regions instead of guessing it ourselves. Someone should definitely tell Viktor to explore it in one of the next videos.

Let's apply that change,...

```sh
kubectl --namespace infra apply \
    --filename examples/$HYPERSCALER.yaml
```

...and `describe` the claim again.

```sh
kubectl --namespace infra describe sqlclaim my-db
```

The output is as follows (truncated for brevity).

```
...
Status:
  Conditions:
    Last Transition Time:  2024-09-25T14:08:10Z
    Reason:                ReconcileSuccess
    Status:                True
    Type:                  Synced
    Last Transition Time:  2024-09-25T14:08:10Z
    Message:               Claim is waiting for composite resource to become Ready
    Reason:                Waiting
    Status:                False
    Type:                  Ready
    Last Transition Time:  2024-09-25T14:13:59Z
    Message:               So far so good
    Reason:                
    Status:                True
    Type:                  Developer
...
```

> If you do not see the *So far so good* message, Crossplane probably did not yet to a new round of reconciliation. Wait for a few moments and *describe* the claim again.

This time, we are greeted with the `So far so good` message indicating that everything seem to be working correctly, for now. Crossplane is, probably, creating the resource group, the firewall rule, the server, and all other resources we, the developers, should not care about.

A while later, if we list all the `sqlclaims`...

```sh
kubectl --namespace infra get sqlclaims
```

The output is as follows.

```
NAME    SYNCED   READY   CONNECTION-SECRET   AGE
my-db   True     True                        18m
```

...we can see that it is now `READY`.

> If the column `READY` is not `True`, some of the resources are still being created. It might take anything between a minute and ten minutes or more to create all the resources depending on which hyperscaler you're using.

Hurray!

The database is up-and-running. It was easy for the developer not only to create it, but also to see whether it's working and, if it's not, what the problem is and whether it is something they can fix themselves or contact someone. The service owner managed to propagate information that matters, and keep the details that don't to where they belong.

They lived happily ever after.

We saw the result of propagation of statuses to parent resources. Now it's time to see how it's done.

## How It's Done

The whole logic that propagates statuses from children to parent or root resources is done through the [Status Transformer Crossplane Function](https://github.com/crossplane-contrib/function-status-transformer). I'll show you how it works in a second, right after we go through a bit of history.

A while ago, I complained about Kubernetes event and status propagation. Those complaints resulted in the [Kubernetes Events Are Broken (If You Are Building a Developer Portal)](https://youtu.be/xAl3TAfFE_M) post which, essentially, claimed that without a mechanism to propagate events and statuses **we cannot build a developer platform** on top of Kubernetes without limiting ourselves to day zero operations. Long story short, that resulted in [this proposal](https://github.com/crossplane/crossplane/issues/5643) which was eventually picked up by the community and the end result in the function that enables us to filter, transform, and propagate statuses all the way to the top of the hierarchy tree to Composition resources and Claims.

Let's take a look at the Composite Definition that made it possible to create claims.

```sh
cat package/compositions.yaml
```

The output is as follows (truncated for brevity).

```yaml
...
  - step: statuses
    functionRef:
      name: crossplane-contrib-function-status-transformer
    input:
      apiVersion: function-status-transformer.fn.crossplane.io/v1beta1
      kind: StatusTransformation
      statusConditionHooks:
      - matchers:
        - resources:
          - name: resourcegroup
          conditions:
          - type: Synced
        setConditions:
        - target: CompositeAndClaim
          force: true
          condition:
            type: Developer
            status: 'True'
            message: So far so good
      - matchers:
        - resources:
          - name: resourcegroup
          conditions:
          - type: Synced
            status: 'False'
            reason: ReconcileError
            message: (.*)cannot get referenced ProviderConfig(.*)
        setConditions:
        - target: CompositeAndClaim
          force: true
          condition:
            type: Developer
            status: 'False'
            reason: FailedToConnect
            message: providerConfig is missing. Contact service owner.
      - matchers:
        - resources:
          - name: resourcegroup
          conditions:
          - type: Synced
            status: 'False'
            reason: ReconcileError
            message: (.*)cannot get referenced ProviderConfig(.*)
        setConditions:
        - target: CompositeAndClaim
          force: true
          condition:
            type: Developer
            status: 'False'
            reason: FailedToConnect
            message: providerConfig is missing. Contact service owner.
      - matchers:
        - resources:
          - name: resourcegroup
          conditions:
          - type: Synced
            status: 'False'
            reason: ReconcileError
            message: (.*)The specified location '(?P<Region>.*)' is invalid(.*)
        setConditions:
        - target: CompositeAndClaim
          force: true
          condition:
            type: Developer
            status: 'False'
            reason: FailedToConnect
            message: selected region {{ .Region }} is not available. Double check the `spec.parameters.region` value.
      - matchers:
        - resources:
          - name: resourcegroup
          conditions:
          - type: Synced
            status: 'False'
            reason: ReconcileError
            message: (.*)The provided location '(?P<Region>.*)' is not available for resource group(.*)
        setConditions:
        - target: CompositeAndClaim
          force: true
          condition:
            type: Developer
            status: 'False'
            reason: FailedToConnect
            message: selected region {{ .Region }} is not available. Double check the `spec.parameters.region` value.
  - step: automatically-detect-ready-composed-resources
    functionRef:
      name: crossplane-contrib-function-auto-ready
  writeConnectionSecretsToNamespace: crossplane-system
```

As I mentioned earlier, I won't go into details how Crossplane Compositions are built nor how Crossplane functions work. I already explored that in quite a few videos on this channel. Instead, I'll just explain the logic behind the Status Transformer Function.

There is a number of `statusConditionHooks`. They all rely on `matchers` to find conditions in specific resources and `setConditions` to create or update conditions in the Composition Resource and the Claim.

The first one, in this case, is a catch all matcher. If the Azure `resourcegroup` resource has the status type set to `Synced`, it sets the status of the `type` `Developer` to `True` with the message `So far so good`. Since conditions can be overwritten, that one will be displayed if none of the other matchers are met. In other words, if we do not find any issue in managed resources, that is the message developers will see.

Further on we have the second matcher that looks for the `reason` `ReconcileError` and the message that contains `cannot get referenced ProviderConfig`. That one uses RegEx so that we don't need to look for the exact message. If that condition is met, it will overwrite the previous status of the type `Developer` with the message `providerConfig is missing. Contact service owner.`

The rest of the matchers follow the same pattern.

The one with `The specified location` `message` is an interesting one since it extracts the `Region` which is later used to generate a dynamic `message` to end users. That message contains the value of the `Region` extracted earlier.

There are quite a few other ways we can propagate statuses and even events. I invite you to check the documentation.

For now, what matters, is that we can design services in Kubernetes based on CRDs and controllers or operators and that we need to think not only about day zero operations but the whole experience which, among other things, includes custom developer-friendly statuses and events. Those can be visualized in many different ways, be it through *kubectl* as we did today, or through Wen UIs or any other interface. As long as the data is available in the top resources, it should not be a problem to present it.

If you're creating CRDs and operators yourself from scratch, you might want to implement a similar logic. If you're using Crossplane, the Status Transformer Function does the heavy lifting and all you have to do is define matchers and set conditions that should be propagated to the top resource.

Thank you for watching.
See you in the next one.
Cheers.

## Destroy

```sh
chmox +x examples/destroy.nu

./examples/destroy.nu

yq --inplace \
    '.spec.package = "xpkg.upbound.io/devops-toolkit/dot-sql:v0.8.132"' \
    config.yaml
```

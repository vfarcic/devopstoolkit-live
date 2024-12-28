
+++
title = 'Is This the End of Crossplane? Compose Kubernetes Resources with kro'
date = 2024-12-30T14:00:00+00:00
draft = false
+++

Whomever is building developer platforms is bound to come to the conclusion that there is **a need to compose resources and expose those compositions through APIs**. If a developer needs a database, they should be able to specify what that database should be without having to deal with subnets, VPCs, internet gateways, and other lower-level components that are required, but are not important for the vast majority of people who just want a database. We are likely to come to a similar conclusion if, for example, a developer wants to run an application. That developer might want to specify a container image, a port, and a host without having to worry about Kubernetes Deployments, Services, Ingresses, Scalers, VirtualServices, and other lower-level Kubernetes types of objects.

<!--more-->

{{< youtube 8zQtpcxmdhs >}}

So, we need a way to create new **API endpoints** that will represent the right level of abstraction and that will, at the same time, compose lower-level resources based on higher-level resources created through those endpoints.

All of us trying to enable developers to be self-sufficient need something like that; me included. That's why I was very excited when I discovered a new project called [kro](https://kro.run).

> Do not try execute the commands in this section. They are only a preview of what's coming. We'll set up everything soon.

It allows us to enable developers to define something like this.

```sh
cat silly-demo-ingress.yaml
```

The output is as follows.

```yaml
apiVersion: kro.run/v1alpha1
kind: Application
metadata:
  name: silly-demo
spec:
  name: silly-demo
  image: ghcr.io/vfarcic/silly-demo
  tag: "1.4.305"
  ingress:
    enabled: true
    host: silly-demo.127.0.0.1.nip.io
```

When those ten or so lines of YAML are applied, we would get something like this.

```sh
kubectl --namespace a-team get all,ingresses
```

The output is as follows.

```
NAME                              READY   STATUS    RESTARTS   AGE
pod/silly-demo-58445dff96-t6rg2   1/1     Running   0          32s
pod/silly-demo-58445dff96-ttwjd   1/1     Running   0          32s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/silly-demo   ClusterIP   10.96.144.73   <none>        8080/TCP   26s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/silly-demo   2/2     2            2           32s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/silly-demo-58445dff96   2         2         2       32s

NAME                                   CLASS   HOSTS                 ADDRESS     PORTS   AGE
ingress.networking.k8s.io/silly-demo   nginx   silly-demo...nip.io   localhost   80      29s
```

That simple manifest resulted in a creation of a Deployment, a Service, and an Ingress. Each of those alone would require more work than that single *Application* manifest, and that was a simple example. If we added a database to the mix, benefits would be even greater. If we expanded it to infrastructure, we would get even more.

So, what is [kro](https://kro.run)?

Kro allows us to **create abstractions that encapsulate Kubernetes resources**. What that means is that kro enables us to create **new API endpoints** in Kubernetes clusters and define which resources will be created when someone creates resources based on those new endpoints. It simplifies **creation of Custom Resource Definitions (CRDs) and controllers**. The idea is awesome, yet somehow familiar.

Based on that description, you can think of kro as a tool that serves a similar purpose as [KubeVela](https://kubevela.io/) or [Crossplane](https://crossplane.io) Compositions. As a matter of fact, when I heard about kro, the first thought that passed through my head was "Hey, this is very similar to Crossplane Compositions. Is this going to replace it? Should I give up on Crossplane now that there is a new project that does something similar? It must be better than Crossplane Compositions. Otherwise, why would they start a project like that?"

So, today is the day we'll not only explore kro but also see whether it is time for me to give up on Crossplane. I do not want to stay with a project that gets superseeded by some other. So, depending on how this goes, this might be my last day working on Crossplane.

All in all, there are two questions we'll try to answer. "**What is kro?**" and, if applicable, is it "**a Crossplane Compositions killer?**"

> Before we proceed, let me state that I am involved in the Crossplane project. As such, you might think that this video is dishonest. I do my best to always give an honest opinion based on experience and never to say "it depends", no matter whether I'm talking about a project I use or a project I work on, or both. Still, the truth is that I am activelly involved with Crossplane and it's up to you to decide whether that makes me dishonest in what I'm about to say.

Here we go.

## Setup

```sh
git clone https://github.com/vfarcic/kro-demo

cd kro-demo
```

> Make sure that Docker is up-and-running. We'll use it to create a KinD cluster.

> Watch [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell
```

> Please watch [The Future of Shells with Nushell! Shell + Data + Programming Language](https://youtu.be/zoX_S6d-XU4) if you are not familiar with Nushell. Alternatively, you can inspect the `setup.nu` script and transform the instructions in it to Bash or ZShell if you prefer not to use that Nushell script.

```sh
chmod +x setup.nu

./setup.nu

source .env
```

## The Application Defined As Low-Level Kubernetes Resources

Here's the definition of an app I'd like to convert to a CRD and a controller with kro.

```sh
cat k8s/app.yaml
```

The output is as follows.s

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: silly-demo
  name: silly-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app.kubernetes.io/name: silly-demo
  template:
    metadata:
      labels:
        app.kubernetes.io/name: silly-demo
    spec:
      containers:
      - env:
        - name: DB_ENDPOINT
          valueFrom:
            secretKeyRef:
              key: host
              name: silly-demo-app
        - name: DB_PORT
          valueFrom:
            secretKeyRef:
              key: port
              name: silly-demo-app
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              key: username
              name: silly-demo-app
        - name: DB_PASS
          valueFrom:
            secretKeyRef:
              key: password
              name: silly-demo-app
        - name: DB_NAME
          value: app
        image: ghcr.io/vfarcic/silly-demo:1.4.301
        livenessProbe:
          httpGet:
            path: /
            port: 8080
        name: silly-demo
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /
            port: 8080
        resources:
          limits:
            cpu: 500m
            memory: 512Mi
          requests:
            cpu: 250m
            memory: 256Mi
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: silly-demo
  name: silly-demo
spec:
  ports:
  - name: http
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app.kubernetes.io/name: silly-demo
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  labels:
    app.kubernetes.io/name: silly-demo
  name: silly-demo
spec:
  rules:
  - host: silly-demo.com
    http:
      paths:
      - backend:
          service:
            name: silly-demo
            port:
              number: 8080
        path: /
        pathType: ImplementationSpecific
---
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  labels:
    app.kubernetes.io/name: silly-demo
  name: silly-demo
spec:
  instances: 1
  storage:
    size: 1Gi
---
apiVersion: db.atlasgo.io/v1alpha1
kind: AtlasSchema
metadata:
  labels:
    app.kubernetes.io/name: silly-demo
  name: silly-demo-videos
spec:
  credentials:
    database: app
    host: silly-demo-rw.a-team
    parameters:
      sslmode: disable
    passwordFrom:
      secretKeyRef:
        key: password
        name: silly-demo-app
    port: 5432
    scheme: postgres
    user: app
  schema:
    sql: |
      create table videos (
        id varchar(50) not null,
        title text,
        primary key (id)
      );
      create table comments (
        id serial,
        video_id varchar(50) not null,
        description text not null,
        primary key (id),
        CONSTRAINT fk_videos FOREIGN KEY(video_id) REFERENCES videos(id)
      );
```

There is nothing special about it. It is a relatively simple set of manifests that should result in an application and a database running inside a Kubernetes cluster.

What I want is to convert it into a new CRD *Application* that people should be able to use to create instances of their applications without having to worry about Deployments, Services, Ingresses, PostgreSQL, database schemas, and whichever other low-level details are required to run "stuff" in Kubernetes. They should focus on what they need to do rather than how to do it.

I expect people to be able to specify only the things that matter to them, and not much more. Everything else should be implementation details.

In that spirit, I would want them to be able to set a *name* of the application, container *image*, including release tag, and a *port* the application is exposing. Further on, there should be an optional *Ingress* since not all apps need to be exposed to the outside world. If ingress is enabled, they should be able to specify the *host*.

They should also be able to optionally have a PostgreSQL *Cluster* and, if they do, they should be able to specify SQL *schema*.

All those pieces of information are already available in the "raw" manifests. The problem is that everything else is "noise".

Now, I could wrap it all up into, let's say, a Helm chart, and instruct everyone to only manage the values file. That's silly. We are way past that. By now we should be aware of the benefits of using APIs to expose services as opposed to giving people "random" files and instructions what to touch and what to keep intact.

There are many benefits to exposing services through APIs and I won't go through them today. I already covered it quite a few times in this channel.

What matters is that we will use kro to create a Custom Resource Definition (CRD) with the schema we discussed and to compose the low-level resources whenever someone creates a Custom Resource (CR).

Now that we know what we're trying to accomplish, let's just do it; step by step.

## kro Resource Groups

Kro comes with a CRD called *ResourceGroup* that allows us to define a schema which will ultimately become a Kubernetes CRD as well as a list of resources that should be composed whenever someone creates or updates a Custom Resource based on that CRD.

We'll start easy by trying to define a part of the schema for the CRD and compose only the Deployment and the Service. If that works out, we'll progress towards additional resources and, maybe, a more complicated setup.

Here's the first iteration of the ResourceGroup.

```sh
cat resource-group.yaml
```

The output is as follows.

```yaml
apiVersion: kro.run/v1alpha1
kind: ResourceGroup
metadata:
  name: application
spec:
  schema:
    apiVersion: v1alpha1
    kind: Application
    spec:
      name: string
      image: string
      tag: string
      port: integer | default=8080
  resources:
    - id: deployment
      template:
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          labels:
            app.kubernetes.io/name: ${schema.spec.name}
          name: ${schema.spec.name}
        spec:
          replicas: 2
          selector:
            matchLabels:
              app.kubernetes.io/name: ${schema.spec.name}
          template:
            metadata:
              labels:
                app.kubernetes.io/name: ${schema.spec.name}
            spec:
              containers:
              - name: ${schema.spec.name}
                image: ${schema.spec.image}:${schema.spec.tag}
                livenessProbe:
                  httpGet:
                    path: /
                    port: ${schema.spec.port}
                ports:
                - containerPort: ${schema.spec.port}
                readinessProbe:
                  httpGet:
                    path: /
                    port: ${schema.spec.port}
                resources:
                  limits:
                    cpu: 500m
                    memory: 512Mi
                  requests:
                    cpu: 250m
                    memory: 256Mi
    - id: service
      template:
        apiVersion: v1
        kind: Service
        metadata:
          labels:
            app.kubernetes.io/name: ${schema.spec.name}
          name: ${schema.spec.name}
        spec:
          ports:
          - name: http
            port: ${schema.spec.port}
            protocol: TCP
            targetPort: ${schema.spec.port}
          selector:
            app.kubernetes.io/name: ${schema.spec.name}
          type: ClusterIP
```

We are defining a `ResourceGroup` with a `schema`. That schema will eventually become a CRD. We can think of it as a simplified way to define a CRD.

There is the `apiVersion` and the `kind` of that future CRD, as well as the `spec`. Bear in mind that we can have anything as *schema.kind*. In this example it is `Application` but in yours it could be *Database*, *Cluster*, *Unicorn*, or *Rainbow*. You're in charge of what the name, as well as schema of your CRDs will be.

Further on, we are defining that we would like the *spec* of the CRD to contain fields `name`, `image`, `tag`, and `port`. They should be self explanatory.

Each of those has a mandatory type of the field like `string` and `integer`. If we need to define additional properties, we can do that by adding pipe (`|`) after the type and write whichever property we would like to have. For now, the only one we're using is the `default` value of the *port* which we're setting to `8080`.

As a result of that *schema*, people should be able to define custom resources of the *kind* *Application* with mandatory fields *name*, *image*, and *tag*, and the optional field *port* which, if not set, will default to *8080*.

What we saw so far is awesome. That was maybe the easiest way to define a CRD I've seen among any of the tools.

Next are `resources`.

They are an array that defines templates kro will use to assemble the resources.

For now, we are focusing on Deployments and Services.

So, we have a resource with the `id` `deployment` and a template that I copied from the original definition we saw earlier, and then modified to make some values dynamic.

Those values are defined using [CEL (Common Expression Language)](https://github.com/google/cel-spec), which is gaining traction lately, especially since it became the language used by Kubernetes Validating Admission Policies.

It's fairly simple. We are putting the value of future Custom Resources. There is the `spec.name`, `spec.image`, `spec.port`, and so on and so forth.

The same logic is repeated with the `service`.

So far, I must say that I'm loving it. Kro ResourceGroups are very simple to define. The syntax is easy and there is no boiler code. It's beautiful.

Let's see whether it works by applying the ResourceGroup we explored,...

```sh
kubectl --namespace a-team apply --filename resource-group.yaml
```

...and retriving `resourcegroups`.

```sh
kubectl --namespace a-team get resourcegroups --output wide
```

The output is as follows.

```
NAME          APIVERSION   KIND          STATE    TOPOLOGICALORDER           AGE
application   v1alpha1     Application   Active   ["deployment","service"]   10s
```

It's nice that it shows which resources will be assembled in the `TOPOLOGICALORDER` column. I suspect that will become a mess if we start having ten or more but, for now, it's great that we are able to easily deduce what will be assembled.

If everything worked as expected, we should have gotten a new CRD `applications`. Let's confirm that.

```sh
kubectl get crds | grep kro
```

The output is as follows.

```
applications.kro.run    2024-11-18T23:54:12Z
resourcegroups.kro.run  2024-11-18T23:53:07Z
```

It's there (`applications.kro.run`). I'm happy.

Finally, let's see whether we can create Custom Resources based on that CRD.

Here's an example.

```sh
cat silly-demo.yaml
```

The output is as follows.

```yaml
apiVersion: kro.run/v1alpha1
kind: Application
metadata:
  name: silly-demo
spec:
  name: silly-demo
  image: ghcr.io/vfarcic/silly-demo
  tag: "1.4.305"
```

That's the experience I was looking for. There's no need to waste time defining Deployments and Services when we can focus only on the data that matters. In this case, we defined the `name`, the `image`, and the `tag` as `spec` properties of the `Application` we defined earlier. There is no *port* in this example meaning that the default value of *8080* will be used.

Let's apply it,...

```sh
kubectl --namespace a-team apply --filename silly-demo.yaml
```

...and retrieve all `applications`.

```sh
kubectl --namespace a-team get applications
```

The output is as follows.

```
NAME         STATE    SYNCED   AGE
silly-demo   ACTIVE   True     8s
```

That's brilliant. We can see that the `silly-demo` was `SYNCED` and that it is now `ACTIVE`. It would be nice if we could have defined additional "print" columns, but that's okay. So far I love it.

Let's see the `tree` of that `application`.

```sh
kubectl tree --namespace a-team application silly-demo
```

The output is as follows.

```
No resources are owned by this object through ownerReferences.
```

That's bad. It means that kro is not adding owner references that establish relations between resources. As a result, other tools like, in this case, `kubectl tree` cannot deduce which resource created what.

Still, kro is a project in its infancy. I'm sure that will be corrected soon.

Let's check what it created the "old fashion" way by listing all the resources in the `a-team` Namespace.

```sh
kubectl --namespace a-team get all
```

The output is as follows.

```
NAME                              READY   STATUS    RESTARTS   AGE
pod/silly-demo-58445dff96-8xrx8   1/1     Running   0          65s
pod/silly-demo-58445dff96-qpf9f   1/1     Running   0          65s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/silly-demo   ClusterIP   10.96.57.149   <none>        8080/TCP   62s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/silly-demo   2/2     2            2           65s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/silly-demo-58445dff96   2         2         2       65s
```

We can see that it did what it's supposed to do. It created the `service` and the `deployment` which, in turn, created the `replicaset` which created the Pods (`pod`).

So far kro is amazing. It's easy. It's simple. I love it!

## kro ResourceGroup Conditionals

We're about to add Ingress to the mix, but there is a slight complication. It needs to be optional since some applications might need be accessible from outside the cluster while others are not.

To do that, we'll add `ingress.enabled` boolean to the schema and use it to deduce whether Ingress should be composed or not. While we're add it, we'll also add `ingress.host` parameter that will allow users to specify the host through which their app should be exposed.

Here's the next iteration of the Resource Group that includes those changes.

```sh
cat resource-group-ingress.yaml
```

The output is as follows (truncated for brevity).

```yaml
apiVersion: kro.run/v1alpha1
kind: ResourceGroup
metadata:
  name: application
spec:
  schema:
    apiVersion: v1alpha1
    kind: Application
    spec:
      ...
      ingress:
        enabled: boolean | default=false
        host: string | default="devopstoolkit.live"
  resources:
    ...
    - id: ingress
      includeWhen:
        - ${schema.spec.ingress.enabled}
      template:
        apiVersion: networking.k8s.io/v1
        kind: Ingress
        metadata:
          labels:
            app.kubernetes.io/name: ${schema.spec.name}
          name: ${schema.spec.name}
        spec:
          ingressClassName: nginx
          rules:
          - host: ${schema.spec.ingress.host}
            http:
              paths:
              - backend:
                  service:
                    name: ${schema.spec.name}
                    port:
                      number: ${schema.spec.port}
                path: /
                pathType: ImplementationSpecific
```

We added `ingress` parameter with sub parameters `enabled` and `host`. Apart from doing nested parameters, there's nothing new over there.

Further on, we added a new resource which is also following the same pattern like the previous two. The only new instruction we're adding is `includeWhen` which will result in that resource being composed only if `ingress.enabled` parameter is set to *true*.

There's a complaint here. The *includeWhen* instruction is not documented in kro docs. It's featured in one of the examples and it is certainly present in the code, but it's not explained anywhere how it works or that it even exists. Still, it's a new project and new projects tend to focus on code and leave the documentation somewhat forgotten. Given how green the project is, we can forgive the maintainers on not focusing on the docs.

While I can ignore missing information in the docs, I cannot ignore attempts to convert YAML into something it is not. I am fully onboard using CEL as YAML values, but I do not like the idea using YAML do implement conditionals, loops, and other similar imperative constructs. If those are needed, and in this case they are, we should be looking into using something other than YAML. KCL, CUE, Go Templating, ytt, or any other similar language or templating engine would be a better choice. Even generic languages like Python, TypeScript, or Go would be better choices.

Don't get me wrong. I am not against using YAML. I love the idea of using CEL as values. However, I am completely against adding conditionals, loops, and other imperative constructs into YAML. That inevitably leads into an abomination since that puts kro on a path of adding an infinite number of such constructs.

This is the first thing I truly do not like. All other negative things I saw so far are minor and can be attributed to kro being a very young project in very early stages. Those can be improved over time. The choice of adding imperative logic as YAML is an architectural choice that is hard to undo.

Still, I'll ignore that for now, so let's continue by applying the new iteration of the Resource Group,...

```sh
kubectl --namespace a-team apply \
    --filename resource-group-ingress.yaml
```

...and retrieving `resourcegroups` from the `a-team` Namespace.

```sh
kubectl --namespace a-team get resourcegroups --output wide
```

The output is as follows.

```
NAME        APIVERSION KIND        STATE  TOPOLOGICALORDER                   AGE
application v1alpha1   Application Active ["deployment","ingress","service"] 3m49s
```

We can see that the `TOPOLOGICALORDER` now shows `ingress` as one of the resources that will be composed. We extended the number of resources that will be composed. Great!

Now, let's remove the application and create a new one based on the new Resource Group.

```sh
kubectl --namespace a-team delete --filename silly-demo.yaml
```

It will probably take a bit of time until all the resources are removed. Patience.

Waiting... Waiting... Waiting...

It seems that it got stack, and that's very dissapointing. We probably did something wrong.

> Stop deleting the Application by pressing `ctrl+c`.

Let's stop the delete process and take a look at kro logs.

```sh
kubectl --namespace kro logs \
    --selector app.kubernetes.io/name=kro --tail 50
```

The output is as follows (truncated for brevity).

```
...
2024-11-19T00:03:24.167Z        DEBUG   dynamic-controller      Syncing resourcegroup instance request  {"gvr": "kro.run/v1alpha1/applications", "namespacedKey": "a-team/silly-demo"}
2024-11-19T00:03:24.178Z        DEBUG   dynamic-controller      Finished syncing resourcegroup instance request {"gvr": "kro.run/v1alpha1/applications", "namespacedKey": "a-team/silly-demo", "duration": "11.577375ms"}
2024-11-19T00:03:24.178Z        ERROR   dynamic-controller      Error syncing item, requeuing with rate limit   {"item": {"NamespacedKey":"a-team/silly-demo","GVR":{"Group":"kro.run","Version":"v1alpha1","Resource":"applications"}}, "error": "failed to create runtime resource group: failed to evaluate static variables: failed evaluating expression schema.spec.ingress.host: no such key: ingress"}
github.com/awslabs/kro/internal/dynamiccontroller.(*DynamicController).processNextWorkItem
        github.com/awslabs/kro/internal/dynamiccontroller/dynamic_controller.go:277
github.com/awslabs/kro/internal/dynamiccontroller.(*DynamicController).worker
        github.com/awslabs/kro/internal/dynamiccontroller/dynamic_controller.go:229
k8s.io/apimachinery/pkg/util/wait.JitterUntilWithContext.func1
        k8s.io/apimachinery@v0.31.0/pkg/util/wait/backoff.go:259
k8s.io/apimachinery/pkg/util/wait.BackoffUntil.func1
        k8s.io/apimachinery@v0.31.0/pkg/util/wait/backoff.go:226
k8s.io/apimachinery/pkg/util/wait.BackoffUntil
        k8s.io/apimachinery@v0.31.0/pkg/util/wait/backoff.go:227
k8s.io/apimachinery/pkg/util/wait.JitterUntil
        k8s.io/apimachinery@v0.31.0/pkg/util/wait/backoff.go:204
k8s.io/apimachinery/pkg/util/wait.JitterUntilWithContext
        k8s.io/apimachinery@v0.31.0/pkg/util/wait/backoff.go:259
k8s.io/apimachinery/pkg/util/wait.UntilWithContext
        k8s.io/apimachinery@v0.31.0/pkg/util/wait/backoff.go:170
```

It seems that it got confused with the `ingress.host` key and that's very worrying since we are not yet trying to create a new Application resource but only to delete the ones we were running since before we updated the Resource Group. That inability to delete the Application resource and all the child resources it spun up points to the problem I mentioned earlier. It seems that it is not tracking which resources are spun up by the root resource but "blindly" trying to delete resources it composed. I might be wrong. It might be something else. At the end of the day, it does not matter. What matters is that it got "confused" and now we cannot even delete the root resource. That's very dissapointing. Still, it's a new project, so let's not get dissapointed just yet. I'm sure that will be fixed soon.

In the meantime, let's delete the whole Namespace.

```sh
kubectl delete namespace a-team
```

Waiting... Waiting... Waiting...

That does not seem to work and I suspect we need to remove Application finalizers to progress.

> Stop deleting by pressing `ctrl+c`.

So, let's stop the deletion and apply a `patch` that will remove the `finalizers`.

```sh
kubectl --namespace a-team patch application silly-demo \
    --patch '{"metadata":{"finalizers":null}}' --type=merge
```

Now the whole Namespace and everything inside it is gone and we can move on as if nothing happened.

Let's create the `a-team` Namespace again,...

```sh
kubectl create namespace a-team
```

...apply the last version of the Resource Group,...

```sh
kubectl --namespace a-team apply \
    --filename resource-group-ingress.yaml
```

...and apply our `silly-demo` Application.

```sh
kubectl --namespace a-team apply --filename silly-demo.yaml
```

We should be at the same state as we were before, except that the kro Resource Group now includes Ingress. Bear in mind that the Application definition is still the same. We did not yet enable Ingress. Since Ingress is disabled by default in the Resource Group, the end result should be the same as before.

Let's check the `applications` in the `a-team` Namespace.

```sh
kubectl --namespace a-team get applications
```

The output is as follows.

```
NAME         STATE   SYNCED   AGE
silly-demo                    11s
```

This is very dissapointing. Something is not working since the `SYNCED` column is empty. That probably means not only that it was not synced but that kro got completely confused.

I suspect that it did not compose any resources, but let's check it out just in case.

```sh
kubectl --namespace a-team get all,ingresses
```

The output is as follows.

```
No resources found in a-team namespace.
```

Yep. There's nothing there.

Let's assume that we (to be more precise I) did something wrong. Events in the `application` should give us a clue as to what's going on.

```sh
kubectl --namespace a-team describe application silly-demo
```

The output is as follows.

```
Name:         silly-demo
Namespace:    a-team
Labels:       <none>
Annotations:  <none>
API Version:  kro.run/v1alpha1
Kind:         Application
Metadata:
  Creation Timestamp:  2024-11-19T00:05:01Z
  Generation:          1
  Resource Version:    1367
  UID:                 be2c7355-f73e-44fe-ac39-fdf97cc3c75b
Spec:
  Image:  ghcr.io/vfarcic/silly-demo
  Name:   silly-demo
  Port:   8080
  Tag:    1.4.305
Events:   <none>
```

My personal state is getting converted from dissaspointment to frustration. `Events` are a cornerstone of debugging issues with Kubernetes resources, and there's nothing. Someone forgot to implement events in the kro controller. To make things even worse, there are no statuses either. It's literally doing nothing and showing no signs of life.

Let's see whether we can deduce what's going on from the `resourcegroup` itself.

```sh
kubectl --namespace a-team describe resourcegroup application
```

The output is as follows (truncated for brevity).

```
Name:         application
...
Spec:
  ...
Status:
  Conditions:
    Last Transition Time:  2024-11-19T00:04:54Z
    Message:               micro controller is ready
    Reason:                
    Status:                True
    Type:                  ReconcilerReady
    Last Transition Time:  2024-11-19T00:04:54Z
    Message:               Directed Acyclic Graph is synced
    Reason:                
    Status:                True
    Type:                  GraphVerified
    Last Transition Time:  2024-11-19T00:04:54Z
    Message:               Custom Resource Definition is synced
    Reason:                
    Status:                True
    Type:                  CustomResourceDefinitionSynced
  State:                   Active
  Topological Order:
    deployment
    ingress
    service
Events:  <none>
```

If we take a look at `Status` `Conditions`, everything seem to be working correctly, even though it's obvious that it isn't. The `Events` are nowhere to be seen. My frustration is growing.

Let's do one last check before we give up on it by taking a look at the logs.

```sh
kubectl --namespace kro logs \
    --selector app.kubernetes.io/name=kro --tail 50
```

The output is as follows (truncated for brevity).

```
...
2024-11-19T00:07:00.790Z        DEBUG   dynamic-controller      Syncing resourcegroup instance request  {"gvr": "kro.run/v1alpha1/applications", "namespacedKey": "a-team/silly-demo"}
2024-11-19T00:07:00.795Z        DEBUG   dynamic-controller      Finished syncing resourcegroup instance request {"gvr": "kro.run/v1alpha1/applications", "namespacedKey": "a-team/silly-demo", "duration": "4.661042ms"}
2024-11-19T00:07:00.795Z        ERROR   dynamic-controller      Error syncing item, requeuing with rate limit   {"item": {"NamespacedKey":"a-team/silly-demo","GVR":{"Group":"kro.run","Version":"v1alpha1","Resource":"applications"}}, "error": "failed to create runtime resource group: failed to evaluate static variables: failed evaluating expression schema.spec.ingress.host: no such key: ingress"}
github.com/awslabs/kro/internal/dynamiccontroller.(*DynamicController).processNextWorkItem
        github.com/awslabs/kro/internal/dynamiccontroller/dynamic_controller.go:277
github.com/awslabs/kro/internal/dynamiccontroller.(*DynamicController).worker
        github.com/awslabs/kro/internal/dynamiccontroller/dynamic_controller.go:229
k8s.io/apimachinery/pkg/util/wait.JitterUntilWithContext.func1
        k8s.io/apimachinery@v0.31.0/pkg/util/wait/backoff.go:259
k8s.io/apimachinery/pkg/util/wait.BackoffUntil.func1
        k8s.io/apimachinery@v0.31.0/pkg/util/wait/backoff.go:226
k8s.io/apimachinery/pkg/util/wait.BackoffUntil
        k8s.io/apimachinery@v0.31.0/pkg/util/wait/backoff.go:227
k8s.io/apimachinery/pkg/util/wait.JitterUntil
        k8s.io/apimachinery@v0.31.0/pkg/util/wait/backoff.go:204
k8s.io/apimachinery/pkg/util/wait.JitterUntilWithContext
        k8s.io/apimachinery@v0.31.0/pkg/util/wait/backoff.go:259
k8s.io/apimachinery/pkg/util/wait.UntilWithContext
        k8s.io/apimachinery@v0.31.0/pkg/util/wait/backoff.go:170
```

There's the problem. It is trying to evaluate `ingress.host` key and that's failing because we did not specify `ingress` in the Application. Still, that should not be an issue for multiple reasons. First of all, the whole Ingress resource should not be composed if *ingress.enabled* is NOT set to *true*. We did that through the *includeWhen* instruction. Furthermore, we defined that it is set in the schema to *false* by default. So, if we do not specify *ingress.enabled* field in our Custom Resouces*, kro should simply not compose Ingress. Or, at least, that is the intention. There is always a chance that I did something wrong. I doubt that's the case but even if it is, kro should be a bit more helpful pointing us to the origin of the issue, not to mention that docs do not provide any clue as to what to do in a case like this one.

Still... This is a young project. I expect it to have issues, so let's ignore this one and assume that it will be polished with time. Today we are evaluating the idea rather than the implementation.

If kro gets confused when we don't specify *ingress* in our Custom Resource and ignores default values in the schema, let's fix the issue with a modified version of our application.

```sh
cat silly-demo-ingress.yaml
```

The output is as follows.

```yaml
apiVersion: kro.run/v1alpha1
kind: Application
metadata:
  name: silly-demo
spec:
  name: silly-demo
  image: ghcr.io/vfarcic/silly-demo
  tag: "1.4.305"
  ingress:
    enabled: true
    host: silly-demo.127.0.0.1.nip.io
```

The difference is that, this time, we are setting `ingress.enabled` to `true` and defining the `ingress.host`.

Let's apply the updated definition,...

```sh
kubectl --namespace a-team apply \
    --filename silly-demo-ingress.yaml
```

...and list all the `applications` in the `a-team` Namespace.

```sh
kubectl --namespace a-team get applications
```

The output is as follows.

```
NAME         STATE    SYNCED   AGE
silly-demo   ACTIVE   True     4m26s
```

This time it seem to be working correctly. We can double check that by retrieving `all` the core Kubernetes resources and `ingresses`.

```sh
kubectl --namespace a-team get all,ingresses
```

The output is as follows.

```
NAME                              READY   STATUS    RESTARTS   AGE
pod/silly-demo-58445dff96-t6rg2   1/1     Running   0          32s
pod/silly-demo-58445dff96-ttwjd   1/1     Running   0          32s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/silly-demo   ClusterIP   10.96.144.73   <none>        8080/TCP   26s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/silly-demo   2/2     2            2           32s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/silly-demo-58445dff96   2         2         2       32s

NAME                                   CLASS   HOSTS                 ADDRESS     PORTS   AGE
ingress.networking.k8s.io/silly-demo   nginx   silly-demo...nip.io   localhost   80      29s
```

That's it. It's working. Now we are getting not only `service` and `deployment` but also `ingress` resources composed through kro.

Now that Ingress is in the mix, we should be able to send requests to our application,...

```sh
curl "http://silly-demo.127.0.0.1.nip.io"
```

...and get the `This is a silly demo` response.

Brilliant. We made it work, somehow. We would still need to figure out how to overcome the situation when we do not want to have Ingress, but I'll leave that for some later time. Right now we are going to move into adding the database to the mix.

## kro ResourceGroup With a More Complex Setup

There are three things we'll try to add. First, we should be able to optionally add CNPG Postgress that should be applied only if a user chooses to enable the database. Second, we should add, also optional, schema management with the Atlas Operator. Finally, we should enhance our deployment so that our application gets the information how to connect to the database as environment variables generated from the Secret CNPG will create for us.

Let's start with the last task by adding the environment variables that should appear only if the consumer chooses to enable database usage.

Here's the updated definition of the Resource Group.

```sh
cat resource-group-db-envs.yaml
```

The output is as follows (truncated for brevity).

```yaml
apiVersion: kro.run/v1alpha1
kind: ResourceGroup
metadata:
  name: application
spec:
  schema:
    apiVersion: v1alpha1
    kind: Application
    spec:
      ...
      db:
        enabled: boolean | default=false
  resources:
    - id: deployment
      template:
        apiVersion: apps/v1
        kind: Deployment
        ...
        spec:
          ...
          template:
            ...
            spec:
              containers:
              - name: ${schema.spec.name}
                ...
                env:
                - name: DB_ENDPOINT
                  valueFrom:
                    secretKeyRef:
                      key: host
                      name: ${schema.spec.name}-app
                - name: DB_PORT
                  valueFrom:
                    secretKeyRef:
                      key: port
                      name: ${schema.spec.name}-app
                - name: DB_USER
                  valueFrom:
                    secretKeyRef:
                      key: username
                      name: ${schema.spec.name}-app
                - name: DB_PASS
                  valueFrom:
                    secretKeyRef:
                      key: password
                      name: ${schema.spec.name}-app
                - name: DB_NAME
                  value: app
    ...
```

We made changes to the `schema` by adding `db.enabled` property that defaults to `false`. That way, database usage is opt-in. Further on, we added the `env` section to the *Deployment*. It provides the environment variables like the `DB_ENDPOINT`, `DB_PORT`, and others required for the application to connect and authenticate to the database.

We have a problem though. The *includeWhen* statement, as far as I could figure by reading the code since the documentation does not even mention it, works only on the resource level. We can choose which resources to exclude when composing resources, but we cannot decide which parts of resources to exclude. This is yet another proof that YAML is not a good choice for anything but very simple scenarios. As it is now, those variables will be fetched from the secret with database credentials no matter whether we enable database usage or not. As a result, we would have to always have the database. Otherwise, the Pods created through that Deployment would end up in an infinite pending state forever waiting for the Secret to appear.

We'll cross that brindge later. For now, let's apply the updated definition of the Resource Group before we move on and add CNPG to the mix.

```sh
kubectl --namespace a-team apply \
    --filename resource-group-db-envs.yaml
```

Now, I expect that the change to the Resource Group will result in changes to all the resources it manages. I expect that the addition of environment variables to the composed Deployments should have been performed.

Let's check that out by outputing the existing `deployment` that Resource Group created earlier.

```sh
kubectl --namespace a-team get deployment silly-demo \
    --output yaml
``` 

The output is as follows (truncated for brevity).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  ...
  name: silly-demo
  ...
spec:
  ...
  template:
    ...
    spec:
      containers:
      - image: ghcr.io/vfarcic/silly-demo:1.4.305
        ...
```

The *env* section is nowhere to be found. The Deployment is exactly the same as it was. The changes to the Resource Group were not propagated to the resources that are already in the cluster.

That's bad since that means that every time we improve our Resource Groups, without changing the API version, we need to delete and create all the instances of it. That's just silly. Still, I'm keeping my excitement high. I love the idea of kro and I'll blame this on it being a project in its infancy. I'm sure this will be improved soon.

So, let's delete the Application,...

```sh
kubectl --namespace a-team delete \
    --filename silly-demo-ingress.yaml
```

...and apply it again.

```sh
kubectl --namespace a-team apply \
    --filename silly-demo-ingress.yaml
```

Let's see whether it works now.

```sh
kubectl --namespace a-team get deployment silly-demo \
    --output yaml
```

The output is as follows (truncated for brevity).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  ...
  name: silly-demo
  ...
spec:
  ...
  template:
    ...
    spec:
      containers:
      - env:
        - name: DB_ENDPOINT
          valueFrom:
            secretKeyRef:
              key: host
              name: silly-demo-app
        - name: DB_PORT
          valueFrom:
            secretKeyRef:
              key: port
              name: silly-demo-app
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              key: username
              name: silly-demo-app
        - name: DB_PASS
          valueFrom:
            secretKeyRef:
              key: password
              name: silly-demo-app
        - name: DB_NAME
          value: app
        ...
```

We can see that, this time, it worked correctly. The `env` variables are there. So, changes to Resource Groups are not automatically propagated to instances of it. We had to delete our Application and create it again to make it work. That's unacceptable, but we'll overlook it assuming that it'll be fixed in one of the next releases of kro.

We are left with two pending tasks. We should add the CNPG Cluster resource that will be creating PostgreSQL database and the Atlas Operator resources that will handle database schema. Let's do CNPG first.

Here's the updated Resource Group.

```sh
cat resource-group-db.yaml
```

The output is as follows (truncated for brevity).

```yaml
apiVersion: kro.run/v1alpha1
kind: ResourceGroup
metadata:
  name: application
spec:
  ...
  resources:
    ...
    - id: postgresql
      includeWhen:
        - ${schema.spec.db.enabled}
      template:
        apiVersion: postgresql.cnpg.io/v1
        kind: Cluster
        metadata:
          labels:
            app.kubernetes.io/name: ${schema.spec.name}
          name: ${schema.spec.name}
        spec:
          instances: 1
          storage:
            size: 1Gi
```

This time we did not change the *schema*. The only thing we're doing is adding one more resource with the `id` `postgresql`. It's a CNPG `Cluster`and there is nothing special about it. It's following the same pattern as the Ingress resource we added earlier with the `includeWhen` set to include it only if `db.enabled` is set to *true*.

This was an easy one so let's just apply it,...

```sh
kubectl --namespace a-team apply \
    --filename resource-group-db.yaml
```

...and retrieve all `resourcegroups` from the `a-team` Namespace.

```sh
kubectl --namespace a-team get resourcegroups --output wide
```

The output is as follows.

```
NAME          APIVERSION   KIND          STATE      TOPOLOGICALORDER   AGE
application   v1alpha1     Application   Inactive                      7m45s
```

What the f**k! As soon as I think that we're out of woods or, to be more precise, as soon as we start ignoring one issue, another one appears out of nowhere. This change made the Resource Group state `Inactive`. What the heck is the problem now. We haven't done anything new. We added a resource using the same patterns and the same constructs as the resources we added earlier.

Let's hope that, this time, we'll see what's wrong by describing the `resourcegroup`. Hopefully, it will not be like the last time when it claimed that everything works correctly only to discover that it stopped working altogether.

```sh
kubectl --namespace a-team describe resourcegroup application
```

The output is as follows (truncated for brevity).

```
Name:         application
...
Status:
  Conditions:
    Last Transition Time:  2024-11-19T00:12:30Z
    Message:               Directed Acyclic Graph is synced
    Reason:                failed to build resource 'postgresql': failed to generate dummy CR for resource postgresql: error generating field spec: error generating field env: error generating field valueFrom: error generating field resourceFieldRef: error generating field divisor: schema type is empty and has no properties
    Status:                False
    Type:                  GraphVerified
    ...
```

We can see that it `failed to generate dummy CR`. I have no idea what that means and, frankly, I would not know what to do to fix it. I know for certaint that the resource definition of the CNPG *Cluster* works correctly. I know that it works when applied without kro ResourceGroup.

I give up!

Kro is not yet ready for prime, and that's okay. It's a new project. Having a bunch of issues is normal. At this stage it is the idea that matters and idea is...

Let's talk about the idea behind kro. I have a very important question to ask.

## Why kro?

Here's the "big" question. **Why?** Why was kro created? We already have projects that allow us to create Custom Resource Definitions and compose resources when Custom Resources based on those definitions are created. That's what KubeVela does. That's what Crossplane Compositions are for. There are other projects that serve a similar purpose. So, why was kro created?

Before I tried the project, I was certain that kro was born because its authors came up with a better way to compose resources. I was sure that the authors saw what Crossplane Compositions are doing. As a matter of fact, I saw in the early commits that they were inspired by Crossplane Compositions. So, they must have came up with a better way to compose resources. However, I don't see what that better way is.

Crossplane Compositions started as YAML-only, just as kro is now. Kro's syntax is, arguably, a bit easier and with less boiler plate code, but that's to be expected since it offers only a fraction of the features Crossplane provides. CEL is a good choice for simple scenarios, but Crossplane now supports functions that allow it to Compose resources using any language. Arguably, CEL is not one of the languages currently supported, but that would be fairly easy to add.

The only tangible benefit I see with kro is that everything is Namespace-scoped, while Crossplane resources, excluding claims, are cluster-scoped. Crossplane 2.0 is planning to change that, and kro folks might not be aware of that plan. Still, is that the reason good enough to start a new project that does a fraction of what Crossplane Compositions do?

Could it be that kro authors think that Crossplane is a bad choice? That could be it. Crossplane is certainly not for everyone, nothing is. Still, there is KubeVela and there are other similar projects that already do what kro does, just better.

All in all, I don't have an answer to the simple question. **Why?** Why was kro created? What is the mission of the project? What does it try to do that hasn't been done before?

At the same time, kro is going though the same phases as other projects went through. It's green, just as Crossplane Compositions were green years ago. It's missing features and it's buggy, just as Crossplane was missing features and was buggy years ago.

Kro started as YAML-only, with CEL values, and is bound to end up with an abomination as the need for imperative constructs emerges like, as we saw before, the need for conditional statements applied to parts of resources, loops, and what so not. The alternative to becoming a YAML abomination is to extend itself to other languages, just as Crossplane did years ago.

All in all, kro is serving, more or less, the same function as other tools created a while ago, without any compelling improvement. If anything, it is likely going to end up going through the same stages as other projects went. I wanted to see it being radically different or having a different mission or solving the problem in a different way. That's not the case, or I'm failing to see it.

So, my question still stands. **Why?** Why was kro created? I'm all for new projects that solve new problems or are trying to solve problems that are already solved in a better way. I just don't see kro being one of them. I hope I'm wrong. Actually, I'm sure I'm wrong. I'm sure that the kro authors have a good reason why the project was created and that they just did not get to document those reasons just yet.

I'll get back to the project in a few months and, by then, I'm sure a lightbulb will turn on in my head and I'll say: "Now I get it!"

## Destroy

```sh
chmod +x destroy.nu

./destroy.nu

exit
```


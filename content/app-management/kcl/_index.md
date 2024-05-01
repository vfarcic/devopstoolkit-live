+++
title = 'Exploring KCL: Configuration and Data Structure Language; CUE and Pkl Replacement?'
date = 2024-04-22T16:00:00+00:00
draft = false
+++

**I'm in pain**... and it's self-inflicted... and I like it.

I tend to go through an endless number of tools, services, and format in search for better ways to do my job. I'm never satisfied. I always think that there is something better out there. So I go through pain of learning a new tool or a language only to jump into a new one shortly afterwards. Spending endless hours going through new stuff does not make sense, but I can't help myself. Hopefully, I might save you from doing it yourself. That's my goal. Go through the pain of trying out everything so that you don't have to.
<!--more-->

{{< youtube Gn6btuH3ULw >}}

Latelly, I've been exploring different ways to manage manifests, mostly for Kubernetes resources, but, in reality, for anything that can be defined as data, meaning YAML, JSON, XML, and so on.

Today I want to explore yet another language designed to work  with data structures and that language is... [KCL](https://kcl-lang.io). It is a constraint-based record and functional language and, as I'm pronouncing it, I realize that such a description might not make sense to most of use, so here's a simplified version.

KCL is a language designed to work with data with the goal to produce YAMl or JSON. From that perspective it is similar to Helm, Kustomize, Jsonnet, Carvel ytt, Pkl, and, my current favorites, CUE.

Now, you might be asking "**why do we need another one of those?**", and that's what I've been asking myself and others as well. Nevertheless, I'm here to explore it and see if it makes sense to use it.

Now, before we proceed, let me tell you what I'm looking for.

I want to use a language or a DSL that is designed for **data structures**. The reason for that is that YAML or JSON I might be producing is data. That requirement alone discards Helm since Helm does not understand data. Helm is a free-text templating engine that has no notion of data. It is based on Go templating that is equally bad at genering HTML pages as generating data. Nevertheless, Helm is the de-facto standard for third-party apps so I have to use it for that, but not for my apps.

Then there is Kustomize that is my favorite when simple scenarios are concerned. It's not a language but, rather, a mechanism that allows us to overlay YAML files with other YAML files. It is simple and effective, but it fails miserably for anything but very simple scenarios. When things are done right, scenarios are simple so I use it heavily, yet, there are cases when I need more.

TODO: Thumbnails: m6g0aWggdUQ, bbE1BFCs548, Nm1ioWPRRVQ

Then there is Jsonnet, Carvel ytt, and a myriad of other solutions which, for one reason or another, I gave up on. Currently, I'm torn between CUE with Timoni and Pkl. I explored all those so I won't go into them today. Check them out if you're not familiar with them. The links are in the description.

Both of those are doing just what I need them to do, and that's where mazochism kicks in. I don't need another one, yet I'm exploring KCL. There are a few reasons for doing that besides enjoying self-inflicted pain.

Going back to my requirements...

Besides the need for a language or a DSL that understands data-structures instead of being general-purpose anything goes, I also need to be able to define **schemas** but also to import schemas. It would be silly for me to reinvent the wheel by creating schemas for, let's say, Kubernetes APIs. I expect those to be readily available and, in case of CRDs, to have the option to import them from a cluster or a Git repo.

I also insist on **immutability**. I experienced too many issues when working with mutable data. I want to avoid that at all costs.

Finally, I don't want to spend weeks trying to learn it. A day is more than enough. If it takes more than that, excluding special "advanced" features, it's too complicated for my tiny brain. It needs to be **easy**.

So, I'm looking for a data-structure language that comes with pre-built schemas but also allows me to create them myself or import them from somewhere, I need it to be immutable and it cannot be hard to learn.

With that in mind, let's take a look at KCL and see whether it fits those requirements and whether it might convince me to drop at CUE or Pkl.

Let's see what KCL is all about.

## Setup

Install `kcl` CLI https://kcl-lang.io/docs/user_docs/getting-started/install#1-install-kcl

Install `kcl` Language Server and IDE extension from https://kcl-lang.io/docs/user_docs/getting-started/install#2-install-kcl-ide-extension.

```sh
git clone https://github.com/vfarcic/crossplane-app

cd crossplane-app

git pull

git checkout kcl
```

## KCL in Action

Before we begin, I must say that KCL is a **CNCF** project meaning that it is not in the hands of a single company making it's future depend less on the whims of that company. It generated a lot of buzz and it has a very active community. There must be something in there, right?

Right now, I'm inside a project that requires a rather large amount of YAML. That YAML is big enough and with enough complexity and repetition that it makes no sense to write it directly. I need to generate it somehow, and I was about to switch to CUE when I got introduced to KCL so I though "What the heck. Let's give it a try."

As you can expect, I can execute `kcl` with the path to the code, hit the enter key, and...

```sh
kcl kcl/backend.k
```

The output is as follows.

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: app-backend
  labels:
    type: backend
    location: local
spec:
  compositeTypeRef:
    apiVersion: devopstoolkitseries.com/v1alpha1
    kind: App
  patchSets:
  - name: metadata
    patches:
    - fromFieldPath: metadata.labels
  resources:
  - name: kubernetes
    base:
      apiVersion: kubernetes.crossplane.io/v1alpha1
      kind: ProviderConfig
      spec:
        credentials:
          source: InjectedIdentity
    patches:
    - fromFieldPath: spec.id
      toFieldPath: metadata.name
    readinessChecks:
    - type: None
  - name: deployment
    base:
      apiVersion: kubernetes.crossplane.io/v1alpha1
      kind: Object
      spec:
        forProvider:
          manifest:
            apiVersion: apps/v1
            kind: Deployment
            spec:
              selector: {}
              template:
                spec:
                  containers:
                  - livenessProbe:
                      httpGet:
                        path: /
                        port: 80
                    name: backend
                    ports:
                    - containerPort: 80
                    readinessProbe:
                      httpGet:
                        path: /
                        port: 80
                    resources:
                      limits:
                        cpu: 250m
                        memory: 256Mi
                      requests:
                        cpu: 125m
                        memory: 128Mi
    patches:
    - fromFieldPath: spec.id
      toFieldPath: metadata.name
      transforms:
      - type: string
        string:
          fmt: '%s-deployment'
    - fromFieldPath: spec.id
      toFieldPath: spec.forProvider.manifest.metadata.name
    - fromFieldPath: spec.parameters.namespace
      toFieldPath: spec.forProvider.manifest.metadata.namespace
    - fromFieldPath: spec.id
      toFieldPath: spec.forProvider.manifest.metadata.labels.app
    - fromFieldPath: spec.id
      toFieldPath: spec.forProvider.manifest.spec.selector.matchLabels.app
    - fromFieldPath: spec.id
      toFieldPath: spec.forProvider.manifest.spec.template.metadata.labels.app
    - fromFieldPath: spec.parameters.image
      toFieldPath: spec.forProvider.manifest.spec.template.spec.containers[0].image
    - fromFieldPath: spec.parameters.port
      toFieldPath: spec.forProvider.manifest.spec.template.spec.containers[0].ports[0].containerPort
    - fromFieldPath: spec.parameters.port
      toFieldPath: spec.forProvider.manifest.spec.template.spec.containers[0].livenessProbe.httpGet.port
    - fromFieldPath: spec.parameters.port
      toFieldPath: spec.forProvider.manifest.spec.template.spec.containers[0].readinessProbe.httpGet.port
    - fromFieldPath: spec.id
      toFieldPath: spec.providerConfigRef.name
  - name: service
    base:
      apiVersion: kubernetes.crossplane.io/v1alpha1
      kind: Object
      spec:
        forProvider:
          manifest:
            apiVersion: v1
            kind: Service
            spec:
              ports:
              - name: http
                port: 8008
                protocol: TCP
              type: ClusterIP
    patches:
    - fromFieldPath: spec.id
      toFieldPath: metadata.name
      transforms:
      - type: string
        string:
          fmt: '%s-service'
    - fromFieldPath: spec.id
      toFieldPath: spec.forProvider.manifest.metadata.name
    - fromFieldPath: spec.parameters.namespace
      toFieldPath: spec.forProvider.manifest.metadata.namespace
    - fromFieldPath: spec.id
      toFieldPath: spec.forProvider.manifest.metadata.labels.app
    - fromFieldPath: spec.id
      toFieldPath: spec.forProvider.manifest.spec.selector.app
    - fromFieldPath: spec.parameters.port
      toFieldPath: spec.forProvider.manifest.spec.ports[0].port
    - fromFieldPath: spec.parameters.port
      toFieldPath: spec.forProvider.manifest.spec.ports[0].targetPort
    - fromFieldPath: spec.id
      toFieldPath: spec.providerConfigRef.name
  - name: ingress
    base:
      apiVersion: kubernetes.crossplane.io/v1alpha1
      kind: Object
      spec:
        forProvider:
          manifest:
            apiVersion: networking.k8s.io/v1
            kind: Ingress
            metadata:
              annotations:
                ingress.kubernetes.io/ssl-redirect: 'false'
            spec:
              rules:
              - http:
                  paths:
                  - backend:
                      service:
                        name: acme
                    path: /
                    pathType: ImplementationSpecific
    patches:
    - fromFieldPath: spec.id
      toFieldPath: metadata.name
      transforms:
      - type: string
        string:
          fmt: '%s-ingress'
    - fromFieldPath: spec.id
      toFieldPath: spec.forProvider.manifest.metadata.name
    - fromFieldPath: spec.parameters.namespace
      toFieldPath: spec.forProvider.manifest.metadata.namespace
    - fromFieldPath: spec.id
      toFieldPath: spec.forProvider.manifest.metadata.labels.app
    - fromFieldPath: spec.parameters.host
      toFieldPath: spec.forProvider.manifest.spec.rules[0].host
    - fromFieldPath: spec.id
      toFieldPath: spec.forProvider.manifest.spec.rules[0].http.paths[0].backend.service.name
    - fromFieldPath: spec.parameters.port
      toFieldPath: spec.forProvider.manifest.spec.rules[0].http.paths[0].backend.service.port.number
    - fromFieldPath: spec.id
      toFieldPath: spec.providerConfigRef.name
    - type: ToCompositeFieldPath
      fromFieldPath: spec.forProvider.manifest.spec.rules[0].host
      toFieldPath: status.host
```
There we go.

The output is over 150 lines of YAML.

Now that you saw the monstruosity that was generated, let's take a look at the KCL definition that made that possible.

```sh
cat kcl/backend.k
```

The output is as follows.

```
import .common
import .deployment
import .service
import .ingress
import .kubernetesProviderConfig

common.Composition {
    metadata = common.Metadata {
        name = "app-backend"
        labels = common.Labels {
            type = "backend"
            location = "local"
        }
    }
    spec = common.Spec {
        resources = [
            kubernetesProviderConfig.KubernetesProviderConfig {}
            deployment.Deployment {}
            service.Service {}
            ingress.Ingress {}
        ]
    }
}
```

At the very top, I am importing schemas, functions, global variables, and a few other things. There's `common` that contains, as you might expect from the name, common stuff that I could not place in a specific "box". Then there are `deployment`, `service`, and `ingress` which are my abstractions that match corresponding Kubernetes resources, even though this YAML is generating Crossplane Compositions and not Kubernetes resources directly. Finally, there's `kubernetesProviderConfig` that is a Crossplane ProviderConfig.

Such an organization allows me to avoid repetition since, as you will see soon, those imports are repeated across other KCL manifests.

The actual output is generated by invoking `Composition` schema defined in common, we'll see it soon. That schema already contains all the values that are the same for all variations as well as variables that need to be defined explicitly. In this case, I have to define only the `name`, the `labels`, and the `resources` array that, in this case, contains `deployment`, `service`, and `ingress` schemas also imported at the top.

Here's a variation of that manifest that generates a similar output but with a few differences.

```sh
cat kcl/backend-db-remote.k
```

The output is as follows.

```
import .common
import .deployment
import .service
import .ingress

common.Composition {
    metadata = common.Metadata {
        name = "app-backend-db-remote"
        labels = common.Labels {
            type = "backend-db"
            location = "remote"
        }
    }
    spec = common.Spec {
        resources = [
            deployment.Deployment{
                _dbEnabled = True
                _dbSecretName = "spec.parameters.dbSecret.name"
                _providerConfigName = "spec.parameters.kubernetesProviderConfigName"
            },
            service.Service{
                _providerConfigName = "spec.parameters.kubernetesProviderConfigName"
            },
            ingress.Ingress{
                _providerConfigName = "spec.parameters.kubernetesProviderConfigName"
            },
        ]
    }
}
```

This KCL definition will generate a different `Composition`. Besides a different `name` and `labels`, customized versions of the `Deployment`, `Service`, and `Ingress` by passing variables to their respective schemas. We'll see those soon. For now, let's take a look at the output of that KCL definition.

```sh
kcl kcl/backend-db-remote.k
```

The output is as follows.

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: app-backend-db-remote
  labels:
    type: backend-db
    location: remote
spec:
  compositeTypeRef:
    apiVersion: devopstoolkitseries.com/v1alpha1
    kind: App
  patchSets:
  - name: metadata
    patches:
    - fromFieldPath: metadata.labels
  resources:
  - name: deployment
    base:
      apiVersion: kubernetes.crossplane.io/v1alpha1
      kind: Object
      spec:
        forProvider:
          manifest:
            apiVersion: apps/v1
            kind: Deployment
            spec:
              selector: {}
              template:
                spec:
                  containers:
                  - env:
                    - name: DB_ENDPOINT
                      valueFrom:
                        secretKeyRef:
                          key: endpoint
                    - name: DB_PASSWORD
                      valueFrom:
                        secretKeyRef:
                          key: password
                    - name: DB_PORT
                      valueFrom:
                        secretKeyRef:
                          key: port
                          optional: true
                    - name: DB_USERNAME
                      valueFrom:
                        secretKeyRef:
                          key: username
                    - name: DB_NAME
                    livenessProbe:
                      httpGet:
                        path: /
                        port: 80
                    name: backend
                    ports:
                    - containerPort: 80
                    readinessProbe:
                      httpGet:
                        path: /
                        port: 80
                    resources:
                      limits:
                        cpu: 250m
                        memory: 256Mi
                      requests:
                        cpu: 125m
                        memory: 128Mi
    patches:
    - fromFieldPath: spec.id
      toFieldPath: metadata.name
      transforms:
      - type: string
        string:
          fmt: '%s-deployment'
    - fromFieldPath: spec.id
      toFieldPath: spec.forProvider.manifest.metadata.name
    - fromFieldPath: spec.parameters.namespace
      toFieldPath: spec.forProvider.manifest.metadata.namespace
    - fromFieldPath: spec.id
      toFieldPath: spec.forProvider.manifest.metadata.labels.app
    - fromFieldPath: spec.id
      toFieldPath: spec.forProvider.manifest.spec.selector.matchLabels.app
    - fromFieldPath: spec.id
      toFieldPath: spec.forProvider.manifest.spec.template.metadata.labels.app
    - fromFieldPath: spec.parameters.image
      toFieldPath: spec.forProvider.manifest.spec.template.spec.containers[0].image
    - fromFieldPath: spec.parameters.port
      toFieldPath: spec.forProvider.manifest.spec.template.spec.containers[0].ports[0].containerPort
    - fromFieldPath: spec.parameters.port
      toFieldPath: spec.forProvider.manifest.spec.template.spec.containers[0].livenessProbe.httpGet.port
    - fromFieldPath: spec.parameters.port
      toFieldPath: spec.forProvider.manifest.spec.template.spec.containers[0].readinessProbe.httpGet.port
    - fromFieldPath: spec.parameters.dbSecret.name
      toFieldPath: spec.forProvider.manifest.spec.template.spec.containers[0].env[0].valueFrom.secretKeyRef.name
    - fromFieldPath: spec.parameters.dbSecret.name
      toFieldPath: spec.forProvider.manifest.spec.template.spec.containers[0].env[1].valueFrom.secretKeyRef.name
    - fromFieldPath: spec.parameters.dbSecret.name
      toFieldPath: spec.forProvider.manifest.spec.template.spec.containers[0].env[2].valueFrom.secretKeyRef.name
    - fromFieldPath: spec.parameters.dbSecret.name
      toFieldPath: spec.forProvider.manifest.spec.template.spec.containers[0].env[3].valueFrom.secretKeyRef.name
    - fromFieldPath: spec.parameters.dbSecret.name
      toFieldPath: spec.forProvider.manifest.spec.template.spec.containers[0].env[4].value
    - fromFieldPath: spec.parameters.kubernetesProviderConfigName
      toFieldPath: spec.providerConfigRef.name
  - name: service
    base:
      apiVersion: kubernetes.crossplane.io/v1alpha1
      kind: Object
      spec:
        forProvider:
          manifest:
            apiVersion: v1
            kind: Service
            spec:
              ports:
              - name: http
                port: 8008
                protocol: TCP
              type: ClusterIP
    patches:
    - fromFieldPath: spec.id
      toFieldPath: metadata.name
      transforms:
      - type: string
        string:
          fmt: '%s-service'
    - fromFieldPath: spec.id
      toFieldPath: spec.forProvider.manifest.metadata.name
    - fromFieldPath: spec.parameters.namespace
      toFieldPath: spec.forProvider.manifest.metadata.namespace
    - fromFieldPath: spec.id
      toFieldPath: spec.forProvider.manifest.metadata.labels.app
    - fromFieldPath: spec.id
      toFieldPath: spec.forProvider.manifest.spec.selector.app
    - fromFieldPath: spec.parameters.port
      toFieldPath: spec.forProvider.manifest.spec.ports[0].port
    - fromFieldPath: spec.parameters.port
      toFieldPath: spec.forProvider.manifest.spec.ports[0].targetPort
    - fromFieldPath: spec.parameters.kubernetesProviderConfigName
      toFieldPath: spec.providerConfigRef.name
  - name: ingress
    base:
      apiVersion: kubernetes.crossplane.io/v1alpha1
      kind: Object
      spec:
        forProvider:
          manifest:
            apiVersion: networking.k8s.io/v1
            kind: Ingress
            metadata:
              annotations:
                ingress.kubernetes.io/ssl-redirect: 'false'
            spec:
              rules:
              - http:
                  paths:
                  - backend:
                      service:
                        name: acme
                    path: /
                    pathType: ImplementationSpecific
    patches:
    - fromFieldPath: spec.id
      toFieldPath: metadata.name
      transforms:
      - type: string
        string:
          fmt: '%s-ingress'
    - fromFieldPath: spec.id
      toFieldPath: spec.forProvider.manifest.metadata.name
    - fromFieldPath: spec.parameters.namespace
      toFieldPath: spec.forProvider.manifest.metadata.namespace
    - fromFieldPath: spec.id
      toFieldPath: spec.forProvider.manifest.metadata.labels.app
    - fromFieldPath: spec.parameters.host
      toFieldPath: spec.forProvider.manifest.spec.rules[0].host
    - fromFieldPath: spec.id
      toFieldPath: spec.forProvider.manifest.spec.rules[0].http.paths[0].backend.service.name
    - fromFieldPath: spec.parameters.port
      toFieldPath: spec.forProvider.manifest.spec.rules[0].http.paths[0].backend.service.port.number
    - fromFieldPath: spec.parameters.kubernetesProviderConfigName
      toFieldPath: spec.providerConfigRef.name
    - type: ToCompositeFieldPath
      fromFieldPath: spec.forProvider.manifest.spec.rules[0].host
      toFieldPath: status.host
```

That's a longer one with around 200 lines of YAML.

Let's shift focus on the imports. One of those was `common`.

```sh
cat kcl/common.k
```

The output is as follows.

```
schema Composition:
    apiVersion = "apiextensions.crossplane.io/v1"
    kind = "Composition"
    metadata: Metadata
    spec: Spec

schema Metadata:
    name: str
    labels: Labels

schema Spec:
    compositeTypeRef = {
        apiVersion = "devopstoolkitseries.com/v1alpha1"
        kind = "App"
    }
    patchSets = [{
        name = "metadata"
        patches = [{fromFieldPath = "metadata.labels"}]
    }]
    resources: []

schema Labels:
    type: str
    location: str

schema KubernetesObject:
    name: str
    base = {
        apiVersion = "kubernetes.crossplane.io/v1alpha1"
        kind = "Object"
        spec: KubernetesObjectSpec
    }
    patches: []

schema KubernetesObjectBase:
    apiVersion = "kubernetes.crossplane.io/v1alpha1"
    kind = "Object"
    spec: KubernetesObjectSpec

schema KubernetesObjectSpec:
    forProvider: KubernetesObjectForProvider

schema KubernetesObjectForProvider:
    manifest: any

Patches = lambda name: str -> [] {
    [
        {
            fromFieldPath = "spec.id"
            toFieldPath = "metadata.name"
            transforms = [{type = "string", string = { fmt = "%s-{}".format(name)}}]
        },
        {fromFieldPath = "spec.id", toFieldPath = "spec.forProvider.manifest.metadata.name"},
        {fromFieldPath = "spec.parameters.namespace", toFieldPath = "spec.forProvider.manifest.metadata.namespace"},
        {fromFieldPath = "spec.id", toFieldPath = "spec.forProvider.manifest.metadata.labels.app"},
    ]
}

ManifestSpec = "spec.forProvider.manifest.spec"
```

Most of that file contains a schema I'm using to generate output I need. There is `Composition` with a few hard-coded values like `apiVersion` and `kind`, and a few sub-schemas like `Metadata` and `Spec`. `Metadata`, on the other hand, defines a `str` variable `name`. That's the name we saw at the very beginning. Whoever is using that schema has to define it or KCL will throw an error.

That continues on and on with different schemas containing either fields that are hard-coded or variables that need to be defined by whoever is using them.

Now, the syntax itself is pretty much Json with key values being separate by `=` and values and types being separated by `:`. Objects are defined with curly braces `{}` and arrays with square brackets `[]`. There are a few more things that are different from Json but, in general, if you know Json or CUE, you should be able to pick it up quickly. If you're not familiar with either of those, you'll still be able to learn it relatively fast. KCL is one of the most powerful yet one of the easiest data-structure languages I used. We'll talk about that later.

We can also use functions like, in this case, `Patches`. In KCL, functions are called `lambda` and this one defines a single argument `name` that is a `str` and returns an array (`[`, `]`) of objects (`{`).

Finally, that definition defines a variable `ManifestSpec` that I'm using as a way to avoid typing that path.

There's more though. Much much more than we can explore in this video, so I'll focus on only a few other aspects of KCL which we can see in the `deployment.k` file which is one of the imports we saw at the start.

```sh
cat kcl/deployment.k
```

The output is as follows.

```
import .common
import k8s.api.apps.v1 as k8sapps

schema Deployment(common.KubernetesObject):
    _dbEnabled: bool = False
    _dbSecretName: str = "spec.id"
    _providerConfigName: str = "spec.id"
    _container = "{}.template.spec.containers[0]".format(common.ManifestSpec)
    name = "deployment"
    base = common.KubernetesObjectBase{
        spec.forProvider.manifest = k8sapps.Deployment{
            spec = {
                selector = {}
                template = {
                    spec = {
                        containers = [{
                            name = "backend"
                            ports = [{containerPort = 80 }]
                            livenessProbe = {httpGet = {path = "/", port = 80 }}
                            readinessProbe = {httpGet = {path = "/", port = 80 }}
                            resources = {
                                limits = {cpu = "250m", memory = "256Mi" }
                                requests = {cpu = "125m", memory = "128Mi" }
                            }
                            if _dbEnabled:
                                env = [
                                    {name = "DB_ENDPOINT", valueFrom.secretKeyRef.key = "endpoint" },
                                    {name = "DB_PASSWORD", valueFrom.secretKeyRef.key = "password" },
                                    {name = "DB_PORT", valueFrom.secretKeyRef = {key = "port",  optional = True }},
                                    {name = "DB_USERNAME",  valueFrom.secretKeyRef.key = "username" },
                                    {name = "DB_NAME" },
                                ]
                        }]
                    }
                }
            }
        }
    }
    patches = common.Patches("deployment") + [
        {
            fromFieldPath = "spec.id",
            toFieldPath = "{}.selector.matchLabels.app".format(common.ManifestSpec)
        }, {
            fromFieldPath = "spec.id",
            toFieldPath = "{}.template.metadata.labels.app".format(common.ManifestSpec)
        }, {
            fromFieldPath = "spec.parameters.image",
            toFieldPath = "{}.image".format(_container)
        }, {
            fromFieldPath = "spec.parameters.port",
            toFieldPath = "{}.ports[0].containerPort".format(_container)
        }, {
            fromFieldPath = "spec.parameters.port",
            toFieldPath = "{}.livenessProbe.httpGet.port".format(_container)
        }, {
            fromFieldPath = "spec.parameters.port",
            toFieldPath = "{}.readinessProbe.httpGet.port".format(_container)
        },
        if _dbEnabled:
            {
                fromFieldPath = _dbSecretName,
                toFieldPath = "{}.env[0].valueFrom.secretKeyRef.name".format(_container)
            }, {
                fromFieldPath = _dbSecretName,
                toFieldPath = "{}.env[1].valueFrom.secretKeyRef.name".format(_container)
            }, {
                fromFieldPath = _dbSecretName,
                toFieldPath = "{}.env[2].valueFrom.secretKeyRef.name".format(_container)
            },
            {
                fromFieldPath = _dbSecretName,
                toFieldPath = "{}.env[3].valueFrom.secretKeyRef.name".format(_container)
            }, {
                fromFieldPath = _dbSecretName,
                toFieldPath = "{}.env[4].value".format(_container)
            },
        {
            fromFieldPath = _providerConfigName,
            toFieldPath = "spec.providerConfigRef.name"
        },
    ]
```

Look at that `import`. KCL comes with pre-built schemas for all core Kubernetes APIs, and many others. In this case, we're importing `k8s.api.apps.v1` and using it to define `k8sapps.Deployment` which, surprise surprise, is a Kubernetes Deployment.

*Don't be confused that the Deployment is defined inside spec.forProvider.manifest. That's a part of a Crossplane Composition which I'm not exploring in this video but only using as an example of what KCL can do.*

The `Deployment` schema I'm defining here is inheriting from the `KubernetesObject` schema I defined in the `common`. That means that it inherits everything from that one and I can define only the things that are different.

Inside that schema, I defined a few variables with names prefixed with `_`. Those are not exported and, as such, are mutable. That's a great feature of KCL. Exported variables are immutable, non-exported are not. That might be one of the things I like the most about KCL. It's immutable when exported data is concerned, but it still allows us to mutate non-exported data. If you're not familiar with terms "exported" and "non-exported", think of them as "public" and "private" in other languages. Only exported data is output to YAML or JSON.

We can see, for example, that `_dbEnabled` is a `bool` variable with the default value of `False`. The rest follows the similar pattern.

The last non-exported variable shows the usage of the `format` function that will replace open-closed curly braces (`{}`) with the value of `common.ManifestSpec`.

What else...

The `patches` value is an interesting one. it showcases unioning of collections. Over there I'm saying that the value should be a combination of a list defined in common `Patches("deployment")` and whatever is defined below. There are some common entries that should be defined in all patches and by including items from that collection with whatever is defined below, I'm avoiding repetition.

Patches itself, if you remember, is a function that returns an array of objects. That's the one we saw when we explore common.k file.

The last thing I want to show is the ability to define expressions. In this case, it is a simple `if` conditional that will include a few more entries into the list if `_dbEnabled` is True.

We saw only a fraction of what KCL can so, but I think that was enogh syntax for today. KCL is massive and you should spend a bit of time going though the docs yourself.

Outside the syntax itself, there are a couple of other things I feel are worth mentioning.

The CLI itself is not overwhelming, yet it does have a few important features outside of the obvious one that allows us to generate YAML or JSON from KCL definitions.

```sh
kcl --help
```

The output is as follows.

```
The KCL Command Line Interface (CLI).

KCL is an open-source, constraint-based record and functional language that
enhances the writing of complex configurations, including those for cloud-native
scenarios. The KCL website: https://kcl-lang.io

Usage:
  kcl [command]

Available Commands:
  clean       KCL clean tool
  completion  Generate the autocompletion script for the specified shell
  doc         KCL document tool
  fmt         KCL format tool
  help        Help about any command
  import      KCL import tool
  lint        Lint KCL codes.
  mod         KCL module management
  play        Open the kcl playground in the browser.
  registry    KCL registry management
  run         Run KCL codes.
  server      Run a KCL server
  test        KCL test tool
  version     Show version of the KCL CLI
  vet         KCL validation tool

Flags:
  -h, --help      help for kcl
  -v, --version   version for kcl

Use "kcl [command] --help" for more information about a command.
```

We can, for example, use `import` to convert existing JSON, YAML, Go structs, Terraform, OpenAPI, Kubernetes CRDs, and a few other formats into KCL. That's great as a starting point.

We can use `mod` to initiate a KCL project, add dependencies like those we saw earlier when I showed Kubernetes schemas, to package KCL, or to pull it or push it into a registry.

There is the option to `play` with KCL by spining up a local server that will allow us to interact with KCL in a browser.

And so on and so forth.

IDE support is supperb. I'm using VS Code and the KCL extension is great. It provides **syntax highlighting**, **auto-complete**, **go-to-definition**, and a few other features that make working with KCL a breeze.

There is also integration and support for a bunch of other tools like Argo CD and Flux, CI pipelines, Hashi Vault, Terraform, and a few others.

Okay. That's it as far as walkthrough it concerned. Let's talk about pros and cons.

## KCL Pros and Cons

As a reminder, my requirements are to have a language or a DSL that is focused on **data-structures**, the ability to work with **schemas**, **immutability**, and for it to be **easy to learn**.

KCL meets all those and so much more.

As a matter of fact, I could not find a single thing I don't like except that I was initially confused with the documentation but that was my fault. I was too hasty.

If I would have to nitpick, I'd say that the only potential, but minor, issue is that the docs contain quite a few spelling errors in the English version. I can only guess that Chinese version is better but, for obvious reasons, I cannot confirm that. Nevertheless, that's a minor issue that does not affect the quality of the docs.

That's it. That's the only negative thing I could find. KCL is awesome, and here are only a few out of many reasons why I think so.

**Documentation** is amazing, even though I had initial trouble understanding how it is organized. It is clear that a lot of attention is put into the design of the language and documenting every detail that mateers. I disliked it at first because of the docs but now that I went through most of it, I can safely say that was my fault. The docs are great.

Having all exported data **immutable** is just what I think we need and that's what KCL provides. The addition of mutable non-exported data is a great feature.

It's (relatively) **easy to learn**. You'll be up-and-running in no time, a day at most. You will not know everything KCL offers, that takes time, but you'll be able to do most things in a day.

What else... It's **powerful**. I did not find anything missing. Everything I needed so far is there, and I know that there's so much more so it my needs change in the future, I'm confident that KCL will be able to accommodate them.

Next, the **VS Code** plugin is great. Syntax highlighting, auto-complete, goto definitions, and all other features we might expect are there. It's a pleasure to work with KCL in VS Code.

Finally, KCL is a project donated to Cloud Native Computing Foundation (**CNCF**). That makes it less prone to future license changes and makes it more likely to have a vibrant and diverse community.

Personally, I cannot sit on more than two chairs at the same time. Until now, I was torn between CUE and Pkl. I'll kick one of those out to gain space for KCL. Pkl, you're out. KCL is in. I might be biased though. I love CUE and KCL looks very similar to it except that it's easier to learn. It's those two now.

Thank you for watching.
See you in the next one.
Cheers.

## Destroy

```sh
git checkout main
```


+++
title = 'Argo CD GitOps Promotions with Kargo (by Akuity): A Brilliant Idea with Flawed Execution?'
date = 2024-11-25T01:00:00+00:00
draft = false
+++

Most of the time, I split projects into "**it provides value**" and "**it's not good enough**". However, every once in a while a project comes along that produces both types of impressions.

There are projects that show a lot of potential. Projects that might represent some important steps toward a better way to manage or operate systems. Yet, those same projects might be showing the vision rather than being something we should adopt today.

Such a vision could be huge like, for example, **Kubernetes which was horrible** when it was first released and, at that time, I did not recommend it to anyone. Yet, its vision proved to be a valid one for all of us to pursue. The end result is what we have today. Almost everyone is using or, at least, planning to use Kubernetes in some capacity or another. It started as a vision that took a while to materialize.

There are examples of projects that have much smaller scope, yet also started as "this is not good enough today but we can see where that might take us". One of such projects is, in my opinion, Kargo by Akuity. I think that it has a lot of potential. It might be an important step forward in decoupling our workflows. It might be just what we need to orchestrate promotions of releases managed by GitOps or, to be more precise, Argo CD. The idea behind it is great, yet, I am not sure that there are enough compeling reasons to use it today. It's great. It truly is. Yet... Something might be missing.

<!--more-->

{{< youtube RoY7Qu51zwU >}}

## Intro

You see, the problem that many of us are facing today are promotions of releases managed by GitOps tools like Argo CD, Flux, and others. That is the case in big part because **GitOps broke our workflows**.

![](/logo/jenkins.png?height=15vw&classes=inline)
![](/logo/github-actions.png?height=15vw&classes=inline)
![](/logo/circle-ci.webp?height=12vw&classes=inline)

In the past, we were used to having a **single workflow** with Jenkins, GitHub Actions, CircleCI, or any other among dozens of similar tools.

We were defining workflows that automate all or, at least, part of our Software Development Lifecycle or SDLC. We would build stuff, we would run some tests, we would do static analysis, and whatever else needed to be done. That also included deployments. Jenkins, or whichever other tool we used would, at some point, deploy a new release to a test environment, then it would execute some functional, integration, and other tests. From there on, we would deploy to some other environment and perform some other tasks, and such a process would continue all the way until a new release is running in production, with yet another round of tasks that should be executed afterwards.

Then came GitOps and said "You know what? You should not be deploying stuff directly but store the **desired state in Git** and let other tools that are completely unrelated with your pipelines do the synchronization." That broke workflows and we failed to figure out a commonly used way to stitch them back together. We tried, but we shouldn't have. It was time to break the monolithic approach to workflows into smaller independent pieces that are **loosely coupled** and are **operating independely** of each other. It was time to start applying lessons learned from microservices into workflows.

![](/logo/cloud-events.png?height=8vw&classes=inline)
![](/logo/cd-events.webp?height=10vw&classes=inline)

The problem is that I don't think we got there. There are many pieces missing and, the biggest one, in my opinion, is unwillingness to adopt event-based approach. More often than not, third-party apps we use today are failing to understand that they are not alone but parts of a bigger ecosystem. Many tools are not equipped to receive signals and react. That is unfortunate since we even built standards to do so. We have [CloudEvents](https://cloudevents.io) and [CDEvents](https://cdevents.dev) that are a standard how processes can talk to each other without being coupled. Just as [OpenTelemetry](https://opentelemetry.io) defined a standarad for almost all observability so that they can all "play nicely" with each other, CloudEvents tends to do the same for communication between processes and tasks.

Long story short, I'm on a lookout for tools that can do something very well, yet "play nicely" with the rest of the ecosystem so that I can plug them into it.

That's why I got attracted to Kargo. It solves a real, yet very specific problem. It can help us define promotions of our releases managed with GitOps. Now, the important thing to keep in mind is that promotions do not work in isolation. We still need to build stuff, and push stuff, and test stuff, and do whatever else needs to be done besides deployments themselves. Promotions are a part of SDLC. Keep that in mind while reading the rest of this post.

Going back to Kargo... It's been a while since I discovered it and used it without planning to make a video about it. I was waiting for it to mature. That happened in October 2024 when it reached version 1. It is not alpha or beta any more. It is, at least according to its maintainers, ready for prime time. So, I'll treat it as such.

Specifically, there are two questions I will try to answer today. We'll try to figure out whether it solves a problem and whether it can be used together with the rest of the tooling we have in our systems. In other words, we'll see whether it is good at what it does, at promotions, and whether we can actually use it.

Let's dig into it.

## Setup

> Replace `[...]` with your GitHub organization or username

```sh
export GITHUB_ORG=[...]
```

> Watch the [GitHub CLI (gh) - How to manage repositories more efficiently](https://youtu.be/BII6ZY2Rnlc) video if you are not familiar with GitHub CLI.

```sh
gh repo fork vfarcic/kargo-demo --clone --remote \
    --org $GITHUB_ORG

cd kargo-demo

gh repo set-default
```

> Select the fork as the default repository.

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

> Install [kargo CLI](https://kargo.akuity.io/quickstart#installing-the-kargo-cli)

## Argo CD ApplicationSet

I have a fairly simple application defined as Kubernetes Kustomize.

Most of the files are in the `base` directory.

```sh
ls kustomize/base/
```

The output is as follows.

```
-- deployment.yaml
-- hpa.yaml
-- ingress.yaml
-- kustomization.yaml
-- service.yaml
```

The environment specific files are in `overlays` directories.

```sh
ls kustomize/overlays/
```

The output is as follows.

```
-- dev
-- pre-prod
-- prod
```

For example, let's take a quick look at `pre-prod` `kustomization.yaml`.

```sh
cat kustomize/overlays/pre-prod/kustomization.yaml
```

The output is as follows.

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- ../../base
patches:
- target:
    kind: Ingress
    name: silly-demo
  patch: |-
    - op: replace
      path: /spec/rules/0/host
      value: silly-demo.pre-prod.127.0.0.1.nip.io
```

We can see that it overwrites `Ingress` by setting the `host` to `silly-demo.pre-prod`. That way, the same application could run in different environments yet be accessible through unique hosts.

Normally, I would have more customisations like, for example, running a single replica in *dev* but a minimum three repliacas in *prod*. Nevertheless, that simple customization of the host should be enough for today's demo.

I will be synchronizing the actual into the desired state using Argo CD. More specifically, I have an ApplicationSet that will do the heavy lifting.

```sh
cat application-set.yaml
```

The output is as follows.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: silly-demo
spec:
  generators:
  - list:
      elements:
      - stage: dev
      - stage: pre-prod
      - stage: prod
  template:
    metadata:
      name: silly-demo-{{stage}}
      annotations:
        kargo.akuity.io/authorized-stage: silly-demo:{{stage}}
    spec:
      project: default
      source:
        repoURL: https://github.com/vfarcic/kargo-demo
        targetRevision: stage/{{stage}}
        path: .
      destination:
        server: https://kubernetes.default.svc
        namespace: silly-demo-{{stage}}
      syncPolicy:
        syncOptions:
        - CreateNamespace=true
```

We can see that there are three stages, `dev`, `pre-prod`, and `prod` so the `ApplicationSet` will create three Argo CD applications. Each of them will have the desired state in a different branch `stage/` whatever-the-stage-name-is. I'm not fond of using branches for GitOps, but, as you'll see later, those will not be branches you're used to. Kargo will do something very special with them.

Finally, the resources of each app will be in a separate `silly-demo-` whatever-the-stage-name-is Namespace.

The important thing to note here is that I did not set the `syncPolicy` to be automated. As a result, Argo CD will not be syncing new commits to those branches automatically. We'll see later why is that so.

Let's apply the ApplicationSet,...

```sh
kubectl --namespace argocd apply --filename application-set.yaml
```

...and take a look at what we got.

```sh
kubectl --namespace argocd get applications
```

The output is as follows.

```
NAME                  SYNC STATUS   HEALTH STATUS
silly-demo-dev        OutOfSync     Missing
silly-demo-pre-prod   OutOfSync     Missing
silly-demo-prod       OutOfSync     Missing
```

We can see that the `SYNC STATUS` is `Unknown`. Argo CD detected that there is a difference between what's in those branches but, since we did not set the sync policy to automated, it is not allowed to do anything about it. It detected a drift but it is not reconciling it just yet.

I will assume that you are familair with Argo CD so I won't go deeper into Argo CD in this post. If that's not the case, there are two things I will say to you.

First of all, be ashamed. Argo CD is one of the most important tools in the Kubernetes ecosystem. Second, once the shame wears off, go and watch the videos in the [GitOps playlist](https://youtube.com/playlist?list=PLyicRj904Z99dJk8bOygbov5up5YYvoZV) to get familiar with the subject.

Now we are ready to explore Kargo.

## Kargo (by Akuity) Promotion Definitions

To understand Kargo, we need to understand a few different concepts, all represented by different kinds of resources.

To begin with, we have a project.

A project is a collection of other Kargo resources that, when combined, describe one or more delivery pipelines.

Here's an example.

```sh
cat kargo-manifests/project.yaml
```

The output is as follows.

```yaml
apiVersion: kargo.akuity.io/v1alpha1
kind: Project
metadata:
  name: silly-demo
spec:
  promotionPolicies:
    - stage: dev
      autoPromotionEnabled: true
```

There's not much happening over there since, as I already mentioned, a project is a way to group other resources. The only important thing to note is that, in this example, there is a promotion policy (`promotionPolicies`) that spacifies that the stage `dev` should have `autoPromotionEnabled`. As a result, as we'll see later, whenever there is a new release that matches certain criteria, Kargo will promote it to the *dev* environment automatically.

Let's apply the project before we move into other, arguably more interesting resources.

```sh
kubectl --namespace silly-demo apply \
    --filename kargo-manifests/project.yaml
```

The important thing to note is that a project is represented as a Namespace. Let's confirm that by outputting all the Namespaces in the cluster.

```sh
kubectl get namespaces
```

The output is as follows.

```
NAME                 STATUS   AGE
argo-rollouts        Active   3h33m
argocd               Active   3h33m
cert-manager         Active   3h34m
default              Active   3h34m
ingress-nginx        Active   3h34m
kargo                Active   3h32m
kube-node-lease      Active   3h34m
kube-public          Active   3h34m
kube-system          Active   3h34m
local-path-storage   Active   3h34m
silly-demo           Active   39s
```

There we go. We can see that the Namespace `silly-demo` has been created. That's where the resources that define Kargo promotions will be living.

Now, before we proceed, we should create a Secret with the credentials that will allow Kargo to interact with the Git repository where are manifests are.

```sh
echo "
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: silly-demo-repo
  labels:
    kargo.akuity.io/cred-type: git
stringData:
  repoURL: ${GITHUB_REPO_URL}
  username: ${GITHUB_USERNAME}
  password: ${GITHUB_PAT}
" | kubectl --namespace silly-demo apply --filename -
```

The only thing that matters related to the secret is that it needs to have the `cred-type` label so that Kargo knows that's the secret with the credentials.

Now we can move to more interesting resources, starting with the  warehouse.

Think of warehouses as... Well... Warehouses. That's where we keep goods that are, later on, transported by freights. In the context of Kargo, a warehouse is where we keep our releases.

Here's an example.

```sh
cat kargo-manifests/warehouse.yaml
```

The output is as follows.

```yaml
apiVersion: kargo.akuity.io/v1alpha1
kind: Warehouse
metadata:
  name: silly-demo
spec:
  subscriptions:
  - image:
      repoURL: ghcr.io/vfarcic/silly-demo
      semverConstraint: ^1.x
      discoveryLimit: 5
```

That warehouse subscribes (`subscriptions`) to releases that can be coming from Helm chart repos, Git repositories, or, as in this case, `image` repos.

Today, we are subscribing to images in the `silly-demo` repo and we are interested only in `1.x` releases. That means that any release with semantic version starting from 1.0.0 all the way until, but not included, version 2. So, if we make any release with major version one, that release will be shipped into that warehouse.

There are many ways we can configure the range of releases we are interested in and the only requirement is to use semantic versioning or [SemVer](https://github.com/masterminds/semver). If you're not, you should anyways.

Finally, the `discoveryLimit` specifies how many versions we're interested in. In this case, I see no reason to go back further than five versions in the past.

Now, that we saw how to fill a warehouse with releases, let's see how to define places where they should be delivered through Kargo Stages.

Typically, we would be delivering releases to environments, but Kargo prefers to call them stages. That makes sense since many of us associate environments with servers or clusters where we run specific versions of our applications. However, there can be many other definitions like instances of specific versions of an application, infrastructure in different phases, and so on. Stages make more sense since we are really talking about different phases through which we might need to pass until something is in production, and that something can be... Well... Anything. It could be a release of an application used for testing purposes, it could be a database running in production, it could be a cluster that we are upgrading and validating before we apply the same process to production, and so on and so forth.

The important note is that even though we will be using an application in today's examples, it can really be anything that needs to move through different stages until it reaches production.

Here are definitions of a few stages through which we'll move our application.

```sh
cat kargo-manifests/stage-*.yaml
```

The output is as follows.

```yaml
apiVersion: kargo.akuity.io/v1alpha1
kind: Stage
metadata:
  name: dev
spec:
  requestedFreight:
  - origin:
      kind: Warehouse
      name: silly-demo
    sources:
      direct: true
  promotionTemplate:
    spec:
      steps:
      - uses: git-clone
        config:
          repoURL: https://github.com/vfarcic/kargo-demo
          checkout:
          - branch: main
            path: ./src
          - branch: stage/dev
            create: true
            path: ./out
      - uses: git-clear
        config:
          path: ./out
      - uses: kustomize-set-image
        as: update-image
        config:
          path: ./src/kustomize/base
          images:
          - image: ghcr.io/vfarcic/silly-demo
      - uses: kustomize-build
        config:
          path: ./src/kustomize/overlays/dev
          outPath: ./out/manifests.yaml
      - uses: git-commit
        as: commit
        config:
          path: ./out
          messageFromSteps:
          - update-image
      - uses: git-push
        config:
          path: ./out
          targetBranch: stage/dev
      - uses: argocd-update
        config:
          apps:
          - name: silly-demo-dev
            sources:
            - repoURL: https://github.com/vfarcic/kargo-demo
              desiredCommitFromStep: commit
apiVersion: kargo.akuity.io/v1alpha1
kind: Stage
metadata:
  name: pre-prod
spec:
  requestedFreight:
  - origin:
      kind: Warehouse
      name: silly-demo
    sources:
      stages:
      - dev
  promotionTemplate:
    spec:
      steps:
      - uses: git-clone
        config:
          repoURL: https://github.com/vfarcic/kargo-demo
          checkout:
          - branch: main
            path: ./src
          - branch: stage/pre-prod
            create: true
            path: ./out
      - uses: git-clear
        config:
          path: ./out
      - uses: kustomize-set-image
        as: update-image
        config:
          path: ./src/kustomize/base
          images:
          - image: ghcr.io/vfarcic/silly-demo
      - uses: kustomize-build
        config:
          path: ./src/kustomize/overlays/pre-prod
          outPath: ./out/manifests.yaml
      - uses: git-commit
        as: commit
        config:
          path: ./out
          messageFromSteps:
          - update-image
      - uses: git-push
        config:
          path: ./out
          targetBranch: stage/pre-prod
      - uses: argocd-update
        config:
          apps:
          - name: silly-demo-pre-prod
            sources:
            - repoURL: https://github.com/vfarcic/kargo-demo
              desiredCommitFromStep: commit
apiVersion: kargo.akuity.io/v1alpha1
kind: Stage
metadata:
  name: prod
spec:
  requestedFreight:
  - origin:
      kind: Warehouse
      name: silly-demo
    sources:
      stages:
      - pre-prod
  promotionTemplate:
    spec:
      steps:
      - uses: git-clone
        config:
          repoURL: https://github.com/vfarcic/kargo-demo
          checkout:
          - branch: main
            path: ./src
          - branch: stage/prod
            create: true
            path: ./out
      - uses: git-clear
        config:
          path: ./out
      - uses: kustomize-set-image
        as: update-image
        config:
          path: ./src/kustomize/base
          images:
          - image: ghcr.io/vfarcic/silly-demo
      - uses: kustomize-build
        config:
          path: ./src/kustomize/overlays/prod
          outPath: ./out/manifests.yaml
      - uses: git-commit
        as: commit
        config:
          path: ./out
          messageFromSteps:
          - update-image
      - uses: git-push
        config:
          path: ./out
          targetBranch: stage/prod
      - uses: argocd-update
        config:
          apps:
          - name: silly-demo-prod
            sources:
            - repoURL: https://github.com/vfarcic/kargo-demo
              desiredCommitFromStep: commit
```

The first `Stage` is called `dev`. That's a stage that should always have the latest release of the application that developers can use to experiment with whatever they're doing.

We can see that it requests a freight (`requestedFreight`) from the `Warehouse` `silly-demo`.

To deliver that release we'll use a `promotionTemplate` with specific steps.

First, we'll clone the code from git (`git-clone`). We'll take whatever is in the `main` branch of the repo and `create` a branch `stage/dev`.

Next, we'll execute `git-clear` to remove anything that might already exist in that branch. As you'll see later, Kargo tends to use branches in a very specific way that might be confusing at first but, kind of, makes sense once we get used to it.

Next, since we're using Kustomize today, we'll `update-image` tag inside *kustomize.yaml* stored in the `kustomize/base` directory. That tag will be applied to the `silly-demo` image.

After that we are building YAML from Kustomize manifests (`kustomize-build`). That part is important since Kargo assumes that we might be defining our manifests through Kustomize overlays, Helm templates, or any other way, but that we want the desired state to be defined as "pure" YAML. I love that idea since I believe that writing YAML is horrible and not flexible enough, yet having YAML as the final desired state is what we should do since YAML alone, even though might not be a good format to define something, is great to deduce what something is. Argo CD follows the same pattern even though we might not be aware of it. When we give it, let's say, a Helm chart, it converts it into YAML before sending it to Kubernetes API. Kargo takes it a step further by storing that generated YAML in Git and instructing Argo CD to apply it. It's doing the conversion that Argo CD would normally do.

Anyways... The final output will be stored as `manifests.yaml` file in the specified branch.

Further on, we are committing (`git-commit`) and pushing (`git-push`) the manifest back to Git.

Finally, we are instructing Argo CD to update (`argocd-update`) Argo CD Application `silly-demo-dev`. As you saw earlier, we configured the Argo CD Applications not to synchronize manifests automatically. This is the reason we did that. Kargo will initiate the sync at the end of the process.

The other two stages are following the same pattern.

The `pre-prod` stage uses `dev` as the source meaning that only after we deployed the release to the development environment it will become eligible for promotion to *pre-production*.

The `promotionTemplate` in this stage is mostly the same as in the previous one except that the branch is different (`stage/pre-prod`), that it will build the manifests based on the Kustomization in the `overlays/pre-prod` directory, and that we are, this time, updating Argo CD Application `silly-demo-pre-prod`.

The last `Stage` is `prod` which, as you can guess, should promote the release to production. It follows the same pattern as the previous two stages so there is probably no need to go through it.

One thing you should understand is that we are not limited to releases from a single warehouse. We could combine them if instead of independently deployable microservices we are working with distributed monoliths (while still calling them microservices).

Let's apply all those manifests we explored, and see the whole promotion process in action.

```sh
kubectl --namespace silly-demo apply --filename kargo-manifests/
```

## Kargo Promotions in Action

We got a project,...

```sh
kubectl get projects
```

The output is as follows.

```
NAME         READY   STATUS                                     AGE
silly-demo   True    Project is initialized and ready for use   21m
```

...and a warehouse.

```sh
kubectl --namespace silly-demo get warehouses
```

The output is as follows.

```
NAME         SHARD   AGE
silly-demo           88s
```

There's nothing special about those two beyond what we saw when we looked at the manifests.

Stages are much more interesting.

```sh
kubectl --namespace silly-demo get stages
```

The output is as follows.

```
NAME       SHARD   CURRENT FREIGHT                            HEALTH    PHASE           AGE
dev                0608f9513259128d32af62bb4a5dae892f712d41   Healthy   Steady          106s
pre-prod           0/1 Fulfilled                                        NotApplicable   106s
prod               0/1 Fulfilled                                        NotApplicable   106s
```

We can see that the `dev` stage is already `Healthy` and in the `Steady` phase. That means that Kargo detected an image that matches our criteria and already instructed Argo CD to synchronize it into the cluster. That happened because we set, in the project, the promotion policy for the *dev* stage to *autoPromotionEnabled*. As a result, Kargo, which is constantly monitoring, in our case, images from a registry, detected that there is at least one that matches the criteria and put it into the Warehouse. Since the *dev* environment is set to auto-promotion, it did just that. It promoted the latest release to the *dev* environment.

We can see that's what really happened by listing all the `promotions` in the *silly-demo* Namespace.

```sh
kubectl --namespace silly-demo get promotions
```

The outpus is as follows.

```
NAME                                     SHARD   STAGE   FREIGHT                                    PHASE       AGE
dev.01jaxj2ct03tmbedp7yda4fxxw.0608f95           dev     0608f9513259128d32af62bb4a5dae892f712d41   Succeeded   2m14s
```

There is it. The promotion to the `dev` stage was indeed created automatically.

As a result, we should have all the resources that belong to the application running in the `silly-demo-dev` Namespace.

```sh
kubectl --namespace silly-demo-dev get all,ingresses
```

```
NAME                             READY   STATUS    RESTARTS   AGE
pod/silly-demo-7454d8bbb-99j4t   1/1     Running   0          2m58s

NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/silly-demo   ClusterIP   10.96.1.238   <none>        8080/TCP   2m58s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/silly-demo   1/1     1            1           2m58s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/silly-demo-7454d8bbb   1         1         1       2m58s

NAME                                   CLASS   HOSTS                             ADDRESS     PORTS   AGE
ingress.networking.k8s.io/silly-demo   nginx   silly-demo.dev.127.0.0.1.nip.io   localhost   80      2m58s
```

There they are.

We can even confirm that the application is indeed running by sending a request to it.

```sh
curl "http://silly-demo.dev.127.0.0.1.nip.io"
```

The output is as follows.

```
This is a silly demo version 1.4.299
```

A truly interesting and a unique way Kargo handles promotions can be observed by taking a look at the `manufests.yaml` file in the `stage/dev` branch.

```sh
gh browse --repo $GITHUB_ORG/kargo-demo \
    --branch stage/dev manifests.yaml
```

We can observe three important things.

![](github-manifests.png)

First, Kargo created a pure YAML manifest made specifically for that environment.

It's not Kustomize that we have in the main branch. Instead, it built it into YAML. If we used Helm, it would do the same. It would convert it into YAML.

That's probably the part I like the most about Kargo. One of the reasons we use GitOps is the ability to observe the desired state. If, in it's final form, it would be in Kustomize, or Helm, or whatever we might be using, it could be hard to deduce what the intention is. We would need to do a mental exercise of applying patches and overlays in case of Kustomize, deduce what would be the output of all the conditionals, loops, variable substitutions in case of Helm, or something similar in other format. This way we get the best of both worlds. We get to keep defining what we want in whichever format we like using, as long as it's supported by Kargo, and we get a nice and clean YAML with the final desired state.

That's something I've been doing for a while now, but without Kargo. If, for example, I'm using GitHub Actions to build images, run tests, and do whatever else I'm doing in my CI/CD proceses, I would, at some point, execute `kustomize build`, `helm template`, `kcl run`, or whateer the command is to convert my stuff into YAML, store it in Git, and once I do that, let Argo CD do the rest. Kargo does the same, but in a separate branch.

Speaking of branches...

![](github-branch.png)

The second thing we can observe is that `manifests.yaml` is the only file in that branch. It's not a clone of the main branch. It has nothing to do with it. It's only a place where the whole desired state is defined in a single file.

Finally, even though we cannot observe that just yet, Kargo always wipes the branch clean before it generates the manifest. It does not disrupt the main branch in any form or way.

All in all, this is a very unusual usage of branches and I'm not even sure that it makes sense, yet it works.

There's another concept we did not yet explore. There are fraights.

Just as "real" freights transport goods from one place to another, Kargo freights are a sort of meta-artifacts that transition releases from the warehouses to stages.

Since we saw that one release is already running in the *dev* environment, we should be able to see it.

```sh
kubectl --namespace silly-demo get freights
```

The output is as follows.

```
NAME                                       ALIAS                  ORIGIN (KIND)   ORIGIN (NAME)   AGE
0608f9513259128d32af62bb4a5dae892f712d41   oldfashioned-penguin   Warehouse       silly-demo      8m4s
```

For now, we have only one freight since only one release was "transported" to one environment.

Now, let's take a look at the status of that freight by retrieving its name and storing it into an environment variable, and...

```sh
export FREIGHT_NAME=$(kubectl --namespace silly-demo \
    get freights --output jsonpath="{.items[0].metadata.name}")
```

...using that variable to `get` that `freight` and output the `status`.

```sh
kubectl --namespace silly-demo get freight $FREIGHT_NAME \
    --output jsonpath="{.status}" | jq .
```

The output is as follows.

```json
{
  "verifiedIn": {
    "dev": {}
  }
}
```

We can see that, for now, that freight has been verified (`verifiedIn`) only in the `dev` environment.

Now, let's imagine that we are happy with that release and that we would like to promote it to the pre-production environment for some serious testing.

To do that, first we need to login into Kargo running in the cluster.

```sh
kargo login https://127.0.0.1:31444 \
    --admin --password admin --insecure-skip-tls-verify
```

Now we can execute `kargo promote` specify the `--project`, the `--freight` that already delivered that same release to the *dev* environment, and the `--stage` where we want it which, in this case, is `pre-prod`.

```sh
kargo promote --project silly-demo --freight $FREIGHT_NAME \
    --stage pre-prod
```

The output is as follows.

```
promotion.kargo.akuity.io/pre-prod.01jaxk0q68sa955v1f9kgz0asx.0608f95 promotion created
```

That's it. We manually promoted the release that was running in the *dev* environment to pre-production.

We can confirm that's truly the case by listing all the promotions in the `silly-demo` Namespace.

```sh
kubectl --namespace silly-demo get promotions
```

The output is as follows.

```
NAME                                          SHARD   STAGE      FREIGHT                                    PHASE       AGE
dev.01jaxj2ct03tmbedp7yda4fxxw.0608f95                dev        0608f9513259128d32af62bb4a5dae892f712d41   Succeeded   16m
pre-prod.01jaxk0q68sa955v1f9kgz0asx.0608f95           pre-prod   0608f9513259128d32af62bb4a5dae892f712d41   Succeeded   18s
```

We can see that the `pre-prod` promotion `succeeded`.

We should be able to see the details of that promotion by describing the resource. However, there is one really annoying thing about it. Kargo does not set any labels in the promotions it is generating so we cannot execute something like `kubectl describe` with `--selector` to select the one we need. Labels are the cornerstone of all Kubernetes resources. We use them in all sort of ways, one of them being the ability to select or filter resources we're interested in. Kargo maintainers did not bother adding them, so we have to do silly workarounds like copy the promotion name from the previous output, paste it as the value of an environment variable,...

> Replace `[...]` with the `NAME` of the promotion in the `pre-prod` `STAGE`

```sh
export PROMOTION_NAME=[...]
```

...and use it to describe that specific promotion.

```sh
kubectl --namespace silly-demo describe promotion $PROMOTION_NAME
```

The output is as follows (truncated for brevity).

```
Name:         pre-prod.01jaxk0q68sa955v1f9kgz0asx.0608f95
...
Spec:
  Freight:  0608f9513259128d32af62bb4a5dae892f712d41
  Stage:    pre-prod
  Steps:
    Config:
      Checkout:
        Branch:  main
        Path:    ./src
        Branch:  stage/pre-prod
        Create:  true
        Path:    ./out
      Repo URL:  https://github.com/vfarcic/kargo-demo
    Uses:        git-clone
    Config:
      Path:  ./out
    Uses:    git-clear
    As:      update-image
    ...
Status:
  ...
  Freight:
    Images:
      Digest:    sha256:2b1af44edf0ddb4dd2d75b2c9c4cd5e68e7c374dd5960fffa24e15d359ab9d61
      Repo URL:  ghcr.io/vfarcic/silly-demo
      Tag:       1.4.299
    ...

- ghcr.io/vfarcic/silly-demo:1.4.299
Events:
  Type    Reason              Age    From                  Message
  ----    ------              ----   ----                  -------
  Normal  PromotionCreated    4m16s  api                   Promotion created for Stage "pre-prod" by "admin"
  Normal  PromotionSucceeded  4m11s  promotion-controller  Promotion Succeeded
```

The output of the promotion provides all the important information we might need.

We can see which `Freight` was involved in the promotion, which `Stage` it was promoted to, and which `Steps` were executed to get us there.

If we take a look at the `Status`, we can see which image `Digest` was used, what the `Repo URL` is, what the image `Tag` is, and so on and so forth.

Finally, there are `events` depicting what happened and when it happened.

If you ever need to deduce how was a release created, where it was deployed, which tag is used, or anything else related to a specific release in a specific environment, the Promotion resource is where you will find the information.

That's it. The release was promoted from dev to pre-production and we can now run tests and do whatever else we need to do to gain confidence we might need to promote it to production, or not.

If we do indeed decide to promote it to prod, all we have to do is execute another `kargo promote` command, just as we did before, but, this time, set the `--stage` to `prod`.

```sh
kargo promote --project silly-demo --freight $FREIGHT_NAME \
    --stage prod
```

That's it. We're done, at least when the first release is concerned. We started with automatic promotion to the development environment, and, from there on, promoted it manually to pre-production and production.

```sh
kargo dashboard
```

> Type `admin` as the password.

I should probably say that Kargo comes with a Web UI which is nothing special, yet, it you like pretty colors, you can use it to see essentially the same information we've been observing from the terminal, just in a more colorful way.

It even allows us to create resources directly from the UI and, by doing that, contradicting itself when stating that it is a GitOps tool since those resources will be created directly in the cluster completely bypassing Argo CD for, in my opinion, no good reason other than "we did not bother adding the ability to push manifests of those resources to Git."

I feel that we do not need yet another dashboard dedicated to a very specific part of the overal SDLC process. I would be much happier if Kargo extended Argo CD UI, especially since the tool itself is tightly coupled to it anyway.

From now on, all we would have to do is build and push new images and Kargo, with the setup we used in this demo, will be releasing them automatically to the development environment and waiting for us to promote it manually to others.

Now that we saw Kargo in action, let's do a bit of trash talk.

## Kargo Critique

I love the idea behind Kargo. It's fanstactic, yet it makes very wrong assumptions. It's as if it thinks that the only tool people use in their developer workflows is Argo CD. If that would be the case, Kargo would make a lot of sense. However, that is not the case.

Let me explain.

When we make changes to code, we tend to run workflows that run unit tests, build and publish artifacts, deploy those artifacts to one environment, run some tests, do something else, deploy to some other environment, execute more tasks, and so on and so forth, all the way until a release is running in production and we start observing how it behaves.

Keep that in mind when you try to answer the following questions.

How will you run tasks that should be run after Kargo promotes a release to an environment? Will you execute `kargo promote` and then go to GitHub Actions, Jenkins, or whichever other workflow tool you're using to execute whatever you need to execute to, let's say, validate that the new release is performing as expected? Since Kargo is not sending any events to anyone, execution of that workflow would need to be performed manually. If that's the case, we can just as well add promotion logic into that workflow. It's not really hard to execute `yq` to modify a tag in a manifest and push it back to Git so that Argo CD does the synchronization.

Here's another question. Once, let's say, you run tests that confirm that a release is ready for production, how will you execute promotion with Kargo? If those tests are executed through a workflow, isn't it easier to, again, run `yq` to modify the manifests describing production than to stop the workflow, fire up a terminal, and execute `kargo promote` again?

Now, you might say that you will execute `kargo promote` commands from your workflows, but how will you find out what the autogenerated name of the promotion is? Remember, there are no labels that can help us filter what we need through `kubectl` selectors.

What I'm trying to say is that there are some operations we need to perform before and after promotions and Kargo does not offer hooks we could use to trigger it or hooks that could be fired from Kargo to trigger some other workflows.

Kargo was not designed to "play" nicely with tools we might be using. The only assumption it makes is that we have Argo CD and nothing else. It's not even trying to "play" with Argo Workflows which is in the same family of projects.

Another critique is that defining steps required for each stage is not much better or even simpler to write than performing similar operations through whichever workflow tool you might be using. It is, essentially, replacing `yq` and `git` commands.

Normally, I would not consider that as a downside since Kargo brings structure and predefined proces to otherwise random Shell commands we were executing from workflows. That's a good thing, but only if it would assume that there are workflows, which it doesn't.

Here's what I would like Kargo to do, besides the things it is already doing.

I would want Kargo to be able to subscribe to events and to publish events. That could be to NATS or any other pub/sub messaging server.

If it would do that, I could, for example, execute a workflow that does whatever it does and, when it reaches a part that would normally promote a release to an environment, just publish an event which would be picked up by Kargo and initiate a promotion. Similarly, I would like Kargo to publish events indicating that promotions are done so that workflows can subscribe to them and execute whichever tasks we would normally have to run to test those promotions or whatever else we might be doing.

What I'm trying to say is that Kargo tries to solve only a small part of development lifecycle. Promotions cannot be considered something that runs in isolation but a part of a much bigger process. As such, Kargo must provide ways to be plugged into that lifecycle. It cannot assume that all we're doing is promoting releases. Those hooks, or integrations, or wharever you want to call it, need to be a part of it and need to be documented. Otherwise, I honestly do not know how to integrate Kargo into what's we're doing in a way that it provides a return of investment.

Okay, now we can jump into my favorite section and talk about pros and cons.

## Kargo Pros and Cons

Let's start with negatives, with cons.

**Cons**
* No integrations
* Documentation is bad
* Branches
* Opinionated
* Separate workflow
* Ignores GitOps

The biggest issue is that there are **no integrations** with workflow tools beyond simply executing *kargo promote* commands which pose difficulties by themselves. I'm not sure how those integrations should look like but I am sure that there should be a way to trigger a promotion after some tasks were executed and that some other tasks might need to run after a promotion is finished.

**Documentation is bad**. Quickstart, for example, does not work, at least not at the time of this writing. For example, the stages in the quickstart are not set correctly so all three of those showcased there are using the same *stages/test* Kustomize directory. At the end of it, it says "we leave it as an exercise to the reader to use the dashboard to progress the Freight from test to uat and again from uat to prod." That won't work since Services in all three environments will try to use the same port and fail. If you don't believe me, I dare you to do what quickstart says and do the exercise. It won't work, unless the quickstart is rewritten by the time you read this.

That's not the only issue in the documentation but a symptom that shows that not enough attention was put into it. It's not easy to deduce what each part of specs are used for, there are not enough examples of different use cases, and so on and so forth. The documentation is very barebone and limited to the minimum, without even being tried out by the authors.

Now, I would not be so negative on the documentation for such a young project. However, the maintainers chose to promote it to version 1. It's supposed to be mature and stable. I don't think that is the case and I would consider Kargo beta, at best. Still, if it's version 1, it should be treated as such.

I'm not sure I'm sold on Kargo's usage of **branches** that contain only the built manifest. I'm not sure that's a better choice than simply using a separate repository. Those branches are not related to the mainline anyway. I assume that's not the only way we can use Kargo, but it is not clear from the documentation what other ways are.

Branches lead me to the next closely related issue. Kargo is very opinionated. That's not necessarily a bad thing for everyone. Some might benefit from a tool that tells them that there is one way and no other way to do something. However, many of us are already using Argo CD for a while now and I feel that a higher level of flexiblity is needed to fit Kargo into existing workflows and organization.

Kargo is, effectively, doing the same job as a workflow would do, including Argo Workflows. It is a set of steps that are executed sequentially. That's not necessarily a bad thing, except that it messes up with other steps in our workflows, those not directly related to promotions. In other words, Kargo is a **separate workflow** that does not fit into existing workflows yet it cannot avoid them.

I also have an issue with Kargo calling itself "Multi-Stage GitOps Continuous Promotion" even though GitOps is not applied to it in any form or way, at least not in the documentation. The docs instruct us to install Argo CD, but then use *kubectl apply* to install the rest, including resources we explored in this post. Shouldn't Argo CD setup everything required to use Kargo if Kargo is *GitOps Continuous Promotion* tool? It's a GitOps tool with documentation that **ignores GitOps**.

I think that most of my issues come from Kargo promoting itself as version 1 while it should be beta at best, maybe even alpha. It's a relatively young project and there is nothing wrong in it being alpha or beta for a while longer until the community figures out what it really is. If it wasn't promoted to version 1, I would not even talk about pros and cons but about the idea behind it, and the idea is great. We need a tool like Kargo. We need a way to manage promotions in a more GitOps-friendly way. So, if you choose to ignore the claims that it is ready, you can just as well ignore all my cons.

Let's jump into the good stuff, into pros.

**Pros**
* Standardized promotions
* Visibility
* Guardrails

To begin with, there is promotion itself with its predefined steps. I see it as an attempt to have **standardized promotions**, and we need that instead each of us fiddling with `yq` commands in our workflows. There is a need for something like that, hence there is the need for Kargo.

The next great thing is the **visibility**. With Kargo, we have almost complete visibility at what was promoted, where it was promoted, when it was promoted, which steps were executed in the promotion, and so on and so forth. We can have that information by inspecting Kargo's Custom Resources (CRs) as well as through the Web UI. The latter is something I'm not happy about since I would want to see information about promotions either in lifecycle workflows or, if that's not an option, in Argo CD itself. Having a separate UI only for promotions means that the visibility it not very visible in the context of everything else we're doing. Still, I'm giving this one a thumbs up with a note that I expect it to integrate its data into other tools.

Finally, there are **guardrails**. Being able to define what can be promoted and under which conditions is great, especially when promoting manually. The guardrails like that are not as useful in the scanerios where everything is automated, but that's not the goal of Kargo. It is, in my opinion, designed for those who want do perform some, if not all promotions manually and, in those scenarios, guardrails are essential.

All in all, I think that Kargo is off to a **great start** and I can imagine it growing to a very important and useful project, especially for those practicing GitOps. However, it's **not there yet**. We have, what I would consider, an early beta implementation that should serve as a starting point of a vision that is not accomplished just yet. It is not mature just yet, in spite of the fact that it claims otherwise. It's a great start and I'm eager to see where it will go from here.

Before we leave, let me give you an important note. My "judgement" of Kargo is based exclusively on the open source version of it. Kargo is also a part of the Akuity platform where some, if not all, of my concerns might have been addressed. Do not judge me for judging only Kargo open source.

## Destroy

```sh
chmod +x destroy.nu

./destroy.nu

exit
```


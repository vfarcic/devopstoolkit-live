
+++
title = 'Full Application Setup in Internal Developer Platform (IDP) with Crossplane'
date = 2025-01-20T14:00:00+00:00
draft = false
+++

What if I tell you that fifty of so lines of YAML will get you everything you might need to work on an application. What if everything includes **Git repo**, **branches**, **pull requests**, **CI workflows**, **scripts** needed both for local development and CI, **Dockerfile**, a bunch of **Kubernetes resources** and quite a few other things you might need. What if everything is really everything and, at the same time, tailor-made for **your specific needs**?

<!--more-->

{{< youtube WpgiVlODt4I >}}

## Intro

Let's talk about the only thing that truly matters in Developer Platforms; **applications**.

Now, that might sound as an overreaction. Sure, applications are important, but so is infrastructure and third-party apps. Right? Well... Yes. Everything is important. Applications cannot run without physical infrastructure and infrastructure needs third-party apps to be of any use. On top of that, we often need databases and other third-party apps to be combined with whatever we are developing.

Still, it's all about our applications. We have servers so that our apps can run on them. We have Kubernetes so that we can manage our applications. We are running databases so that our applications can store data.

You get the point? Everything we do is a result of the need to have our applications up and running and serving our users.

Hence, applications are what really matters making them, arguably, **the focus of developer platforms**. We want to be able to deploy, manage, and observe our apps. To accomplish that, we need quite a few things. We certainly need a repository where we'll store application code, scripts, workflow definitions, manifests, and whatever else we might need. More often than not, we also need a place to store application state meaning that we should have a database. Finally, we might need some infrastructure where all that will be running.

That's the subject of today's post. We'll explore how we can add everything we might need to manage applications in an Internal Developer Platform (IDP). More specifically, we'll see how we can enable everyone to easily get a **code repository** with all the files needed to work and manage applications, how we can add third-party **applications and infrastructure** our applications might need, and, finally, how we can define **our applications**. All that needs to be easy and accessible to anyone working in our organization. Simplicity is the key today.

Let's start with repositories.

## Setup

```sh
git clone https://github.com/vfarcic/idp-full-demo

cd idp-full-demo

git fetch

git checkout apps
```

> Make sure that Docker is up-and-running. We'll use it to create a KinD cluster.

> Watch [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell
```

> Watch [The Future of Shells with Nushell! Shell + Data + Programming Language](https://youtu.be/zoX_S6d-XU4) if you are not familiar with Nushell. Alternatively, you can inspect the `setup/kubernetes.nu` script and transform the instructions in it to Bash or ZShell if you prefer not to use that Nushell script.

```sh
chmod +x platform

platform setup apps

source .env
```

## GitHub Repository with Source Code

It all starts with a repository. We need a place where we'll store code, scripts, configurations, and anything else we might be using to work on a project.

Now, I'm sure that almost anyone knows how to create a Git repository and how write some files locally and push them there. Still, it would be nice to have all that as a unified yet customizable and, above all, simple process. So, I defined a Crossplane Composition that does just that.

*I'll be using Crossplane Compositions to demonstrate some principles and show how we can accomplish certain objectives. You should not be limited to Crossplane. You should be able to accomplish the same using other tools. The important note is to focus on what we're trying to accomplish here rather than on specific technologies that help us do what needs to be done.*

Here's a Claim based on that definition.

```sh
cat crossplane/repo.yaml
```

The output is as follows.

```yaml
apiVersion: devopstoolkit.live/v1alpha1
kind: GitHubClaim
metadata:
  name: idp-full-app
spec:
  id: idp-full-app
  parameters:
    public: true
    app:
      language: go
```

This is as simple as it can get. We have the `spec.id` that is a unique identifier which will, eventually, become the repository.

Further on, we are setting the `public` parameter to `true` thus choosing to make the repository accessible to anyone and we choose to use `go` as the `app` `language`. If we ignore the fact that I implemented `go` as the only choice (for now), people should be able to get different results depending on their language choice.

*Here comes and important note. Typically, a developer portal would not only have APIs, like the one we just saw, and controllers that would manage the state of resources, but also some user interfaces. People might be writing YAML like the one we're exploring here, or they might be using a Web UI, like Backstage or Port that would generate that YAML, or we would have a custom CLI, or whichever user interface we might be building. We will not explore those today and you will have to imagine that there are some user interfaces meaning that writing YAML is only one of the options. All in all, we'll use YAML and apply it with *kubectl* and you have to imagine that your end users might have other avenues to interact with the platform.*

*Another important note is that we're exploring examples I generated to prove a point. They do not represent everything you might need. You are in charge of the interfaces (APIs) and controllers that should match your needs. Also, I will not show how I did what I did, but, rather, the end result. I want to show what's possible. I'll leave the links to the repositories with the Compositions used in this post in the description of this video (the one at the top of this post).*

Let's apply that manifest, while imagining that end users generated it through some custom UI or a CLI,...

```sh
kubectl --namespace a-team apply --filename crossplane/repo.yaml
```

...and take a look at what was generated.

```sh
crossplane beta trace --namespace a-team githubclaim idp-full-app
```

The output is as follows.

```
NAME                                          SYNCED   READY   STATUS
GitHubClaim/idp-full-app (a-team)             True     False   Waiting: Claim is waiting for composite resource to become Ready
└─ GitHub/idp-full-app-wknl2                  True     False   Creating: ..., idp-full-app-file-devbox, idp-full-app-file-go-mod, and 4 more
   ├─ Branch/idp-full-app-init                False    False   ReconcileError: ...ll-app/git/ref/heads/main: 409 Git Repository is empty. []  []}]
   ├─ PullRequest/idp-full-app-init           False    False   ReconcileError: ...} {Resource:PullRequest Field:head Code:invalid Message:}]  []}]
   ├─ RepositoryFile/idp-full-app-devbox      False    False   ReconcileError: ...-app/contents/devbox.json: 409 reference already exists []  []}]
   ├─ RepositoryFile/idp-full-app-gitignore   True     True    Available
   ├─ RepositoryFile/idp-full-app-go-mod      False    False   ReconcileError: ...te the resource: [{0 unexpected status code: 404 Not Found  []}]
   ├─ RepositoryFile/idp-full-app-go-sum      False    False   ReconcileError: ...te the resource: [{0 unexpected status code: 404 Not Found  []}]
   ├─ RepositoryFile/idp-full-app-main-go     False    False   ReconcileError: ...te the resource: [{0 unexpected status code: 404 Not Found  []}]
   ├─ RepositoryFile/idp-full-app-readme      False    False   ReconcileError: ...te the resource: [{0 unexpected status code: 404 Not Found  []}]
   └─ Repository/idp-full-app                 True     True    Available
```

We can see that it created or will soon create a `Repository` and, inside it, a `Branch` with a few `RepositoryFiles`. Further on, once it's done with those, it will create a `PullRequest` which we could review, approve, and merge to the mainline.

After a while all those resources should be available. Let's confirm that.

```sh
crossplane beta trace --namespace a-team githubclaim idp-full-app
```

The output is as follows.

```
NAME                                          SYNCED   READY   STATUS
GitHubClaim/idp-full-app (a-team)             True     True    Available
└─ GitHub/idp-full-app-wknl2                  True     True    Available
   ├─ Branch/idp-full-app-init                True     True    Available
   ├─ PullRequest/idp-full-app-init           True     True    Available
   ├─ RepositoryFile/idp-full-app-devbox      True     True    Available
   ├─ RepositoryFile/idp-full-app-gitignore   True     True    Available
   ├─ RepositoryFile/idp-full-app-go-mod      True     True    Available
   ├─ RepositoryFile/idp-full-app-go-sum      True     True    Available
   ├─ RepositoryFile/idp-full-app-main-go     True     True    Available
   ├─ RepositoryFile/idp-full-app-readme      True     True    Available
   └─ Repository/idp-full-app                 True     True    Available
```

That's it when the repository with initial files is concerned. Much more is comming but, for now, we should clone the newly created repo,...

```sh
git clone "https://github.com/$GITHUB_USER/idp-full-app"
```

...and enter into it.

```sh
cd idp-full-app
```

We confirmed that the new repo was created. Let's see whether there is a pull request as well.

```sh
gh pr list
```

The output is as follows.

```
Showing 1 of 1 open pull request in vfarcic/idp-full-app

ID  TITLE    BRANCH  CREATED AT         
#1  Initial  init    about 2 minutes ago
```

We can see that the PR is indeed there. How about the files?

```sh
gh pr view init --json files
```

The output is as follows.

```json
{
  "files": [
    {
      "path": "README.md",
      "additions": 3,
      "deletions": 0
    },
    {
      "path": "go.mod",
      "additions": 34,
      "deletions": 0
    },
    {
      "path": "go.sum",
      "additions": 89,
      "deletions": 0
    },
    {
      "path": "main.go",
      "additions": 25,
      "deletions": 0
    }
  ]
}
```

Not much was created, yet enough to get us going before we start defining the application and third-party services. For now, there is a `README.md` and Go files (`go.mod`, `go.sum`, `main.go`). That makes sense given that, in this example, we choose Go as the programming language.

Let's merge the pull request,...

```sh
gh pr merge init --rebase
```

...and pull it to the local repo,...

```sh
git pull
```

...before we move to the next phase.

## Third-Party Apps and Infrastructure for a Database Server

Source code stored in a Git repository is only a fraction of what we might need when working on an application. We might need some scripts, workflows, Dockerfile, and quite a few other things. We'll get to those later. For now, we'll focus on third-party apps. More specifically, we'll see how we can add a PostgreSQL, or any other database to the mix.

We'll start by creating a directory in the newly created repo.

```sh
mkdir apps
```

Next, we'll copy two manifests I created earlier into the `apps` directory.

```sh
cp ../crossplane/$HYPERSCALER-sql.yaml apps/silly-demo-db.yaml

cp ../crossplane/$HYPERSCALER-sql-password.yaml \
    apps/silly-demo-db-password.yaml
```

Now we can see the definition of the database people can claim without going into nitty-gritty details of everything required to run it.

```sh
cat apps/silly-demo-db.yaml
```

The output is as follows.

```yaml
apiVersion: devopstoolkitseries.com/v1alpha1
kind: SQLClaim
metadata:
  name: silly-demo-db
spec:
  id: silly-demo-db
  compositionSelector:
    matchLabels:
      provider: aws
      db: postgresql
  parameters:
    version: "16.2"
    size: small
    region: us-east-1
    databases:
      - db-01
      - db-02
```

If you read posts in this side you probably already saw this or a very similar manifest, so I won't go into much detail. We're claiming a `postgresql` database server in `aws`. It should be version `16.2`, it should be `small`, whatever that means in AWS, it should be in the `us-east-1` region, and it should have two `databases`.

The second manifest is the initial password. I should have used External Secrets Operator to pull it from a secrets store but I was too lazy to do that so it's in plain sight available for any malicious person to find.

Let's apply both the password...

```sh
kubectl --namespace a-team apply \
    --filename apps/silly-demo-db-password.yaml
```

...and the claim,...

```sh
kubectl --namespace a-team apply \
    --filename apps/silly-demo-db.yaml
```

...and take a look at what we got.

```sh
crossplane beta trace --namespace a-team sqlclaim silly-demo-db
```

The output is as follows.

```
NAME                                            SYNCED   READY   STATUS
SQLClaim/silly-demo-db (a-team)                 True     False   Waiting: Claim is waiting for composite resource to become Ready
└─ SQL/silly-demo-db-krxrd                      True     False   Creating: ...es: gateway, mainRouteTableAssociation, rdsinstance, and 14 more
   ├─ InternetGateway/silly-demo-db             False    -       ReconcileError: ...enced field was empty (referenced resource may not yet be ready)
   ├─ MainRouteTableAssociation/silly-demo-db   False    -       ReconcileError: ...enced field was empty (referenced resource may not yet be ready)
   ├─ RouteTableAssociation/silly-demo-db-1a    False    -       ReconcileError: ...enced field was empty (referenced resource may not yet be ready)
   ├─ RouteTableAssociation/silly-demo-db-1b    False    -       ReconcileError: ...enced field was empty (referenced resource may not yet be ready)
   ├─ RouteTableAssociation/silly-demo-db-1c    False    -       ReconcileError: ...enced field was empty (referenced resource may not yet be ready)
   ├─ RouteTable/silly-demo-db                  False    -       ReconcileError: ...enced field was empty (referenced resource may not yet be ready)
   ├─ Route/silly-demo-db                       False    -       ReconcileError: ...enced field was empty (referenced resource may not yet be ready)
   ├─ SecurityGroupRule/silly-demo-db           False    -       ReconcileError: ...enced field was empty (referenced resource may not yet be ready)
   ├─ SecurityGroup/silly-demo-db               False    -       ReconcileError: ...enced field was empty (referenced resource may not yet be ready)
   ├─ Subnet/silly-demo-db-a                    False    -       ReconcileError: ...enced field was empty (referenced resource may not yet be ready)
   ├─ Subnet/silly-demo-db-b                    False    -       ReconcileError: ...enced field was empty (referenced resource may not yet be ready)
   ├─ Subnet/silly-demo-db-c                    False    -       ReconcileError: ...enced field was empty (referenced resource may not yet be ready)
   ├─ VPC/silly-demo-db                         True     False   Creating
   ├─ ProviderConfig/silly-demo-db-sql          -        -       
   ├─ ProviderConfig/silly-demo-db-sql          -        -       
   ├─ Object/silly-demo-db-secret               False    -       ReconcileError: ...om referenced resource: status.atProvider.address: no such field
   ├─ Database/silly-demo-db-db-01              False    -       ReconcileError: ... cannot get credentials Secret: Secret "silly-demo-db" not found
   ├─ Database/silly-demo-db-db-02              False    -       ReconcileError: ... cannot get credentials Secret: Secret "silly-demo-db" not found
   ├─ ProviderConfig/silly-demo-db              -        -       
   ├─ SubnetGroup/silly-demo-db                 False    -       ReconcileError: ...enced field was empty (referenced resource may not yet be ready)
   └─ Instance/silly-demo-db                    False    -       ReconcileError: ...enced field was empty (referenced resource may not yet be ready)
```

That's a bunch of AWS resources like `InternetGateway`, `SecurityGroup`, RDS `Instance`, and others, combined with two `Databases`, and a few other things.

It will take a bit of time until all those are up-and-running. We won't wait for them. Instead, we'll jump right into defining the application itself, and quite a few other things the application might need.

## Kubernetes Resources, CI Workflow, Dockerfile, Scripts, etc.

Let's copy a manifest I prepared in advance,...

```sh
cp ../crossplane/app.yaml apps/silly-demo.yaml
```

...and take a look at it.

```sh
cat apps/silly-demo.yaml
```

The output is as follows.

```yaml
apiVersion: devopstoolkitseries.com/v1alpha1
kind: AppClaim
metadata:
  name: silly-demo
  labels:
    app-owner: vfarcic
spec:
  id: silly-demo
  compositionSelector:
    matchLabels:
      type: backend
      location: local
  parameters:
    namespace: a-team
    image: ghcr.io/vfarcic/idp-full-demo
    tag: FIXME
    port: 8080
    host: silly-demo.127.0.0.1.nip.io
    ingressClassName: nginx
    db:
      secret: silly-demo-db
    repository:
      enabled: true
      name: idp-full-app
    ci:
      enabled: true
```

This one is a bit more complicated, yet very simple compared to what it does.

We are specifying information one might expect to have when defining an app.

There is the `namespace`, `image`, `tag`, `port`, `host`, and `ingressClassName`.

The value of the `tag` is interesting. It's currently set to `FIXME` which obviously does not look like a tag. The reason for that is simple. We did not yet build a single image and, therefore, there is no tag we could use. We cuild fix that by executing *docker image build* and pushing the image to the registry, but we won't do that. That would be silly. Instead, we'll build images automatically whenever we push changes to the source code of the application. The problem, however, is that we do not have a workflow that will do that. We do not have a CI process, just yet. We'll see that will change in a few minutes. For now, let's see what else we have over there.

Since we want that application to talk to the database we claimed earlier, we're telling it where the `secret` with the credentials that will be created is.

Next, we're telling the claim that the usage of the `repository` with the code is `enabled` and what the repo `name` is. With that information, the claim should be able to push to that repo additional files related to the application we're defining.

Finally, we're saying that `ci` should be `enabled`. As a result, the claim should create all the files we might need to build images and perform any other CI steps.

As a result of all that, we should not only have the application up-and-running but also everything else, including CI workflow, as well.

Let's apply it.

```sh
kubectl --namespace a-team apply --filename apps/silly-demo.yaml
```

To begin with, that claim should have created Kubernetes resources required to run the application. Let's see whether that's really the case.

```sh
kubectl --namespace a-team get all,ingresses
```

The output is as follows.

```
NAME                              READY   STATUS         RESTARTS   AGE
pod/silly-demo-6b5974ff5c-wcvxx   0/1     ErrImagePull   0          6s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/silly-demo   ClusterIP   10.96.167.228   <none>        8080/TCP   6s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/silly-demo   0/1     1            0           6s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/silly-demo-6b5974ff5c   1         1         0       6s

NAME                                   CLASS   HOSTS                         ADDRESS   PORTS   AGE
ingress.networking.k8s.io/silly-demo   nginx   silly-demo.127.0.0.1.nip.io             80      6s
```

There we go. We got the `deployment` which created the `replicaset` which, in turn, spin up a `pod` with the container where the application should be running. That Pod, however, is failing with the `ErrImagePull` message. That's normal since it's trying to use an image that does not exist. We did not build it just yet.

There's also a `service` for communication with the Pods of the application and `ingress` which allows us to talk to the app from outside the cluster.

Typically, there would be quite a few other components if that would be "production-ready", but I did not want to complicate it more then necessary for this demo.

There's more though, and we can see all the components composed from that claim by running a `trace`.

```sh
crossplane beta trace --namespace a-team appclaim silly-demo
```

The output is as follows.

```
NAME                                              SYNCED   READY   STATUS
AppClaim/silly-demo (a-team)                      True     False   Waiting: Claim is waiting for composite resource to become Ready
└─ App/silly-demo-v2c9s                           True     False   Creating: ..., silly-demo-file-dockerfile, silly-demo-file-dot-nu, and 2 more
   ├─ Object/silly-demo-deployment                True     True    Available
   ├─ Object/silly-demo-ingress                   True     True    Available
   ├─ Object/silly-demo-service                   True     True    Available
   ├─ ProviderConfig/silly-demo-app               -        -       
   ├─ Branch/silly-demo-branch-ci                 True     True    Available
   ├─ PullRequest/silly-demo                      False    False   ReconcileError: ... [{Resource:PullRequest Field:head Code:invalid Message:}]  []}]
   ├─ RepositoryFile/silly-demo-file-devbox       False    False   ReconcileError: ...te the resource: [{0 unexpected status code: 404 Not Found  []}]
   ├─ RepositoryFile/silly-demo-file-dockerfile   False    False   ReconcileError: ...te the resource: [{0 unexpected status code: 404 Not Found  []}]
   ├─ RepositoryFile/silly-demo-file-dot-nu       False    False   ReconcileError: ...te the resource: [{0 unexpected status code: 404 Not Found  []}]
   └─ RepositoryFile/silly-demo-file-gha          False    False   ReconcileError: ...te the resource: [{0 unexpected status code: 404 Not Found  []}]
```

Besides those that we already saw, all of them represented as `object` resources, we can see that a `Branch` was created and that some `RepositoryFiles` were pushed to that branch and that, finally, a `PullRequest` was created. Those are the interesting ones but, before we see what exactly they are, we should wait for a few moments and confirm that all the Managed Resources are available.

```sh
crossplane beta trace --namespace a-team appclaim silly-demo
```

The output is as follows.

```
NAME                                              SYNCED   READY   STATUS
AppClaim/silly-demo (a-team)                      True     True    Available
└─ App/silly-demo-v2c9s                           True     True    Available
   ├─ Object/silly-demo-deployment                True     True    Available
   ├─ Object/silly-demo-ingress                   True     True    Available
   ├─ Object/silly-demo-service                   True     True    Available
   ├─ ProviderConfig/silly-demo-app               -        -       
   ├─ Branch/silly-demo-branch-ci                 True     True    Available
   ├─ PullRequest/silly-demo                      True     True    Available
   ├─ RepositoryFile/silly-demo-file-devbox       True     True    Available
   ├─ RepositoryFile/silly-demo-file-dockerfile   True     True    Available
   ├─ RepositoryFile/silly-demo-file-dot-nu       True     True    Available
   └─ RepositoryFile/silly-demo-file-gha          True     True    Available
```

Now that all the Managed Resources are `Available` we can confirm that what we got is what we actually want.

To begin with, there should be a new pull request, so lets list PRs.

```sh
gh pr list
```

The output is as follows.

```
Showing 1 of 1 open pull request in vfarcic/idp-full-app

ID  TITLE  BRANCH         CREATED AT        
#2  CI     silly-demo-ci  about 1 minute ago
```
We can see that a pull request suspiciously named `CI` was created.

Let's see what's inside it.

```sh
gh pr view silly-demo-ci --json files
```

```json
{
  "files": [
    {
      "path": ".github/workflows/ci.yaml",
      "additions": 44,
      "deletions": 0
    },
    {
      "path": "Dockerfile",
      "additions": 15,
      "deletions": 0
    },
    {
      "path": "devbox-ci.json",
      "additions": 12,
      "deletions": 0
    },
    {
      "path": "dot.nu",
      "additions": 44,
      "deletions": 0
    }
  ]
}
```

Now we're talking. We got four files that, hopefully, contain everything we'll need to run CI workflows every time we push changes to that repository. We'll see what those files are in a moment.

*As I already mentioned, this is only a demo. You should create Compositions that match your own needs. You might use GitLab CI instead of GitHub Actions, you might be building images using Buildpacks instead of Dockerfile, and so on and so forth. This is me showing you what you can and should do rather than giving you the final solution.*

Let's merge the PR,...

```sh
gh pr merge silly-demo-ci --rebase
```

...and pull latest changes into the local repo.

```sh
git pull
```

Now we can look at those four files that were created and pushed to the repo automatically when we claimed an application.

The first in line is `ci.yaml`.

```sh
cat .github/workflows/ci.yaml
```

The output is as follows.

```yaml
name: ci
run-name: ci
on:
  push:
    branches:
      - main
jobs:
  all:
    runs-on: ubuntu-latest
    env:
      TAG: 0.0.${{ github.run_number }}
      FORCE_COLOR: 1
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3
      - name: Login to ghcr
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: vfarcic
          password: ${{ secrets.REGISTRY_PASSWORD }}
      - name: Install devbox
        uses: jetify-com/devbox-install-action@v0.11.0
        with:
          project-path: devbox-ci.json
          enable-cache: 'true'
      - name: All
        run: |
          devbox run --config devbox-ci.json -- ./dot.nu run ci $TAG
        env:
          REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
      - name: Commit changes
        run: |
          git config --local user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git config --local user.name "github-actions[bot]"
          git add .
          git commit -m "Release ${{ env.TAG }} [skip ci]"
      - name: Push changes
        uses: ad-m/github-push-action@master
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          branch: ${{ github.ref }}
```

As you can probably guess, that's GitHub Actions workflow. Most of it is typical stuff you would normally do like code `Checkout`, `Login` to a container image registry at the top and `Commit` and `Push` changes back to the repo at the bottom.

The middle part is a bit "special". Instead of defining all the dependencies and all the operations, I have it all in a single step called `All`. Over there, `devbox` will make sure that all the tools needed are there and, after that, it will run a script `dot.nu`. I prefer having all the automation defined as a script so that tasks can be executed in workflows but also locally or anywhere else. Otherwise, we'd need to define the same thing multiple times, and that's just silly. The script itself is written as Nushell which you should be familiar with if you're reading posts here. I love it. It's awesome. I'm using it for all scripts and CLIs.

Next, we got `Dockerfile`.

```sh
cat Dockerfile
```

The output is as follows.

```Dockerfile
FROM golang:1.23.3-alpine AS build
RUN mkdir /src
WORKDIR /src
ADD ./go.mod .
ADD ./go.sum .
ADD ./*.go ./
RUN GOOS=linux GOARCH=amd64 go build -o app
RUN chmod +x app
FROM scratch
ARG VERSION
ENV VERSION=$VERSION
ENV DB_PORT=5432 DB_USERNAME=postgres DB_NAME=my-db
COPY --from=build /src/app /usr/local/bin/app
EXPOSE 8080
CMD ["app"]
```

You should be familiar with Dockerfile unless you live in a cave without internet or you just joined the ranks of software nerds. If it's the former, I suggest moving under a bridge in a city where you can steal WiFi from someone. If it's the former, I have two things to tell you. First of all, welcome. Second, I don't understand how you got this far into this post. All this might sounds Chinese to you, unless you are from Chine in which case I'll change it to Japanese.

We also got a `devbox` file.

```sh
cat devbox-ci.json
```

The output is as follows.

```json
{
  "packages": [
    "nushell@0.100.0",
    "go@1.22.3"
  ],
  "shell": {
    "init_hook": [
      "chmod +x dot.nu"
    ],
    "scripts":   {}
  }
}
```

That one defines all the packages needed to execute CI operations no matter where we're executing them from. This one is short with only `nushell` and `go`. It's a demo. I'm sure you'll have more.

Finally, there is that script that defines all the commands for running CI processes as a whole or separately.

```sh
cat dot.nu
```

The output is as follows (truncated for brevity).

```nu
#!/usr/bin/env nu
def main [] {}
# Runs all CI tasks
def "main run ci" [
    tag: string # The tag of the image (e.g., 0.0.1)!
    manifest = "apps/silly-demo.yaml" # The path to the manifest file
    --run_tests = true # Whether to run tests
    --build_images = true # Whether to build images
] {
    if $run_tests {
        main run tests --language go
    }
    if $build_images {
        main build image $tag
    }
    main update claim $tag $manifest
}
# Runs tests
def "main run tests" [
    --language = "go" # The language of the project (e.g., go)
] {
    if $language == "go" {
        go test -v $"(pwd)/..."
    }
}
...
```

I won't go into the details of that script since it should be easy to understand what it's doing if you're familiar with Nushell. If you're not, please watch [The Future of Shells with Nushell! Shell + Data + Programming Language](https://youtu.be/zoX_S6d-XU4).

Let's go back to the Kubernetes cluster where the application is running.

```sh
kubectl --namespace a-team get all,ingresses
```

The output is as follows.

```
NAME                              READY   STATUS             RESTARTS   AGE
pod/silly-demo-6b5974ff5c-wcvxx   0/1     ImagePullBackOff   0          6m34s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/silly-demo   ClusterIP   10.96.167.228   <none>        8080/TCP   6m34s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/silly-demo   0/1     1            0           6m34s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/silly-demo-6b5974ff5c   1         1         0       6m34s

NAME                                   CLASS   HOSTS                         ADDRESS     PORTS   AGE
ingress.networking.k8s.io/silly-demo   nginx   silly-demo.127.0.0.1.nip.io   localhost   80      6m34s
```

As we can see, the `pod` is still not running. It's status is `ImagePullBackOff`, which was to be expected since we did not yet build the image nor we configured it to use it. However, now we have everything we need to do that, so let's just do it. Let's run the first build of our CI.

## Continuous Integration (CI) in Action

Before we run CI, we need to add a secret. That could be part of the Composition as well but I left it as a manual action as a security precaution. That sounds like a better excuse than admitting that I was too lazy to add it.

```sh
gh secret set REGISTRY_PASSWORD --body $GITHUB_TOKEN \
    --repo $GITHUB_USER/idp-full-app
```

Actually, there is one more manual action we should perform before we run CI workflow. We need to open repository settings,...

```sh
echo "https://github.com/$GITHUB_USER/idp-full-app/settings/actions"
```

...and select `Read and write permissions` in the `Workflow permissions` section.

> Open the URL from the output of the previous command in a browser.
> Select `Read and write permissions` in the `Workflow permissions` section.
> Click the `Save` button.

![](github-settings.png)

That's it. Now we're ready.

From here on, all we'd be doing is writing code of the application and pushing changes to the repo. However, since we already made a few modifications to the files, we can skip working on the code and just `add`,...

```sh
git add .
```

...commit,...

```sh
git commit -m "Apps"
```

...and push changes.

```sh
git push
```

Now we ned to wait for a few moments. Since it can be boring to stare at a blank screen, we'll `watch` the progress of the GitHub Actions workflow.

```sh
gh run watch
```

> Select the (only) run `ci, ci (main)`

Look at it go! It did repo `checkout`, it `Set up QEMU` required to build container images with Docker, it did the `Login to ghcr`, the container image repository we're using, and it installed `devbox`. That was all preparation and, from there on, it executed `All` the steps required for CI since they are all in the Nushell script. Among other things, that script built and pushed the image and made changes to the application manifest. To finish it off, it committed those `changes` and pushed them to the registry. The rest is GitHub Actions automatic cleanup process.

The final output is as follows.

```
? Select a workflow run * ci, ci (main) 5s ago
✓ main ci · 12343859933
Triggered via push about 1 minute ago

JOBS
✓ all in 1m38s (ID 34445579523)
  ✓ Set up job
  ✓ Checkout
  ✓ Set up QEMU
  ✓ Login to ghcr
  ✓ Install devbox
  ✓ All
  ✓ Commit changes
  ✓ Push changes
  ✓ Post Install devbox
  ✓ Post Login to ghcr
  ✓ Post Checkout
  ✓ Complete job

✓ Run ci (12343859933) completed with 'success'
```

It's a `success`. We did it. Hooray!

Should we confirm that what we think happened really happened? Let's do it. Let's `pull` the changes from GitHub.

```sh
git pull
```

The output is as follows.

```
remote: Enumerating objects: 8, done.
remote: Counting objects: 100% (8/8), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 5 (delta 3), reused 5 (delta 3), pack-reused 0 (from 0)
Unpacking objects: 100% (5/5), 1.02 KiB | 348.00 KiB/s, done.
From https://github.com/vfarcic/idp-full-app
   5c24852..fbd3ca9  main       -> origin/main
Updating 5c24852..fbd3ca9
Fast-forward
 .github/workflows/ci.yaml |   3 +-
 apps/silly-demo.yaml      |   2 +-
 devbox.lock               | 101 ++++++++++++++++++++++++++++++++++++++++++++++++
 dot.nu                    |   0
 4 files changed, 103 insertions(+), 3 deletions(-)
 create mode 100644 devbox.lock
 mode change 100644 => 100755 dot.nu
```

We can see that `apps/silly-demo.yaml` changed automagically. That's really the only output of the pipeline. That's the final outcome, so let's take a look at it.

```sh
cat apps/silly-demo.yaml
```

The output is as follows.

```yaml
apiVersion: devopstoolkitseries.com/v1alpha1
kind: AppClaim
metadata:
  name: silly-demo
  labels:
    app-owner: vfarcic
spec:
  id: silly-demo
  compositionSelector:
    matchLabels:
      type: backend
      location: local
  parameters:
    namespace: a-team
    image: ghcr.io/vfarcic/idp-full-demo
    tag: 0.0.1
    port: 8080
    host: silly-demo.127.0.0.1.nip.io
    ingressClassName: nginx
    db:
      secret: silly-demo-db
    repository:
      enabled: true
      name: idp-full-app
    ci:
      enabled: true
```

The `spec.parameters.tag` is what changed in that manifest. At the very end of the CI pipeline, after it built the image, it updated that file by adding the tag of the image it built.

As a reminder, let's take another look at the resources in the cluster.

```sh
kubectl --namespace a-team get all,ingresses
```

The output is as follows.

```
NAME                              READY   STATUS             RESTARTS   AGE
pod/silly-demo-6b5974ff5c-wcvxx   0/1     ImagePullBackOff   0          25m

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/silly-demo   ClusterIP   10.96.167.228   <none>        8080/TCP   25m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/silly-demo   0/1     1            0           25m

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/silly-demo-6b5974ff5c   1         1         0       25m

NAME                                   CLASS   HOSTS                         ADDRESS     PORTS   AGE
ingress.networking.k8s.io/silly-demo   nginx   silly-demo.127.0.0.1.nip.io   localhost   80      25m
```

We can see that the `pod` is still failing with the `ImagePullBackOff` status. Back when we applied those manifests, we did not have the image of the application. Now we do and our manifest is up to date.

Before we proceed, we should make the registry public to avoid having to setup credentials to the private registry inside the cluster.

```sh
echo https://github.com/users/$GITHUB_USER/packages/container/idp-full-app/settings
```

Please open the URL from the output of the previous command in a browser. Click the `Change visibility` button, select `Public`, type `idp-full-demo` as the name of the registry, and click the `I understand the consequences, change package visibility` button.

Normally, we would not need to do anything to deploy the new release of the application. We should have Argo CD or Flux watching that repository and applying changes to the cluster. However, I did not prepare that part for this demo assuming that you are already using or, at least, familiar with GitOps. So, instead of relying on auto-synchronization through Argo or Flux, we'll `apply` that manifest...

```sh
kubectl --namespace a-team apply --filename apps/silly-demo.yaml
```

...and take another look at the resources in the cluster.

```sh
kubectl --namespace a-team get all,ingresses
```

The output is as follows.


```
NAME                              READY   STATUS    RESTARTS   AGE
pod/silly-demo-696bb6c475-s6nxv   1/1     Running   0          33s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/silly-demo   ClusterIP   10.96.167.228   <none>        8080/TCP   26m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/silly-demo   1/1     1            1           26m

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/silly-demo-696bb6c475   1         1         1       33s
replicaset.apps/silly-demo-6b5974ff5c   0         0         0       26m

NAME                                   CLASS   HOSTS                         ADDRESS     PORTS   AGE
ingress.networking.k8s.io/silly-demo   nginx   silly-demo.127.0.0.1.nip.io   localhost   80      26m
```

There we go. The Pod is `Running`. The application is serving our users, and all it took is a few lines of YAML that gave us a new repository with all the scripts, workflow, Dockerfile, the initial source code, and anything else we might need. We also got a PostgreSQL database server in the hyperscaler of choice as well as all the Kubernetes resources needed to run the application.

The only thing that would make developers happier than what we did today is a tap on a back followed with the words "you're doing great, here's a substantial increase of your salary."

If you're new to Crossplane, you'll find a link to the playlist with the full tutorial. Otherwise, if you're already experience with it, you might want to take a look at the code of the Compositions I used today, the links are in the description as well.

## Destroy

```sh
cd ..

platform destroy apps

git checkout main

exit
```


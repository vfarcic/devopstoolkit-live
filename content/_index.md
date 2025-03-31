+++
archetype = "home"
title = ""
+++

# Latest Posts

<!-- <img src="/development/remote-environments-with-dev-containers-and-devpod-are-they-worth-it/thumbnail.jpg" style="width:50%; float:right; padding: 10px">

## [Remote Environments with Dev Containers and Devpod: Are They Worth It?](/development/remote-environments-with-dev-containers-and-devpod-are-they-worth-it)

Today we are going to explore running remote ephemeral development environments. We are going to see (potentially) the best solution you should (probably) NOT use. If that sounds confusing, you're not alone. 

We'll explore the Development Containers spec as well as Devpod as an implementation of that spec. Together, they provide a way to run ephemeral development environments.

There is a hidden reason for going through those. I have serious doubts about the story behind remote environments, at least in a specific form, and I want to discuss what we really want them for. But, to do that, we need to go through a few practical examples to be on the same page before I go off the beaten path and start questioning it all.

Buckle up. You're in for a ride that starts with some important questions that turn into excitement and finish... Well... I do not yet know how it will finish.

**[Full article >>](/development/remote-environments-with-dev-containers-and-devpod-are-they-worth-it)**

--- -->

<!-- <img src="/crossplane/crossplane-v2-simplified-compositions-namespace-scoped-resources-and-more/thumbnail.jpg" style="width:50%; float:right; padding: 10px">

## [Debunking Myths and Simplifying Compositions with Crossplane v2](/crossplane/crossplane-v2-simplified-compositions-namespace-scoped-resources-and-more)

**Crossplane v2** is here with some very cool features that I want to go through.

We'll see the **changes to Crossplane Composition schemas**, a shift to **Namespace-scoped resources**, **direct composition of any resources** without the need to rely only on Crossplane Managed resources, **new API versions**, **removal of deprecated features**, and more.

**[Full article >>](/crossplane/crossplane-v2-simplified-compositions-namespace-scoped-resources-and-more)**

---
-->

<img src="/internal-developer-platforms/debunking-myths-and-simplifying-compositions-with-crossplane-v2/thumbnail.jpg" style="width:50%; float:right; padding: 10px">

## [Debunking Myths and Simplifying Compositions with Crossplane v2](/internal-developer-platforms/debunking-myths-and-simplifying-compositions-with-crossplane-v2)

"**Crossplane is too complicated!**" "**Crossplane is only for infrastructure!** I need something else for applications."

I hear those and other similar statement very often so I decided to clarify a few things and, hopefully, eliminate some missconceptions.

To be more precise, today I want to debunk one missconception about Crossplane and, at the same time, show how Crossplane addressed one semi-legitimate complaint in the recent v2 release.

So, debunk one missconception and show one improvement with both of those being intertwined.

**[Full article >>](/internal-developer-platforms/debunking-myths-and-simplifying-compositions-with-crossplane-v2)**

---

<img src="/internal-developer-platforms/kubevela-oam-the-resurrection-of-simplified-app-management/thumbnail-03.jpg" style="width:50%; float:right; padding: 10px">

## [KubeVela & OAM: The Resurrection of Simplified App Management?](/internal-developer-platforms/kubevela-oam-the-resurrection-of-simplified-app-management)

Imagine that you are building an Internal Developer Platform.

What would be a good user experience if, for example, one would like to deploy and manage a backend application without spending five years trying to understand all the details about Kubernetes?

**[Full article >>](/internal-developer-platforms/kubevela-oam-the-resurrection-of-simplified-app-management)**

---

<img src="/kubernetes/why-most-kubernetes-dashboards-are-failing-you-and-whats-the-future/thumbnail-02.jpg" style="width:50%; float:right; padding: 10px">

## [Why Most Kubernetes Dashboards Are Failing You (and What Is The Future)](/kubernetes/why-most-kubernetes-dashboards-are-failing-you-and-whats-the-future)

Most **Kubernetes dashboards got it wrong**. Most did not understand the **limitations of Kubernetes API** and, as a result, act as glorified file explorers rather than tools that help us find what we're looking for, especially when dealing with large scale.

Today we are going to explore the **mistakes** Kubernetes dashboards are making and try to figure out how they might be able to get onto the right path, even though that might require some drastic changes in their design.

By the end of today's story you should see what we need to **navigate**, **search**, **debug**, and do whatever we might need to do in Kubernetes clusters. We'll feel like we are using Google Search rather than navigating through Kubernetes resources in a similar way we are traversing files and directories.

**[Full article >>](/kubernetes/why-most-kubernetes-dashboards-are-failing-you-and-whats-the-future)**

---

<img src="/db/neon---never-share-databases-again/thumbnail-03.jpg" style="width:50%; float:right; padding: 10px">

## [Neon - Never Share Databases Again!](/db/neon---never-share-databases-again)

Take a look at this.

<img src="/db/neon---never-share-databases-again/gha.png" style="width:40%; float:left; padding: 10px">

That is GitHub actions run that executed hopefully typical tasks anyone is executing every time a pull request is opened. It run unit tests, it build a container image and pushed it to a registry, it update manifests so that, later on Argo CD or Flux can deploy the new release to production when that PR is merged to mainline. But, before that happens, it created an ephemeral Kubernetes cluster and run integration tests to confirm that everything works as expected.

I hope that's what everyone is doing and, if that's not the case, please let me know in the comments and I'll do my best to make a video about the whole process.

There is one "special" part of that workflow run that I did not yet mention. There is something that hadly anyone does, yet something very important.

**[Full article >>](/db/neon---never-share-databases-again)**

---

<img src="/internal-developer-platforms/past-present-and-future-of-internal-developer-platforms/thumbnail-02.jpg" style="width:50%; float:right; padding: 10px">

## [Past, Present, and Future of Internal Developer Platforms](/internal-developer-platforms/past-present-and-future-of-internal-developer-platforms)

Can you guess which tool I used to build my first Internal Developer Platform?

Let me give you a few tips before you try to guess.

1. It performed most of the **tasks people needed** to do, like deployments of applications, execution of tests, and creation of new and management of existing servers and databases.
2. It was very **easy to use**.
3. It had a **graphical user interface** but it could be used without it through an API or a CLI.
4. It had **nothing to do with Kubernetes, or containers, or Cloud**.
5. The tool used to create it was one of the **most popular** tools back then, and is still one of the most widely tools used in enterprises.
6. Almost **no one used that platform** after the initial excitement dried out.

Which tool do you think I used? Which tool fits that description?

**[Full article >>](/internal-developer-platforms/past-present-and-future-of-internal-developer-platforms)**

---

<img src="/containers/say-goodbye-to-tedious-docker-commands-embrace-docker-bake/thumbnail-01.jpg" style="width:50%; float:right; padding: 10px">

## [Say Goodbye to Tedious Docker Commands: Embrace Docker to Bake Images](/containers/say-goodbye-to-tedious-docker-commands-embrace-docker-bake)

Building and pushing container image with Docker is **easy**. Right? We define a Dockerfile and we execute a command like `docker image build ...`. Docker file is easy to define and the rest is just a CLI command. How hard can it be?

Well... It can be hard or, at least, **tedious**.

Imagine that we have to build images for **multiple platforms**, that each of those images should be released both as a **specific version** but also as **latest**. Then add to that the situation that we need to build **more than one image**, let's say a backend and a frontend.

How many commands do we need to execute and how many arguments should each of those commands have? Can we remember all those arguments and are we willing to execute a bunch of commands?

That simple example already shows that building and pushing container images can be hard and tedious. The good news is that there is a better way. There is a declarative way to do all that.

**[Full article >>](/containers/say-goodbye-to-tedious-docker-commands-embrace-docker-bake)**

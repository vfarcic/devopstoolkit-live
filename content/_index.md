+++
archetype = "home"
title = ""
+++

# Latest Posts

<img src="/internal-developer-platforms/is-this-the-end-of-crossplane-compose-kubernetes-resources-with-kro/thumbnail-03.jpg" style="width:50%; float:right; padding: 10px">

## [Is This the End of Crossplane? Compose Kubernetes Resources with kro](/internal-developer-platforms/is-this-the-end-of-crossplane-compose-kubernetes-resources-with-kro)

Whomever is building developer platforms is bound to come to the conclusion that there is **a need to compose resources and expose those compositions through APIs**. If a developer needs a database, they should be able to specify what that database should be without having to deal with subnets, VPCs, internet gateways, and other lower-level components that are required, but are not important for the vast majority of people who just want a database. We are likely to come to a similar conclusion if, for example, a developer wants to run an application. That developer might want to specify a container image, a port, and a host without having to worry about Kubernetes Deployments, Services, Ingresses, Scalers, VirtualServices, and other lower-level Kubernetes types of objects.

**[Full article >>](/internal-developer-platforms/is-this-the-end-of-crossplane-compose-kubernetes-resources-with-kro)**

---

<img src="/kubernetes/mastering-kubernetes-volumes-emptydir-csi-drivers-storage-classes-persistent-volumes-and-persistent-volume-claims-configmaps-and-secrets/thumbnail-02.jpg" style="width:50%; float:right; padding: 10px">

## [Mastering Kubernetes: Volumes (emptyDir, CSI Drivers, Storage Classes, Persistent Volumes and Persistent Volume Claims, ConfigMaps, and Secrets)](/kubernetes/mastering-kubernetes-volumes-emptydir-csi-drivers-storage-classes-persistent-volumes-and-persistent-volume-claims-configmaps-and-secrets)

Today we'll go through Kubernetes volumes. We'll explore local volumes like emptyDir, CSI Drivers, Storage Classes, Persistent Volumes and Persistent Volume Claims, ConfigMaps, and Secrets. It does not matter what you do in Kubernetes, **volumes are unavoidable**. You will use them, so you need to understand what they're used for, when to use them, and how to use them.

**[Full article >>](/kubernetes/mastering-kubernetes-volumes-emptydir-csi-drivers-storage-classes-persistent-volumes-and-persistent-volume-claims-configmaps-and-secrets)**

---

<img src="/kubernetes/your-cluster-isnt-safe-the-dark-side-of-backups/thumbnail-01.jpg" style="width:50%; float:right; padding: 10px">

## [Your Cluster Is Not Safe: The Dark Side of Backups](/kubernetes/your-cluster-isnt-safe-the-dark-side-of-backups)

We all want to feel safe. That's why we create backups. We want to know that our systems will survive no mather what happens.

The feeling of safety is very very important, and I need to appologise in advance for what I'm about to say.

You are **NOT safe**. If the disaster happens, you might not be able to survive it.

Here's why.

**Backups alone are not enough!**; especially if its only one type of a backup.

When the day of reckoning comes, **you will NOT be saved!**

Let me prove that to you.

**[Full article >>](/kubernetes/your-cluster-isnt-safe-the-dark-side-of-backups)**

---

<img src="/kubernetes/master-kubernetes-backups-with-velero-step-by-step-guide/thumbnail-03.jpg" style="width:50%; float:right; padding: 10px">

## [Master Kubernetes Backups with Velero: Step-by-Step Guide](/kubernetes/master-kubernetes-backups-with-velero-step-by-step-guide)

Do you think your Kubernetes clusters are safe and able to withstand anything and everything?

Truth be told, Kuberentes is **very resilient**. It was designed to be highly available and fault tollerant. You hopefully know all that and, if you do, you probably also know that nothing is 100% resilient. Bad things happen. Unexpected things happen, and we should be **prepared for anything and everything**. Backups are a part of that preparation. We are or, at least, we should be creating them regularly in hope that we'll never have to restore any of them. Yet, we might be in a situation when restoring a backup is our last option, especially in cases when a cluster is completely gone and the only option left is to create a new one and restore a backup which, hopefully, should result in the new cluster running the same resources, having the same data, showing the same logs, and everything else that we had in the old cluster that is now dead.

**[Full article >>](/kubernetes/master-kubernetes-backups-with-velero-step-by-step-guide)**

---

<img src="/misc/say-goodbye-to-direct-communication-event-driven-pub-sub-with-nats/thumbnail-02.jpg" style="width:50%; float:right; padding: 10px">

## [Say Goodbye to Direct Communication! Event-Driven Pub/Sub With NATS](/misc/say-goodbye-to-direct-communication-event-driven-pub-sub-with-nats)

Imagine a world where **everyone minds their own business**. A world where no one tells anyone what to do and where no one is forcing anyone to do something. A world where everyone is focused only and exclusively on **their own tasks** and nothing else. A world where we work in complete isolation yet a world where everything is done more officiently.

In such a world, each of us would do a task and, when finished, announce to the world that we are done. After that, we would pick another task from the pile that corresponds to our specific skilset instead of being told what to do.

The process would be very simple and apply to everyone. Pick a task, do it, say that you are done, repeat.

Now, that might not be the world we would like to live in, but that might be a perfect world for processes. That might be just the way we should organize our applications.

**[Full article >>](/misc/say-goodbye-to-direct-communication-event-driven-pub-sub-with-nats)**

---

<img src="/ci-cd/argo-cd-gitops-promotions-with-kargo-by-akuity-a-brilliant-idea-with-flawed-execution/thumbnail-01.jpg" style="width:50%; float:right; padding: 10px">

## [Argo CD GitOps Promotions with Kargo (by Akuity): A Brilliant Idea with Flawed Execution?](/ci-cd/argo-cd-gitops-promotions-with-kargo-by-akuity-a-brilliant-idea-with-flawed-execution)

Most of the time, I split projects into "**it provides value**" and "**it's not good enough**". However, every once in a while a project comes along that produces both types of impressions.

There are projects that show a lot of potential. Projects that might represent some important steps toward a better way to manage or operate systems. Yet, those same projects might be showing the vision rather than being something we should adopt today.

Such a vision could be huge like, for example, **Kubernetes which was horrible** when it was first released and, at that time, I did not recommend it to anyone. Yet, its vision proved to be a valid one for all of us to pursue. The end result is what we have today. Almost everyone is using or, at least, planning to use Kubernetes in some capacity or another. It started as a vision that took a while to materialize.

There are examples of projects that have much smaller scope, yet also started as "this is not good enough today but we can see where that might take us". One of such projects is, in my opinion, Kargo by Akuity. I think that it has a lot of potential. It might be an important step forward in decoupling our workflows. It might be just what we need to orchestrate promotions of releases managed by GitOps or, to be more precise, Argo CD. The idea behind it is great, yet, I am not sure that there are enough compeling reasons to use it today. It's great. It truly is. Yet... Something might be missing.

**[Full article >>](/ci-cd/argo-cd-gitops-promotions-with-kargo-by-akuity-a-brilliant-idea-with-flawed-execution)**

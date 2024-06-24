+++
archetype = "home"
title = ""
+++

# Latest Posts

<!-- <img src="/kubernetes/scaling-explained-through-kubernetes-hpa-vpa-keda--cluster-autoscaler/thumbnail.png" style="width:50%; float:right; padding: 10px">

## [Scaling Explained Through Kubernetes HPA, VPA, KEDA & Cluster Autoscaler](/kubernetes/scaling-explained-through-kubernetes-hpa-vpa-keda--cluster-autoscaler)

Scaling is probably one of the most important aspects of computing, and a common cause of bankrupcy.

If our processes use more memory and CPU than what they need, they are wasting money or "stealing" those resources from others thus making them less efficient. On the other hand,if we give processes less memory and CPU than what they need, their performance will be affected negatively making user experience suck. And that's the good outcome. Much worse situation is that underpowered processes might crash with "out of memory" and other similar exceptions.

Hence, the goal is to assign just the right amount of resources to processes. Not too much, not to few, but just right. We do that through scaling, and we need to answer three questions.

* **What do we scale?**
* **Where do we scale?**
* **Who scales?**

**[Full article >>](/kubernetes/scaling-explained-through-kubernetes-hpa-vpa-keda--cluster-autoscaler)**

--- -->

<img src="/internal-developer-platforms/mastering-developer-portals-discover--integrate-api-schemas-with-port/thumbnail.png" style="width:50%; float:right; padding: 10px">

## [Mastering Developer Portals: Discover & Integrate API Schemas with Port](/internal-developer-platforms/mastering-developer-portals-discover--integrate-api-schemas-with-port)

I am so **dissaspointed with developer portal consoles** for ignoring the fact that almost **everything is discoverable through APIs**. Why should I design forms and fields in Backstage if Backstage should be able to ask an API for a schema? Even if that schema is not exactly what I need and I might have to remove parts of it, that's still easier and better than starting from scratch. It's a waste of time to do the same thing over and over again. We should define what we have, expose that through an API, and all other tools should just "discover" it. Just as that's true for, let's say, `kubectl`, it should be true for graphical user interfaces.

Hence, I spent months complaining about it and... I got it.

**[Full article >>](/internal-developer-platforms/mastering-developer-portals-discover--integrate-api-schemas-with-port)**

---

<img src="/cloud/single-pane-of-glass-for-everything-aws-azure-gcp-kubernetes-with-steampipe/thumbnail.png" style="width:50%; float:right; padding: 10px">

## [Single Pane of Glass for Everything (AWS, Azure, GCP, Kubernetes, ...) with Steampipe](/cloud/single-pane-of-glass-for-everything-aws-azure-gcp-kubernetes-with-steampipe)

Right now, I am connected to a PostgreSQL database and you are thinking that is boring. Who wants to see me querying a database?

Well... Wait until you see what I can query from such a database.

**[Full article >>](/cloud/single-pane-of-glass-for-everything-aws-azure-gcp-kubernetes-with-steampipe)**

---

<img src="/wasm/unleashing-webassembly-in-kubernetes-with-kwasm/thumbnail.png" style="width:50%; float:right; padding: 10px">

## [Unleashing WebAssembly in Kubernetes with Kwasm](/wasm/unleashing-webassembly-in-kubernetes-with-kwasm)

I am going to show you something you've seen many times before. Yet, looks can be deceiving. You will likely come to a wrong conclusion. What I will show is not what it might look like.

**[Full article >>](/wasm/unleashing-webassembly-in-kubernetes-with-kwasm)**

---

<img src="/ci-cd/from-makefile-to-justfile-or-taskfile-recipe-runner-replacement/thumbnail.png" style="width:50%; float:right; padding: 10px">

## [From Makefile to Justfile (or Taskfile): Recipe Runner Replacement](/ci-cd/from-makefile-to-justfile-or-taskfile-recipe-runner-replacement)

When I work locally, if I need to create a cluster I just execute `cluster-create`, wait for a few moments, and a local cluster with everything I need is running.

**[Full article >>](/ci-cd/from-makefile-to-justfile-or-taskfile-recipe-runner-replacement)**

---

<img src="/infrastructure-as-code/ansible-vs-terraform-vs-crossplane/thumbnail.png" style="width:50%; float:right; padding: 10px">

## [Terraform vs. Crossplane vs. Ansible - Rivals or Allies?](/infrastructure-as-code/ansible-vs-terraform-vs-crossplane)

I am often asked to compare Crossplane with Terraform, or Pulumi, or Ansible, or any other tool that primarily manages resources, be it those in hyperscalers like AWS, Google Cloud, and Azure, or in Kubernetes, or anywhere else. Well... Today I'm here to tell you that none of those tools are going away any time soon. We need all of those. We need configuration management tools like Ansible, we need Infrastructure-as-Code (IaC) tools like Terraform and Pulumi, and we need control planes, be it opinionated ones like AWS, Google Cloud, and Azure, or those that allow us to build our own control planes like Crossplane.

**[Full article >>](/infrastructure-as-code/ansible-vs-terraform-vs-crossplane)**

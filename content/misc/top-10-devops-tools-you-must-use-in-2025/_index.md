
+++
title = 'Top 10 DevOps Tools You MUST Use in 2025!'
date = 2025-01-06T14:00:00+00:00
draft = false
+++

Today is the day I announce which tools I recommend using in 2025.

This is the list of the best of the best.

Creme de la creme.

I'm lying. This is not the list of the best tools but, rather, the list of the tools that made impact in 2024, the tools that matured enough that I recommend as permanent tools in your toolbelt. It's the list of tools that proved their worth and should be adopted in 2025. Some of them are the best in their respective category, while others made a significant shift in the way we work that I believe deserve to be in the spotlight. Some are small, while others are massive. Some are widely known while others are obscure.

<!--more-->

{{< youtube gYl3moYa4iI >}}

So, the best of 2025 is more like "the best of the new tools and services or those that matured anough to be used in 2025".

That being said, every single tool I will mention today deserves to be in the spotlight. All those I will present should be adopted or, at least, considered by you. Nevertheless, I will choose a winner in each category.

Another important note is that I will not cover every single category. That would be impossible since there is close to an infinite number of ways we can categorize tools. Instead, I will focus on the categories that were my focus throughout the past year.

There are ten categories, presented in alphabetical order.

We'll go through **Artificial Intelligence (AI)**, **Continuous Integration and Continuous Delivery (CI/CD)**, **Containers**, **Developer Portals**, **GitOps**, **Infrastructure as Code and Control Planes**, **Manifests Management**, **Terminals**, **Miscellaneous**, that contains the tools I believe are valuable but I couldn't place in any specific category, and, finally, there is the **Users Choice** which contains the tools and services you chose.

Finally, before we jump right into it, let me note that, in case I covered a tool mentioned here in some other video, there will be a link to it in the description in case you want to explore it further.

That's enough chit-chat. I'm sure you prefer that we jump right into it, so let's do just that, starting with AI.

## Artificial Intelligence (AI)

AI is everywhere. AI is a hype. Everyone is jumping into it. Everyone is investing money into it. Yet, no one is sure what it's for. It's a mess!

Nevertheless, this is neither the time nor place to dive into the current state of AI.

What matters is that there's a bunch of AI chats like **[OpenAI](https://openai.com)** better known as Chat GPT, Google **[Gemini](https://gemini.google.com)**, **[Claude](https://claude.ai)** by Anthropic, and other SaaS solution, **[Ollama](https://ollama.com)** and an infinite number of other models that can be self-hosted. If you are a software engineer, which you almost certainly are if you're watching this video, you are likely already using **[GitHub Copilot](https://github.com/features/copilot)** or a similar solution that helps you write code, even if that code is YAML.

All of those are great but, my choice, goes to something much more obscure, much smaller and, mostly irrelevant to most.

Over the past year or so, the AI related tool I used the most is **[Fabric](https://github.com/danielmiessler/fabric)**. It helps with prompts. Now, that might sound silly, but only until we see the differences in AI responses between "normal" and really good prompts. With Fabric, we don't have to think how to use different prompts, how to communicate with different AIs, how to write prompts, and so on and so forth. All we have to do is use one of the existing patterns directly or copy and modify them to serve our specific needs. From there on, whenever we need something from an AI, we just pass our input to one of Fabric's patterns and get what we need. It's awesome and I use it all the time and I am proclaiming it the winner in this category. It's a small project that could not even imagine competing with the "big guys" I mentioned earlier, yet it is the tool I use all the time.

The next category is CI/CD.

## Continuous Integration / Continuous Delivery (CI/CD)

Continuous Integration and Delivery tools or, as I like to call them, workflows, haven't changed in decades. There is no innovation in that area but only small, often marginal improvements. **GitHub Actions**, **Argo Workflows**, **GitLab CI/CD**, **Tekton**, and other workflow tools are all, essentially the same. None of them made any significant innovation since Jenkins. They all do the same thing. They all execute tasks one after another or in parallel. They might be using a different syntax to define workflows, they might have different maintenance cost and performance benefits, yet, they are all the same. I lost interest in them. Whichever you are using right now is probably good enough.

I am much more interested in what's happening outside workflow tools.

We saw **[Dagger](https://dagger.io)** as an attempt to define workflows that can run both locally and inside "traditional" workflow tools. We can define what we need in TypeScript, Python, or Go, and run it locally or inside workflows like GitHub Actions. The main advantage is that we can define execution of tasks once and run it anywhere and, from that perspective, it effectively replaces Shell scripts or Makefile. Dagger is potentially great, yet I am not yet convinced that it's the way to go.

We also saw rise and adoption of quite a few tools that, essentially, try to replace Makefile.

**[Earthly](https://earthly.dev)** is certainly interesting. It combines Dockerfile and Makefile into a single definition. That's a great idea since those two are one of the most commonly used formats so combining them makes a lot of sense.

Then there are **[Task](https://taskfile.dev)** and **[Just](https://github.com/casey/just)** that are trying to reinvent Makefile which, frankly, should have been retired long time ago. Both Taskfile and Justfile are, arguably, better formats than Makefile. I use both mainly since I cannot yet make up my mind which I like more.

We also saw the rise of **[Kargo](https://kargo.akuity.io)** which tends to solve a real problem of promotions of releases managed by GitOps tools, specifically Argo CD. Promotions became kind of complicated with GitOps and we definitely need a tool that will help us in that area. Yet, I think that Kargo, even though it features some very interesting new concepts is still too young and not polished enough to be widely used. It will probably get there sometime, but not today.

The biggest improvement in the CI/CD area are **[Nix](https://nixos.org)** packages. Nix in general is awesome and its packaging system is amazing. We finally got packages that work everywhere, be it Linux, Mac, or Windows. We finally have a way to define all the tools that a project needs and the capability to get those tools no matter whether we need them on a laptop, in workflows, or anywhere else. Now, when I talk about Nix, I do not refer to Nix OS. It came too late. Most of us do not care about operating systems running on top of servers and are not willing to change macOS or Windows as operating systems on our laptops. Nix packages are what attracted me to Nix and we get them through Nix Shell that can run anywhere. The problem, however, is that Nix syntax is just too complicated for a user who did not choose to dedicate its life to Nix. It is a programming language, of sorts, that is just too complicated and obscure to learn quickly.

My love for Nix packages and hate for its syntax led me to **[Devbox](https://jetify.com/devbox)**. It's awesome. It gives us all the advantages of Nix Shell and packages with a very simple and straightforward syntax. All we have to do is define *devbox.json* either manually or through *devbox* CLI. You'll notice that every single repository I created during the last year or two has the *devbox.json* file. *devbox init* is the first command I execute when starting a new project and *devbox shell* is the first command I execute when I want to work on a project. I use it to install the tools I need to work on a project no matter whether that's on my laptop, or inside workflows like GitHub Actions, or anywhere else. Devbox is awesome. I cannot recommend it enough and it is a clear winner in this category. It is not a tool that you will use to run CI/CD workflows, but it should be used inside the workflows to install all the tools you'll need. That alone would not make it "special" but when we combine it with the usage of those same packages locally or anywhere else where you might be working with a project, it becomes indispensable. Also, if you need those same packages as a container or DevContainer, it can also generate *Dockerfile* and *devcontainer.json* files. If you haven't already, I strongly recommend adopting Devbox. If you are watchign my videos and following along my instructions, you are almost certainly already using Devbox since it's in every one of my setup instructions.

Next, we'll talk about containers.

## Containers

Containers are now boring, no matter whether we're talking about running them locally or in "real" servers.

Most of us do not care any more which container engine is used inside Kubernetes, we care even less which one is used with managed services like Google Cloud Run, Azure Container Apps, Lambda, or whichever other you might be using. It's a commodity or low implementation detail.

Similarly, I don't think it matters what you're using on your laptop. Is it **[Docker Desktop](https://docker.com/products/docker-desktop)** or one of the alternatives like **[Rancher Desktop](https://rancherdesktop.io/)**, **[Podman](https://podman.io)**, **[Nerdctl](https://github.com/containerd/nerdctl)**, or something else. I know that some of you have strong feelings and claim that one is much better than the other. If you're one of those, all I can say is that I don't care any more, and I don't think that many do. They all work fine. They all run containers. Some of them have additional bangs and whistles, but I don't think any of them are life changing.

There is something important and very successful going on around containers, even though it might not seem that important at the first glance. That something is "slim and safe images". Over time, most of us learned about the importance of having slim images. We learned that containers are not VMs; that containers do not need the whole operating system. As a result, we stopped using base images like Ubuntu, CentOS, and others. We learned that the less we put into container images, the better. We learned that the best operating system for images is not to have an operating system or, to be more precise, to use distroless images when possible, and minimal images when not. We learned that most of security vulnerabilities come from "stuff" we don't use.

With that in mind, the winner this year is **[Chainguard Images](https://chainguard.dev/chainguard-images)**. They are the images you should be using as base images for whatever you're building. They are slim and they are safe. If you use them as the base, your images will be small in size. They will be more performant and will be safe with zero CVEs (unless the code of your application introduces them).

I strongly recommend using Changuard images as your base images.

Next in line are Developer Portals.

## Developer Portals

Last couple of years we saw resurgence of platform engineering. I intentionally said "resurgence" since we've been building developer platforms for decades now. Every bigger company I worked with had some kind of a platform that helped develers be more productive. That could be anything from a server with scripts, Jenkins jobs, some kind of a UI that provides links to services, observability dashboards, or anything else. Developer platforms have been around for a long long time, even though they might not have been called like that nor people working on them have been called platform engineers. Most of them failed or were only moderately successful.

Here's the important question. Is it going to be any different this time? I think it will.

This time we have standards, mainly in the form of Kubernetes. This time we finally all agreed what is the underlying technology sitting below those platforms. This time we know how to create control planes on top of Kubernetes, how to extend APIs with CRDs, how to make all the tools interoperable by forcing them to be Kubernetes-native, how to do service discovery by querying APIs, and so on and so for. Ultimately, most of us made a choice to adopt Kubernetes and most of the vendors choose Kubernetes as the base on top of which their tools are running. Kubernetes is the base on top of which we should be building platforms. There is no doubt about that. Kubernetes is what might make us successful in building platforms. It's what might make decades of failures a success. Kubernetes future depends on that. Kubernetes was not designed to be used directly but as a base on top of which we build the final solution and that final solution is a platform fine-tuned to the needs of a company.

Now, this category is NOT about developer platforms, mostly because having a platform means combining an infinite number of tools and processes. This category is about Developer Portals defined as user interfaces through which developers can communicate with the platform.

Within this category of portals or simply graphical user interfaces sitting on top of platforms typically built by extending Kubernetes, we have quite a few tools to choose from.

There is **[Backstage](https://backstage.io/)** as one of the fastest-growing CNCF projects. It's widely adopted both by end-users and vendors who are using it as the interface for whichever tool they are selling. Everyone is jumping onto Backstage, but also realizing that it is very expensive to manage and not really built to leverage Kubernetes. It is based mostly on static files and has quite a few issues related to the dynamic nature of what we do today. The shining star of Backstage is its extensibility through the plugin systems. It has a massive number of ready-to-go plugins which are, for most part, of very poor quality. It allows us to create our own plugins which can do anything we need them to do, but at a significant development and maintenance cost. It's great, yet bad at the same time.

There are also managed Backstage offerings like **[Roadie](https://roadie.io/)** that solve some of the problems, mainly maintenance cost. If you chose Backstage and you can use SaaS and you can afford it, Roadie is great.

There are many alternatives to Backstage, mostly as commercial, often SaaS offerings. There is **[Cortex](https://cortex.io)**, **[OpsLevel](https://opslevel.com)**, and many others.

The one I ended up using, the one I strongly recommend, and the one that is the winner is **[Port](https://getport.io)**. When it was created, it understood that the main purpose of portals is NOT to pretend that they are platforms but, rather, to sit on top of platforms. As such, it was focused on providing a data model, ways to visualize it, and to trigger events when we fill in forms based on those data models. It hasn't been doing much more, and that was a good thing. I liked that it had a clear focus. What I did not like is that it ignored Kubernetes. Just as with Backstage, Kubernetes is more of an afterthought than the design decision. I understand why that is so. Port, like many others, focused on where we are instead of focusing on where we will be. It focused on the present rather than the future. That was somehow mitigated with the ability to "discover" Kubernetes CRDs and CRs.

I chose Port for two main reasons. First, it's a data model with the ability to present data as a graphical user interface, rather than trying to be everything to everyone. That fit perfectly into my idea that portals are not platforms but, rather, graphical representations of platform. The second reason was the addition of discoverability. The moment they added the option to "discover" CRDs sitting in my cluster that serves as a control plane, I could stop redefining the same things over and over again. Instead of telling Port what is what it could simply discover services sitting in my platforms.

The problem, however, is that integration with Kubernetes does not seem to get much love and is still in early stages. On top of that, Port's API is bad forcing me to use the UI to manage the portal while I want the UI mostly for consumers, not for service providers or portal builders.

Nevertheless, among all the bad choices, Port is still, in my opinion the best, as long as using SaaS and its price are acceptable. That being said, I still hope that one of the existing solutions will get it right or that a new one will emerge given that I don't want to choose the less bad option but one that is really good.

Next, let's talk about GitOps.

## GitOps

There was a "GitOps war". We had, and still have, **[Argo CD](https://argoproj.github.io/cd)** and **[Flux](https://fluxcd.io)** as dominant tools and a few failed attempts like **[Rancher Fleet](https://fleet.rancher.io)**. We were spending tremendous amount of time trying to deduce whether Argo CD is better than Flux. Those conversations are now mostly over. Argo CD won. The main patron of Flux was WeaveWorks which went bankrupt leaving Flux with no finances to continue. There are still contributions flowing in but they cannot be compared to the "love" Argo CD is getting. Intuit, the company that started Argo CD or, to be more precise, acquired it, continues investing in it, even though it never had any commercial interests in it. Those contributions are now much smaller, mainly because many other companies started investing in it. RedHat was there for a long time, Codefresh joined the effort afterwards, Akuity was formed by core Argo CD maintainers, mostly ex-employees of Intuit, and quite a few others. Even though I would argue that Flux has a better design, better architecture, and is based on better ideas, it is, unfortunately, slowly dying leaving Argo CD as the last man standing. Hence, the winner is Argo CD.

If you are already using Flux, there might not yet be a compelling reason to change. If you're using something other than Argo CD or Flux, you might be making a terrible mistake. If you are just starting, Argo CD is a clear choice. Don't even bother looking for an alternative, at least not today. That might change in the future but, today, if you are looking for a GitOps tool, choose Argo CD.

Next, let's talk about Resource Management.

## Resource Management

People tend to talk about Configuration Management, Infrastructure-as-Code, and, these days, Control Planes, as very different concepts. They are partly right. Those are very different, yet they serve the same purpose. They are just different approaches to Resource Management. That is becoming clearer since the lines between configurations, applications, and infrastructure are becoming blurred. It is often close to impossible to separate configuration from resources. It's hard to dstinguish what is an application and what is infrastructure. We are moving towards immutability and everything managed through APIs, no matter whether that's networking, or processes, or hyperscalers, or anything. It's all about managing resources and what changed over time is how we manage resources.

Long long time ago, we would write scripts that would manage resources. Scripts were complicated to write and close to impossible to maintain given the number of permutations between the desired and the actual state of resources. As a result, we got Configuration Management tools like **[CFEngine](https://cfengine.com)** and, later on, **[Chef](https://chef.io/)** and **[Puppet](https://puppet.com)**, only for most of them to be eventually replaced by **[Ansible](https://ansible.com)**. They all have two things in common. They are based on promise theory allowing us to specify what we want instead of specifying how to do something. It was the job of those tools to convert the actual into the desired state. The second thing they all had in common is that they were designed to work directly with servers and that everything is mutable. We call those tools Configuration Management tools.

Two things happened afterwards. We started adopting immutability and, more importantly, APIs. That's when **[Terraform](https://terraform.io)** and, later on, **[Pulumi](https://pulumi.com)** were born. Instead of trying to SSH into servers and do whatever needs to be done, they focused on APIs. We would define what we want, the desired state, and those tools would reconcile it into the actual state by talking with APIs, be it AWS, Azure, Google Cloud, or any other. We call those Infrastructure-as-Code tools.

Now, to be clear, Ansible and other Configuration Management tools could talk to APIs, just as Terraform and other Infrastructure-as-Code tools could SSH. Still, the primary focus of Configuration Management tools is SSH and Infrastructure-as-Code tools communication with existing APIs.

What all those tools mentioned so far have in common is that they all assume that there are some definitions and some binary sitting somewhere, let's say a laptop. We execute that binary that takes those definitions and does whatever needs to be done with the resources on the other end so that those resources are in the same state as what's defined in those manifests.

Today we have a third wave of Resource Management tools. There is not yet a clear winner on how we call them, so I'll call them Control Plane tools. Their primary focus on creating Custom APIs. Instead of defining what we want on the same level as the resources we are trying to manage, those tools allow us to become service providers. They accomplish that by enabling us to create APIs that consumers can talk to. Now we can define what an application or a database or a cluster or anything else is as an API endpoint with its schema. Consumers do not need to deal with low-level details like VPC, subnets, or whatever else providers like AWS or Kubernetes give us. There is a clear internal separation of concerns with some people in a company creating services and others consuming those services. Consumers consume by invoking APIs and providers provide by building services behind those APIs. While Infrastructure-as-Code tools are based on lessons learned from consuming services created by others, Control Plane tools are based on lessons learned from building services.

There are many tools in this category. One of the older ones is Open Application Model or **[OAM](https://oam.dev/)** which started with the focus on applications but, since then, has been extended to any type resources. The most commonly used implementation of OAM is **[KubeVela](https://kubevela.io)**. It is the tool I used heavily in the past. However, it never managed to get over the initial Open Application Model spec resulting in a failure to extend itself beyond the initial scope. As a result, it had a great start but, since then, its popularity and usage dropped.

We also got **[Cluster API](https://cluster-api.sigs.k8s.io)** which enabled us to create Cluster-as-a-Service. Cluster API is amazing, but is focused only on Kubernetes clusters. It is solving only a fraction of the problem. At the same time, it is a very opinionated way to manage Kubernetes clusters which might or might not fit everyone's needs.

We also got **[Crossplane](https://crossplane.io)** which, just as KubeVela, allows us to create our own APIs endpoints as well as processes that listen to those endpoints and perform the reconciliation needed to convert the actual into the desired state of resources. What makes Crossplane more powerful is its flexibility. It allows us to create any type of a definition with very few if any restrictions. The major downside of Crossplane or, to be more specific, Crossplane Composition was that it tried to define processes that do the reconciliation as YAML. While YAML is great for defining the desired state, it is a very bad solution for defining the reconciliation of the states. That was fixed later on with the addition of Functions that allow us to use any programming language to define Compositions.

The newest addition to the family whose members build or contribute towards building control planes is **[Kro](https://kro.run)**. It is very similar to Crossplane Compositions and might, one day, be a better solution but, right now, it is in its infancy.

All those tools within the control plane sub-category are based on Kubernetes. It would not make sense to pick anything else since Kubernetes already has built-in and, in a way standard mechanism that provides the base all those tools should leverage. It has the machine that allows creation of CRDs which extend the "core" API and it has controllers that are affectively processes that listen to events in the API and perform whichever operations they should perform to reconcile the states. Hence, we can consider control planes extended versions of Kubernetes clusters.

The winner in this category must be from the Control Plane sub-category. That's where we are going. We cannot continue ignoring the option of building our own APIs and providing services to others in our company. We could, just as well, call this category Developer Platforms, except that there is more to platforms than managing resources. Nevertheless, we are moving from "Here are the files. Execute this and something will happen." to "I built a service for managing this and that, it is exposed through this API, use it."

The winner in this category is obviously Crossplane with a note that **Kro** might take over if it grows from an infant into something more mature.

*Now, before I continue, let me stress that I am part of the Crossplane project. You can interpret that any way you like. You can say: "Viktor is biased. Crossplane is not the best choice." Alternatively, you can also say "Viktor would not be involved with Crossplane if he does not think it's the best project in its category." It's up to you to choose which interpretation you'd like to take or even to come up with a third one. The reason I'm saying all this is transparency. I want to be clear when there might be some conflict of interest or when I might be subjective.*

All in all, this year's winner for the Resource Management category is Crossplane, and now we can move to State Management Formats.

## State Management Formats

We've seen explosion of the tools and formats that ultimately all do the same thing. I'm referring to tools that allow us to define data in some format and output that data into XML, Json, YAML, or whichever other format we need. We had such tools for a long time and their number increased drastically with the adoption of Kubernetes which expects us to define the desired state in YAML or Json.

One of the first, if not the first such tool to emerge is no other than **[Helm](https://helm.sh)**. It went through some drastic changes over its lifetime. The current model is based on Go Templating which is, essentially, a templating engine that allows us to produce the output in any format. As such, it is completely oblivious that we are, in case of Helm, dealing with data and that is its biggest weakness. We can, essentially, put anything as a template and Go Templating will happily convert it to text. The fact that we use Helm to generate YAML is not of much importance. Go Templating creates text output. That's why we need some silly constructs like *indent* to ensure that text can be interpreted as YAML.

The main strenghts of Helm are its age and adoption. It's been around forever and everyone using Kubernetes is familiar with it. It also became an unofficial standard resulting in almost all open source projects and vendors creating Helm chart as a way to distribute their "stuff". Whichever third-party app we need to run, there is almost certainly a Helm chart for it.

On top of all that, Helm is also a packaging tool. It allows us to package those template, those charts, and publish them to a registry. That simplifies distribution a lot and not all other tools we'll discuss today have it.

If we would be picking a winner for the format that should be used for third-party apps, we could stop here. Helm is the undisputable king in that area. However, I won't make it that easy so, instead, we'll limit the scope to your apps. Which format should you use for your applications?

In that scanrio, I would argue that Helm is the worst option. The sole idea that we should use free-text templating to generate data sounds silly. YAML is structured data and it makes perfect sense to use a language or a format that understands data.

Not long after Helm emerged, we got **[Kustomize](https://kustomize.io)** as a project maintained in one of Kubernetes SIGs or Special Interest Groups. The executable was even added to *kubectl* so we do not even need to install anything. It's simply there.

Kustomize takes a different approach when compared with Helm. Instead of applying templates, it applies patches. With Kustomize, we work with "pure" YAML and then patch that YAML to get variations we need. Those patches could be applied to ever changing tags, or to differences present in various environments, or anything else.

Kustomize is great, but only on relatively small projects. That fits well with many internal applications. If we define an application, we are likely not going to have many variations of the manifests of that app. We would typically change tags with each release, and have some differences in variaous environments. We might be changing the host, number of replicas, and a few other pieces of data when moving the application from, let's say, staging to production environment.

All in all, if what you have does not require great number of variations or some complex logic, Kustomize might be the winner. Still, that is often not the case, so let's continue.

We also go an explosion of formats or, to be more precise, languages that were designed, in some form or another, to work with data structures. There is **[Carvel ytt](https://carvel.dev/ytt)** which is a great format but failed to see adoption outside Tanzu users.

There is also **[CUE](https://cuelang.org)** which tends to be very popular among Go developers since it is a language built on top of Go. To make things even more interesting, we got **[Timoni](https://timoni.sh)** which introduced patterns that make working with CUE more focused on Kubernetes as well as a packaging mechanism that publishes it into OCI images. The problem with CUE and, through it, Timoni, is that it has its quirks. It is not always easy to understand what should be done and how it should be done and it can be a very challenging language to be picked up by people not interested in dedicating a lot of time to master it.

We also got **[KCL](https://kcl-lang.io)** which is similar to CUE, but easier to learn and without some of the CUE sillyness. Unlike CUE, anyone can pick it up in no time.

Then there is **[Pkl](https://pkl-lang.org)** which looked promising. It's great for those used to Java or Groovy, but failed to gain my trust. There is nothing truly wrong with it. It's just that I liked some other languages more.

Finally, we also got **[cdk8s](https://cdk8s.io)** which allows us to use general purpose languages to output data, mainly YAML. Cdk8s can be great or horrible depending on which language we choose. For example, the experience with TypeScript is excellent while Go results in too much boiler plate code, mostly related to transformations.

There are, ofcourse, many other formats and languages. It would take too much time to go through all of them. Those I mentioned are likely best contenders to be the formats you are already using or that you should switch to.

I used all those formats extensivelly, especially throughout 2024. My goal was to gain enough experience with all of them to be able to make a decision which one will be my choice. That turned out to be KCL, so I'm proclaiming it a winner. It took me a while to rewrite almost everything I have but I finally did it. I still keep simple "stuff" as "pure" YAML but everything else is now in the KCL format. To clarify, when I say "everything", I mean everything that is not being distributed outside my organization and excluding third-party apps I use. Those are still Helm, and are likely going to continue being Helm charts, whether I like it or not.

Next, we'll talk about terminals.

## Terminals

It's not secret that I spend significant amount of time in terminals. I think that everyone should use terminals and I'm sure that many do. It's much easier, more reliable, and reproducible to execute commands than to click buttons. UIs are great for observing, not necessarily for operations.

It would be impossible to compile the list of all the tools worth using inside terminals, so I'll focus only on those that made a significant impact on me and my workflow lately.

**[Starship](https://starship.rs)** is one of those that I fel in love the moment I discovered it. It's a very easy, yet very effective way to generate dynamic prompts. It's now on all my machines and I could not live without it, unless someone shows me a better one.

Then there is **[Charm](https://charm.sh)** which is a group of projects that do not necessarily have much in common, except that they are all somehow related to terminals. If you want to convert your Go code into an amazing CLI, there is *Huh*. If we'd like to send emails through terminal, there is *Pop*. If we would like to convert a Shell script into an interactive CLI, there is *Gum*. There are many other projects and I cannot recommend them enough. Charm is amazing and you should check it out if you haven't already.

The next in line is **[Zellij](https://zellij.dev)** that makes terminal multiplexing easy. I like easy. My brain is not developed enough to grasp tools that require a lot of learning. With Zellij, I was up and running with all my terminal tabs and panes in no time. I love it and use it all the time.

Finally, there is **[Nushell](https://nushell.sh)** which arguably made the biggest impact on me. When creating scripts and CLIs, I was always torn between the simplicity of Bash and the power of Go. Bash is great for very simple scripts and Go is amazing when working on complicated CLIs. Much of what I do is somewhere in the middle. Much of it is more than dozen or so lines when Bash becomes silly, but less then thousands of lines when Go is a great choice. Nushell changes all that. It is a Shell, and a language, and data processor at the same time. With it, we effectively write a script that compiles so we detect errors right away. It outputs everything as data instead of text, so piping outputs as inputs and processing those inputs is a breeze. It works everywhere. It's awesome and easily this years pick as a winner. After a while, I started using it exclusively for all my scripts. If you're following along my instructions from other videos, you likely already saw it in action, even though at simpler scenarios where it doesn't shine as much as it could. Try it. Use it. You'll love it.

The second to last category is Miscellaneous.

## Miscellaneous

This is the category with the tools that I feel are important but I could not fit into any other category.

There are three that made a splash.

**[NATS](https://nats.io)** has been around for a while, more often than not used by third-party tools. It's small, lean, fast, and easy to setup and use pub/sub server. Since I think that we should all be moving our applications to the publish and subscribe model, at least when more complex systems are concerned, NATS is a great choice. It might not be as advanced as, let's say, Kafka and other similar tools. Still, more often than not, we don't need complex tools that do much more than what we need. NATS is often just what is needed.

Then there is **[wasmCloud](https://wasmcloud.com)** that was moved from Sandbox to Incubation status in CNCF. I'm not yet convinced that WASM is ready for prime but if you are already using WASM, wasmCloud might be a good choice of a tool to define and manage your apps.

Finally, the last one in this somehow random category is **[Dapr](https://dapr.io)**, a CNCF project which was always awesome. What makes it special is that it recently graduated. Now it is in the group of "elite" projects that proved itself worthy to be alongside other projects CNCF considers mature.

It's close to impossible to declare a winner in this category since all those are completely different. They are all great in their own merit. Still, I promised to pick one, and that one is NATS. I'm picking it mostly because I feel that, even though it is commonly used by third-party apps, it is not getting as much love from end users as it should.

There is only one category left, and that one might be the most interesting one. Let's see what are your choices.

## Users Choice

When I asked you to send me your favorite tools and services, I did not expect such a massive response. You wrote quite a few public comments and sent me a massive number of private messages. Thank you all for participating.

I compiled a list of hundreds of tools and services you are using. It would take a lot of time to list all of them, so I'll limit it the top ten, sorted in descending order, from most to least mentioned.

That would be **Argo CD**, **Kubernetes**, **Cilium**, **Crossplane**, **Git**, **ESO**, **Terraform** or **OpenTofu**, **Cloud**, **Port**, **vCluster**.

According to you, **Argo CD** is a clear winner. It's the tool that most of you recommend. However, I won't proclaim it a winner of this category simply because I already did that when we talked about GitOps.

Then there is **Kubernetes** which I assume all of you are using and like, including all those who voted for Argo CD. I'll ignore that one as well, simply because it's a base that everyone uses these days. If Kubernetes would be a contenter, it would win every single year, at least in the foreseable future.

The next in line is **Cilium**, and that's the one I'll proclaim the winner. Cilium is amazing. It became the CNI of choice for most Kubernetes clusters. There are many people who run Kubernetes without even knowing that its networking is handled by Cilium. For many of us, Cilium was the first introduction to eBPF. Many other projects adopted eBPF thanks, in part, to the success of Cilium.

For some, Cilium removed the need for a service mesh. For others it provides valuable source of networking metrics. Then there are those who use it primarily to manage network policies. And so on and so forth. Almost everyone is using Cilium for some reason or another, some without even knowing they're using it. It is truly the de-facto standard when networking is concerned and it rightfully deserves to be the winner for the user choice award.

## The Winners

There you have it folks. Those are the winners for 2025.

We got a relatively insignificant **AI** tool **[Fabric](https://github.com/danielmiessler/fabric)**. It is by no means as important or as big as what we've been seeing in the AI space yet it is the tool I'm using all the time. It shows the tremendous difference between bad and good promts.

With the **CI/CD** space I chose **Nix Packages**, specifically **[Devbox](https://jetify.com/devbox)**. It is not, stricly speaking, a CI/CD tool. We can use it to define packages, mostly CLIs needed to work on a project. Those can be used when working on a project locally or inside CI/CD workflows. We can also use it to build container images or DevContainers if we work in remote environments. It's amazing and I use it in every single project I work on.

When **Containers** are concerned, I also made an unusual choice. Instead of choosing between Docker, Rancher Desktop, and other container engines, I choose **[Chainguard Images](https://chainguard.dev/chainguard-images)**. Chainguard made a significant change by providing slim and zero-vulnerabilities base images.

**Developer Portals** are one of the biggest trends right now. To be more precise, Developer Platforms are the area where we see companies putting a lot of investment, and portals are an important part of them. Unfortunately, I don't think we have a great solution in that area. Among mostly bad choices, I think that **[Port](https://getport.io)** is the best choice, so I proclaimed it a winner in that category.

**GitOps** was one of the areas where we had a lot of discussions in the past. We would often enter into heated arguments where some would be defending Argo CD, others would be advocating for Flux, and some would be desperatly trying to introduce other solutions. I think those debates are now over. **[Argo CD](https://argoproj.github.io/cd)** is the winner in general as well as my choice in this category. That was an easy one, even though I think that Flux is better in quite a few aspects.

When it comes to **Resource Management**, it is now clear that resources should be managed by Kubernetes, no matter whether those resources run in the same cluster or are elsewhere. Kubernetes is becoming the standard and the default choice for managing any type of resources, anywhere. Within that category, there is nothing even close to **[Crossplane](https://crossplane.io)**. New projects are emerging so that might change in the future but, for now, it is an easy choice as well.

Another area where we see a lot of changes are **State Management Formats**. There is an explosion of languages and formats that, ultimatelly, all serve the same purpose. They all transform something into structured data, predominantly YAML. New languages are being born and wrappers around existing languages are being created. It's as if everyone is trying to replace Go Templating used in Helm. The one that impressed me the most by being relatively simple yet providing everything I think we might need is **[KCL](https://kcl-lang.io)**, it is is the winner and my recommendation. I moved all my "complex" definitions to KCL.

When it comes to working in **Terminals**, **[Nushell](https://nushell.sh)** is a language I ended up using almost every day. It is a Shell and a language and a data processor at the same time. All the scripts I'm writing, be it simple or complex, are now Nushell. It's awesome, and I strongly recomment is.

The **Miscellaneous** category is a weird one since it contains unrelated tools I could not place anywhere else but I feel deserve to be considered. Even though it is not very new and is widely used among third-party projects, I think that **https://nats.io** does not get the attention and adoption it deserves among the end-users, so proclaiming it a winner and recommending it is my way to put it under the spotlight.

Finally, your choice, the **Users Choice** is **[Cilium](https://cilium.io/)**. There's probably no need to say what it is nor why it was chosen. It's the winner. I'm glad you chose it. Otherwise, I'd need to add eleventh category Networking only to ensure that it is one of the winners.

That's all folks. That was the list of the tools you should consider adopting in 2025. You probably use at least some, if not all of those. If you aren't, get to it. Put them into your toolkit.

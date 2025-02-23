
+++
title = 'Past, Present, and Future of Internal Developer Platforms'
date = 2025-02-24T15:00:00+00:00
draft = false
+++

Can you guess which tool I used to build my first Internal Developer Platform?

Let me give you a few tips before you try to guess.

1. It performed most of the **tasks people needed** to do, like deployments of applications, execution of tests, and creation of new and management of existing servers and databases.
2. It was very **easy to use**.
3. It had a **graphical user interface** but it could be used without it through an API or a CLI.
4. It had **nothing to do with Kubernetes, or containers, or Cloud**.
5. The tool used to create it was one of the **most popular** tools back then, and is still one of the most widely tools used in enterprises.
6. Almost **no one used that platform** after the initial excitement dried out.

Which tool do you think I used? Which tool fits that description?

<!--more-->

{{< youtube WAm3ypS0_wg >}}

You probably won't guess it, so let me reveal it right away.

<img src="/logo/jenkins.png" style="width:30%; float:right; padding: 10px">

The tool I used to build my first Internal Developer Platform is Jenkins some 20 years ago.

Some of those statements might sound incorrect or confusing.

You might say that Jenkins is not the tool designed to build Developer Platforms. But, before you say that, please think about it first.

It has a customizable user interface we can use to define forms with buttons and it can display data in different formats. Isn't that the most rudimentary description of a portal we would put on top of a platform? It can execute tasks that will deploy applications, run tests, setup database servers, create and maintain clusters, or anything else. Doesn't that sound like a description of services that perform operations we need a platform to perform?

The correct question should be different. It's not whether Jenkins is the right tool to build a platform but rather whether it is a good choice to be the tool for such a platform today. The answer to that one is a huge **no**. We have better tools for that task today. Technology changes, and the fact that a tool is not a good choice for a specific task today does not mean that it wasn't a good choice in the past. Back when I was young, Jenkins was a new project that revolutionized quite a few things. It was a great choice as the base of a platform.

Another confusing thing I said was my statement that I built an Internal Developer Platform 20 years ago. How can that be if the term **Internal Developer Platform** was coined less than 10 years ago and gained traction in early 2020s? Am I writing this post in 2040s? Did I invent a time machine?

The truth is that we did not call it Internal Developer Platform. I'm not sure what we called it. It was probably called platform or we did not have a name for it and called it for what it is; Jenkins.

Names do not matter. What matters is that we were building a platform that was supposed to be used by developers internally decades ago. I'm sure we were not the first though. There were surrely others before us, and there were certainly others after us.

The point is that since early days of software engineering people were building tools that **help developers do their work** faster, or easier, or better. When we combine those tools, we get a platform. Such platforms that were built decades ago are certainly not looking like those we are building now, yet, their primary purpose is still the same. Enable software engineers to do whatever they need to do.

Details are changing as our industry is changing, but the fundamental objectives are still the same. We were building developer platforms in the past, we are building them now, and we are likely to be building them in the future.

That's the subject of today's video. We are going to explore **the past, the present, and the future of developer platforms.**

By understanding the past, we can better understand the present and knowing what we're doing today should help us understand where we might be going in the future.

## What Is Internal Developer Platform?

The term Internal Developer Platform gives us a good clue as to what it is.

It is **Internal** meaning that it is not meant to be used by general public but internally within an organization. In theory, any employee of a company might be a good candidate to use it.

Then there is **Developer** which further limits the users of it. Assuming that any software engineer is a developer, any softwre engineer inside a company is a potential user. As a such, we are limiting the user base from "anyone in a company" to "developers in a company".

Finally, it is a **Platform** which, in turn is a collection of applications or, as we call them today, services that perform certain tasks or provide certain information. What those tasks and information are differs from one company to another. That's the main difference between it being internal and more generic platform like AWS, Azure, or Google Cloud. Internal Developer Platforms are tailor-made to match our needs. Unlike other platforms, we are not trying to convince the whole world to use it.

In other words, an Internal Developer Platform is a set of applications or services that **help developers perform tasks** specific to the company they work in.

Keep that definition in mind while we go back in time and see how it all started, what it is today, and how it might look like in the future.

While we're going down the memory lane, please suspend your "modern" preconcived ideas what a platform is and stick with the broad "definition".

## Internal Developer Platforms of the Past

It all started with **scripts**.

Someone would get bored by having to perform the same tasks over and over again. Those tasks were typically focused on operating servers, deploying applications, or observing the state of our systems.

How did that look like? The answer to that question depends on what you used back in twentieth century. For those of us who used Windows, in my case that would be Windows NT, everything was done through a graphical user interface of Windows itself or an application running in Windows. It was all colorful and full of forms and buttons. I spent a lot of money on a therapist trying to forget that period of my life, so I'll skip reliving it.

Others were using Unix or Linux or BSD or Mainframe or whatever else we might have had at that time. Some of those had graphical user interfaces mostly designed to make Windows refugees feel somewhat at home. Natives, on the other hand, were using terminals. Type something, press the enter key, see the output, repeat.

Most of the operations were being done with SSH. We would SSH into a server, copy some and edit other files, run, shut down, or restart a process, see the output, repeat. That's it. It was a black terminal with green font. SSH was how we accessed servers and *Vi* was the editor. Operating servers meant starting processes and changing their configuration. Applications were being deployed by copying binaries into servers and running them. Observing the state of a server meant watching logs, parsing them to find what we're looking for, and restarting servers when we could not figure out a better way to fix an issue.

There was a lot to know and even more to learn. Experienced people would type commands from memory when in presence of other colleagues, and copying and pasting commands when no one was watching.

Eventually, someone would ask themselves: "Why I am wasting my time by copying and pasting instructions from a document? I have better things to do than act as a copy & paste expert. Let me create a script that will do that for me." Eventually, that person would end up with a bunch of scripts. Some would be configuring specific aspects of servers, others would be deploying applications, and there would be those that would *cat* log files, pipe the output to RegEx expressions to find what we're looking for.

What did we get when we combine all those scripts? We got an Internal Developer Platform. Those scripts were useful only to people in that company, they were being executed by developers and since it is a collection of them, they are essentially very simple applications that form a platform.

We eventually started putting those scripts into **Cron Jobs**.

Instead of trying to remember to restart a process every morning in an attempt to avoid fixing memory leaks, we would define a cron job that would execute one of our scripts.

That was the era of Cron Jobs. When we needed something on-demand, we would execute a script and when we needed something to be scheduled, we would define a Cron Job that would run a script for us. Life was good, at least when compared to the life we had before.

Then something wonderful happened. Kohsuke Kawaguchi decided to make a graphical user interface for Cron Jobs and scripts and named it Hudson.

Later on, due to some dispute with Oracle, Hundson became [Jenkins](https://jenkins.io) which, in turn, became the primary tool for building Internal Developer Platforms, even though the term itself was to be invented a few decates later.

So, we got Jenkins which is, essentially, a Cron Job written in Java and with a graphical user interface. It's primary use case was to "automate" execution of tasks. "Which tasks?" you might be asking. Well... Anything a developer needs and that everything is essentially the same as what we were doing before. There would be a Jenkins job that would configure a server, another that would deploy an application, yet another that would generate some data we would use to observe the state of something, and so on and so forth. Later on we associated Jenkins with CI. Still, it was a tool that could execute tasks. Some of those tasks were being executed periodically, just as Cron Jobs are, while others were being executed on-demand. That, on-demand would be as a result of pushing something to CVS. That's what we had before Git, which came after SVN and Perforce which came after CVS. If you are in your twenties or thirties you are almost certainly not familiar with any of them. If you are still using one of those you are probably working in a company that made a decision to defy physics by stopping the flow of time and stay stuck in 80s and 90s.

All that was nothing special. What made Jenkins "special" is its graphical user interface. Many were missing pretty colors, forms, and buttons; especially Windows refugees. That's when we added **Portals** to Internal Developer Platforms. All of a sudden, one could open a Jenkins page in Netscape (there was no Chrome nor Firefox back then). Over there, one would be presented with a form with fields that could be filled in, and a Submit button. When pressed, that button would execute a Jenkins job with data from those fields as inputs. Those jobs would do whatever needs to be done, and, at the end of a run, we would see the outcome. Alternatively, we could be watching what's going on in real-time. That was amazing since, back then, we did not have Netflix nor YouTube. Watching a Jenkins job while running was the popular entertainment. We would have something useless to do while our colleagues would think that we are doing something very productive. That's why every job would be streaming huge quantity of useless data. It looked very professional.

![](/logo/cfengine.png?height=5vw&classes=inline)
![](/logo/chef.png?height=5vw&classes=inline)
![](/logo/puppet.png?height=5vw&classes=inline)
![](/logo/ansible.png?height=5vw&classes=inline)

As time passed, we got better and better tools to put on our toolbelt. We got **Configuration Management** tools like [CFEngine](https://cfengine.com/), [Chef](https://chef.io), [Puppet](https://puppet.com/), and [Ansible](https://ansible.com) that helped greatly as a replacement for those brittle scripts. That gave rise to the concept of the **Desired State**.

Until those tools appeared, we had scripts full of conditionals. If a server is down, start it. If it's up, SSH into it. If an application is running, shut it down. If it's not running, or if it was just shut down, copy the binary with the new release and start it again. Half of all scripts were *if*, *else*, and *else if* statements.

Configuration Management tools tought us that we do not have to do that. Instead, we could focus on defining what we want, and they would figure out whether something should be started or shut down, whether the correct release is running and, if it's not, to replace the binaries, whether specific entry in a configuration file is set to a specific value and, if it's not, set it, and so on and so forth.

<img src="/logo/aws.png" style="background: white; width:30%; float:right; padding: 10px">

Time kept moving and we kept being surprised. The next surprise was AWS. Now, that surprise was not what you might think it is. The "innovation" that it brought us is not that we can have storage and VMs on demand, but that everything can be operated using **HTTP API**.

Now, to be clear, AWS did not invent HTTP APIs. It did, however, make them a norm when operating systems. All of a sudden, we did not need to SSH into servers to accomplish something. Instead, we could just send a request to its API demanding something. It was like being a boss on a phone call with one of its employees. "I want 3 storages and 5 virtual machines", followed by him hanging up. There is no discussion. No brain-storming. No questions whether it can be done, nor how it should be done, nor when it will be done. It was a command that contained the desired state. "This is what I want. Do it now, and make sure it stays that way." Now, to be clear, we already had the ability to demand something based on the desired state. That's what Puppet, Chef, Ansible, and other tools were for. The new invention was that phone call or, in case of AWS, that HTTP request to its API. APIs changed everything.

As a side note, none of the tools we used so far went away. We still had scripts that were executed by Jenkins. Now we added Configuration Management tools that were also executed through Jenkins jobs and, sometimes, talked to AWS API.

Over time, HTTP APIs became a norm. Everyone adopted them so now we could operate stuff also through Azure API, Google Cloud API, VMWare API, or API of any other provider, no matter whether that provider gave us Something-as-a-Service or installed something in our on-prem data center.

![](/logo/terraform.png?height=10vw&classes=inline)
![](/logo/pulumi.png?height=10vw&classes=inline)

Soon after the "AWS revolution", we got tools that were dedicated on interaction with APIs of Cloud providers. Good examples of such tools would be [Terraform](https://terraform.io) and [Pulumi](https://pulumi.com). We call such tools **Infrastructure-as-Code** tools. The major difference between them and those we had before is that they were based on the idea that everything can be managed through APIs. Conceptually, they still allowed us to define the desired state just as Configuration Management tools did. What made them different is that they were translating that state to API calls.

That leads me to today.

## Internal Developer Platforms Today

We got a few very important innovations in the last ten years or so, namely containers and container orchestrators like Kubernetes. Those, however, did not make a significant impact on Internal Developer Platforms. They changed the industry, but in a different way.

The ability to orchestrate containers is not what made Kubernetes great. It's something else. Standardization and extensibility is what made Kubernetes one of the most significant projects in the history of software engineering. All of a sudden, projects and vendors could extend Kubernetes in a standard way. As a result, we can use and combine all sorts of different tools without having to learn new syntax. Everything works the same.

From the user's perspective, a project adds one or more Custom Resource Definitions (CRDs) and controllers, and we write Custom Resources to instruct those controllers what to do. Those CRDs are effectivelly extending Kubernetes API.

On top of that, they are all interoperable. We can combine a service mesh with observability tools and plug in a GitOps tool into the mix without any of those tools having to do anything "special" to make it happen. They do not need to know that others exist and they do not have to do anything special for us to combine them.

The ability to extend Kubernetes API and instruct it what to do when someone creates a resource based on those extensions is what made it all possible.

Why does that matter for those of us building Internal Developer Platforms? What is it that we can do now that we couldn't do before?

We can do internally what AWS and other public service providers did globally. We can provide users of our platform with APIs through which they can request services we are offering. That worked exceptionally well for AWS and, given how easy it is to do it in Kubernetes, it works for us as well. Instead of giving people scripts and files to tinker with, as we were doing all this time, we can now give them API endpoints to consume our services. On top of that, we can create Kubernetes controllers that are acting as those services. When someone requests something on our API, the one we are building by extending Kubernetes API, controllers can make sure that something is done. CRDs are used to build the API and controllers are services we are offering in our platform.

Now, to be clear, it's not as if we could not create APIs and services before. We could, but it was complicated and it wasn't standardized so everyone ended up with their own snowflakes. Now it is democratized. Now we can do, with relative ease, what AWS and others did. We can have our own tailor-made control planes.

![](/logo/kubevela.png?height=5vw&classes=inline)
![](/logo/kratix.png?height=5vw&classes=inline)
![](/logo/crossplane.png?height=5vw&classes=inline)

To make things even simpler, we got tools that make a relatively easy job even easier. Early on, when CRDs were introduced to Kubernetes, we had to write a bunch of Go code to make controllers that react when CRs are created, updated, or deleted. Since then, we got projects like [KubeVela](https://kubevela.io/), [Crossplane](https://crossplane.io), [Kratix](https://kratix.io), and [kro](https://kro.run).

They might be using different ways to describe what they're doing but, in a nutshell, they are all, among other things, enabling us to create those CRDs and controllers, often with the objective to compose low-level resources or execute specific processes.

We got a new category called **Control Plane** tools.

With both the past and the present in mind, today we have a few good practices when building developer platforms.

1. We manage the **state** of resources by extending Kubernetes with tools like Crossplane.
2. We execute **tasks** with tools like GitHub Actions.
3. We expose our services through **APIs** created as Kubernetes CRDs.
4. We leverage the rest of Kubernetes ecosystem to implement **RBAC**, **policies**, **GitOps**, and whatever else we might need.
5. We create **user interfaces** like Web UIs with tools like Backstage, CLIs with whichever progremming language you prefer, or any other type of UIs. All those do one and only one thing. They all provide a way for users to interact with our APIs.

So, think of the current era of platform engineering as a period focused on doing whatever we were doing before, but with the additional focus on exposing services through HTTP APIs. Kubernetes is becoming **invisible** for developers. It is an implementation detail that matters to those creating platforms. They use it as the base on top of which they build a platform which, from the end user perspective, does not expose low-level Kubernetes details.

Right now, Kubernetes is moving to the background, just as hyperscalers did a while ago.

That's where we are today.

Where are we going from here? The answer to that question should not surprise anyone.

## The Future of Internal Developer Platforms

Nobody can predict the future, least of all me. But, if I would have to place a bet, that would be on **AI**, but in a very different form than what we are seeing today.

AI, right now, is focused on providing information. We can hook it into our platform, and let users ask questions like "What's wrong with my application?" We can also let it go through the logs and find the parts that matter. We can let it evaluate the state of our system and give us information about the parts that are missbehaving or give us suggestions what we can improve. We can even ask it to suggest how we should define something and write code for us.

It all boils down to the same principles. We give AI models access to our data which it combines with more general data it already has. With all that information, it can help us out by answering our questions about the system as a whole or specific parts of the system.

I don't think that makes a significant difference. It's good, but not great. It's not the "revolution" we are all hoping for.

I am much more interested in potential ability of AI to perform actions.

AI can already detect if something is wrong. While I like being informed, I don't really want it to tell me what's going on, but to fix issues. It can already detect pattern anomalies, but I want it to act on them. It can already suggest how to define a database, but I want it to create it instead.

We are not there yet.

AI is not even a reliable chat partner today. Its recommendations are often wrong or not optimal and, until that changes, I certainly don't want it to perform any actions on our behalf. But that will change. At some point it will stop giving suggestions on the level of an undergraduate student and reach a level of a senior engineer. When that time comes, we can start thinking about the next phase. We can start giving it more and more control over our systems and let it perform actions on our behalf. When that day comes, our role will change. When AI starts acting instead of only suggesting, we will be overlords. We will act in a way closer to how managers act today. We, software engineers, will manage and overview AI. We will act more like PR reviewers than coders.

Just as today a junior developer might write code and create a PR that is reviewed by a more senior engineer, AI will be doing something similar. We will say what we need, and it will act on our behalf. We will observe the outcomes just as PR reviewers or observability experts are observing code and live systems. We will be doing all that through Internal Developer Platforms, or, to be more specific, through the API of the platform.

Such an AI would likely be an agent. Given that our systems can span many servers, accounts, clusters, and whatever else, it would be silly to put an agent everywhere. Instead, it is likely going to be sitting in a single place and perform actions through APIs with the one built on top of our platform being the main one since it acts as a kind of a gateway to all the others.

What comes after that? Nobody knows. We might end up living in Matrix, or we might be facing terminators, or we might all live like gods. That's a far away future that cannot be predicted and none of us is likely to be alive by then. What matters, for now, is that shorter-term future is in AI performing actions on our behalf. That's not today, but that is coming.

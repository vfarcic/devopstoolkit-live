+++
archetype = "home"
title = ""
+++

# Latest Posts

<a href="/internal-developer-platforms/stop-setting-up-developer-portals-manually-feat-port-mcp"><img src="/internal-developer-platforms/stop-setting-up-developer-portals-manually-feat-port-mcp/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Stop Setting Up Developer Portals Manually! (feat. Port MCP)](/internal-developer-platforms/stop-setting-up-developer-portals-manually-feat-port-mcp)

Setting up a developer portal properly takes weeks. Blueprints, actions, workflows, GitOps integration, scorecards, relationships between entities. It's a massive undertaking, and most teams underestimate how much work it actually is.

I wanted to set up Port, my favorite commercial internal developer portal. If you're building an IDP, Port sits on top of it and gives you the UI layer. I've covered Port in depth before in [How To Build A UI For An Internal Developer Platform (IDP) With Port?](https://youtu.be/ro-h7tsp0qI) and [Mastering Developer Portals: Discover & Integrate API Schemas with Port](https://youtu.be/PV1sBiC85Yc), so I won't rehash what it does here.

Here's the situation. My Port account is completely empty. I destroyed everything so we can start fresh. I want to set up everything: blueprints, actions, workflows, GitOps integration, scorecards, the works. But I don't have weeks. I don't even have days.

Clicking through the web UI isn't an option. Writing endless JSON and figuring out dozens of API calls isn't either. Port is awesome, but it's not magic. It doesn't just materialize results out of thin air.

Or... maybe it does.

Let me show you. We'll start from the platform builder's perspective, then switch to how platform users can interact with it. 

**[Full article >>](/internal-developer-platforms/stop-setting-up-developer-portals-manually-feat-port-mcp)**

---

<a href="/containers/my-production-dockerfile-rules-how-i-build-docker-images"><img src="/containers/my-production-dockerfile-rules-how-i-build-docker-images/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [My Production Dockerfile Rules: How I Build Docker Images](/containers/my-production-dockerfile-rules-how-i-build-docker-images)

Most Dockerfiles I see in production are security nightmares waiting to happen. Running as root. Using `:latest` tags. Copying entire directories including secrets. And the images? Bloated with debugging tools that attackers love.

Here's the thing. Writing a good Dockerfile isn't hard. It's just that nobody taught you the rules. Today, I'm going to show you every best practice you need to build production-ready containers. We'll cover image selection, build optimization, security hardening, and maintainability. And at the end, I'll show you how AI can apply all these rules automatically.

Let's start with the foundation: choosing the right base image.

**[Full article >>](/containers/my-production-dockerfile-rules-how-i-build-docker-images)**

---

<a href="/development/api-gateways-explained-why-your-services-are-a-mess-zuplo-review"><img src="/development/api-gateways-explained-why-your-services-are-a-mess-zuplo-review/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [API Gateways Explained: Why Your Services Are a Mess (Zuplo Review)](/development/api-gateways-explained-why-your-services-are-a-mess-zuplo-review)

Every API you expose to the world needs the same things. Authentication. Rate limiting. Documentation. Analytics. And if you're building AI-powered applications, add cost tracking and provider failover to that list.

You could implement all of this in every single service you build. Or you could centralize it in an API gateway.

Today we're exploring what API gateways actually are, when you need them, and when you don't. We'll look at how they fit into different architectures, from traditional VMs to Kubernetes to serverless. And we'll take this opportunity to explore Zuplo as a possible solution for your API gateway needs.

**[Full article >>](/development/api-gateways-explained-why-your-services-are-a-mess-zuplo-review)**

---

<a href="/misc/top-10-devops-tools-you-must-use-in-2026"><img src="/misc/top-10-devops-tools-you-must-use-in-2026/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Top 10 DevOps Tools You MUST Use in 2026](/misc/top-10-devops-tools-you-must-use-in-2026)

2025 was the year agentic AI went from interesting experiment to daily reality. AI agents stopped being autocomplete tools and started understanding entire codebases, refactoring across files, writing tests, debugging their own mistakes, managing infrastructure, and handling operational tasks. For many developers and ops engineers, the way they work fundamentally changed.

2026 is the year to take this seriously. Not just for application developers, but for DevOps engineers, SREs, and platform teams. The tools are mature enough now. The productivity gains are real. If you're not integrating AI agents into your workflow, you're leaving significant value on the table.

But here's the thing: agentic AI doesn't replace everything else. You still need solid foundations. Internal developer platforms, testing frameworks, scripting languages, development environments. These tools still matter. What's changed is that AI now intersects with all of them.

So this year's recommendations cover both: the AI tools that emerged in 2025 and the non-AI tools that remain essential. I spent 2025 testing all of them in real projects, real workflows, real problems. Not quick demos.

**[Full article >>](/misc/top-10-devops-tools-you-must-use-in-2026)**

---

<a href="/kubernetes/stop-trusting-kubectl-get-all-heres-what-it-hides-from-you"><img src="/kubernetes/stop-trusting-kubectl-get-all-heres-what-it-hides-from-you/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Stop Trusting kubectl get all! Here Is What It Hides From You](/kubernetes/stop-trusting-kubectl-get-all-heres-what-it-hides-from-you)

Ever wonder why Kubernetes has a command called `get all` that doesn't actually retrieve all your resources? Try it yourself and you'll find it conveniently forgets about Ingresses, PersistentVolumeClaims, and potentially many other resource types.

Worse yet, even when you manually list everything in a namespace, you're left staring at a pile of objects with no idea how they fit together. There's no built-in way to say "these five resources form a complete system" or to check if that system is healthy.

Turns out, this is a real problem when navigating clusters and trying to understand what's actually running. So I created a Custom Resource Definition that wraps related resources into logical groups with status, context, and relationships.

In this video, I'll walk you through the problem, explore how Kubernetes ownership and ownerReferences work, and demonstrate a better approach using CRDs.

**[Full article >>](/kubernetes/stop-trusting-kubectl-get-all-heres-what-it-hides-from-you)**

---

<a href="/ai/stop-sitting-on-the-bench-why-ai-resisters-are-getting-kicked-out"><img src="/ai/stop-sitting-on-the-bench-why-ai-resisters-are-getting-kicked-out/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Stop Sitting on the Bench! Why AI Resisters Are Getting Kicked Out](/ai/stop-sitting-on-the-bench-why-ai-resisters-are-getting-kicked-out)

Today we're going to talk about something completely different. It's about betting, not software engineering. Or is it? I guess we'll find out.

**[Full article >>](/ai/stop-sitting-on-the-bench-why-ai-resisters-are-getting-kicked-out)**

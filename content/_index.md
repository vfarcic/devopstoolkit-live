+++
archetype = "home"
title = ""
+++

# Latest Posts

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

---

<a href="/observability/distributed-tracing-explained-opentelemetry--jaeger-tutorial"><img src="/observability/distributed-tracing-explained-opentelemetry--jaeger-tutorial/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Distributed Tracing Explained: OpenTelemetry & Jaeger Tutorial](/observability/distributed-tracing-explained-opentelemetry--jaeger-tutorial)

Your users are complaining that your application is slow. Sometimes it takes 8 seconds to respond, other times 2 seconds. But when you check your metrics, everything looks fine. Average response times are acceptable. All services report healthy. Your dashboards are green.

So either your users are idiots, or you're not capable of capturing what's actually happening with their requests. Now, I tend to assume users are right. Which means I'd have to call you... Well... I'm not going to do that. Instead, I'm going to show you why you can't see what's really happening.

Here's what you're about to learn. You'll see exactly how to track requests as they flow through dozens of microservices, identify which specific operation is causing delays, and understand why your traditional observability tools are lying to you. By the end of this video, you'll know how to implement distributed tracing that actually shows you what's happening in your system.

Let's start with why this problem exists in the first place.

**[Full article >>](/observability/distributed-tracing-explained-opentelemetry--jaeger-tutorial)**

---

<a href="/development/top-10-github-project-setup-tricks-you-must-use-in-2025"><img src="/development/top-10-github-project-setup-tricks-you-must-use-in-2025/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Top 10 GitHub Project Setup Tricks You MUST Use in 2025!](/development/top-10-github-project-setup-tricks-you-must-use-in-2025)

Have you ever seen a GitHub issue that just said "it's broken" with zero context? Or reviewed a pull request where you had no idea what changed or why? How many hours have you wasted chasing down information that should have been provided upfront?

Here's the reality: whether you're maintaining an open source project, building internal tools, or managing commercial software, you face the same problem. People file vague bug reports. Contributors submit PRs without explaining their changes. Dependencies fall months behind. Security issues pile up. And you're stuck playing detective instead of building features.

But here's what most people don't realize: all of this chaos is preventable. GitHub has built-in tools for issue templates, pull request templates, automated workflows, and community governance. The problem is that setting all of this up manually takes hours, and most people either don't know these tools exist or don't bother configuring them properly.

**[Full article >>](/development/top-10-github-project-setup-tricks-you-must-use-in-2025)**

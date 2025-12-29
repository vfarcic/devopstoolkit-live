+++
archetype = "home"
title = ""
+++

# Latest Posts

<a href="/kubernetes/stop-trusting-kubectl-get-all-heres-what-it-hides-from-you"><img src="/kubernetes/stop-trusting-kubectl-get-all-heres-what-it-hides-from-you/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Stop Sitting on the Bench! Why AI Resisters Are Getting Kicked Out](/kubernetes/stop-trusting-kubectl-get-all-heres-what-it-hides-from-you)

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

---

<a href="/kubernetes/deploy-ai-agents-and-mcps-to-k8s-is-kagent-and-kmcp-worth-it"><img src="/kubernetes/deploy-ai-agents-and-mcps-to-k8s-is-kagent-and-kmcp-worth-it/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Deploy AI Agents and MCPs to K8s: Is kagent and kmcp Worth It?](/kubernetes/deploy-ai-agents-and-mcps-to-k8s-is-kagent-and-kmcp-worth-it)

What if you could manage AI agents with kubectl? **kagent** lets you define AI agents as custom resources, give them tools, and run them in your cluster. **kmcp** deploys MCP servers to Kubernetes using simple manifests. Both promise to bring AI agents into the cloud-native world you already know.

The idea sounds compelling. Create agents with YAML, connect them to MCP servers, let them talk to each other through the A2A protocol. All running in Kubernetes, managed like any other resource. It's the kind of integration that platform engineers dream about.

But there's a gap between promise and reality. We're going to deploy both tools to a Kubernetes cluster, create agents, connect them to MCP servers, and see what actually happens when you try to use them. We'll find out if this is the future of AI in Kubernetes, or if we're solving problems that don't need solving.

**[Full article >>](/kubernetes/deploy-ai-agents-and-mcps-to-k8s-is-kagent-and-kmcp-worth-it)**

---

<a href="/ai/gemini-3-is-fast-but-gaslights-you-at-128-tokens-second"><img src="/ai/gemini-3-is-fast-but-gaslights-you-at-128-tokens-second/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Gemini 3 Is Fast But Gaslights You at 128 Tokens/Second](/ai/gemini-3-is-fast-but-gaslights-you-at-128-tokens-second)

Gemini 3 is fast. Really fast. But speed means nothing when the AI confidently tells you it fixed a bug it never touched, or insists a file is updated when it's completely unchanged. That's not laziness. That's gaslighting at 128 tokens per second.

**[Full article >>](/ai/gemini-3-is-fast-but-gaslights-you-at-128-tokens-second)**

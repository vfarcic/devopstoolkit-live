+++
archetype = "home"
title = ""
+++

# Latest Posts

<!-- <a href="/kubernetes/self-healing-kubernetes-when-to-use-ai-vs-traditional-automation"><img src="/kubernetes/self-healing-kubernetes-when-to-use-ai-vs-traditional-automation/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Self-Healing Kubernetes: When to Use AI vs Traditional Automation](/kubernetes/self-healing-kubernetes-when-to-use-ai-vs-traditional-automation)

It's 2 AM Saturday. You're sleeping when your phone rings. You wake up ready to answer with "go to hell you prick," but then you see it's PagerDuty. Pods are crashing in production. You drag yourself to your laptop, still half-asleep, and start digging. Events. Logs. Metrics. Configurations. Forty minutes later, you've found it: an out-of-memory condition. The fix takes thirty seconds. A simple memory limit adjustment. Something that could have been detected and fixed automatically while you slept.

This doesn't have to be your reality. Here's what you'll learn: how to build automation that watches Kubernetes events, analyzes problems, and remediates issues before they ruin your weekend. We'll cover when traditional automation works, when AI adds value, and how to progressively mature your incident response from manual firefighting to intelligent self-healing systems.

The foundation is understanding events themselves.

**[Full article >>](/kubernetes/self-healing-kubernetes-when-to-use-ai-vs-traditional-automation)**

--- -->

<a href="/ai/mcp-server-deployment-guide-from-local-to-production"><img src="/ai/mcp-server-deployment-guide-from-local-to-production/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [MCP Server Deployment Guide: From Local To Production](/ai/mcp-server-deployment-guide-from-local-to-production)

Everyone's using MCP servers these days. They're connecting AI agents to databases, Kubernetes clusters, GitHub repos, cloud resources, you name it. MCP is becoming the standard way to give AI tools access to external systems. But here's the question: how should you actually run these MCP servers?

The documentation typically shows you one way: running it locally with NPX. But is that secure? Is it scalable? Can your team share it? What about production? There are actually multiple deployment options, each with different trade-offs.

So here's what we're going to do: I'll show you **four different ways** to deploy MCP servers, from the dead simple to the enterprise-ready. We'll look at local execution with NPX, Docker containers, Kubernetes deployments, and operator-managed resources. Plus, I'll cover a few notable cloud platforms like Fly.io, Cloudflare Workers, and AWS Lambda. For each approach, I'll show you exactly how it works, what problems it solves, and what new problems it creates.

**[Full article >>](/ai/mcp-server-deployment-guide-from-local-to-production)**

---

<a href="/internal-developer-platforms/why-your-infrastructure-ai-sucks-and-how-to-fix-it"><img src="/internal-developer-platforms/why-your-infrastructure-ai-sucks-and-how-to-fix-it/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Why Your Infrastructure AI Sucks (And How to Fix It)](/internal-developer-platforms/why-your-infrastructure-ai-sucks-and-how-to-fix-it)

Here's the harsh reality: your AI agent is **completely useless** for infrastructure management, and you probably don't even realize it yet.

You've probably tried throwing ChatGPT or Claude at your DevOps problems, thinking AI will magically solve your infrastructure challenges. Maybe you got some generic responses that looked helpful on the surface. But when you actually tried to implement those suggestions, you discovered the painful truth - the AI has no clue about your environment, your standards, or your constraints.

Most organizations are making the same critical mistake: they're treating AI like a search engine instead of building it into their platform properly. They ask vague questions, get generic answers, and wonder why their "AI transformation" isn't working.

**[Full article >>](/internal-developer-platforms/why-your-infrastructure-ai-sucks-and-how-to-fix-it)**

---

<a href="/kubernetes/kubernetes-controllers-deep-dive-how-they-really-work"><img src="/kubernetes/kubernetes-controllers-deep-dive-how-they-really-work/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Kubernetes Controllers Deep Dive: How They Really Work](/kubernetes/kubernetes-controllers-deep-dive-how-they-really-work)

Here's something that might surprise you: **most people using Kubernetes don't actually understand how it works**. They know how to write YAML files and run `kubectl apply`, but when things go wrong - and they always do - they're completely lost. Why? Because they don't understand controllers.

Controllers are the beating heart of Kubernetes. They're what make your pods automatically restart when they crash, what scale your applications up and down, and what make custom resources feel like native parts of the platform. Without understanding controllers, you're just throwing YAML at the wall and hoping it sticks.

In this video, we're going deep into how controllers actually work - not just the basic concept, but the real mechanics. We'll explore how they consume and emit events, how they coordinate with each other, and why understanding this will make you infinitely better at building and debugging Kubernetes systems. Whether you're just using Kubernetes or building your own controllers, this knowledge will transform how you think about the platform.

**[Full article >>](/kubernetes/kubernetes-controllers-deep-dive-how-they-really-work)**

---

<a href="/development/how-i-tamed-chaotic-ai-coding-with-simple-workflow-commands"><img src="/development/how-i-tamed-chaotic-ai-coding-with-simple-workflow-commands/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [How I Tamed Chaotic AI Coding with Simple Workflow Commands](/development/how-i-tamed-chaotic-ai-coding-with-simple-workflow-commands)

You know what's absolutely maddening about working with AI coding agents? They're brilliant one moment, then completely fucking chaotic the next. They'll write perfect code for your database layer, then completely forget about it three prompts later when you're working on the API. They jump from task to task like rabbits on crack, losing context and making decisions that seem smart in isolation but are completely insane when you look at the bigger picture.

Here's the thing: if you're like most developers, you've probably tried to wrangle AI agents with existing tools, and you've probably been disappointed. Maybe you've used Jira or Linear to track what the AI should work on, or tried some of the newer AI-specific task management tools. The problem is that most of these tools either don't work well with AI agents, or the ones that do provide workflows that just don't fit how I actually think about building software.

**[Full article >>](/development/how-i-tamed-chaotic-ai-coding-with-simple-workflow-commands)**

---

<a href="/ai/teaching-ai-your-company-policies-vector-search-enforcement"><img src="/ai/teaching-ai-your-company-policies-vector-search-enforcement/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Teaching AI Your Company Policies: Vector Search + Enforcement](/ai/teaching-ai-your-company-policies-vector-search-enforcement)

Let's start with a fundamental question that most people think they know the answer to, but really don't. **What are policies?** You probably think you know. Hell, you might even have dozens of them implemented in your clusters right now. But here's the thing: most of what you call policies aren't actually policies at all.

A real policy is a business rule, a guideline, a principle. It's "Never use `latest` as an image tag because it makes rollbacks impossible and debugging a nightmare." It's "Databases in Google Cloud must always run in the `us-east1` region, those in Azure must run in `eastus`, and AWS databases go into `us-east-1`." These are policies. They're the rules we've established about how things should be done, why they should be done that way, and what happens when they're not.

**[Full article >>](/ai/teaching-ai-your-company-policies-vector-search-enforcement)**

---

<a href="/ai/terminal-agents-codex-vs-crush-vs-opencode-vs-cursor-cli-vs-claude-code"><img src="/ai/terminal-agents-codex-vs-crush-vs-opencode-vs-cursor-cli-vs-claude-code/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Terminal Agents: Codex vs. Crush vs. OpenCode vs. Cursor CLI vs. Claude Code](/ai/terminal-agents-codex-vs-crush-vs-opencode-vs-cursor-cli-vs-claude-code)

I love Claude Code. But I hate being locked into Anthropic models. What if I want to use GPT5? Or Llama? Or whatever comes out next week? So I went on a quest to find a good terminal-based coding agent that actually works with different models.

The perfect test case? GPT5. Everyone's hyping it as the best coding model ever created. If a terminal agent claims to be model-agnostic, it should work brilliantly with GPT5, right? So I tested every terminal-based agent I could find: Codex from OpenAI themselves, Charm Crush, OpenCode, Cursor CLI.

What I discovered might surprise you. By the end of this video, you'll understand exactly what's wrong with today's terminal agents, why model flexibility is harder than it seems, and whether any of these tools can actually compete with Claude Code. Let's find out.

**[Full article >>](/ai/terminal-agents-codex-vs-crush-vs-opencode-vs-cursor-cli-vs-claude-code)**

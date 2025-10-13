+++
archetype = "home"
title = ""
+++

# Latest Posts

<!-- <a href="/internal-developer-platforms/why-your-infrastructure-ai-sucks-and-how-to-fix-it"><img src="/internal-developer-platforms/why-your-infrastructure-ai-sucks-and-how-to-fix-it/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Why Your Infrastructure AI Sucks (And How to Fix It)](/internal-developer-platforms/why-your-infrastructure-ai-sucks-and-how-to-fix-it)

Here's the harsh reality: your AI agent is **completely useless** for infrastructure management, and you probably don't even realize it yet.

You've probably tried throwing ChatGPT or Claude at your DevOps problems, thinking AI will magically solve your infrastructure challenges. Maybe you got some generic responses that looked helpful on the surface. But when you actually tried to implement those suggestions, you discovered the painful truth - the AI has no clue about your environment, your standards, or your constraints.

Most organizations are making the same critical mistake: they're treating AI like a search engine instead of building it into their platform properly. They ask vague questions, get generic answers, and wonder why their "AI transformation" isn't working.

**[Full article >>](/internal-developer-platforms/why-your-infrastructure-ai-sucks-and-how-to-fix-it)**

--- -->

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

---

<a href="/kubernetes/why-kubernetes-discovery-sucks-for-ai-and-how-vector-dbs-fix-it"><img src="/kubernetes/why-kubernetes-discovery-sucks-for-ai-and-how-vector-dbs-fix-it/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Why Kubernetes Discovery Sucks for AI (And How Vector DBs Fix It)](/kubernetes/why-kubernetes-discovery-sucks-for-ai-and-how-vector-dbs-fix-it)

Kubernetes API is **brilliant for execution**. Once you know exactly which resources to deploy and how they work together, it handles everything flawlessly. Controllers reconcile state, resources get created, and your applications and infrastructure run perfectly.

But it **sucks big time for discovery**. When you need to figure out what's actually possible in your cluster, the API becomes a complete nightmare. You're stuck sifting through hundreds of cryptically named resource types, hoping to stumble across something that might solve your problem.

Even AI struggles with this. Ask it to "create a PostgreSQL database with schema management in AWS," and it either gives you a complex multi-resource approach or completely misses the simple solution that actually exists. The problem isn't missing information. **The problem is that the information is impossible to find**.

Today, I'll show you how **semantic search with vector databases** can finally solve this discovery nightmare. By the end of this video, you'll see how AI can instantly find the perfect resources using natural language, even when the exact keywords don't match.

**[Full article >>](/kubernetes/why-kubernetes-discovery-sucks-for-ai-and-how-vector-dbs-fix-it)**

---

<a href="/ai/stop-blaming-ai-vector-dbs-rag-game-changer"><img src="/ai/stop-blaming-ai-vector-dbs-rag-game-changer/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Stop Blaming AI: Vector DBs + RAG = Game Changer](/ai/stop-blaming-ai-vector-dbs-rag-game-changer)

Let me guess. You tried AI, it hallucinated something completely wrong, and now you're back to doing everything manually while complaining that "AI doesn't work."

Maybe you're a developer who asked it about your codebase, and it confidently explained functions that don't exist. Or suggested using deprecated APIs your team abandoned two years ago. Perhaps it recommended architectural patterns that directly contradict your team's decisions.

Or you're in ops, and it gets even worse. You asked about your backup policies, and it invented procedures you've never implemented. You requested help with a Kubernetes deployment, and it suggested configurations that violate every security standard you have. You wanted it to troubleshoot a production issue, and it gave you generic advice that would take down your entire cluster.

**[Full article >>](/ai/stop-blaming-ai-vector-dbs-rag-game-changer)**

+++
archetype = "home"
title = ""
+++

# Latest Posts

<!-- <a href="/kubernetes/why-kubernetes-querying-is-broken-and-how-i-fixed-it"><img src="/kubernetes/why-kubernetes-querying-is-broken-and-how-i-fixed-it/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Why Kubernetes Querying Is Broken and How I Fixed It](/kubernetes/why-kubernetes-querying-is-broken-and-how-i-fixed-it)

**`kubectl get all` is a lie.** It doesn't get all. It gets maybe 10% of what's actually in your cluster. And if you want the other 90%? You're writing bash loops, waiting forever, and still missing resources because you don't even know what to search for.

This video is about fixing Kubernetes' terrible querying story. The root cause is etcd. It's a key-value store, not a database. It was never designed to answer questions like "show me all databases" or "what's running in this namespace."

I'll show you how to sync Kubernetes metadata into a Vector database, enabling both traditional queries and semantic search. By the end, you'll be able to run proper database queries AND ask questions in plain English, and get answers that actually make sense. **No more grep pipelines. No more guessing resource names.**

**[Full article >>](/kubernetes/why-kubernetes-querying-is-broken-and-how-i-fixed-it)**

--- -->

<a href="/ai/elevenlabs-api-review-a-developers-brutally-honest-take"><img src="/ai/elevenlabs-api-review-a-developers-brutally-honest-take/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [ElevenLabs API Review: A Developer's Brutally Honest Take](/ai/elevenlabs-api-review-a-developers-brutally-honest-take)

I'm a software engineer and I got addicted to AI.

I use it to write code, to operate clusters, to analyze test failures, to pick which pull request I should work on. But here's the thing. I noticed there's so much more we can do with AI. We can use it for almost anything.

Today I want to explore multimedia from a software engineering perspective. Not how to click buttons in some fancy UI. I want to show you how to integrate audio and video APIs into your workflows and applications. And I'll use my own YouTube automation as the example.

That sounds just about right for this channel. I guess I deal with too many languages and sometimes get confused. Speaking of which, that transition you just heard? That wasn't me speaking Spanish or French. That was AI. And by the end of this video, you'll know exactly how to build that yourself.

**[Full article >>](/ai/elevenlabs-api-review-a-developers-brutally-honest-take)**

---

<a href="/observability/ai-agent-debugging-setup-opentelemetry-jaeger-in-kubernetes"><img src="/observability/ai-agent-debugging-setup-opentelemetry-jaeger-in-kubernetes/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [AI Agent Debugging Setup: OpenTelemetry + Jaeger in Kubernetes](/observability/ai-agent-debugging-setup-opentelemetry-jaeger-in-kubernetes)

I sent two requests to the same AI agent, the same endpoint, doing similar work. One took 10 seconds with 10 operations. The other took over a minute with 42 operations. Same agent. **Completely different behavior.**

That's the problem with agentic systems. The LLM decides the path. It chooses which tools to call, how many times to loop, what data to fetch. Every request can fan out differently. You can't predict it. And if you can't see what's happening, you can't debug it, optimize it, or trust it.

The good news? We already solved this problem for distributed systems. OpenTelemetry tracing. And it works for AI agents too.

**[Full article >>](/observability/ai-agent-debugging-setup-opentelemetry-jaeger-in-kubernetes)**

---

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

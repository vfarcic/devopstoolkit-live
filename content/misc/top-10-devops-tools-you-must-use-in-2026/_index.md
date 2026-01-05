
+++
title = 'Top 10 DevOps Tools You MUST Use in 2026'
date = 2025-01-05T16:00:00+00:00
draft = false
+++

2025 was the year agentic AI went from interesting experiment to daily reality. AI agents stopped being autocomplete tools and started understanding entire codebases, refactoring across files, writing tests, debugging their own mistakes, managing infrastructure, and handling operational tasks. For many developers and ops engineers, the way they work fundamentally changed.

2026 is the year to take this seriously. Not just for application developers, but for DevOps engineers, SREs, and platform teams. The tools are mature enough now. The productivity gains are real. If you're not integrating AI agents into your workflow, you're leaving significant value on the table.

But here's the thing: agentic AI doesn't replace everything else. You still need solid foundations. Internal developer platforms, testing frameworks, scripting languages, development environments. These tools still matter. What's changed is that AI now intersects with all of them.

So this year's recommendations cover both: the AI tools that emerged in 2025 and the non-AI tools that remain essential. I spent 2025 testing all of them in real projects, real workflows, real problems. Not quick demos.

<!--more-->

{{< youtube 65o_j4E7_lk >}}

What I'm sharing today are my recommendations for 2026. Not a comprehensive list. Not a neutral comparison. These are the tools I actually use, the ones that survived months of daily work, and the ones I think you should seriously consider. I'll also point out what didn't make the cut and why.

Some of my choices will be obvious. Others might surprise you. A few will probably make you disagree. That's fine. The goal isn't to tell you what to think. It's to give you a practitioner's perspective so you can make better decisions for your own stack.

## Best AI Models for Software Engineering

The AI model landscape is evolving at a pace that makes any definitive ranking obsolete within months. The model that leads benchmarks today will likely be surpassed tomorrow. New releases drop constantly, capabilities leap forward, and pricing changes overnight. Any recommendation here comes with a built-in expiration date.

That said, some patterns are emerging. Certain model families consistently deliver top results for software engineering tasks: understanding code, generating accurate implementations, reasoning through complex debugging scenarios, and working with configurations and manifests. The leaders in coding benchmarks tend to stay near the top even as new models appear, suggesting that some providers have figured out what matters for engineering work.

For open-source options you can run locally or self-hosted on your own servers, [Llama 4](https://llama.meta.com/) offers a massive 10M token context window, [Qwen](https://github.com/QwenLM/Qwen) delivers strong coding performance with an Apache 2.0 license, and [DeepSeek](https://deepseek.com/) provides impressive capabilities at a fraction of the price. These models are good and getting better, but they're still not quite on par with closed models. They might be close, but not there yet. The other challenge is compute requirements. You might not have the hardware needed today, and those requirements are likely to increase as models grow larger.

Among the proprietary alternatives, [OpenAI GPT](https://openai.com/) remains widely used with the largest ecosystem. [Mistral](https://mistral.ai/) offers a strong European alternative with frontier performance at reduced costs. [Cohere](https://cohere.com/) focuses on enterprise RAG use cases. [xAI Grok](https://x.ai/) brings real-time data access.

**[Anthropic](https://anthropic.com/)** models are my choice for software engineering work. Claude consistently leads SWE-bench and other coding benchmarks, often by a meaningful margin. This isn't accidental. Anthropic has made software engineering a core focus in a way that other providers haven't. Their first AI conference was entirely dedicated to coding and developers. Claude Code is the best terminal-based agent available. Cursor, the leading AI-native IDE, uses Claude as its default model.

The results speak for themselves. Claude understands codebases deeply, generates accurate implementations, and reasons through complex debugging scenarios better than alternatives. For infrastructure-as-code, Kubernetes manifests, and configuration files, the difference is noticeable. The model grasps context and produces code that works on the first try more often than competitors. Claude Sonnet 4 now supports a 1M token context window, matching Gemini's previous advantage in this area.

The trade-off is cost. Claude's API pricing is higher than Gemini's, and rate limits on the consumer plans can be frustrating for heavy users. But for professional software engineering work, the quality difference justifies the price.

[Google Gemini](https://deepmind.google/technologies/gemini/) is a close second. Cost-effectiveness is excellent, with a generous free tier and competitive API pricing. Multimodal capabilities are strong if you need to work with images, diagrams, or documentation that includes visuals.

Gemini can feel less refined than Claude for some coding tasks. The output quality is good but not quite at Claude's level for complex software engineering work. That said, the gap has been narrowing, and for many use cases Gemini delivers excellent results at lower cost. If budget is a primary concern, Gemini is a solid choice.

## Best AI Coding Agents

We are witnessing a seismic shift in how software engineers work. AI agents have moved from novelty to necessity for many application developers. They're not just autocompleting lines anymore. They're understanding entire codebases, refactoring across multiple files, writing tests, and even debugging their own mistakes. The productivity gains are real and substantial.

However, this shift hasn't fully reached the ops world yet.

DevOps engineers, SREs, and platform teams are still largely working the way they did before AI agents became mainstream. Part of this is the nature of the work. Ops tasks often involve production systems where mistakes have immediate consequences, complex debugging across distributed systems, and tribal knowledge about why things are configured a certain way. AI agents struggle with context that lives in runbooks, Slack threads, and people's heads.

That said, 2026 is likely the year this changes. The tools are maturing rapidly, context windows are expanding, and agents are getting better at understanding infrastructure-as-code, Kubernetes manifests, and cloud configurations. If you're in ops and haven't started experimenting with AI agents, now is the time to get familiar. The learning curve exists, and you don't want to be climbing it when everyone else has already integrated these tools into their workflows.

Two directions are emerging in this space. The first follows traditional development patterns through IDEs, where AI augments the familiar editing experience with inline completions, chat panels, and context-aware suggestions. The second is terminal-based agents that take a different approach, often operating more autonomously and integrating better with command-line workflows that ops teams already use.

Another key distinction is model flexibility. Some agents are tightly coupled to specific models, which can mean better optimization for that model's strengths but also vendor lock-in. Others are model-agnostic, letting you swap providers or use local models at the cost of potentially less optimization.

In the IDE camp, [GitHub Copilot](https://github.com/features/copilot) remains the most widely adopted with seamless multi-IDE integration and reliable completions, though it struggles with large-scale refactoring. [Windsurf](https://codeium.com/windsurf) offers a great beginner experience with unlimited agent access and planning mode for multi-step tasks. Open-source IDE options like [Continue](https://continue.dev/) and [Cline](https://github.com/cline/cline) provide model-agnostic alternatives but lack the polish of commercial offerings.

For terminal-based agents tied to specific models, [Gemini CLI](https://github.com/google-gemini/gemini-cli) brings Google's massive 1M token context window to the command line with a generous free tier, though benchmark scores lag behind competitors. [OpenAI Codex](https://openai.com/index/openai-codex/) offers flexible reasoning levels and strong GitHub integration but has UX challenges.

Cloud-specific options like [Amazon Q Developer](https://aws.amazon.com/q/developer/) excel within their ecosystems but offer limited value outside them.

My recommendations come down to three tools that cover different use cases. I prefer terminal-based agents, but some people prefer working in IDEs. No judgment on that one, at least not today. Within the terminal camp, some want the best experience regardless of vendor lock-in, while others prioritize model flexibility. All three are valid choices depending on your priorities.

For IDE users, **[Cursor](https://cursor.com/)** is the clear winner. It's a VS Code fork with AI deeply integrated into the editing experience. You get inline completions, chat, and precise context control. The GUI-first experience is polished and fast. The downsides are rate limits that can be frustrating for heavy users and pricing changes that have upset some of the community. But if you live in your IDE and want AI assistance without leaving it, Cursor is the best option available.

For terminal users who want the best experience, **[Claude Code](https://claude.ai/code)** is in a league of its own. It has the highest SWE-bench scores, a true 200K token context window, and excels at autonomous multi-file operations. It understands entire codebases and can work through complex tasks with minimal hand-holding. The catch is that it only works with Anthropic models. If you're comfortable with that lock-in, Claude Code delivers results that other terminal agents can't match. I use it daily.

For terminal users who want model flexibility, **[OpenCode](https://github.com/opencode-ai/opencode)** is the best choice. It's a true open-source Claude Code alternative that works with any model provider. It's still behind Claude Code in capabilities and has a smaller community, but it's the right choice if you want to avoid being locked to a specific family of models. As the model landscape continues to shift, having the freedom to switch providers without changing your tooling has real value.

## Building Custom AI Agents

We've entered a phase where companies need to build their own AI agents. Off-the-shelf agents are great for general-purpose tasks, but every organization has unique workflows, internal tools, and domain knowledge that generic agents can't tap into. This is especially true for Internal Developer Platforms.

If application developers are increasingly using AI agents as their primary interface for getting work done, then platform teams need to expose platform capabilities to those agents. Otherwise, developers will be constantly context-switching between their AI assistant and the platform portal, which defeats the purpose of both.

The way to bridge this gap is through Model Context Protocol (MCP) servers and custom agents. MCP allows AI agents to discover and use tools exposed by your platform. Custom agents can encode your organization's specific workflows, policies, and tribal knowledge. Together, they enable scenarios like developers asking their AI agent to provision a new environment, check deployment status, or investigate a production incident, all without leaving their normal workflow.

Building custom agents requires SDKs and frameworks. The landscape here is still maturing, with options ranging from low-level SDKs that give you maximum control to high-level frameworks that handle orchestration, memory, and tool management for you. The right choice depends on how much complexity you want to manage yourself versus how much you want abstracted away.

Given how fast the model space is moving, I believe we should be building agents in a way that allows us to switch models at any time. That makes SDKs tied to a specific vendor a bad choice, even if they offer features that might not be available elsewhere. Until clear long-term winners in the model space emerge, our agents must be agnostic.

This rules out vendor-specific options like the Anthropic SDK, Google ADK, and OpenAI Agents SDK as primary choices, despite their polish and tight integration with their respective models. Microsoft's [Semantic Kernel](https://github.com/microsoft/semantic-kernel) and [AutoGen](https://github.com/microsoft/autogen) offer more flexibility but still lean heavily into the Azure ecosystem.

For multi-agent orchestration, [CrewAI](https://crewai.com/) has gained traction with its intuitive role-based approach. [LangGraph](https://github.com/langchain-ai/langgraph) extends LangChain with graph-based architecture offering low latency and time-travel debugging. For simpler use cases with type safety, [PydanticAI](https://github.com/pydantic/pydantic-ai) offers a FastAPI-like experience.

If you're building for Kubernetes environments specifically, [Kagent](https://kagent.dev/) is the first open-source agentic AI framework designed for Kubernetes, now a CNCF project with built-in integrations for Argo, Helm, Istio, and Prometheus. However, I think Kagent has a lot left to be desired. It's useful for those wanting to create agents quickly, but not for custom agents that can be taken seriously. There's more to agents than a way to define a system prompt and connect it to MCPs. [kmcp](https://github.com/kagent-dev/kmcp) is more interesting as it helps you build, test, and deploy MCP servers to Kubernetes with proper lifecycle management through CRDs.

For building custom agents, **[Vercel AI SDK](https://ai-sdk.dev/)** stands out as the best choice. The primary reason is model agnosticism. It supports dozens of providers (OpenAI, Anthropic, Google, Cohere, DeepSeek, and more) through a unified API. You can swap providers without changing your code, which aligns with the principle that our agents must not be locked to specific models.

Beyond provider flexibility, Vercel AI SDK offers simplicity that other frameworks lack. Where LangChain requires instantiating objects and managing abstractions, Vercel SDK uses simple function calls with less boilerplate. The learning curve is gentler. Streaming is built-in with React hooks like `useChat` and `useCompletion` that make real-time UIs trivial to implement. It works across frameworks including Next.js, React, Svelte, Vue, Nuxt, and Node.js.

The SDK is actively maintained with AI SDK 5 adding agentic loop control, type-safe chat, and tool enhancements. The documentation is excellent. And if you need LangChain's complex orchestration for specific use cases, Vercel SDK integrates with it, giving you the best of both worlds.

The downside is that it's TypeScript-first. If you're not familiar with TypeScript, you'll need to either learn it or look for an alternative. That said, if you're an experienced developer, picking up a new language shouldn't be a significant barrier.

## AI Code Review Tools

Despite the name, this category isn't just about reviewing application code. It covers AI-powered review of anything you push to Git: Kubernetes manifests, Terraform configurations, Helm charts, CI/CD pipelines, shell scripts, documentation. If it lives in a repository and goes through pull requests, these tools can review it.

AI code review is a no-brainer adoption. The time investment is close to zero since you just enable it on your repositories and it starts working. It's not obtrusive either. While these tools can often suggest fixes you can apply with one click, their main focus is providing recommendations. You can take them into account or dismiss them. There's no forced workflow change, no new tool to learn, no context switching. The AI reviews your PR in parallel with human reviewers, and you decide what feedback is valuable.

For ops teams, this is particularly useful. Misconfigurations in Kubernetes manifests, security issues in Terraform, missing best practices in Helm charts: these are exactly the kinds of issues that slip through human review because reviewers are focused on the logic, not the YAML structure. AI reviewers don't get tired and they don't skip the boring parts.

The options range from simpler tools like [Sourcery](https://sourcery.ai/) and [CodeAnt AI](https://codeant.ai/) that handle basic reviews, to more sophisticated solutions. [Traycer](https://traycer.ai/) identifies edge cases and performance issues but requires a paid plan after the trial. [Greptile](https://greptile.com/) builds a full semantic graph of your repo for cross-file bug detection with SOC2 compliance, though it can be noisy. [Cubic.dev](https://cubic.dev/) focuses on speed and learns from your feedback but is GitHub-only. [Qodo Merge](https://qodo.ai/) (formerly PR-Agent) is open-source and benchmarks as the fastest and most thorough, with RAG-powered searches across repos.

**[CodeRabbit](https://coderabbit.ai/)** is my choice for AI code reviews. The primary reason is ease of adoption. Setup takes minutes with minimal configuration. It works across GitHub, GitLab, Azure DevOps, and Bitbucket. The reviews provide line-by-line feedback that resembles what you'd get from a senior developer, not just high-level summaries. It learns from your interactions over time, adapting to your codebase and team preferences.

What sets CodeRabbit apart is its MCP server integration. You can connect it to Claude Code, Cursor, or other MCP-compatible agents. This means you can write code in your agent, create a PR, get CodeRabbit's review, then ask your agent to fetch those review comments and implement the fixes, all without leaving your workflow. The loop between writing code and addressing review feedback stays within a single agent session. For teams already using AI agents for development, this integration is significant.

Qodo Merge is a solid alternative, especially if you need self-hosting for strict security requirements or want open-source transparency. It uses RAG to search across repositories for context. But CodeRabbit's ease of setup, MCP integrations, and broad platform support make it the better default choice for most teams. The pricing is reasonable with a free tier for basics and paid tiers for more features.

## Vector Databases for AI

The database landscape has shifted. Beyond traditional SQL and NoSQL databases, there's now a pressing need to provide data to AI models through agents. Due to context limitations, you can't just dump all your data into an LLM and hope for the best. You need to find the data that matters for each specific query, which means semantic search through embeddings.

This is where vector databases come in. They store embeddings, numerical representations of text, code, images, or any other content, and let you find semantically similar items quickly. When a developer asks an AI agent about a production incident, the agent can search your runbooks, past incidents, and documentation using semantic similarity rather than keyword matching. The results are dramatically better.

The market has responded in two ways. Dedicated vector databases have emerged, purpose-built for storing and querying embeddings at scale. At the same time, existing databases are adding vector capabilities so you don't have to manage another system. If your data already lives in PostgreSQL, adding pgvector might make more sense than migrating to a dedicated solution.

If you want to add vector capabilities to databases you already run, [pgvector](https://github.com/pgvector/pgvector) extends PostgreSQL, [Redis Vector](https://redis.io/docs/latest/develop/get-started/vector-database/) adds vector search to Redis with sub-millisecond latency, and [Cloudflare Vectorize](https://developers.cloudflare.com/vectorize/) provides edge-native serverless vectors if you're in the Cloudflare ecosystem. These work well for smaller scale or when you want to avoid managing another database.

For dedicated vector databases, [Chroma](https://trychroma.com/) offers the best developer experience for rapid prototyping but is limited to smaller datasets. [Milvus](https://milvus.io/) scales to billions of vectors but requires engineering expertise. [Weaviate](https://weaviate.io/) was first to market with a deep feature set including hybrid search and multi-modal support. [Pinecone](https://pinecone.io/) is the easiest fully managed option with zero ops and strong compliance certifications, but costs can grow quickly at scale.

**[Qdrant](https://qdrant.tech/)** is my choice for vector database work. It hits the right balance between performance, features, and cost. Written in Rust, it delivers excellent query speeds with minimal latency. The filtering capabilities are what set it apart. You can filter on payload values before the vector search happens, not after. This means queries like "find similar documents from the last 30 days" or "find similar incidents in the production environment" are fast regardless of how many vectors you have.

The open-source model matters here. You can run Qdrant locally for development, self-host it in your own infrastructure, or use Qdrant Cloud if you want a managed option. This flexibility is important for platform teams who need to keep data in-house or control costs at scale. Pinecone is easier to get started with, but the costs grow quickly as your data grows. Qdrant is significantly cheaper at scale while delivering comparable performance.

The trade-offs exist. Qdrant has a steeper learning curve than simply adding pgvector to your existing PostgreSQL. If you only need basic vector search on a small dataset, pgvector is simpler.

Weaviate is better if you need sophisticated hybrid search combining text and vector queries. But for most AI agent use cases where you need fast, filtered semantic search at reasonable cost, Qdrant is the best choice.

## Internal Developer Platforms

Internal Developer Platform (IDP) is one of the most misunderstood concepts in our industry. Too many people equate IDP with a portal, a fancy web UI where developers click buttons. That's not a platform; that's just a frontend. To understand what a platform really is, look at public cloud providers like AWS, Azure, or Google Cloud. They follow a clear pattern: services that do something (EC2 spins up VMs, S3 stores objects, RDS manages databases), APIs that expose those services, and user interfaces that consume those APIs (web console, CLI, SDKs, Terraform). The UI is just one of many ways to interact with the platform. It's not the platform itself.

An IDP must follow the same pattern. You need services that actually do things like provision infrastructure, deploy applications, manage secrets, and enforce policies. Those services must be exposed through APIs. Then you can build whatever user interfaces make sense, whether that's a portal, a CLI, GitOps workflows, or all of the above. If you start with the portal, you're building a house starting with the roof.

So where do you run these services? Kubernetes controllers are the most logical choice. They're designed to reconcile desired state with actual state, exactly what platform services need to do. How do you expose APIs? Kubernetes Custom Resource Definitions (CRDs) give you a declarative API for free. How do you interact with those APIs? Any way you already interact with Kubernetes: kubectl, Helm, GitOps tools like Argo CD or Flux, dashboards, or custom web UIs. The portal becomes just another client, not the center of the universe.

Many options in this space offer partial solutions or are built on what I consider obsolete foundations. If you only need a developer portal, [Roadie](https://roadie.io/), [Port](https://getport.io/), and [Cortex](https://cortex.io/) can work, but a portal alone is not a platform. Commercial all-in-one solutions like [Harness](https://harness.io/), [Mia-Platform](https://mia-platform.eu/), and [Qovery](https://qovery.com/) bundle various capabilities together, but often with proprietary architectures that don't align with how modern platforms should be built. [Humanitec](https://humanitec.com/) and [Northflank](https://northflank.com/) get closer to true platform orchestration, but still come with vendor lock-in concerns.

If you're building a platform in 2026, you should be building it on Kubernetes with Kubernetes-native components from the CNCF ecosystem. Services should be controllers, APIs should be CRDs, and the entire stack should follow the patterns that Kubernetes established. Anything else is either a partial solution or a step backward.

The **[BACK Stack](https://github.com/back-stack)** (Backstage, Argo CD, Crossplane, Kyverno) is my choice for building Internal Developer Platforms. Each component has established itself as the leader in its domain.

[Backstage](https://backstage.io/) is probably the only widely adopted portal solution, with massive contributions from thousands of organizations. It provides the developer-facing UI layer. [Argo CD](https://argo-cd.readthedocs.io/) (alongside Flux) is the de-facto standard for GitOps, handling continuous delivery and keeping cluster state synchronized with Git repositories. [Crossplane](https://crossplane.io/) is the most mature and widely adopted solution for building platform services as Kubernetes controllers with APIs exposed through CRDs. [Kyverno](https://kyverno.io/) has established itself as the standard for defining and enforcing policies across Kubernetes resources.

All four projects are open source and owned by CNCF. Argo and Crossplane are graduated projects, while Backstage and Kyverno are incubating and on track for graduation. They are mature, widely adopted, and well maintained. The ecosystem integration between them is strong, with Crossplane providers spanning cloud platforms, databases, and SaaS applications, all manageable through Backstage portals with Kyverno enforcing security policies.

The only significant piece missing from the BACK Stack for a complete IDP is workflows (CI pipelines). But that space is already covered by a plethora of tools that have existed for a long time, and they all do more or less the same thing. Whether you choose GitHub Actions, GitLab CI, Jenkins, Tekton, or any other CI tool matters less than getting the platform foundations right.

The investment required to build a BACK Stack IDP is real, but less daunting than it might seem. Most companies running Kubernetes are already using Argo CD for deployments and Kyverno for policies. Extending their usage beyond current workloads is easier with Crossplane than with non-Kubernetes-native tools since the patterns and workflows are already familiar. And when it comes to developer portals, there's no real alternative to Backstage.

With the BACK Stack, you get full control, no vendor lock-in, and a platform built on the same patterns as public clouds.

## Kubernetes Development Environments

The gap between local development and production Kubernetes environments has always been painful. You can run a local cluster with kind or minikube, but that doesn't help when your service needs to talk to dozens of other services, databases, message queues, and external APIs that only exist in a shared environment. You end up mocking everything, which means you're not actually testing against real dependencies. Or you deploy to a shared dev cluster for every change, which is slow and creates conflicts with other developers.

This category covers tools that bridge that gap. The approaches vary: some create VPN tunnels to remote clusters, some intercept traffic and route it to your local machine, some spin up isolated virtual clusters, and some automate the build-deploy-test cycle. The goal is the same: let developers work locally with the speed and convenience of their own machine while still interacting with real services in a real cluster.

For platform teams, this is an important piece of the developer experience puzzle. If developers can't easily test their changes against a realistic environment, they'll either skip testing (leading to production issues) or demand expensive dedicated environments (leading to infrastructure sprawl). The right tooling here pays for itself quickly.

The oldest approach is VPN-based tunneling. [Telepresence](https://telepresence.io/) pioneered this space but comes with finicky setup, compatibility issues with service meshes and corporate VPNs, and requires root access. [Gefyra](https://gefyra.dev/) was born from Telepresence frustration, offering a simpler Docker-based approach that doesn't modify running workloads.

For automating the build-deploy-test cycle, [Skaffold](https://skaffold.dev/) brings Google-backed maturity with declarative YAML configuration. [Tilt](https://tilt.dev/) offers a browser UI showing build status and logs with Starlark configuration for flexibility. [DevSpace](https://devspace.sh/) provides file sync, port forwarding, and dev containers across all major clouds.

At a higher level, [Signadot](https://signadot.com/) creates intelligent sandboxes integrated with CI/CD for PRs. [vcluster](https://vcluster.com/) takes a different approach by spinning up virtual Kubernetes clusters with faster provisioning and stronger isolation than namespaces.

**[mirrord](https://mirrord.dev/)** is my choice for bridging local development with remote Kubernetes environments. The key difference from Telepresence and similar tools is that mirrord works at the process level rather than the network level. Instead of creating a VPN tunnel to your cluster, mirrord intercepts your local process's system calls and proxies them to a temporary agent running in your cluster.

This approach has several advantages. No cluster installation is required. mirrord uses the Kubernetes API directly, so all you need is a configured kubeconfig. It creates a temporary pod when it runs and cleans up automatically when done. No operators, no daemons, no permanent changes to your cluster.

No root access is needed on your machine either. Unlike Telepresence which requires elevated privileges to create network tunnels, mirrord only affects the running process. The rest of your machine remains untouched. This makes it easier to adopt in corporate environments where developers don't have admin rights.

Traffic mirroring instead of interception is a big deal for shared environments. Telepresence intercepts traffic, meaning requests intended for the remote service get redirected to your local machine. This disrupts others using that environment. mirrord can mirror traffic instead, sending a copy to your local process while the original requests are handled normally by the remote service.

Environment configuration is automatic. mirrord proxies network access, file access, and environment variables uniformly. Your local process sees the same environment variables, can read the same files, and connects to the same services as the remote pod.

## Kubernetes Platform Testing

As the pattern of building platforms on Kubernetes with services exposed through CRDs becomes more prevalent, testing Kubernetes resources becomes critical. This isn't just about testing applications running on Kubernetes. It's about testing the platform itself: the controllers, the operators, the custom resources, the policies, and the integrations between them.

When you combine this with agentic AI that interacts with your platform through these APIs, the stakes get higher. An AI agent provisioning infrastructure or modifying configurations needs to work correctly every time. The APIs it calls need to behave as documented. The controllers behind those APIs need to reconcile state reliably. You can't manually verify this at scale. You need automated end-to-end tests that exercise the full lifecycle of your custom resources.

The tooling in this space has matured significantly. Declarative testing frameworks let you define expected states and assertions in YAML rather than writing test code. Conformance testing ensures your clusters meet Kubernetes standards. The best tools make it easy to turn a bug report into a regression test by simply copying manifests.

For basic Helm chart validation, [Helm test](https://helm.sh/docs/topics/chart_tests/) is built-in but limited to simple pass/fail checks. [Helm-unittest](https://github.com/helm-unittest/helm-unittest) adds BDD-style unit testing as a Helm plugin. [KUTTL](https://kuttl.dev/) was the original declarative Kubernetes test tool but development has slowed significantly.

For cluster conformance testing, [Sonobuoy](https://sonobuoy.io/) ensures your clusters meet Kubernetes standards with non-destructive diagnostics. The official [Kubernetes e2e-framework](https://github.com/kubernetes-sigs/e2e-framework) provides Go-based testing libraries with automatic cluster lifecycle management, though it requires Go knowledge.

**[Kyverno Chainsaw](https://kyverno.github.io/chainsaw/)** is my choice for testing Kubernetes platforms. It builds on the ideas from KUTTL and improves on them significantly. The core idea is declarative testing: you define tests in YAML rather than writing Go code or bash scripts.

This matters for platform teams. When you build a platform with custom controllers and CRDs, you need to test that resources reconcile correctly, that policies are enforced, and that the entire lifecycle works as expected. Writing this in Go means maintaining more test code than actual platform code. Chainsaw lets you define test cases by simply providing the manifests you want to apply and the expected state you want to verify.

The workflow for turning bug reports into regression tests is remarkably simple. Someone reports that a specific manifest causes unexpected behavior. You copy that manifest into a test case, add the expected outcome, and you have a regression test. No code to write.

Each test step is isolated, which makes CI debugging easier. When a test fails, you know exactly which step failed and can see the relevant logs without wading through a monolithic test output. The documentation is detailed and actively maintained with frequent releases.

Complex assertion logic might require scripting blocks rather than pure YAML, but that's a rare edge case. For most platform testing scenarios, declarative YAML definitions are sufficient and dramatically simpler than the alternatives.

## Shell Scripting

Every DevOps engineer faces the same dilemma: should I write this in Bash or use a "real" programming language like Python or Go? Bash is ubiquitous and perfect for quick glue scripts, but it falls apart when dealing with structured data, error handling, or anything beyond simple string manipulation. Python is powerful and readable, but suddenly you're managing virtual environments, dependencies, and wondering if the target system even has the right Python version. Go gives you static binaries but the overhead of writing, compiling, and maintaining compiled code for a simple automation task feels like overkill.

The truth is, modern DevOps work increasingly involves structured data; JSON from APIs, YAML configs, log parsing, cloud CLI outputs. Traditional shells force you into awkward pipelines of jq, awk, sed, and grep. Meanwhile, "real" languages require too much ceremony for what should be a 10-line script. This category explores shells that bridge that gap; offering the immediacy of shell scripting with modern language features like structured data handling, proper error messages, and sane syntax. This isn't a comprehensive list of all shells, just the ones I explored while looking for something better than Bash that doesn't require spinning up a full development environment.

After trying many solutions, I settled on **[Nushell](https://nushell.sh/)** for scripting. It delivers the best of both worlds: quick to write like Bash, but with proper data types and type checking like Go or TypeScript. The key difference is that Nushell treats data as structured tables, records, and lists rather than text streams. You can filter, sort, and transform JSON, YAML, CSV, Excel, and SQLite with the same commands. No more jq/awk/sed pipelines. `where status == "running"` is more intuitive than parsing text with regex.

PowerShell pioneered this structured data approach, and it deserves credit for that. But Nushell improves on the concept. It's faster (PowerShell has historically been slow), the syntax is cleaner and less verbose (no verb-noun conventions forcing awkward naming), and it's built for Unix-like systems first rather than treating them as second-class citizens. Nushell is written in Rust, so it's performant with no runtime dependencies. The error messages actually tell you what went wrong and how to fix it.

I should be clear: I don't use Nushell as my interactive shell. I still use Zsh for that because it's POSIX-compliant and works everywhere. I use Nushell exclusively for scripting, where its structured data handling shines. The fact that Nushell isn't commonly installed on servers isn't a problem for me because I use Nix through DevBox. Every project has a `devbox.json` that brings in all needed tools, including Nushell. That said, I wouldn't use Nushell for scripts that need to run directly on servers where I don't control the environment. I don't have many of those, so it's not an issue.

The trade-offs are real: no job control yet, still pre-v1.0 so syntax can change between versions, and you can't copy-paste Bash scripts. But for DevOps work involving structured data, Nushell has replaced both Bash and Python for most of my scripting needs. Nushell users may not be the majority yet, but it's too good to ignore.

## What to Use in 2026

Here are my recommendations for 2026.

For AI models, go with **Anthropic's Claude** for software engineering work. If budget is a concern, **Gemini** is a strong second choice.

For AI agents, pick based on your workflow. **Cursor** if you prefer IDEs. **Claude Code** if you work in the terminal and want the best experience. **OpenCode** if model flexibility matters more than polish.

For building custom agents, use **Vercel AI SDK**. Model agnosticism is critical when the landscape is shifting this fast.

For automated code reviews, adopt **CodeRabbit**. The MCP integration alone makes it worth it.

For vector databases, choose **Qdrant**. Best balance of performance, features, and cost.

For internal developer platforms, build on the **BACK Stack**. Kubernetes-native, no vendor lock-in, and all CNCF projects.

For Kubernetes development environments, use **mirrord**. It solves the local-to-remote gap without the pain of alternatives.

For platform testing, adopt **Kyverno Chainsaw**. Declarative testing that actually works.

For scripting, try **Nushell**. Structured data handling without the ceremony of "real" languages.

These tools survived real-world use in 2025. They're ready for 2026.

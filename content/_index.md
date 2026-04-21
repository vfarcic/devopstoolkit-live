+++
archetype = "home"
title = ""
+++

# Latest Posts

<a href="/development/ai-wont-kill-your-job-your-customers-using-ai-will"><img src="/development/ai-wont-kill-your-job-your-customers-using-ai-will/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [AI Won't Kill Your Job. Your Customers Using AI Will](/development/ai-wont-kill-your-job-your-customers-using-ai-will)

Over a trillion dollars in software market cap vanished in a single month. Software companies are laying off thousands. Analysts are calling it the SaaSpocalypse. And yet, software engineering jobs are at multi-year highs. What the hell is going on?

If you're a software engineer and you think your job is at risk because of AI, you're right. But probably not for the reasons you think. If you work at a company that *sells* software, yeah, you should be worried. But if you work at a bank, an insurance company, a retailer, a manufacturer, any company that *uses* software rather than selling it, AI might actually make your job more secure than ever. Your company is likely about to build a lot more software than it ever has before

**[Full article >>](/development/ai-wont-kill-your-job-your-customers-using-ai-will)**

---


<a href="/development/ai-just-killed-the-software-engineer-and-created-something-better"><img src="/development/ai-just-killed-the-software-engineer-and-created-something-better/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [AI Just Killed the Software Engineer (And Created Something Better)](/development/ai-just-killed-the-software-engineer-and-created-something-better)

The software engineering profession is being rewritten right now. Not slowly. Not in some distant future. Right now. And most companies are responding to it in exactly the wrong way.

This video is going to cover a lot of ground, so let me tell you upfront what you are getting into.

We will start with the numbers. Hard data on what AI is already doing to productivity, revenue, and the way code gets written. Then we will get into the human side of this, because behind those numbers are real people going through an identity crisis, and I do not think enough people are talking about that honestly.

From there, we will dig into what actually changes about how software gets built. New development cycles, spec-first workflows, how much autonomy you give AI agents, and why context engineering is the single biggest factor most teams are ignoring.

Then comes the part that makes people uncomfortable. How you restructure your company. Org design, team composition, pod models, platform teams, which roles expand, which ones shrink, and what happens to junior developers when nobody is hiring them anymore.

We will wrap up with concrete steps you can take Monday morning. No vague advice. Specific moves.

This is based on my personal experiences, on working with and talking to a lot of companies, seeing what works and what does not, and doing a lot of analysis. Let's get into it.

**[Full article >>](/development/ai-just-killed-the-software-engineer-and-created-something-better)**

---



<a href="/ai/i-built-custom-ai-agents-for-my-team-heres-the-complete-blueprint"><img src="/ai/i-built-custom-ai-agents-for-my-team-heres-the-complete-blueprint/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [I Built Custom AI Agents for My Team (Here's the Complete Blueprint)](/ai/i-built-custom-ai-agents-for-my-team-heres-the-complete-blueprint)

I've spent the last few months building custom AI agents for internal teams, and I can tell you: the gap between using a general-purpose coding assistant and having an agent that truly understands your company is enormous. We're talking about the difference between an intern who's brilliant but knows nothing about your business, and a senior engineer who's been with you for years.

In this video, I'll walk you through the complete architecture for building your own AI agent. Every component: system context, tools, knowledge retrieval, multi-agent orchestration, security, observability, and cost optimization. By the end, you'll have a blueprint you can actually follow. Not just theory, but the real decisions you'll face and how to make them.

**[Full article >>](/ai/i-built-custom-ai-agents-for-my-team-heres-the-complete-blueprint)**

---




<a href="/app-management/kubernetes-serverless-without-the-vendor-lock-in-heres-how"><img src="/app-management/kubernetes-serverless-without-the-vendor-lock-in-heres-how/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Kubernetes Serverless Without the Vendor Lock-In (Here's How)](/app-management/kubernetes-serverless-without-the-vendor-lock-in-heres-how)

Traffic is never constant. Maybe your app gets hammered during business hours and barely touched at night. Maybe it's steady all day but spikes unpredictably. Maybe there are 15 minutes a day when nobody's using it at all. The point is, **a fixed number of replicas is always wrong**. You're either wasting resources or under-provisioned.


What you actually want is an app that scales with demand. More replicas when traffic goes up. Fewer when it drops. And in the extreme case, zero replicas when there's no traffic at all. Now, scaling to zero is easy. Just set the replica count to zero and you're done. The hard part is coming back up without losing any requests. If someone sends a request and nothing is running, that request needs to be held, not dropped.



That's what we're building today. Not with [Knative](https://knative.dev). Not with [AWS Lambda](https://aws.amazon.com/lambda). Just standard Kubernetes with a few smart components wired together. We'll start with a single static replica, add Prometheus-based autoscaling, and then push it all the way to true scale-to-zero with **zero lost requests**.

**[Full article >>](/app-management/kubernetes-serverless-without-the-vendor-lock-in-heres-how)**

---




<a href="/kubernetes/building-inference-as-a-service-on-kubernetes"><img src="/kubernetes/building-inference-as-a-service-on-kubernetes/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Building Inference-as-a-Service on Kubernetes](/kubernetes/building-inference-as-a-service-on-kubernetes)


Every prompt you send to ChatGPT, Claude, or Gemini runs on someone else's GPUs. Your data, your code, your company's secrets, all flowing through infrastructure you don't control. For most people, that's fine. But if you're in healthcare, finance, government, or anywhere that compliance actually matters, "fine" isn't good enough.

Here's the thing. Running AI models on your own infrastructure sounds like it should be straightforward. It's not. GPUs are brutally expensive. Get the setup wrong and you're burning hundreds of thousands of dollars on mistakes. Get it right, and you have **Inference-as-a-Service** that any team in your company can use, with your data never leaving your network.

In this video, we're going to build exactly that. A self-hosted inference platform on Kubernetes, from cluster provisioning to serving your first model. By the end, you'll have a working setup where anyone in your organization can deploy a model with a single custom resource. No GPU expertise required.

**[Full article >>](/kubernetes/building-inference-as-a-service-on-kubernetes)**

---




<a href="ai/ai-coding-agents-are-blind-to-your-company-knowledge-heres-the-fix"><img src="ai/ai-coding-agents-are-blind-to-your-company-knowledge-heres-the-fix/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [AI Coding Agents Are Blind to Your Company Knowledge (Here's the Fix)](ai/ai-coding-agents-are-blind-to-your-company-knowledge-heres-the-fix)

AI coding agents are incredibly capable. They can write code, generate configurations, and solve complex problems. But they have a blind spot. They only know what is publicly available. They have no idea how your company actually does things. Your internal standards, your custom abstractions, your architecture decisions, your policies. None of that exists in any AI model's training data.

So what happens? Developers use AI, get perfectly reasonable output, and then spend time fixing it to match how things are actually done in their organization.

Today I am going to show you how to close that gap. We will start with the most common approaches people try, see why they fall short, and then build a proper solution using a RAG pipeline that gives AI access to your entire company knowledge base. The example we will use is Kubernetes deployments, but the concept applies to anything: coding standards, security policies, architecture patterns, onboarding docs, you name it.

**[Full article >>](ai/ai-coding-agents-are-blind-to-your-company-knowledge-heres-the-fix)**

---




<a href="/ai/stop-designing-uis-for-ai-let-the-llm-decide-what-you-see"><img src="/ai/stop-designing-uis-for-ai-let-the-llm-decide-what-you-see/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Stop Designing UIs for AI - Let the LLM Decide What You See](/ai/stop-designing-uis-for-ai-let-the-llm-decide-what-you-see)

Have you ever asked an AI agent a complex question and gotten back a wall of ASCII tables, Markdown headers, and bullet points crammed into a terminal? You squint at it, scroll up and down, and think "there has to be a better way to see this." There is. But it requires rethinking how we build UIs entirely.

Traditional interfaces are designed by humans for predictable data. AI outputs are neither predictable nor structured the same way twice. So how do we visualize something when we don't know what it is until the moment it appears?

Today I'll show you two approaches to solving this: one that ships rendering code from the server, and one that lets AI decide how data should be displayed. By the end, you'll understand why the future of UIs might not be designed by us at all.

**[Full article >>](/ai/stop-designing-uis-for-ai-let-the-llm-decide-what-you-see)**

---




<a href="/ai/why-self-hosting-ai-models-is-a-bad-idea"><img src="/ai/why-self-hosting-ai-models-is-a-bad-idea/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Why Self-Hosting AI Models Is a Bad Idea](/ai/why-self-hosting-ai-models-is-a-bad-idea)

Open weight LLM models are bullshit.

Now, before you write an angry comment telling me I'm just another person who hates open source, let me be clear. Open weight is NOT open source. The licenses are screwed and can change for worse at any time. You can't use the truly powerful models unless you have money to burn. And you definitely can't build a sustainable business on them.

That is, if you want models that actually compete with the best. Not the toys that are fun to play with but aren't the real deal.

Did those statements just put me on top of your hate list? Are you already typing a furious response? Good.

I get it though. You think using AI is expensive. You want to save money, so you have a brilliant idea: self-host free models. Right?

Stick around because I'm going to show you the actual math. And by the end of this video, you might realize that the "open" in open weight is costing you more than the "closed" alternatives ever would.

**[Full article >>](/ai/why-self-hosting-ai-models-is-a-bad-idea)**

---




<a href="/ai/commands-vs-mcp-vs-skills-what-i-use"><img src="/ai/commands-vs-mcp-vs-skills-what-i-use/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Commands vs MCP vs Skills (What I Use)](/ai/commands-vs-mcp-vs-skills-what-i-use)

When working with coding agents, we can extend them with Commands, MCP tools, MCP prompts, Skills, subagents, rules, hooks, plugins, memories... It's madness. We seem to be getting something new every week. And every agent does things slightly differently.

So let's cut through the noise. I'll show you what each of these actually does, how they work under the hood, and most importantly, which ones you should actually use and when.

**[Full article >>](/ai/commands-vs-mcp-vs-skills-what-i-use)**

---




<a href="/kubernetes/why-kubernetes-querying-is-broken-and-how-i-fixed-it"><img src="/kubernetes/why-kubernetes-querying-is-broken-and-how-i-fixed-it/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Why Kubernetes Querying Is Broken and How I Fixed It](/kubernetes/why-kubernetes-querying-is-broken-and-how-i-fixed-it)

**`kubectl get all` is a lie.** It doesn't get all. It gets maybe 10% of what's actually in your cluster. And if you want the other 90%? You're writing bash loops, waiting forever, and still missing resources because you don't even know what to search for.

This video is about fixing Kubernetes' terrible querying story. The root cause is etcd. It's a key-value store, not a database. It was never designed to answer questions like "show me all databases" or "what's running in this namespace."

I'll show you how to sync Kubernetes metadata into a Vector database, enabling both traditional queries and semantic search. By the end, you'll be able to run proper database queries AND ask questions in plain English, and get answers that actually make sense. **No more grep pipelines. No more guessing resource names.**

**[Full article >>](/kubernetes/why-kubernetes-querying-is-broken-and-how-i-fixed-it)**

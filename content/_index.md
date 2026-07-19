+++
archetype = "home"
title = ""
+++

# Latest Posts

<a href="/infrastructure-as-code/one-control-plane-for-every-gpu-cluster-modeplane"><img src="/infrastructure-as-code/one-control-plane-for-every-gpu-cluster-modeplane/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [One Control Plane for Every GPU Cluster (Modeplane)](/infrastructure-as-code/one-control-plane-for-every-gpu-cluster-modeplane)


We've been working on something new. A project called Modelplane. It's early, it's rough... but I think it's ready to fly.

But before I show you what it does, let me back up and explain the problem it solves. Because that's really where this whole thing starts.


Serving a single model on a single cluster is more or less a solved problem. Pick a serving engine, hand it a GPU, point some traffic at it, and you're done. The hard version is serving models at scale. GPUs are scarce and expensive, and they're scattered all over the place, across regions, across clouds, and across your own on-prem hardware, wherever you could actually get your hands on them. And the models people really care about, the big ones, won't even fit on a single machine. So you don't end up with a cluster. You end up with a whole fleet of GPU clusters.


And managing that fleet by hand is miserable. Every cloud provisions clusters differently. Every one of those clusters needs the exact same serving stack installed on it. And on top of all that, you're constantly playing matchmaker, figuring out which model should run on which hardware, across every cluster you've got. Do that for one cluster and it's a chore. Do it for a fleet and it's a full-time job nobody wants.


And if you look closely, there are really two very different jobs tangled up in here. On one side, somebody owns the hardware and the fleet. They decide which GPU types are blessed, which clusters exist, and where those clusters run. That's the platform side. On the other side is everyone who actually needs to serve a model. An app developer bolting inference onto a product, a data scientist, a product team, whoever it is. They don't want to think about any of the fleet stuff. They just want to say "give me a GPU with this much memory" and ship. They shouldn't need to know, or care, which clusters or instance types exist underneath. I'll call that side the developers, and I mean software engineers of any kind, not just the machine learning crowd.


Blurring those two roles together is exactly where most platforms go wrong. You end up forcing developers to understand infrastructure, or forcing the platform team to babysit every model.


This is where Modelplane comes in, and its whole shape is built around keeping those two roles apart. The platform side defines hardware classes and registers clusters. Developers declare what they need. And a scheduler sits in the middle, bridging the two.


That split, platform side on one end, developers on the other, is the spine of everything we're about to do. So keep it in the back of your mind as we go, because you'll watch it play out in the manifests, in the way the pieces reference each other, and in who's responsible for what.

**[Full article >>](/infrastructure-as-code/one-control-plane-for-every-gpu-cluster-modeplane)**

---


<a href="/development/how-i-review-ai-written-code-without-reading-a-single-line"><img src="/development/how-i-review-ai-written-code-without-reading-a-single-line/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [How I Review AI-Written Code Without Reading a Single Line](/development/how-i-review-ai-written-code-without-reading-a-single-line)


The first thing I do in the morning is watch videos on YouTube. Still in bed. No time to lose. It might look like I'm being entertained, but I'm actually working. These aren't videos you'd ever want to watch. You'd get bored at best or, more likely, say "what the fuck is this?" if you ever saw one. Yet I find them genuinely engaging, real time-savers, and they've become my morning routine. They tell me more about my day than anything else.

I'll get to what those videos actually are. But first I need to show you how I build software now, because that's the reason they exist. This is about two things. How agentic AI can write genuinely good code. And how I can **review and confirm a whole feature the agents built on their own**, in seconds, without reading a single line of it.

**[Full article >>](/development/how-i-review-ai-written-code-without-reading-a-single-line)**

---



<a href="/ai/how-i-built-a-server-that-runs-ai-agents-24-7-full-setup"><img src="/ai/how-i-built-a-server-that-runs-ai-agents-24-7-full-setup/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [How I Built a Server That Runs AI Agents 24/7 (Full Setup)](/ai/how-i-built-a-server-that-runs-ai-agents-24-7-full-setup)


If you've started using AI coding agents, you've probably felt the pull to run more than one. To have several going at once, in parallel, each chewing through a different task while you orchestrate the lot. That's the goal we're working toward. But the moment you reach for it, you run into a handful of problems, and solving them is what this whole video is about.


The first is **persistence**. These agents run on a machine, and machines sleep, reboot, lose power. The instant that happens, every agent stops dead, and hours of work can vanish with them.


The second is **accessibility**. The agents run wherever they run, but we're not always sitting right next to them. You close the laptop at home, you're working from an airport café an hour later, you're over an ocean by nightfall. And through all of it, you still want to reach them, to check in, to redirect them.

Those two are the big ones. There are also a couple of bonus problems, the kind that aren't dealbreakers on their own but quietly make everything worse.

One is **dedication**. If the agents are grinding through builds and tests on the very machine you're trying to work on, everything ends up fighting over the same CPU and RAM. You and your agents, elbowing each other for resources.

The other is **isolation**. Agents execute code. They run commands. They install things, sometimes things you'd never install yourself. Keeping all of that well away from your daily-driver machine limits the blast radius when something inevitably goes sideways.

**[Full article >>](/ai/how-i-built-a-server-that-runs-ai-agents-24-7-full-setup)**

---



<a href="/infrastructure-as-code/infrastructure-with-ai-agents-for-dummies"><img src="/infrastructure-as-code/infrastructure-with-ai-agents-for-dummies/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Infrastructure with AI Agents for Dummies](/infrastructure-as-code/infrastructure-with-ai-agents-for-dummies)


AI agents are amplifiers. If you're good at your job, agents make you better. You do more great things, faster. But if you're bad at your job, agents amplify that too. Where you used to cause a slow trickle of shit, now you have the means to unleash a full-blown **shitstorm**, at scale, in minutes.

Now, AI is all the rage these days, and for good reason. So of course people are using agents to manage real resources: infrastructure, databases, applications, all of it. The question is what happens when they do. That's what we're looking at today: an agent managing actual cloud resources, what goes wrong, why it goes wrong, and what it takes to make it work properly.

**[Full article >>](/infrastructure-as-code/infrastructure-with-ai-agents-for-dummies)**


---



<a href="/development/why-one-ai-agent-is-never-enough"><img src="/development/why-one-ai-agent-is-never-enough/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Why One AI Agent Is Never Enough](/development/why-one-ai-agent-is-never-enough)


When I used to give an AI agent a task, it would finish in one go. Write the code, declare victory, done. Now, with th

e setup I'm about to show you, the same task can take tens of iterations before the work is considered finished. The output is **dramatically better**, and I'm spending **less time** on it, not more.

The reason is that there's no longer a single agent doing the work. There's a team. One agent writes the code. Another reviews it. A third audits it for security. A fourth ships it. They run on different models, with fresh context each time, and they push back on each other until the work actually holds up. I just play games until something genuinely needs me.

In this video, I'll walk through what that pipeline looks like, 

why each role exists, and how I run all of it end-to-end. By the end, you'll have a complete picture of how to set this up yourself, and a slightly uncomfortable realization about what your job becomes when the agents do the coding.

**[Full article >>](/development/why-one-ai-agent-is-never-enough)**

---





<a href="/ai/why-ai-code-review-goes-first-and-humans-go-second-feat-coderabbit"><img src="/ai/why-ai-code-review-goes-first-and-humans-go-second-feat-coderabbit/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Why AI Code Review Goes First (And Humans Go Second) (feat: CodeRabbit)](/ai/why-ai-code-review-goes-first-and-humans-go-second-feat-coderabbit)



Code review was the safety net. The last check before something shipped. The place where bad ideas got caught, sloppy work got pushed back, and someone with fresh eyes made sure the change actually made sense.

On most teams, that net is breaking. Not because reviewers got worse. Not because standards dropped. Something fundamental about how code gets written changed, and the review process never caught up.


You can feel it if you've been paying attention. Pull requests sitting open for days. Approvals coming back so fast nobody could have read the diff. Small mistakes slipping through that would've been caught two years ago. The cracks are showing.

In this video, I'll show you what's actually breaking and why, the workflow that closes the gap, and the specific tool I use on every pull request to make it real.

**[Full article >>](/ai/why-ai-code-review-goes-first-and-humans-go-second-feat-coderabbit)**

---






<a href="/ai/how-i-access-every-ai-model-without-the-lock-in"><img src="/ai/how-i-access-every-ai-model-without-the-lock-in/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [How I Access Every AI Model Without the Lock-In](/ai/how-i-access-every-ai-model-without-the-lock-in)

New models keep dropping all the time, and I want to try them all. I want to see which one is better for which tasks, which one is cheaper, which one is faster. There's [OpenAI](https://openai.com), [Anthropic](https://anthropic.com), [Google Gemini](https://gemini.google.com), [Moonshot AI](https://moonshot.ai), [xAI](https://x.ai), and the list just keeps growing. I do not want to be locked into one model or one family of models forever. So I got subscriptions to some, I'm paying for API access to others, and then there are models that I would typically have to host myself. I don't want to do that. Even if I did, I neither have the hardware nor the patience for it.

**[Full article >>](/ai/how-i-access-every-ai-model-without-the-lock-in)**

---






<a href="/observability/i-stopped-staring-at-dashboards-ai-reads-my-grafana-metrics-now"><img src="/observability/i-stopped-staring-at-dashboards-ai-reads-my-grafana-metrics-now/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [I Stopped Staring at Dashboards. AI Reads My Grafana Metrics Now.](/observability/i-stopped-staring-at-dashboards-ai-reads-my-grafana-metrics-now)


Something just broke in production. An alert fired. You open Grafana, click through three dashboards trying to find one that matches what's actually happening, and twenty minutes later you're still squinting at panels and switching tabs to grep logs.

That entire workflow is about to disappear.


AI agents can now read your metrics, your logs, and your traces directly. They draw conclusions. They build custom dashboards on the spot. They tell you what's wrong while you're still typing the question. And the best part: **you don't have to leave your terminal to do any of it.**

In this video, I'll show you how. We'll start with Grafana Assistant inside the [Grafana](https://grafana.com) UI, then move to [Claude Code](https://claude.com/claude-code) wired up to the Grafana MCP server. By the end, you'll see how to query observability data from a chat prompt, build dashboards fitted to whatever you're investigating, and connect your own agents to live runtime data, all without touching a browser.

**[Full article >>](/observability/i-stopped-staring-at-dashboards-ai-reads-my-grafana-metrics-now)**

---







<a href="/ai/the-4-modes-of-ai-coding-and-why-your-tool-picks-itself"><img src="/ai/the-4-modes-of-ai-coding-and-why-your-tool-picks-itself/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [The 4 Modes of AI Coding (And Why Your Tool Picks Itself)](/ai/the-4-modes-of-ai-coding-and-why-your-tool-picks-itself)



Working with AI agents is management. Most people think the IDE versus TUI debate is about tools. It is not. It is about management style. And like any good manager, you adjust based on two things. How well can you specify the task? And how much does the situation warrant trust? The worse you are at specifying and the less trust is warranted, the more you need to intervene.


I think of it as four modes of working with AI agents. Four management styles. Mode 1: "I trust you only to complete what I started." Mode 2: "I need to review every single thing you do." Mode 3: "I observe what you are doing and intervene when needed." Mode 4: "I trust you with this task." These are not ranked from worst to best. They are different styles for different situations.

Both IDEs and TUIs can operate in all four modes. But each has modes where it is native and modes where it is fighting its own architecture. And that gap widens as autonomy increases. I used to be IDE-only. Then I combined IDE and TUI. Now I use TUI exclusively. That was not a tribal choice. It followed my shift through these four modes. As my center of gravity moved toward higher autonomy, the IDE stopped being the right tool for most of what I do.

So let me walk you through each mode, how IDE and TUI handle it differently, and where each paradigm hits its structural limits.

**[Full article >>](/ai/the-4-modes-of-ai-coding-and-why-your-tool-picks-itself)**

---








<a href="/ai/how-i-hooked-ai-video-generation-into-my-dev-workflow-with-higgsfield"><img src="/ai/how-i-hooked-ai-video-generation-into-my-dev-workflow-with-higgsfield/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [How I Hooked AI Video Generation Into My Dev Workflow (with Higgsfield)](/ai/how-i-hooked-ai-video-generation-into-my-dev-workflow-with-higgsfield)

Real productivity out of an AI agent starts with where it lives. Agents that run inside your terminal, your IDE, or your desktop beat agents that sit behind a web UI. [Codex](https://github.com/openai/codex), [Cursor](https://cursor.com), [Claude Desktop](https://claude.com). The surface around the model is what decides how much work you actually get out of it.

And it's not just code. Calendars, email, web research, summarizing long PDFs, drafting writing. All of it routinely happens inside the same agent session, through MCP connectors that wire the agent into Gmail, Google Calendar, Notion, Slack, and whatever else you've plugged in.

**[Full article >>](/ai/how-i-hooked-ai-video-generation-into-my-dev-workflow-with-higgsfield)**

---

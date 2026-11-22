+++
archetype = "home"
title = ""
+++

# Latest Posts

<a href="/observability/agent-observability-41-problems-my-ai-agents-never-told-me-about"><img src="/observability/agent-observability-41-problems-my-ai-agents-never-told-me-about/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Agent Observability: 41 Problems My AI Agents Never Told Me About](/observability/agent-observability-41-problems-my-ai-agents-never-told-me-about)


When I work with one agent, I watch it. I read what it's doing, approve what it wants to run, and answer its questions the moment it asks them. If a build is slow, I see it. If a check keeps failing for the same silly reason, I see that too, and I fix whatever causes it. I don't need metrics for any of that. I am the metrics.

That's not how I work anymore. I give an agent a task, let it rip, move to the next one, and do the same thing again, until dozens of them are running, many of them while I'm asleep. The slow builds, the failing checks, the questions waiting for an answer: I stopped seeing any of it.


Here's one example. One of my projects has a CI alert that's supposed to tell me when the main branch breaks. It was added a couple of months ago, and the secrets it needs to send anything were never registered. For seven weeks, every time main went red, the alert fired and reached nobody. Once, main stayed red for nineteen hours. And the instructions my agents follow say that this alert is exactly how a broken main gets noticed. Working next to one agent, I'd have seen a red build within minutes. With a fleet, nobody noticed that the alarm itself was dead.

That's what this video is about: improving agent workflows when nobody is watching. Not tuning the one agent you supervise, but finding what's wrong, what's slow and what's wasteful in a fleet of mostly unsupervised agents working around the clock. I'll use real data from a few days of agents building a real project, but the point is what you can take from it: a handful of practices that work for your own agents, whatever tools you use.


**[Full article >>](/observability/agent-observability-41-problems-my-ai-agents-never-told-me-about)**

---


<a href="/devops/ai-agents-in-kubernetes-without-laptop-credentials-feat-kaop"><img src="/devops/ai-agents-in-kubernetes-without-laptop-credentials-feat-kaop/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [AI Agents in Kubernetes Without Laptop Credentials (feat. KAOP)](/devops/ai-agents-in-kubernetes-without-laptop-credentials-feat-kaop)

Why would anyone put a platform between an AI agent and a Kubernetes cluster? Claude Code can fix clusters on its own. It finds the cause, rules out the red herrings, changes only what it should, and verifies the fix. No platform, no special setup. Just a terminal, a prompt, and my credentials. That works for one person. Does it work for a company, where it means every laptop holding the keys to production, nobody else approving what the agents change, no shared record of what they did or what it cost, and nothing happening unless someone sits at a keyboard?


To find out, I tried a platform built for exactly that: [Komodor](https://komodor.com)'s Agentic Operations Platform, KAOP for short. It lets you define agents and run them inside your clusters instead of on someone's laptop, as services with their own identities. It sends their risky actions to people for approval, and it keeps a record of what they did and what it cost. It does more than that, like a catalog of ready-made agents, guardrails on the tools agents call, and side-by-side testing of a new model or prompt against the current one, and I'll come back to those at the end.

So, do we need a platform at all? What changes when there is one? And is KAOP the one to pick? To answer that, we'll fix the same incident four times: with Claude Code alone, with Claude Code going through KAOP, and twice with nobody at a laptop. One of those fixes goes wrong in a way that says something about agents in general, not only about KAOP.

**[Full article >>](/devops/ai-agents-in-kubernetes-without-laptop-credentials-feat-kaop)**

---



<a href="/development/how-i-manage-ai-agents-at-scale-prompting-is-no-longer-the-problem"><img src="/development/how-i-manage-ai-agents-at-scale-prompting-is-no-longer-the-problem/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [How I Manage AI Agents at Scale (Prompting Is No Longer the Problem)](/development/how-i-manage-ai-agents-at-scale-prompting-is-no-longer-the-problem)

Agents that manage agents, and those agents manage workers of their own. That's how most of my work gets done these days. And the whole team is made entirely of disposable context windows.

That last part changes everything. With one agent, I write a prompt. With dozens, prompts stop being the problem. The problem becomes what each agent is allowed to know. Who keeps the plan. Who sees the logs. Who starts with a clean slate. That is the same problem every manager has, and it turns out to be most of what makes a large agent team work.

That is what this video is about: how the way we instruct agents changes as we give them more autonomy, more work, and more responsibility. By the end, we will get to where I think this is going next: systems that decide how much organization each task actually deserves.

**[Full article >>](/development/how-i-manage-ai-agents-at-scale-prompting-is-no-longer-the-problem)**

---




<a href="/bedtime/the-true-origin-of-kubernetes-and-why-google-gave-it-away"><img src="/bedtime/the-true-origin-of-kubernetes-and-why-google-gave-it-away/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [The True Origin of Kubernetes (And Why Google Gave It Away)](/bedtime/the-true-origin-of-kubernetes-and-why-google-gave-it-away)


Gather round. Get comfortable, kids. Tonight, I'm going to read you a bedtime story.


This is the story of Kubernetes.


You already know Kubernetes. That's exactly why you never asked where it came from. You wrote YAML this week. You know roughly what a pod is. You have been on a call where somebody said "we'll just put it in the cluster," and everybody nodded, and nobody said out loud what that was going to cost.

But that is the last ten minutes of a much longer story. Before Kubernetes, machines had names. There was a system inside Google that nobody outside Google had ever laid eyes on. There was a genuinely better answer out of Berkeley, and it lost. And there was a meeting where the man in charge of Google's infrastructure listened to this whole plan and could not believe what he was hearing.

So tonight I'll tell you where Kubernetes actually came from, why Google gave away the thing it called a competitive advantage, and which companies ended up collecting.

**[Full article >>](/bedtime/the-true-origin-of-kubernetes-and-why-google-gave-it-away)**

---




<a href="/development/how-i-manage-45-ai-agents-across-3-machines"><img src="/development/how-i-manage-45-ai-agents-across-3-machines/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [How I Manage 45 AI Agents Across 3 Machines](/development/how-i-manage-45-ai-agents-across-3-machines)

Right now, I have **forty-five agents working across three machines**: my laptop, a mini PC in my office, and a server in [InMotion Cloud](https://inmotioncloud.com). I call it my agent fleet. In a few hours, that fleet produces what would take me a week or two to write myself. Getting it to do the work is not the hard part anymore. Seeing what it's doing is.

We all know how to run an agent and work with it. Many of us know how to do the same with a few agents running in parallel. But what happens when there are dozens of them, spread across multiple machines? How do we manage a fleet like that?

Because that's what I am these days. A manager. And a good manager's work comes down to three things: delegating work, resolving conflicts, and removing obstacles so the team can keep going. To do that well, a manager needs a team that can work on its own most of the time, and a way to see what that team is doing when it can't.

With AI, that team is made of agents. They're already very good at almost everything, and if I did my part, they know how I like to work, what the rules are, and what they should and shouldn't do. That's what `AGENTS.md` and skills are for. I'll assume those are in place, since that's a whole different topic.

What I want to show you is how that team grows. From a single agent I have to babysit, to teams of agents spread across machines I don't even need to be near. And what happens to the manager along the way, because the agents scale much better than I do. The beginning is often the best place to start, so let's go back a year or two.

> It would be close to impossible to reproduce the exact steps from this post. They depend on my machines, my repositories, my issues and my agent setup. Instead, try something similar with [Agent Deck](https://agent-deck.devopstoolkit.ai): start with one agent, add a dispatcher, and grow from there onto additional machines.

**[Full article >>](/development/how-i-manage-45-ai-agents-across-3-machines)**

---





<a href="/ai/how-i-run-ai-agents-in-the-cloud-without-leaving-claude-code-with-aether"><img src="/ai/how-i-run-ai-agents-in-the-cloud-without-leaving-claude-code-with-aether/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [How I Run AI Agents in the Cloud Without Leaving Claude Code (with Aether)](/ai/how-i-run-ai-agents-in-the-cloud-without-leaving-claude-code-with-aether)

I run AI agents on a machine that does nothing else. Not my laptop. A dedicated box, sitting in the corner, whose entire job is to run agents while I get on with something else. It's a good setup, and it's cheap. I paid for that box once. Renting something equivalent by the month would have cost more than the box did within a few months, and then it would have kept on costing.

It also has a ceiling. Not a ceiling on how many agents I can have open, though. The number that matters is how many of them are compiling, running tests, or otherwise doing real work at the same moment. Once that gets to five or six, they start fighting each other for CPU, memory, and disk, and everything grinds. Or it breaks completely.

So what do we do when we want to run more agents than the laptop or the server they're on can handle?

The obvious answer is to buy a bigger machine. I don't want to buy a bigger machine. I hit that ceiling maybe twice a week. Most days four or five of them are actually building anything at once, and buying hardware that sits idle six days out of seven to cover the seventh is exactly the mistake we spent a decade learning not to make with servers.

The other answer is the one we landed on with servers: keep the steady state on hardware you own, and rent the overflow when you need it. That works for agents too. What's much less obvious is how you keep control of the rented ones, because the moment your agents are somewhere else, most tools want you to go and look at them somewhere else too. In a browser. In their dashboard. Not in the agent you actually use.


And I'm not switching. I use [Claude Code](https://claude.com/claude-code). I use [OpenCode](https://opencode.ai). I've put real time into getting good at both, into the prompts, the skills, the muscle memory. If you want me to drop all of that and work inside your web interface instead, it had better be bloody spectacular. Not adequate. Not a reasonable alternative. Better than what I already have, or I'm not moving. A dashboard with a list of tasks on it is not a reason to abandon the tool I live in all day. It's a reason to close the tab.

That's what I wanted to find out. Whether I can borrow somebody else's machines without giving up the thing I actually work in. The answer turned out to be mostly yes, and the "mostly" is the interesting part.

**[Full article >>](/ai/how-i-run-ai-agents-in-the-cloud-without-leaving-claude-code-with-aether)**

---





<a href="/ai/ai-pentesting-found-real-kubernetes-exploits-in-3-hours"><img src="/ai/ai-pentesting-found-real-kubernetes-exploits-in-3-hours/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [AI Pentesting Found Real Kubernetes Exploits in 3 Hours for $2](/ai/ai-pentesting-found-real-kubernetes-exploits-in-3-hours)

A few hours of an agent's work, and a few minutes of mine. That's all it took to find a way into my own Kubernetes cluster. Not a maybe buried in a PDF. A real one, proven, with my session token stolen and my cluster's secrets read back out to an attacker. A human doing the same thing would have taken weeks, if I could even get one booked.

And if you're a developer, in DevOps or SRE, or anyone who ships software and has already decided security testing isn't your job, don't go anywhere. **This one's for you.** It's a way to offload the work you don't want to do, or never get around to, and still get findings back in a shape you can act on.

Here's the situation most of us are actually in. AI writes half our code now. Sometimes all of it. We ship faster than we ever have. And penetration testing, the thing that's supposed to catch the holes before someone else does, hasn't kept up. It's scarce, it's expensive, and it happens once a year, if it happens at all. By the time the report lands, you've shipped a dozen more releases, the version it actually tested isn't even running anymore, and half the findings are too late to fix cleanly. And what you get is usually a list of problems, not solutions. It's no wonder pentesters aren't most developers' favorite people to hear from.

So this video is about what changes when that pentester is an AI. I'm going to point one at an app I actually own, let it genuinely break in, and see what that's worth.

**[Full article >>](/ai/ai-pentesting-found-real-kubernetes-exploits-in-3-hours)**

---





<a href="/bedtime/heroku-is-dead-and-it-taught-us-everything-we-know"><img src="/bedtime/heroku-is-dead-and-it-taught-us-everything-we-know/thumbnail.png" style="width:50%; float:right; padding: 10px"></a>

## [Heroku Is Dead (And It Taught Us Everything We Know)](/bedtime/heroku-is-dead-and-it-taught-us-everything-we-know)


Gather round. Get comfortable, kids. Tonight, I'm going to read you a bedtime story.


This is the story of Heroku. The best place to run software that anybody ever built, and one of the least important things its owner has ever owned.


Your company has a platform team. Maybe you're on it. They're building an internal developer platform. It has a name with "launch" or "forge" or "runway" in it, a portal, a golden path, and a YAML file you fill in to get a database. It has been in progress for about two years.

Hold onto that. Because everything on that roadmap already shipped. Finished, priced and documented, in 2011. And the thing that shipped it is still running tonight — you can sign up in under a minute.

Your platform team is going to build it anyway.

Some of you deployed to it every day. Some of you have never heard the name.

Those two facts are the same story.

**[Full article >>](/bedtime/heroku-is-dead-and-it-taught-us-everything-we-know)**

---






<a href="/ai/kubernetes-gpu-autoscaling-why-scale-to-zero-costs-you-10-minutes-per-request"><img src="/ai/kubernetes-gpu-autoscaling-why-scale-to-zero-costs-you-10-minutes-per-request/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Kubernetes GPU Autoscaling: Why Scale to Zero Costs You 10 Minutes Per Request](/ai/kubernetes-gpu-autoscaling-why-scale-to-zero-costs-you-10-minutes-per-request)

The same question, to the same model, on the same cluster. Four seconds one time, and over ten minutes the next, with nothing broken in between and nobody having touched a line of configuration.

That is the price of turning a GPU off when nobody is using it, and turning it off is the only way to stop paying for it, because the cloud charges you for the machine whether or not anything is running on it. The same ten minutes governs the other direction too. A second replica has to be asked for long before the traffic that needs it arrives, because it will not turn up in time to serve it.

So this is that trade, measured rather than argued. What scale to zero actually saves, what those ten minutes are made of, which parts of them you can attack, and when you have to ask for more.

**[Full article >>](/ai/kubernetes-gpu-autoscaling-why-scale-to-zero-costs-you-10-minutes-per-request)**

---







<a href="/bedtime/stack-overflow-is-dying-and-chatgpt-killed-it"><img src="/bedtime/stack-overflow-is-dying-and-chatgpt-killed-it/thumbnail.png" style="width:50%; float:right; padding: 10px"></a>

## [Stack Overflow Is Dying and ChatGPT Killed It](/bedtime/stack-overflow-is-dying-and-chatgpt-killed-it)


Gather round. Get comfortable, kids. Tonight, I'm going to read you a bedtime story.

This is the story of Stack Overflow. The machine that taught a generation of programmers how to copy and paste.


And you know Stack Overflow. Unless you started programming very recently, you have found the answer to a problem on Stack Overflow, copied the solution into your code, and accepted the praise when everything worked. That's all right. I won't tell anyone.

For a generation, Stack Overflow was the most important tool developers pretended they were not using. Now it is fading, and nobody knows whether it will reinvent itself or become a footnote in the history of the industry it helped build.

So tonight, I will tell you how Stack Overflow came into being, why it worked so well, what went wrong, and what it is trying to become next.

**[Full article >>](/bedtime/stack-overflow-is-dying-and-chatgpt-killed-it)**

---

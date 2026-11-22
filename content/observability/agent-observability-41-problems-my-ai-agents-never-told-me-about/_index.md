
+++
title = "Agent Observability: 41 Problems My AI Agents Never Told Me About"
date = 2026-11-23T16:00:00+00:00
draft = false
+++


When I work with one agent, I watch it. I read what it's doing, approve what it wants to run, and answer its questions the moment it asks them. If a build is slow, I see it. If a check keeps failing for the same silly reason, I see that too, and I fix whatever causes it. I don't need metrics for any of that. I am the metrics.

That's not how I work anymore. I give an agent a task, let it rip, move to the next one, and do the same thing again, until dozens of them are running, many of them while I'm asleep. The slow builds, the failing checks, the questions waiting for an answer: I stopped seeing any of it.


Here's one example. One of my projects has a CI alert that's supposed to tell me when the main branch breaks. It was added a couple of months ago, and the secrets it needs to send anything were never registered. For seven weeks, every time main went red, the alert fired and reached nobody. Once, main stayed red for nineteen hours. And the instructions my agents follow say that this alert is exactly how a broken main gets noticed. Working next to one agent, I'd have seen a red build within minutes. With a fleet, nobody noticed that the alarm itself was dead.

That's what this video is about: improving agent workflows when nobody is watching. Not tuning the one agent you supervise, but finding what's wrong, what's slow and what's wasteful in a fleet of mostly unsupervised agents working around the clock. I'll use real data from a few days of agents building a real project, but the point is what you can take from it: a handful of practices that work for your own agents, whatever tools you use.


<!--more-->

{{< youtube E9EekxrP3j8 >}}

## Why Unsupervised AI Agents Need Observability


Supervising a single agent is observability. It's just the human kind. You see every step of one run, you notice whatever annoys you, and you fix it. That works, but only while you're watching, and only for the run in front of you.

Letting agents run on their own changes three things. First, you're not there, and most problems never announce themselves. An agent waiting for your answer, you'll stumble on eventually. A build that's far slower than it should be, a test suite that keeps growing, checks that fail a few times before an agent quietly fixes whatever broke them: those you never find out about. The run ends green, and whatever it cost to get there is invisible. Second, costs multiply. A minute wasted once is nothing. Spread across dozens of agents working in parallel all day, it adds up. Third, some problems exist only across many runs. Something that goes wrong in one run out of seventeen looks like bad luck when you happen to see it, and like a problem only when you count it.

Worse, a lot of those problems come from the agents themselves. When agents write the code, they also write the tests, the build scripts and the CI. When a team of humans makes its own build slower, the same humans feel it and fix it. When agents do it, the agents that come after them pay the price, and agents don't complain. They wait, and then they do the same thing again. The feedback a supervised setup gets for free is gone, so it has to be built.


So where does that feedback come from? Watching isn't an option. Even if I wanted to, I couldn't follow dozens of agents working in parallel around the clock. Asking agents to report their own problems doesn't work, because they don't complain, and an agent's account of its own run isn't evidence I should take on faith. Checking in on a few runs misses everything that only shows up across many. We've been here before. Nobody runs a fleet of applications by watching them. We instrument them, collect metrics and traces in one place, ask questions across all of them, and set up alerts that reach someone when something breaks. A fleet of autonomous agents is another fleet of workloads. Of all the options, **metrics and traces are the only one that scales**.

That doesn't make evals pointless. Evals are to agents what tests are to applications: a fixed set of tasks that tells you whether a new model, prompt or skill does better or worse. Telemetry tells you what happens on real work, including problems that have nothing to do with the agent itself, like that alert. You need both, but this video is about the second.

## Instrument the System, Not Just the Agent


The first practice is to capture whatever your eyes used to see, and that's a lot more than tokens and cost. You need four layers. The run itself: which agent, in which role, every status change between working, waiting and blocked, every time a human steps in, every hand-off between agents, and how it ended. What the agent does: every shell command as a span, with how long it took. The pipeline around it: the checks it runs before pushing, the CI on its pull requests, the reviews, and whether main stays green. And the machine: CPU, memory and disk per run.


Most coding agents already export their own telemetry too. Claude Code, Codex, OpenCode and Pi all speak OpenTelemetry. What makes it useful is gluing it together: every span and every event carries the same run ID, so one query returns everything one run did, whichever layer it came from. And keep it to metadata. No prompts, no file contents, no agent output. **Every layer, one run ID, metadata only.**


Something has to send all that, and what that is depends on how you run your agents: a tool built for it, an agent's own exporters, or your own scripts. What you're about to see comes from [Agent Deck](https://agent-deck.devopstoolkit.ai), the open-source app I use to run all my agents. It sends the first three layers itself, a collector that comes with it covers the machine, and it's all configured through the [standard OpenTelemetry variables](https://agent-deck.devopstoolkit.ai/docs/telemetry), so it works with whichever backend you prefer. If you want to try it, the link is in the description, and if you find it useful, star it on [GitHub](https://github.com/vfarcic/dot-agent-deck). It helps more people find it. Still, Agent Deck is only where my examples come from. The four layers, and everything that follows, apply to any setup, and that's what this video is about.


The project I measured is one of mine, and it's built entirely by agents: teams with an orchestrator, a coder, a tester, an auditor and a reviewer, running on Claude Code, Codex, OpenCode and Pi, plus solo agents working through a queue of issues, day and night. Everything goes to an OpenTelemetry backend, which in my case happens to be [Honeycomb](https://www.honeycomb.io), together with the history of CI, pull requests and tests from GitHub. And I didn't analyse any of it myself. An agent did, and what it found is behind every practice that follows.

## Make AI Agents Come and Find You


The second practice is about the moments that need a human. A supervised agent has you right there. A fleet has to come and find you.


That dead alert from the start left only a note on the CI run, and nobody is notified about notes. Over sixty days, one in ten of the times main went red took nine hours or more to get back to green. Once, two agents fixed the same broken main in parallel, each with its own pull request, and nothing told either of them about the other. There were no other alerts either. Agents ran into the monthly spending limit of their API account, and I found out only from the data, after the fact.

Questions are the same story. Four of them sat on screen for hours, the longest for six. Some waiting is the price of not watching. The problem was that nobody knew. The hook that holds a question gave up after an hour, the deck kept showing the agent as waiting, and a question that had been there for hours looked exactly like one that had just arrived.


So first, make fewer of those moments. Look at what agents wait for, and when the same permission or the same kind of question keeps coming up, fix the permissions, the skills or the instructions, so the next agent doesn't have to ask. Then alert only on what truly needs a human: an agent waiting longer than it should, a main that stays red, agents stopped by a limit. An alert for every question in a fleet of dozens is noise, and noise gets ignored. Send the alerts that remain to wherever you actually are, not to a dashboard nobody opens, and test each one end to end, because an alert that has never reached anyone is no alert at all. **Agents don't escalate on their own.**

## Count, Then Fix the Cause


The third practice is to count. Some problems look like bad luck in one run and like a pattern across hundreds.


Before pushing, every agent runs the checks: formatting, linting and tests. More than one in five of those runs failed, and one in seventeen failed on formatting alone, something a single command fixes automatically. Seen once, that's a shrug. Counted, it's a step every agent should run before the checks, and nobody had told them to.


More than half of the change requests from the reviewer agent were about a finding from a review bot that the coder never answered. The instructions told agents to resolve every review thread, and they did. But one of the bots puts its findings in a summary comment, not in threads. Nobody would spot that reading pull requests one by one. Across hundreds of them, it's obvious.

Counting also shows where the human goes. The orchestrator needed me about five times an hour. The Codex and OpenCode workers didn't need me once in more than sixty runs. So steering I keep repeating to the orchestrator belongs in its instructions. And counting kills assumptions. I'd have bet approvals were where my time goes, but agents blocked on permission prompts added up to minutes over the whole window, and none of that was an actual permission prompt. Without the data, I'd have spent my effort tuning allowlists.


So measure rates across runs, not anecdotes from one. When the same failure keeps coming back, fix it once, in the tooling or the instructions, instead of letting every agent rediscover it. And check where the time actually goes before you optimise anything. **Fix the cause once, not the symptom every run.**

## Measure the Feedback Loop AI Agents Live In


The fourth practice is to look past the agents, at the loop they work in: change, build, check, CI, review.


Every build in the sandbox where agents run their checks recompiled a big chunk of the project, taking about a minute and a half even when nothing had changed. The cause was a build script, written by an agent months earlier, watching Git files that kept changing under it. To confirm the cause, the agent that did the analysis stopped those files from changing inside the sandbox, and the same build took a third of a second. Multiply a minute and a half by many agents, each running into it many times a day, and it stops being small. Anyone watching would have complained on the first day. Agents just waited, and did it again.


The machine told a similar story. While teams were working, it was overloaded almost a quarter of the time, and the load followed how much CPU the agents used, not how many of them were running. And compiling the same test helpers again for every test binary was about 41 percent of a cold build's compile time.


So measure the loop itself: how long it takes from a change to a check result, how often checks run when nothing relevant changed, and how loaded the machine is per run. Agents won't complain about a slow build, so the numbers have to. If you only observe the agents, you miss it, because **most of what slows agents isn't the agents.**

## Treat AI Agent Instructions as Something You Measure


The fifth practice is about instructions. When you supervise an agent, you steer it with words. When nobody is watching, the instructions are the only steering left, so measure them like any other input.


That project has an instructions file, CLAUDE.md, that every agent loads before it does anything. By a rough estimate, it was close to a third of everything an agent sent to the model in its very first request. The obvious reaction is to shrink it. The data said otherwise. A session sends far more than that first request, and even removing the whole file would have saved at most about five percent of the cost. The issue to shrink it was replaced with a review for rules that conflict, repeat each other, or no longer apply. A long instruction that agents need stays long.

Other measurements were missing. Every skill activation was recorded under the same generic name, so I couldn't tell which skills ever fire and which are dead weight. And some defaults only make sense with a human at the keyboard. Claude Code's prompt suggestions were three percent of its cost, in a setup where most agents have nobody typing at all.


So record which instructions and skills are actually used, measure before you cut anything, and look for defaults built for a person who isn't there. **Measuring tells you where to look, not what to do.**

## Don't Trust AI Agent Telemetry You Haven't Checked


The sixth practice is the one people skip. Once telemetry replaces your eyes, bad telemetry is bad eyesight, and you can't notice it by watching.


In the first round of data, a few dozen shell commands accounted for 88 percent of all recorded shell time. One of them, an `rm` followed by an `ls`, supposedly ran for 38 hours. It didn't. The command failed, the agent reported the failure through an event nobody was listening to, and the span stayed open until the run ended. Any question about where shell time goes was answered mostly by those ghosts. And it didn't stop there. A third of all human interventions belonged to runs with no agent recorded. Pi reported two status changes in 52 runs. And thousands of events came from agents started outside the deck, by other agents, mixed in with the real ones.


So before acting on anything, check the outliers, check that every agent reports what it should, and check that you're counting only the runs you think you are. Fix the measurements first, and only then act on them. **Data you haven't verified is only an opinion with charts.**

## Make Improving AI Agents a Routine


The last practice ties the rest together. A supervisor fixes what they see on the spot. A fleet needs a process, so that improvements happen on a schedule instead of on someone's attention.

Start by deciding what each problem costs. Time, because an agent waiting for an answer or re-running a failed check finishes its work later, and so does everything queued behind it. Tokens, because a failed check means an agent reading the output, reasoning about it and running everything again. Compute, because a saturated machine slows everything on it. Quality, because ignored findings and a silent red main let broken things through. And human attention, the scarcest resource in the whole system and the one a fleet is supposed to save. Weigh that against what the fix costs, and you know what to do first.


Then run it as a loop. First, measure: fix the telemetry until each signal arrives and means what it says. Second, collect: let normal work run, and don't change anything while the data comes in, so it's a clean baseline. Third, analyse: in my case, an agent went through the data one source at a time, from agent runs and checks to pull requests, CI, tests, builds and instructions, and wrote down every query and every number. Fourth, improve: rank the findings and file each one as its own issue. Fifth, schedule the next round, which checks each fix against its baseline and finds whatever is wrong now. The data keeps flowing all the time, and the analysis repeats every few months.


The first round, a single pass over those few days of data, produced forty-one issues. Each one says which metric it should move, preferably a rate rather than a total, its value now, the conditions it was measured under, which direction is better, and which numbers it must not make worse. There are no targets, because these numbers move with how many agents run at once and on what hardware. A fix isn't an improvement until the next round's data says so. Some will work, some won't move anything, and some turn out to be the wrong idea before anyone measures them, like shrinking that instructions file.

Keep the limits in mind. This is one project, mostly one machine, and a little over three days of data, so treat the numbers as a direction rather than a verdict. The numbers were never the point. If your agents run while you're doing something else, they have their own versions of all of these problems.


So, to sum it up. Instrument the whole system, joined by one ID. Make your agents need you less, and when they do, make them come and find you. Count, and fix causes once. Measure the loop your agents live in. Measure your instructions. Verify the telemetry before you trust it. And make all of it a routine with baselines, not a one-off audit.

How many agents do you run without watching them, and how would you know if something was wrong with them? Let me know in the comments.


When you stop watching your agents, something else has to. **Metrics and traces are the supervisor you no longer are.**

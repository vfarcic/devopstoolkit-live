
+++
title = "How I Run AI Agents in the Cloud Without Leaving Claude Code (with Aether)"
date = 2026-10-19T16:00:00+00:00
draft = false
+++

I run AI agents on a machine that does nothing else. Not my laptop. A dedicated box, sitting in the corner, whose entire job is to run agents while I get on with something else. It's a good setup, and it's cheap. I paid for that box once. Renting something equivalent by the month would have cost more than the box did within a few months, and then it would have kept on costing.

It also has a ceiling. Not a ceiling on how many agents I can have open, though. The number that matters is how many of them are compiling, running tests, or otherwise doing real work at the same moment. Once that gets to five or six, they start fighting each other for CPU, memory, and disk, and everything grinds. Or it breaks completely.

So what do we do when we want to run more agents than the laptop or the server they're on can handle?

The obvious answer is to buy a bigger machine. I don't want to buy a bigger machine. I hit that ceiling maybe twice a week. Most days four or five of them are actually building anything at once, and buying hardware that sits idle six days out of seven to cover the seventh is exactly the mistake we spent a decade learning not to make with servers.

The other answer is the one we landed on with servers: keep the steady state on hardware you own, and rent the overflow when you need it. That works for agents too. What's much less obvious is how you keep control of the rented ones, because the moment your agents are somewhere else, most tools want you to go and look at them somewhere else too. In a browser. In their dashboard. Not in the agent you actually use.


And I'm not switching. I use [Claude Code](https://claude.com/claude-code). I use [OpenCode](https://opencode.ai). I've put real time into getting good at both, into the prompts, the skills, the muscle memory. If you want me to drop all of that and work inside your web interface instead, it had better be bloody spectacular. Not adequate. Not a reasonable alternative. Better than what I already have, or I'm not moving. A dashboard with a list of tasks on it is not a reason to abandon the tool I live in all day. It's a reason to close the tab.

That's what I wanted to find out. Whether I can borrow somebody else's machines without giving up the thing I actually work in. The answer turned out to be mostly yes, and the "mostly" is the interesting part.

<!--more-->

{{< youtube s-XCTNf06XQ >}}

## Setup

> The demo below runs against open issues in my own repository. Those issues will be closed or changed by the time you read this, so you can't reproduce it as written. To follow along you'll need your own repository, or a fork, and your own issues. Most of the specifics change with it.

> The project ID in the `aether delegation install` command is mine. Yours comes from `aether project list` after you link your own repository.

> If the machine running your agents is headless, which is the whole point of a dedicated machine, the browser steps below won't work as written. Both `aether login` and the in-client MCP authentication finish an OAuth callback on a port on *that* machine. Tunnel each one to wherever your browser is.

```sh
curl -fsSL https://app.runaether.dev/install.sh | sh

aether login

aether delegation install --runtime claude-code --root . -p dcd89323-2193-4ef5-bf22-27a5ce107e60
```

> Use the full project ID. The short one that `aether project list` prints is rejected here.

## Running Agents On One Machine


My dedicated machine runs [Agent Deck](https://agent-deck.devopstoolkit.ai), which is a thing I built to manage a fleet of agents from one place. It gives every agent a pane and a worktree of its own, and lets me jump into any of them whenever I want to see what it's doing.


So let's give it some work. A product requirements document, and a couple of issues that I'll let it choose for itself.

```text
Dispatch work on the PRD 1223 as well as 2 issues (your picks) to work on.
```



Each of those became its own agent, in its own worktree, cut from the same commit on `main`.

![](dad-dispached.png)

Now, the PRD is not one agent. A document like that gets split across roles, because asking a single agent to write the code, test it, review it, audit it and release it tends to produce an agent that marks its own homework.


So there's an orchestrator, a coder, a reviewer, an auditor, a tester, and a release agent, all working on the same PRD.

![](dad-dispatched-orchestration.png)


Add it up. One PRD and two issues turned into nine agents, all on that one machine, and I can attach to any of them and watch.

Nine sounds like a lot, and it's the wrong number to look at. Check the counters at the bottom. Only some of those agents are working at any given moment. The rest are idle, because a reviewer has nothing to review until the coder is finished, and a release agent has nothing to release until the tester says so. What actually costs me is the handful that are compiling and running tests at the same time, and that is what's close to the limit here.

So what I need is somewhere to send the overflow. Not a different way of working, not a different tool, not a browser tab. The same agent I'm already talking to, pointed at machines that aren't mine.

## Running Cloud Agents With Aether


[Aether](https://runaether.dev) runs coding agents on machines you don't own. One microVM per task, created when the work starts and destroyed when it's finished.

Think serverless, except the workload isn't a request that comes back in milliseconds. It's an agent that runs for minutes, or hours, or occasionally days, and that might need to ask you something halfway through. That last part turns out to matter more than I expected.

I tried it a while back and gave up almost immediately, because using it meant leaving Claude Code and doing everything in their web UI. It's changed since then, and now I can drive their agents from whichever agent I'm already using, which is the only reason we're having this conversation.


So, same session, same agent, same window. I just point the next batch somewhere else.

```text
Pick another 10 issues to work on and dispatch them to Aether.
```



Ten tasks accepted, covering twelve issues, each one on its own branch cut from `main`, each with a link to the machine that's working on it.

![](data-dispatched-aether-summary.png)

Notice what just happened, because it matters later. I knew I needed more capacity. What I never got to say is how much. All ten of those tasks went out at whatever size the project happens to be set to. The same size for every one of them. Maybe that's the right size for all ten. Nobody worked that out, though. It's just the number that was already sitting in the box.

Nobody asked me, and my agent couldn't ask either, because the size isn't something it can see. If that were a parameter on the tool it calls, I'd expect it to notice, and then either pick one itself or turn around and ask me which I wanted. It had just read all ten issues, and it knows what this repository puts every single commit through. It could make a far better guess than a project-wide default can.

And then I went to bed.

## Checking On Cloud Agents



The next morning, this was waiting for me. There are two things in it worth looking at. The first is Agent Deck telling my agent, directly, that a unit it dispatched has finished, and handing over the report. That's the easy case though. Those agents are running on the same machine as the agent that dispatched them, so there's a daemon in the middle that can just walk over and tap it on the shoulder.

The second is the Aether check firing when the session stopped, and failing. Aether delegation check failed. Check your connection and Aether login. Unread updates remain available.

![](data-dispatcher-notification.png)

There were updates waiting, and my agent never saw them. That matters more than a missed notification. I live in these agents now, and what I want from them isn't a heads-up. It's action. Review the pull requests that came back. Merge the clean ones. Cut a release. None of that happens if the agent doesn't know the work finished. And if one of those remote agents had got stuck overnight instead, I wouldn't have found out by looking at the place I actually look, and nothing would have moved until I noticed and went digging.


So I went looking myself, starting with my email, because there was nowhere else obvious to start.

![](gmail.png)

That one's from GitHub, not from Aether. One of the remote agents had left a comment on its own pull request, and a good comment at that: every check green, the one review finding fixed and its thread resolved.

And I only read it because I was already hunting for something. On any other morning that email gets deleted unread, along with every other GitHub notification that isn't a human opening an issue or a pull request. So the one signal that made it through overnight is a signal I routinely throw away without looking at it.

I'll cut them some slack on that. Driving Aether from a client agent instead of their web UI is new, and I'd expect this edge to get sanded down soon.

So I went back to my own agent and asked it directly.


```text
What's the current status of the work dispatched to Aether?
```



Ten issues, ten pull requests, every one of them green.

![](data-aether-status.png)

That answer is the argument for driving all of this from your own agent instead of somebody's website. Not because a web UI couldn't show me that table. It has the same connection to my repository that I do, and somebody could build that screen.

It's that I never asked for a table. I asked a question, in whatever words came to mind at the time, and I could have asked a completely different one. Which of these touch the daemon protocol. Which are safe to merge first. Merge the clean ones and tell me what's left. A screen gives you what somebody decided in advance that you'd want to see. An agent gives you what you actually asked for, and then does the next thing you ask.

## Cloud Agent Pros And Cons

None of what I like or dislike here is really about Aether. It's about the model underneath it: running the extra agents on machines you don't own, and renting them only for as long as the work takes.


The obvious win is **parallelism** past what any single box will tolerate. Ten agents compiling the same Rust workspace at the same time is not something I'm doing at home.

Close behind it is **isolation**. One agent that decides to run the full test suite in a loop can't starve the other nine, because it doesn't share anything with them. On one machine, it absolutely can.

Then there's the fact that **my machine stays mine**. The work happens somewhere else, so the thing in front of me doesn't turn into a space heater while I'm trying to use it. And if you're running agents on the machine you actually work on, which is most people, there's a bigger version of that: agents somewhere else can't reach your files, your keys, or the eleven other repositories you have checked out. I solved that part by giving them a box of their own. Renting solves it too.

And the money works the way renting always works. I **pay for the peak** instead of buying for it, and the rest of the week I pay nothing.


The one I feel most is that I **can't just look**. I'm not asking to sit and watch agents work. The more autonomous they get, the less anyone does that. But they ask questions. They ask for approvals. They fail. And when something has gone wrong I want the whole context in front of me, not a summary of it. I can get it by asking my client agent to go and fetch it from the remote one, and that works. It's a round trip and a wait, every single time, where on my own machine it's one keystroke.

Then, **nothing is watching the whole fleet**. Agent Deck knows what's running on my box. Aether knows what's running on theirs. Nothing sees both, so I'm the scheduler. I'm the one who noticed my own machine was saturated, decided the rest had to go somewhere else, and moved it across by hand. I'd be doing exactly the same thing again tomorrow.

Related, and worse: my agent **can't choose the machine**. It read the issue, it knows the repository, and it knows what this particular change will actually cost to verify, which is the only thing that matters. But the tool it calls to dispatch the work doesn't take a machine size, so the task goes out at whatever default someone set for the whole repository. One setting, every task, forever. An agent can only act on what its tools put in front of it. That's the general lesson here, rather than a complaint about one product. The participant with the most context about the work is the one most likely to be left out of the decision.

To be specific about where I hit that, since I only tested one of these: Aether does have machine sizes, and I can set one from the command line. What I can't do is let my agent set one, because the tool it calls to create a task doesn't take that parameter, so every task inherits the project default. Other platforms may well handle it differently. I haven't checked.

And **nothing reaches back**. When work out there finishes, or gets stuck, or needs an answer from me, it has to find its way into the agent I'm actually sitting in front of. That's the other half of the first problem. I can't easily reach in, and it can't reliably reach out, and between those two I end up checking on things that were supposed to free me from checking on things.

And your code is **on somebody else's computer**. To get any of this working you hand a third party a credential that can act as you, against your repositories. That's a trade we make all the time, with CI, with hosted runners, with every service any of us has ever signed up to. That doesn't make it wrong. It does make it a decision.

So here's the trade. I can reach into any pane on my own machine with one keystroke, and I can't reach into a single one of the ten that ran overnight. What I got in exchange is work that actually got done, properly, on machines I never bought.

And it is cheaper, as long as you're honest about what you're using it for. Renting wins on the peak. If the peak turns into every day, that arithmetic flips and you should go and buy yourself another box, exactly the way it has always worked with servers.

Which is the answer to the question I started with. You don't move your agents somewhere else. You keep the ones you want to stay close to, and you rent machines for the rest.

## Destroy

> The machines take care of themselves. Idle workspaces suspend, and stale ones get reclaimed without you doing anything.

> The credentials don't. What outlives the demo is a long-lived token that can run as you, a GitHub App installation on your repository, and an account. Clean those up yourself.

```sh
aether delegation remove --runtime claude-code

aether auth logout
```

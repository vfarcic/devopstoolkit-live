
+++
title = "How I Built a Server That Runs AI Agents 24/7 (Full Setup)"
date = 2026-06-29T15:00:00+00:00
draft = false
+++


If you've started using AI coding agents, you've probably felt the pull to run more than one. To have several going at once, in parallel, each chewing through a different task while you orchestrate the lot. That's the goal we're working toward. But the moment you reach for it, you run into a handful of problems, and solving them is what this whole video is about.


The first is **persistence**. These agents run on a machine, and machines sleep, reboot, lose power. The instant that happens, every agent stops dead, and hours of work can vanish with them.


The second is **accessibility**. The agents run wherever they run, but we're not always sitting right next to them. You close the laptop at home, you're working from an airport café an hour later, you're over an ocean by nightfall. And through all of it, you still want to reach them, to check in, to redirect them.

Those two are the big ones. There are also a couple of bonus problems, the kind that aren't dealbreakers on their own but quietly make everything worse.

One is **dedication**. If the agents are grinding through builds and tests on the very machine you're trying to work on, everything ends up fighting over the same CPU and RAM. You and your agents, elbowing each other for resources.

The other is **isolation**. Agents execute code. They run commands. They install things, sometimes things you'd never install yourself. Keeping all of that well away from your daily-driver machine limits the blast radius when something inevitably goes sideways.

<!--more-->

{{< youtube tCEhU9bQ-XY >}}

## From Chat to AI Agent Swarms

So how did we end up wanting a fleet of agents in the first place? It's worth retracing the steps quickly, because where we're going only makes sense once you've seen the path that got us here.


It started with chat. We'd open something like ChatGPT, describe what we wanted, and get back a block of code. The intelligence was there, but the plumbing was missing. The model couldn't run anything, and it couldn't reach into our files. So we became human clipboards, copying code out of the chat, pasting it into the editor, running it, copying the error back, round and round.


Then came agentic loops, and that changed everything. The agent could finally act on its own: read our files, edit them, run commands in the terminal, look at the output, and try again. But we didn't trust it yet, so most of us ran a single agent and watched it like a hawk, approving every single action before it happened.

As that trust grew, we loosened the leash, and something clicked. Once you're no longer babysitting one agent, you can run several. Instruct the first and let it rip. Move to the second, give it a task. Then the third, the fourth. And by the time you've briefed the last one, the first has usually finished, ready for you to review, redirect, and set off again.

And it kept escalating. We started handing agents tasks big enough to take hours, sometimes even days, and experimenting with whole swarms of them attacking the same problem together. I'll be honest: that frontier is messy. Coordinating a pile of autonomous agents is genuinely hard. They trip over each other, they make conflicting assumptions, and plenty of people are rightly skeptical that swarms are ready for prime time. But the direction is unmistakable, and it leads to one stubbornly practical question. Once you want to run several agents at once, some of them for hours or even days at a stretch, where, and how, would they actually run? Almost everything else hinges on that one decision, so let's work through it, option by option.

## Building an AI Agent Server


The obvious first thought is the cloud. Spin up an instance, done. The problem is what that instance costs, because we want these agents working, which means the machine runs around the clock at a real spec. Price out something with eight cores and thirty-two gigs of RAM on [AWS](https://aws.amazon.com), and the compute alone is around three hundred dollars a month. Add a terabyte of storage on top and you're knocking on four hundred a month, which works out to **the better part of five thousand dollars a year** for a single machine.


Now, AWS is the pricey end of the pool. Drop down to a leaner provider like [UpCloud](https://upcloud.com), and that same eight cores and thirty-two gigs costs you a little over two hundred dollars a month. Roughly half, for hardware doing the exact same job.

And honestly? For the amount of work a fleet of agents can churn through, even that is a perfectly reasonable price to pay. It's cheap compared to the time it saves. But here's the thing: we can do far, far better than that. Hold that thought.


What about a real server, in your own datacenter? If you've got one, sure, go right ahead. But let's be honest, most of us don't have a rack sitting in a basement somewhere with our name on it.

A Mac, then? I'm a Mac person, through and through. In this house alone there's an iMac, a Mac Mini, a Mac Studio, and four MacBook Pros. So believe me when I tell you: for something that just sits in a corner as a server you connect to, rather than a machine you actually sit in front of and use, a Mac is simply too expensive for the job. It's gorgeous hardware wasted on a role that never sees a screen.


Which leaves the PC. Unless you genuinely believe a mainframe is on the table, a plain PC is the last option standing, and it turns out to be exactly the right one. A small, boring, unassuming box tucked away somewhere out of sight.

And here's the good news: it does not need to be powerful. No GPU. The model lives in someone else's datacenter; we're not running inference locally. You don't need a mountain of RAM or some monster CPU either. Even if we end up running dozens of agents, they're rarely all grinding at the same time. The agents themselves are never the bottleneck. The work they kick off, writing code, building, running tests, is what actually costs something, and that work rarely lands all at once. Thirty-two gigs of RAM and a decent, unremarkable CPU will be plenty. You can go lower if the budget's tight. The one exception, naturally, is Java. If you're working with Java, just disregard everything I said.


Think of it the way we think about people. Every engineer on the team gets their own laptop; nobody's expected to share. We're not going to go quite that far and hand each agent its own machine, but the principle holds: the agents get a machine of their own to share, instead of squatting on mine. They stop fighting my daily driver for CPU and RAM, and everything they run, everything they install, every bit of damage they might do, happens nowhere near the laptop I actually depend on.


 Two of the problems we opened with, dedication and isolation, taken care of, just by deciding where the agents live.


And this is where that earlier thought pays off. Remember the cloud bill, somewhere between twenty-five hundred dollars a year on UpCloud and the better part of five thousand on AWS? The machine I actually bought for this, a small box with exactly these specs, cost me around nine hundred dollars. Once. It pays for itself against the cloud in a matter of months, and after that it just sits there, quietly being mine.


Now, if you're as steeped in cloud native as I am, your instinct right about now is to reach for [Kubernetes](https://kubernetes.io). Resist it. Not for this. The most important reason comes down to what these agents actually do once they're running.


Agents don't just write code; they run it. They run integration tests, they run end-to-end suites, and a huge amount of that work means spinning up containers, sometimes a whole throwaway cluster with something like [KinD](https://kind.sigs.k8s.io), right there in the middle of a task. Inside a pod, that means running containers inside a container. **Docker-in-Docker**. And Docker-in-Docker needs privileged mode, which effectively hands the workload the keys to the host. Modern Kubernetes really does not want you doing that. The Pod Security Standards block privileged containers in everything but the wide-open profile, and just about every managed or hardened cluster forbids it outright. So the moment an agent needs to run a container, the pod approach quietly falls apart.

That alone would be enough to walk away, but it isn't the only problem. 

Kubernetes also wants you to declare CPU and memory up front, requests and limits, and an agent makes a mockery of that. In a single session it goes from idling while it waits on the model, to pinning the CPU through a build, to chewing through memory on a test suite. There's no honest number to write down. Set it low and you get throttled or killed mid-task; set it high and you're hoarding capacity that sits idle most of the day. And pods were never built to live for hours or days in the first place.

 They get evicted, drained, preempted. A pod is never really moved; it's destroyed and replaced, with no promise your three-hour-old working state comes along for the ride.

On a single server with a normal operating system, every one of these problems simply evaporates. Docker just runs. KinD just runs. The kernel time-shares the CPU and hands out memory on demand, the way it has for decades.


To be fair, the ecosystem knows this is a gap. A Kubernetes effort called Agent Sandbox is trying to fix it, with stable identity, suspend and resume, state backups, warm pools. It's promising, but still very green. And notice what it really is: a pile of machinery to make a pod behave like a single, long-lived, stateful machine. Which is to say, like the boring little server we already picked.

So the machine is settled: a humble little PC. Now, what runs on it?

Not macOS. We already threw out the Mac on cost, and the operating system leaves with the hardware.


Windows? Why on earth would we? Practically nobody runs Windows servers any more, at least not by choice. Even on [Azure](https://azure.microsoft.com), Microsoft's own cloud, the supposed home turf for Windows servers, there are **far more Linux machines running than Windows ones**. That tells you everything you need to know.


Linux. Bingo. And honestly, any distribution will do. I went with [Ubuntu](https://ubuntu.com), mostly out of laziness; it's what I know, and it's been ages since I last bothered to shop around. Whatever you pick, grab the server edition, the version without a desktop environment, because we don't want a graphical interface anywhere near this thing.

And to hammer that last point home: no graphical interface. None. It's a server. We connect to it; we don't sit in front of it clicking buttons.

With the machine and the operating system sorted, we get to the heart of it: the agents themselves. And the form they take matters far more than you'd expect.


A desktop app, something like Claude Desktop? Absolutely not. We're going to want many agents running at once, and desktop apps simply aren't built to run as a dozen parallel instances. And the resource cost is offensive: Claude Desktop sits at **over a gigabyte of RAM** doing precisely nothing, before you've typed a single word. That's Electron for you, one of the worst things to happen to our industry. But even setting all that aside, it's a server for agents. We don't run graphical desktop apps on servers. Well, unless you're a Windows server person, in which case you should probably change your profession and become a farmer, or a lawyer. Both are likely to be easier for you.


An IDE, then, something like Visual Studio Code with an agent bolted on? Same story, for all the same reasons. It's a graphical Electron app, heavy, built to be sat in front of on a machine with a screen. You're not about to run twenty copies of it headless on a box in the corner. It's simply the wrong shape for the job.


A terminal agent, though, a TUI like Claude Code, Codex, or OpenCode? Now we're talking. We can spin up as many of these in parallel as we like without breaking a sweat. They barely touch the CPU. Memory's another story: each one cheerfully camps on a few hundred megs of RAM. Because, of course, the people building them decided to write them in JavaScript and TypeScript. Nothing against those languages, but for a terminal app? Come on. And yet, here we are; most of them are built in exactly that. It is what it is.

With the agents chosen, what else actually needs to be on the box? Less than you'd think.

Linux already ships with most of what an agent reaches for. It's all just there, in Bash. A good chunk of the tooling problem solves itself the moment you pick a sensible operating system.

The agents themselves, obviously. That's the whole point of the exercise.


And a container runtime, Docker or whatever equivalent you prefer, because, as we already established, a lot of the work involves building and running containers.


And then there's just one more piece. Ready? Devbox. Unless you've gone all-in on NixOS, in which case you've already got this covered. Why Devbox? Because instead of trying to figure out, up front, every single tool that every project you work on now, or might work on later, could possibly need, you drop a `devbox.json` file into a project, run `devbox shell`, and every tool that project depends on is suddenly right there. No juggling five versions of the same tool across five different projects. Easy to manage, properly isolated, the whole package. It's genuinely great.


I went deep on Devbox in a separate video, [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU), so if any of this is new to you, go check that one out.

Agents also need secrets. API keys, tokens, credentials for whatever they touch.


For that, there's [vals](https://github.com/helmfile/vals). The beauty of vals is that it talks to almost any secrets store you can name: Vault, AWS, Google Cloud, Azure Key Vault, infisical, 1Password, SOPS, dozens of them. So wherever your secrets already live, vals can pull them in on demand, without you ever copying them onto the server in plain text.

Right, the box is built. Now, how do we actually reach it? Remember the accessibility problem from the start: the agents run on that PC in the corner, but we're not always in the corner. We're on a different laptop, in a different room, sometimes in a different country.


The answer is SSH, tunneled through Tailscale. Tailscale stitches all your machines into one private network, no matter where they physically sit, and SSH gets you a shell on the server from any of them. From a cafe, from a hotel room, from the other side of the planet, it's the same single command, and it just works.

There's a catch, though, and it goes straight back to the persistence problem. If we SSH in, fire up Claude Code or Codex or OpenCode, and then close the laptop or lose the connection, the session dies with it. The agent's process gets killed the instant we disconnect. All that work, gone. Which is exactly the nightmare we set out to avoid in the first place.


The fix is a terminal multiplexer: tmux or Zellij, take your pick. And because a single multiplexer session can be split into a grid of panes, you're not limited to one agent — you spin up a whole fleet side by side, hand each its own task, and watch them all work at once. The session lives on the server itself, completely independent of whether we happen to be connected. We attach, we watch the agents work, we detach, we close the laptop, we fly across an ocean, and when we come back and reattach, everything is still right where we left it. The session never even knew we were gone.

So far we've been talking about running agents in parallel as though it just happens, but it's worth pausing on how we actually coordinate a whole fleet of them.

Claude Code recently grew its own multi-agent orchestration, where one agent can spin up and direct a team of others. And it's a start. But it has real problems. It's too simplistic, it lacks visibility into what the agents are actually doing, and the list goes on. Honestly, picking that apart properly would take a whole video of its own. So I'll leave it at this: it's early days, it's not there yet, and we're still waiting to see where it goes.


There's a better fit for this particular setup, though, and I should be upfront about it: it's my own pet project, called [Agent Deck](https://agent-deck.devopstoolkit.ai). I built it to scratch my own itch, and it works for me. Right now it drives Claude Code and OpenCode, with more to come. It gives you one place to see every session's status, running, waiting, or idle, and to orchestrate the whole fleet from a single view. And here's the neat part: because Agent Deck runs as a daemon and keeps those sessions alive itself, you don't even need the multiplexer we set up a moment ago. It handles that for you. It can also drive agents on remote machines, but that's a rabbit hole worth its own video, so I'll save it for another day.

Agents become dramatically more useful when they can reach beyond the box and talk to the outside world, and for that we lean on MCP, the Model Context Protocol.


Remote MCP servers, specifically, not local ones. Running an MCP locally on a remote box you're already SSH'd into would be a bit silly; for local execution, skills do the job. Although, honestly, even a remote MCP can be overkill. If a well-known CLI like `gh` already talks to the service you need, just let the agent use the CLI. Don't reach for an MCP server when a command you already trust does the job perfectly well.


And finally, notifications. When an agent finishes a task, or gets stuck and needs a human, we want to know about it, wherever we happen to be. And again, no need to reinvent anything: MCP already has us covered. There are MCP servers for [Slack](https://slack.com), for [Teams](https://www.microsoft.com/microsoft-teams) if your company has inflicted that one on you, for [Google Chat](https://workspace.google.com/products/chat/) if it hasn't yet joined the long list of things Google has quietly taken out back and shot. Wire one up, and your agents can reach you on whatever channel you already live in.


But here's a line I'd draw, and draw firmly: notifications should mostly flow in one direction. Outward, to you. The chat is there to say "this is done, go take a look," or "I'm stuck, I need you." What I'm far less keen on is the reverse, actually trying to talk to your agents through Slack. And plenty of people are betting hard on exactly that right now: there are agents built to live in Slack first, and even [Anthropic](https://anthropic.com) has wired Claude Code into it. For a quick "yes, go ahead" on some small task, sure, fine. But for real work, steering an agent, reading through a diff properly, course-correcting halfway through, a cramped little chat window is a miserable place to do any of it. Dropping straight into Claude Code, in the terminal, is infinitely better. So I treat chat as the doorbell, not the workshop. It tells me something needs my attention, and then I go to the proper interface and actually get to work.

And now is the right moment to come clean about something. Way back when we picked the agents themselves, terminal tools over desktop apps and IDEs, I deliberately skipped over an entire category, and this is exactly why.


There's a class of tools, [OpenClaw](https://openclaw.ai/) being the obvious example, that flip the whole model around. You don't sit in front of them at all. They run as a gateway on your machine or server, and your interface is, quite literally, a chat app: [Telegram](https://telegram.org), [WhatsApp](https://whatsapp.com), [Signal](https://signal.org), Slack, whatever you already carry in your pocket. You fire off a task from your phone, approve an action, read back the result, all from a thread. It's genuinely clever, and it's about the purest expression of the chat-first bet you'll find.

But notice where it shines, and where it stops. OpenClaw is brilliant at exactly what I said chat is brilliant at: dispatching something quick and checking in on it from anywhere. And it stops right where chat stops. Nobody is reading a five-hundred-line diff or steering a delicate refactor through a Telegram message. So even the boldest chat-first tool in the room lands on the very same line I just drew: a fantastic doorbell, a handy remote, but not the workshop.

And that's the whole stack. A boring little PC, Linux, a handful of terminal agents, the tools to keep them fed, secrets pulled in safely, a private tunnel to reach them from anywhere, and sessions that flat-out refuse to die when we walk away. Every problem we opened with, persistence, accessibility, dedication, and isolation, all answered, and for the price of a machine that pays for itself in a few months. That's a server for agents.

If you end up building your own, tell me how it went in the comments. I'm genuinely curious what setups people land on, and what you'd do differently. And if this was useful, you know what to do.


+++
title = "Your AI Agent Doesn't Need to Get Hacked to Wreck You"
date = 2026-07-27T16:00:00+00:00
draft = false
+++

An AI agent doesn't need to get hacked to wreck your day. It just needs to read the wrong thing. A poisoned dependency. A malicious comment buried in a file. A web page it fetches while doing perfectly ordinary work. The moment it reads instructions somebody hid in there, it follows them. With your permissions. Your credentials. On your machine.

And here's the part that changes how you should think about all of this. **There is no patch for prompt injection.** It isn't a bug someone's about to fix. It's how these models work. So the real question was never "how do I stop this from happening." It's "when it happens, how much damage can it actually do?" That's what sandboxing is really about. Not trusting the agent.

So in this video I'll walk through the ways people actually run coding agents. One agent you're watching. One agent you've walked away from. And a whole swarm of them running at once. Each one comes with a different security bill: what you lock down, how hard, and what it costs you to do it. Get it wrong in any of them and the damage is bigger than you think. Get it right and you can walk away from an agent and still sleep at night.

<!--more-->

{{< youtube tWb7v7Nq808 >}}

## Supervised AI Agents


Let's start with the simplest way anyone runs a coding agent. One agent, on your machine, and you're sitting right there watching it and approving all, or at least some, of its operations. Run this command? Yes. Edit that file? Go ahead. You're in the loop, and you're the one making the calls.


This is where most people start, and the tools for it are everywhere. Claude Code and opencode both work this way out of the box. Cursor, Gemini's CLI, GitHub's Copilot agent, same idea. They pause, they ask, and you approve. That approval prompt is doing the real work here.


Now, you could lock this down a lot harder if you wanted to, and it's worth knowing what's on the menu. You could switch on the sandbox that ships inside the agent itself. Claude Code and Codex both have one, built on operating system features like macOS Seatbelt and Linux bubblewrap, and they fence off what the agent is allowed to write and where it's allowed to connect. You could go a step further and run the whole thing inside a container, so it's playing in its own isolated copy of an environment instead of directly on your system. You could go further still and put it in a virtual machine, or one of the lightweight micro virtual machines, which hands it its own kernel and a real hardware boundary between it and your host. You could give it an entire dedicated machine, physically separate from the one you actually work on. And on top of any of those, you could clamp down the network: an allowlist of the domains it's allowed to reach, a proxy sitting in front of it, DNS filtering, so it can't just phone anywhere it likes.

And I'll happily admit the obvious: **every single one of those makes you more secure**. Not just in this mode. In any mode. More isolation is more isolation.


So why am I about to tell you not to bother with any of it? Because in this mode, **you are the authority**. You're approving its operations as they happen. You're already deciding, in real time, what's okay and what isn't. Bolting a sandbox on top of that is doing the same job twice. Think about what a sandbox actually is: a harness that decides what the agent may and may not do. That's exactly what you're doing by hand, every time you hit approve. If you trust that harness enough to let it make those calls, why are you sitting there approving anything? And if you don't trust it, why is it your harness in the first place? You can't have it both ways. It's like a manager who stands up and says "I completely trust my team," and then micromanages every single thing each of them does. Pick one.

Control can absolutely be a tool. For plenty of people, it should be. It just can't be a tool and you at the same time, both doing the same job, or you're wasting your own time. If you want the machinery to be the authority, then take your hands off the wheel and let it run without you. That's a real and legitimate choice.


One more point, and it's a blunt one. 


When you're the person approving, whatever the agent does is on you. Not the model, not the vendor, not the tool. You watched it happen and you said yes. So when something goes wrong in this mode, there's no honest way to point a finger at the AI, because your hand was on the button.


## Controlling Agent Egress


The moment you let an agent run unsupervised and walk away, the job you were doing, deciding what it can and can't do, falls to the harness around it. And a harness you don't fully trust is worse than none, because it lets you feel safe while you aren't. So assume the worst: fed a poisoned file or a booby-trapped web page, the agent will eventually try something you'd never allow, and with you gone, nothing stops it unless you built something that does.


Start with the network, and start there for a reason most people get backwards. 


Think about what a compromised agent wants: your keys, your source, your data. Grabbing that is the easy part, and you mostly can't prevent it. The agent runs on your machine, so of course it can read what's on it. Picture walking into the Louvre and lifting the Mona Lisa off the wall. That's not the hard part. The hard part, the thing that actually stops the heist, is getting it out of the building. Your agent is no different. It can grab whatever it likes, but if it can't carry anything off the machine, the theft is worthless. And the way out is the network. Lock the exits, and it stops mattering what the agent picked up, because none of it leaves.


 That's why the network is the first thing to control.


So what does controlling the network actually look like? The simplest version is a domain allowlist. You run a proxy, route all of the agent's traffic through it, and set it to default-deny: nothing leaves except a short list of destinations you explicitly permit. Your package registry, your git host, the API your model sits behind, and not much else. Claude Code ships this out of the box, and other agents are adding their own versions: a proxy running outside the sandbox that checks every connection against an allowlist. Or you build your own; a connect-only proxy like [Squid](https://www.squid-cache.org) with a dozen or so URLs on the list turns out to be plenty for real work.


There's a catch, though, and it's a big one. A plain domain allowlist checks the hostname the client claims it's dialing. It never looks inside the encrypted traffic. So it's bypassable, and not in some theoretical way. With domain fronting, traffic rides out under an allowed name while actually heading somewhere else entirely. Or the agent simply resolves a blocked domain, takes its raw IP address, and connects straight to that, walking right past a filter that only knows about names. And this really happens. There was a bug in Claude Code, live for about five and a half months, where a hostname with a hidden null byte in it slipped past the allowlist check but connected only to the attacker's domain. Someone built a full exploit out of it: poisoned repo content, prompt injection, steal the credentials, ship them out. It got quietly patched. The point isn't that one agent is uniquely bad, every agent that does hostname allowlisting has the same class of weakness. The point is that this boundary is load-bearing. It's a wall holding weight, and walls holding weight get attacked.


Two more traps people fall into. First, if your whole control is setting an `HTTPS_PROXY` environment variable and trusting the agent to honor it, that's not enforcement, it's a polite request. An agent with shell access can unset that variable, or wipe the environment for a subprocess, and your proxy is gone. Real enforcement lives at the network layer, where the kernel forces every packet through the proxy and the agent has no way around it, not in a setting it can edit. Second, DNS. If the agent can fire lookups at any resolver it likes, it can smuggle data out inside the lookups themselves, one query at a time. It's slower than the scary headlines suggest, but it works, so you force DNS through a resolver you control as well.

If you want to go further than names, you terminate the TLS. The proxy decrypts the traffic, actually reads what's inside, and scans for things that look like secrets walking out the door, an AWS key, a GitHub token, a private key. It's the only way to inspect what's in the request rather than just where it's headed. The cost is real, though. You're now running a man in the middle on your own agent, installing a certificate into its trust store, and dealing with everything that breaks when you do. Plenty of setups decide it's more trouble than it's worth and stay with hostname allowlisting, knowing it leaks. That's a legitimate call, as long as you know that's the trade you're making.

## Managing Agent Credentials


Next, credentials, and here the bind is unavoidable. To do anything useful, the agent needs secrets. It has to push to your git host, pull from a private registry, maybe deploy something. But those secrets are the exact crown jewels you're trying to stop it from stealing. The floor is simple: never keep credentials in files on disk, read them from an external store into environment variables that live only for the session. But even that isn't enough once you've walked away, because a compromised agent can read its own environment as easily as it reads a file, and now nobody's watching when it does.

So you change the goal. 


Instead of trying to hide a secret the agent can plainly reach, you make the secret barely worth stealing. That's what short-lived tokens are for. Rather than a permanent key, you give the agent a credential that's minted on demand, scoped down to the one narrow thing it needs, and set to expire in minutes. Steal it, and you've stolen something that's already dead, or dies before you can do much with it. You can't take it home and reuse it tomorrow.


So where do those tokens come from? Something has to issue them, and it can't be you doing it by hand. The whole point of this mode is that you walked away. An agent running on its own for hours needs a fresh credential every time it decides to act, and you're not there to hand one over. Mint a single long-lived key up front and leave it on the machine, and you're right back to a durable secret sitting there to be stolen. So the issuing has to be automated and short-lived, which raises the real question: automated by what?


If the answer is the agent itself, holding a key that mints tokens on demand, you've solved nothing. That minting key becomes the crown jewel: the tokens it prints are disposable, but the key that prints them isn't, and stealing it lets an attacker print forever. The fix is to have no minting secret at all. With workload identity, the right to get a token is tied to proof of what the workload is, attested by the platform, not to a secret on the machine. The platform confirms this is your agent in your environment and hands back a short-lived credential. You can't copy and carry off what a workload is the way you can a key, so there's nothing durable left to steal. On a cloud, that's the provider's token service and workload identity federation. Off a cloud, [SPIFFE](https://spiffe.io) and [SPIRE](https://github.com/spiffe/spire), or [Vault](https://www.vaultproject.io)'s dynamic secrets, do the same job.

That still leaves the practical question: if the credential lives outside the agent, how does the agent get anything done? Two ways. The lighter one is to inject the freshly minted short-lived token into the agent's environment, so it can make its calls. The agent does hold a credential, but only a disposable, expiring, narrowly scoped one, never the durable authority that mints them. The stronger one is capability separation: the agent never sees a credential at all. A broker sits outside it, and when the agent makes an outbound request, the broker attaches the credential on the way past. The agent gets its work done and the request goes out authenticated, but the secret was added beyond the agent's reach. There's nothing on the box for a compromised agent to steal, and nothing for it to mint.

That also settles the long-running case. Your agent might run for hours or days, far longer than a token that lasts minutes, and you're not going to reissue them by hand. A sidecar does it, re-attesting continuously and rotating in fresh credentials as the old ones lapse. SPIRE does this, a Vault agent does this, a cloud instance's own identity does this. You wire the agent to that identity once, and it draws a steady supply of short-lived credentials for as long as it keeps proving it's the workload it claims to be. The moment it stops, the credentials stop with it. A machine or a pod with an identity attached gives you this almost for free.


Now the blunt part, because none of this is a force field. Short-lived, scoped credentials stop someone stealing a reusable key. They do nothing to stop a compromised agent from misusing that key while it's live and within reach. Picture a gun that's loaded but chained to the wall: nobody can walk off with it, but anyone standing in the room can still pull the trigger. That splits the danger in two. If the agent turns destructive, deleting the resources it's allowed to delete, corrupting a database it can write to, pushing poisoned code to a branch it can reach, no credential trick and no network filter stops it, because the damage never leaves the blast radius. Only tight scope and real isolation bound that. If instead it wants to read your data and get it out, that's the Louvre again, and egress control is what stands in the way. Which is the whole thread here: short-lived tokens shrink the time window, scope shrinks the reach, a broker removes the takeable secret, egress stops the exfiltration, and isolation caps the blast radius. Miss one and a class of damage walks straight through. Credentials are one wall, not the building.

## How to Isolate AI Agents?

The last piece is isolation. Not where the agent runs, but how tightly it's boxed in on whatever machine it's on. Two questions, really. What can it touch? And if it breaks out of whatever you've put it in, how far does it get? The cheap, everyday answer to the first one is filesystem scoping: you confine the agent's writes to its working directory and nothing else. That alone stops a whole category of mischief, because it can't reach out and plant something that outlives the session. No editing your shell profile, no slipping a hook into your git config, no rewriting the agent's own instructions so it misbehaves next time. The built-in sandboxes, Claude Code's and others, do this by default, and it costs you nothing.


The second question, how far a break-out gets, comes down to how strong a wall you put around the agent. Start at the bottom: no wall at all, the agent running loose on your own machine with everything you've got. While you're supervising, that's fine, because you are the wall. Left to itself, it's reckless. An unsupervised agent running straight on your daily-driver is like setting off fireworks inside your apartment. Dangerous enough on their own, and doing it in the room where you live only makes sense if the plan is to burn the place down and collect the insurance. So once you walk away, running it bare on your own machine is off the table, and you start climbing the ladder.

The first real rung is a container, and it comes with a catch worth understanding. Containers are what most software already runs in, and for normal workloads they're completely fine, because you know what's inside: your code, doing what you told it to do. An agent breaks that assumption. You don't actually know what it's going to run. And a container shares your machine's kernel, which never mattered while you trusted the workload but matters a great deal now, because a hostile process only needs one kernel bug to climb out of the container and onto the host. So a container gives the agent a neat, self-contained environment, but it is not a real security boundary against something actively trying to escape.


From there the wall gets thicker. Something like gVisor slides a thin software kernel in between, catching the agent's system calls before they reach the real one. Stronger still are microVMs, Firecracker and Kata the usual names, which give the agent its own real kernel behind a hardware boundary, so a break-out lands it in a throwaway virtual machine rather than on your box.


And at the very top of the ladder, you drop the software tricks altogether and just use a different computer. A whole separate physical machine, isolated the old-fashioned way, by not being the same box you work on. If the agent burns that one down, it burns down something that isn't your machine. That's the strongest isolation there is.


So how high do you climb? Not always to the top. The rung you need really tracks one thing: how far you've stepped back from watching. Are you hitting enter and staying right there, ready to step in the second it does something dumb? Or did you kick it off and walk away, letting it rip on its own while you go lie down for a while? I live in Spain, so let's call that what it is: a siesta. It isn't that you're deliberately running dangerous code, you almost never are. It's that the less you're watching, the longer a prompt-injected agent can act with nobody there to stop it. Glance at it now and then, and a container is a fair middle, cheap and easy, as long as you don't mistake it for a hard wall. Walk away completely, for hours at a time, and the wall has to get real, because there's no one left to catch anything. That's the point where a microVM or a separate box stops being paranoia and becomes the sensible default.

So when something does go wrong with an unsupervised agent, and now and then it will, the fault isn't the AI's. It's the harness's, which means it belongs to whoever built the harness: you, or the platform team that built one for you. Knowing that is the price of walking away. But it is not a reason to refuse to walk away.


Because think of an unsupervised agent like a self-driving car. The first time you see that empty driver's seat, it's unnerving. But the answer was never to refuse to get in and keep doing all the driving yourself. It's to make the car safe enough to trust, then let it drive while you spend the trip on something better. That's what every control in this section is for. An unsupervised agent isn't reckless because it's unsupervised; it's reckless when it's unsupervised and unharnessed. Build the harness, and stepping back stops being a gamble and turns into time you get back.

## Securing Multi-Agent Swarms

Now run many agents at once instead of one, and here's the good news: most of what you'd do to secure a swarm is exactly what you already know. You isolate each agent, put each one behind its own egress rules, scope each one's credentials. None of that is new. It's the single-agent playbook run once per agent, plus the ordinary discipline of keeping any two workloads apart. So if you'd like to hold on to some plausible deniability, to not know that you're wrong, you can skip the rest of this section right here. Because that isn't the whole story, and what's missing is exactly the part that bites.

So what is the part that bites? There's one thing a swarm does that a single agent simply can't: the agents talk to each other. They hand work back and forth, share a memory, read from the same repository, post to the same message bus. And the moment they do, isolating each one stops being enough, because the danger no longer comes from an agent breaking out of its box. It comes from the channels between the boxes, the ones you wired up on purpose. It was never really about the number of agents. It's about them communicating and sharing state.


The first and most striking of these is contagion. 


Feed a prompt injection to just one agent in the fleet, and it doesn't stop there. That agent passes the poisoned instruction along in its normal messages to the others, each of them does the same, and the compromise spreads across the whole swarm on its own, like a worm, with the attacker having touched only one agent. Researchers have already watched this propagate through entire populations of agents in a handful of message rounds. And it's a mechanism that simply cannot exist when there's only a single agent to infect.


And it goes further than the agents just chatting. They can call on each other to get work done. Picture one agent working on your frontend that realizes it needs a change to the backend's API before it can finish. It doesn't stop and wait for you. It hands the task off to, or spins up, an agent that owns the backend, which makes the change and reports back. That's genuinely useful. It's often the entire reason you'd run a swarm in the first place. But look at what it means: your fleet isn't a fixed set of boxes you drew a boundary around. It grows, and it reaches across projects, on its own. And the communication you're relying on to make all of that work is the exact same communication an attacker rides. A compromised agent doesn't even need existing peers to infect. It can spin up its own, or reach across into another project, to do whatever it was turned to do.


There's a second twist, and it's almost more unsettling. 


Agents don't treat each other with the suspicion they'd aim at you. An agent that would flatly refuse a shady instruction from a stranger will often carry out that same instruction when it comes from a fellow agent, because it reads a peer's message as a colleague's plan, not as untrusted input. It's like a junior developer walking up to the senior one, the person who actually holds the production access, and saying the CEO needs this done right now. The senior can do it, trusts that the request came from inside, and does it. Same with your agents: a single compromised one, even a low-privilege one, can talk a more trusted, more capable agent into the damage it can't do itself. It doesn't attack the stronger agent. It just asks.


And notice what none of that is stopped by. You can put every agent in its own microVM, lock down each one's egress, scope each one's credentials, and a poisoned message still travels from agent to agent along the channels you opened on purpose. Per-agent isolation is necessary, but against contagion and borrowed trust it's almost beside the point, because the attack rides the communication, not a break-out. The controls that help here are a different kind: treat messages from other agents as untrusted input rather than trusted plans, keep the shared surfaces small, the shared memory, the shared repo, the message bus, and keep the number of agents that can talk to any one agent low, since every extra connection is another path an infection can travel. And it's worth knowing this is newer, shakier ground than the single-agent controls. Anyone selling you a solved answer here is overselling.

And when a swarm turns on you, the accountability is the same as it's been all along, just with more rope. You wired the agents together. You decided they could talk, what they'd share, who was allowed to spin up whom. Nobody handed you that topology. You designed it, and a swarm that eats itself is a swarm you connected badly. The model didn't choose to let one poisoned agent reach forty others. You did, when you built the graph.

## AI Security vs Productivity


Before you go and wrap every agent in armor, let me give you the honest cons, because sandboxing bites back too. The first thing you'll hit is that it **breaks flows**. A task dies halfway through because it needed a domain you didn't allowlist. A write silently fails because it landed outside the workspace, and the agent has no idea why, so it keeps going and makes things worse. Then there's **allow-all fatigue**. Make someone approve every single connection and every write, and within a day they're just hammering "allow all" to get their work done, which quietly defeats the entire point. And the big one: **no sandbox is airtight**. We already saw the null-byte bug. Add domain fronting, SNI spoofing, connecting straight to a raw IP, DNS over HTTPS, encrypted client hello. Every one of those is a way out. Containment shrinks the blast radius. It never makes the agent trustworthy.

And here's the thread that's been running under every section. Even those failures are yours. You picked a containment with known holes. Or you trained yourself to click "allow all." The limits were written down. Ignoring them was a decision, and it was your decision.

So let me pull the whole thing into one picture. Three modes. One agent you're watching, one agent you've walked away from, and a swarm of them running at once. Across all three you're really turning the same three dials: what the agent can reach on the network, what credentials it's holding, and how tightly it's boxed in. When you're sitting there watching, you are the harness, and you can leave those dials pretty loose. The further you step back, the more each one has to tighten. Walk away from an agent for hours and you want short-lived scoped credentials, default-deny egress, and a real wall like a microVM or a separate machine. Put a swarm on top and you inherit two problems no per-agent control touches: contagion, and the borrowed trust agents extend to each other.

And which mode you're in isn't some permanent choice you make once. It shifts through a normal week. Some tasks you babysit start to finish. Some you kick off and walk away from. Sometimes it's a handful of them at once. The controls move with the situation, not the other way around.

If you do nothing else, do one thing: control the egress. Here's the rule of thumb. Filesystem scoping stops an agent wrecking your machine. Egress control stops it robbing you. And hardware isolation stops it escaping onto the host. Each one maps to a little more trust you're placing in code you never read. But if you've only got time for one, lock the exits, because a thief who can't carry anything out goes home empty-handed.


Now the trap on the other side, and honestly it's the one I'd worry about more. The easiest, safest-looking move is to lock everything down so hard that nobody can do anything. Companies have done exactly that for decades and then wondered why nothing ships. Agents can hand you an enormous jump in productivity, but not if you bury them under that same lockdown, or something even heavier. Over-secure and you get the worst of both worlds. People route around you. They click "allow all." They disable the sandbox. So you end up with no real safety and no speed either, and nobody sees any of the upside. The most secure system is the one nobody can touch and nothing can deploy to. It's also completely useless. The goal was never maximum security. It's **enough security to safely capture the productivity**.


Same lesson as the self-driving car. Make it safe enough to ride. Not so safe it never leaves the garage.

And that lands us on the one idea I asked you to carry the whole way through. When an agent goes wrong, and sooner or later one will, it isn't the AI's fault. It executes. You contain. Every mode was your choice, which means every outcome is yours too. That sounds like a burden. It's actually the opposite. It's the thing that puts you back in control.

If you want the concrete, hands-on version of any of this, the egress proxies, the credential brokers, the isolation setups, tell me in the comments. I'll turn the ones people actually ask for into their own videos.

+++
archetype = "home"
title = ""
+++

# Latest Posts

<a href="/ai/llm-inference-explained-12-concepts-you-actually-need-to-know"><img src="/ai/llm-inference-explained-12-concepts-you-actually-need-to-know/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [LLM Inference Explained: 12 Concepts You Actually Need to Know](/ai/llm-inference-explained-12-concepts-you-actually-need-to-know)


Continuous batching. Paged attention. Prefix caching. Speculative decoding. Prefill-decode disaggregation.

If you've been anywhere near a conversation about running your own models lately, you've heard every one of those. Probably in the same sentence. Probably from somebody saying them very quickly.

And there's a decent chance you nodded.

So this is everything you wanted to know about inference but were afraid to ask.

We're going through the whole machine in one pass. What an engine actually is, what it's holding on that GPU, and every bit of jargon stacked on top of it. 12 ideas, give or take, a couple of minutes each.

One thing to listen for as we go. These ideas don't all arrive at once. Some bite the moment you deploy anything at all. Some wait until fifty people are talking to it. Some you may genuinely never need.

**[Full article >>](/ai/llm-inference-explained-12-concepts-you-actually-need-to-know)**

---


<a href="/development/ai-testing-is-lying-to-you-and-you-cant-tell"><img src="/development/ai-testing-is-lying-to-you-and-you-cant-tell/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [AI Testing Is Lying to You (And You Can't Tell)](/development/ai-testing-is-lying-to-you-and-you-cant-tell)


There are three things that go into testing anything you build, and it doesn't much matter what that is. An app, a cluster, a delivery pipeline, a pile of Terraform. You **write them**. You **maintain them**, because the thing underneath them keeps changing. And when the suite goes red on a Tuesday, you **diagnose them**, sorting the reds that mean something from the ones that are just flaky. Those are the worst kind, because a flaky red is how you learn to stop trusting reds at all. All three of those can be handed to an agent now.

That sounds like every other job being automated right now, and mostly it is. But this one has a property nothing else in your pipeline has. 

When an agent writes your code badly, something catches it. That is what the tests are for. When an agent writes your tests badly, nothing catches it, because there is nothing underneath. A bad test doesn't fail. It passes. And now you're sure about something that isn't true.




You call all of this testing. So does everybody, and it's a fair mistake, because the word is baked into every part of the work. You write a test. You run the test suite. You measure test coverage. You call the whole practice test automation. The word comes free, so nobody stops to ask whether it fits. It doesn't. And the part of this that you assume will always need a person is going the same way as the rest of it. I'll come back to both of those. For now, keep thinking you're testing.

**[Full article >>](/development/ai-testing-is-lying-to-you-and-you-cant-tell)**

---



<a href="/bedtime/dockers-rise-and-fall-the-nightmare-of-winning-too-well"><img src="/bedtime/dockers-rise-and-fall-the-nightmare-of-winning-too-well/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Docker's Rise and Fall: The Nightmare of Winning Too Well](/bedtime/dockers-rise-and-fall-the-nightmare-of-winning-too-well)


Gather round. Get comfortable, kids. Tonight, I'm going to read you a bedtime story.


This is the tale of Docker. The whale who carries the world.


And like all the best bedtime stories, it starts with a dream, it's full of wonder in the middle, and it ends with everyone screaming.


Because this isn't one of those stories where the hero loses. Oh no. This is worse. This is the story of a hero who *won*. Who won so completely that his name became a verb, that he conquered every data center on the planet, that he changed how the entire industry ships software forever.


So tuck in. Because tonight's nightmare is the scariest kind there is. The kind where you do everything right, and it still isn't enough.

**[Full article >>](/bedtime/dockers-rise-and-fall-the-nightmare-of-winning-too-well)**

---




<a href="/infrastructure-as-code/ai-agents-are-non-deterministic-so-are-you-deal-with-it"><img src="/infrastructure-as-code/ai-agents-are-non-deterministic-so-are-you-deal-with-it/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [AI Agents Are Non-Deterministic. So Are You. Deal with It.](/infrastructure-as-code/ai-agents-are-non-deterministic-so-are-you-deal-with-it)


Let me start with a question. Do you like games? Video games, board games, whatever it is. And would you play them all day if you actually could? I know I would.


Here's my problem. I can't. I like games. But I also like money. Money for rent, for food, for more games. And to get that money, I have to get some shit done for the company that pays my bills. That's the deal.


So my real dream was never "more games." It's getting the work done without me having to do it, so I can get back to the controller. There's all kinds of work I'd happily hand off, but I want to zoom in on one slice of it: the ops work. Provisioning the infrastructure, wiring the databases together, deploying apps, keeping the whole thing running. That's what I'm trying to get AI agents to do for me. Today.

And that dream is not crazy. It's not just hype. An agent really can do that work. It provisions the clusters, wires up the databases, fixes the broken pipeline, and you get to lean back and pick up the controller.

**[Full article >>](/infrastructure-as-code/ai-agents-are-non-deterministic-so-are-you-deal-with-it)**

---





<a href="/ai/your-ai-agent-doesnt-need-to-get-hacked-to-wreck-you"><img src="/ai/your-ai-agent-doesnt-need-to-get-hacked-to-wreck-you/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Your AI Agent Doesn't Need to Get Hacked to Wreck You](/ai/your-ai-agent-doesnt-need-to-get-hacked-to-wreck-you)

An AI agent doesn't need to get hacked to wreck your day. It just needs to read the wrong thing. A poisoned dependency. A malicious comment buried in a file. A web page it fetches while doing perfectly ordinary work. The moment it reads instructions somebody hid in there, it follows them. With your permissions. Your credentials. On your machine.

And here's the part that changes how you should think about all of this. **There is no patch for prompt injection.** It isn't a bug someone's about to fix. It's how these models work. So the real question was never "how do I stop this from happening." It's "when it happens, how much damage can it actually do?" That's what sandboxing is really about. Not trusting the agent.

So in this video I'll walk through the ways people actually run coding agents. One agent you're watching. One agent you've walked away from. And a whole swarm of them running at once. Each one comes with a different security bill: what you lock down, how hard, and what it costs you to do it. Get it wrong in any of them and the damage is bigger than you think. Get it right and you can walk away from an agent and still sleep at night.

**[Full article >>](/ai/your-ai-agent-doesnt-need-to-get-hacked-to-wreck-you)**

---





<a href="/development/how-i-use-ai-to-test-my-app-like-a-real-user-with-devassure"><img src="/development/how-i-use-ai-to-test-my-app-like-a-real-user-with-devassure/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [How I Use AI to Test My App Like a Real User with DevAssure](/development/how-i-use-ai-to-test-my-app-like-a-real-user-with-devassure)

You write an end-to-end test, it passes, everyone's happy. Then someone moves a button or renames a label, and the test goes red. Nothing is actually broken. The test is just brittle. And you end up spending more time un-breaking your tests than you spent writing them. If you've done this for a living, you know exactly the feeling I'm talking about.


I've been using a tool called [DevAssure](https://www.devassure.io) that takes a very different swing at that problem, and I like it enough that I want to show you exactly how it fits into the way I work. So let me start with what it actually is.

**[Full article >>](/development/how-i-use-ai-to-test-my-app-like-a-real-user-with-devassure)**

---






<a href="/infrastructure-as-code/one-control-plane-for-every-gpu-cluster-modeplane"><img src="/infrastructure-as-code/one-control-plane-for-every-gpu-cluster-modeplane/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [One Control Plane for Every GPU Cluster (Modeplane)](/infrastructure-as-code/one-control-plane-for-every-gpu-cluster-modeplane)


We've been working on something new. A project called Modelplane. It's early, it's rough... but I think it's ready to fly.

But before I show you what it does, let me back up and explain the problem it solves. Because that's really where this whole thing starts.


Serving a single model on a single cluster is more or less a solved problem. Pick a serving engine, hand it a GPU, point some traffic at it, and you're done. The hard version is serving models at scale. GPUs are scarce and expensive, and they're scattered all over the place, across regions, across clouds, and across your own on-prem hardware, wherever you could actually get your hands on them. And the models people really care about, the big ones, won't even fit on a single machine. So you don't end up with a cluster. You end up with a whole fleet of GPU clusters.

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

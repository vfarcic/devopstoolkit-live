+++
archetype = "home"
title = ""
+++

# Latest Posts

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



<a href="/ai/why-your-gpu-fails-at-3-users-llm-inference-isnt-a-compute-problem"><img src="/ai/why-your-gpu-fails-at-3-users-llm-inference-isnt-a-compute-problem/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Why Your GPU Fails at 3 Users (LLM Inference Isn't a Compute Problem)](/ai/why-your-gpu-fails-at-3-users-llm-inference-isnt-a-compute-problem)

I put a model on a GPU. It fit, with room to spare. It loaded, it answered instantly, and for about ten minutes I looked like a genius.


Then the third person asked it something, and the answers just stopped coming.

The third. Not the three hundredth. Nothing else changed. Same GPU, same model, same prompt. The only difference was how many people were talking to it at once. Does the model fit is the wrong question. It's the check everybody runs before they deploy, and it tells you nothing at all about how many people you can serve.


And the size barely matters here. Whether you're running something small enough to sit on one cheap card, or something so large it needs a rack of them, the arithmetic is the same shape, and the thing that runs out runs out for the same reason.

Now, I've said before that self-hosting your own models is a bad idea, and I still think that. But plenty of you are doing it anyway. Air-gapped environments. Data residency rules. Models you fine-tuned yourself. Those are real reasons. So if you're going to do it, let's do it properly.

**[Full article >>](/ai/why-your-gpu-fails-at-3-users-llm-inference-isnt-a-compute-problem)**

---




<a href="/bedtime/is-jenkins-dead-no-and-thats-much-worse"><img src="/bedtime/is-jenkins-dead-no-and-thats-much-worse/thumbnail.png" style="width:50%; float:right; padding: 10px"></a>

## [Is Jenkins Dead? No, And That's Much Worse](/bedtime/is-jenkins-dead-no-and-thats-much-worse)


Gather round. Get comfortable, kids. Tonight, I'm going to read you a bedtime story.


This is the tale of Jenkins. The butler who never once said no.


Now, this is not one of those stories where the hero loses. Jenkins won. Jenkins won everything. For the better part of a decade, if you wrote software for a living, Jenkins stood between your keyboard and your customers, and it built your code, and it tested it, and it shipped it, every single night, without ever once being thanked.


And here's the thing. 

Almost nobody chooses Jenkins any more. Ask around your office and you will find people who would rip it out tomorrow morning. They can't. Twenty-odd years after a young engineer at Sun Microsystems wrote the first version of it, it is still there, still running the thing that actually ships your product, and everybody has quietly agreed not to fucking touch it.

So tuck in. Because tonight's nightmare isn't that our hero dies at the end. It's that he doesn't. He is still running tonight. And you cannot switch him off.

**[Full article >>](/bedtime/is-jenkins-dead-no-and-thats-much-worse)**

---





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


+++
title = "LLM Inference Explained: 12 Concepts You Actually Need to Know"
date = 2026-08-31T16:00:00+00:00
draft = false
+++


Continuous batching. Paged attention. Prefix caching. Speculative decoding. Prefill-decode disaggregation.

If you've been anywhere near a conversation about running your own models lately, you've heard every one of those. Probably in the same sentence. Probably from somebody saying them very quickly.

And there's a decent chance you nodded.

So this is everything you wanted to know about inference but were afraid to ask.

We're going through the whole machine in one pass. What an engine actually is, what it's holding on that GPU, and every bit of jargon stacked on top of it. 12 ideas, give or take, a couple of minutes each.

One thing to listen for as we go. These ideas don't all arrive at once. Some bite the moment you deploy anything at all. Some wait until fifty people are talking to it. Some you may genuinely never need.

<!--more-->

{{< youtube MVo3P-D7eSc >}}







## What An Engine Is


An inference engine is the program that sits between a model on a disk and somebody typing a question. That's the whole category.

Its job list is short, and it's worth going through, because nearly everything else in this video attaches to one of these lines.


It loads the weights onto the GPU. Weights are just the numbers that came out of training. Billions of them, and we'll come back to why that number matters more than you'd expect.


Separately from those weights, it keeps a running record of every conversation in progress. That record is the expensive part. Not the model. The record.


It decides which request gets worked on next, and how many to work on at the same time.


It runs the actual arithmetic.


And it speaks HTTP, so your code can talk to it roughly the way it talks to OpenAI.


Load, remember, schedule, compute, serve. 5 things.

You've used one, whether or not you've ever run one. If you've started Ollama on your laptop, that was an engine. If you've deployed vLLM, same. And if you've never touched either, every request you've ever sent to Anthropic or OpenAI still landed on an engine at the far end. Somebody is running it. It just isn't you.

They differ, and how they differ is a whole other video, but that job description doesn't change.

And here's the useful part. Every item on that list has a point where it starts to matter to you, and those points are a long way apart. Some bite the moment you deploy anything at all. Some wait until fifty people are talking to it at once. Some you may never hit. Working out which is which is most of what this video is for.


The engine doesn't work alone, though, so before we climb inside it, here's the whole machine in one picture.

At the bottom, kernels — not the operating-system kind. These are the little pieces of code that do the actual arithmetic. Everybody benchmarks them and almost nobody chooses them, so that's the last you'll hear about kernels today.

Above them, the engine. The 5 things we just went through.

Above that, a control plane, which deploys and manages engines, because sooner or later you have more than one.

And on top, a gateway, routing traffic across models and replicas.

Most of this video lives in the engine. Not because the rest doesn't matter — we climb back out at the end — but because almost every word you'll hear thrown around, batching, paging, prefix caching, quantization, lives in there. And none of it makes sense until you know what the engine is trying to do.

## The Two Costs

Everything the engine holds on that card is one of two things, and the two behave nothing alike.

The first is the weights.

A weight, we said, is a number that came out of training. There are billions of them, and a number takes up room. At the precision most models ship at, 2 bytes each.

So the arithmetic isn't complicated. 8 billion parameters, 2 bytes apiece, 16 gigabytes.


And those 16 gigabytes have to sit on the card itself, right next to the thing doing the maths. Not in the machine's ordinary RAM. On the card.


Take the NVIDIA L4 as an example. About the cheapest datacenter GPU you can rent, and 23 gigabytes of memory on the card.


16 of them are gone before anybody has asked a single question.

But here's the thing about that 16. It never changes. One user or a thousand users, the weights cost exactly the same. You load them once and you're done thinking about them.

The second cost is not like that at all.

When a model writes a sentence, it looks back at everything that came before it. Every token, every time. Recomputing all of that for every new word would be ridiculous, so the engine keeps those intermediate values and reuses them.

That's the **KV cache**. Same card, same memory, sitting right next to the weights and competing with them for room.



And it grows in two directions at once. It grows as a conversation gets longer, because there's more to remember. And it grows with every extra person having a conversation at the same time, because every one of them needs their own.

10 people deep in long conversations need 10 times the cache of one person in a long conversation.

One cost is fixed. The other one multiplies.

So when does each of them start to matter to you? The weights, immediately — if they don't fit, you haven't got a service at all. The cache, later, and exactly how much later depends on how many people you have and how long they talk to it. That second one is what almost nobody thinks about until they're already past it.

## Serving More Than One At A Time

A GPU is very good at doing many things at once, and fairly ordinary at doing one thing quickly. So the way to get value out of one is to work on several requests in the same pass.

That's batching, and it comes in two flavours that behave nothing alike.


The old way is to collect a handful of requests, run them together, wait for all of them to finish, and then pick up the next handful. It works. But if one person asked a yes-or-no question and another asked for a thousand-word essay, the yes-or-no finished ages ago, and its seat in that batch has been sitting empty ever since — doing nothing, until the essay is done.


The modern way is **continuous batching**. A request that finishes leaves straight away, and a waiting one takes its place. The batch turns over constantly, no seat sits empty, and nobody is waiting on a stranger's essay.

That's the difference between a queue that drains and a queue that only grows.

The other half of serving many people is fitting them.

Every conversation needs room in that KV cache, and you don't know up front how much. So the obvious approach is to reserve the most a conversation could possibly need, per request, in advance.


Which wastes an enormous amount, because hardly anyone uses their maximum. You end up with a card that's almost entirely reserved and almost entirely empty at the same time.


**Paging** is the fix. Instead of one big reservation per conversation, the cache gets handed out in small fixed-size blocks, as and when they're needed. If you've ever wondered how your laptop runs more programs than it has memory for, it's the same trick.

So when do these start to matter to you? Batching, at your second concurrent user. Literally the second one. Paging, as soon as requests vary much in length, which in practice means immediately.

## Buying Room

If the weights are the fixed cost, the obvious question is whether they have to be that big.

They don't.

Every weight is a number, stored at some precision. Quantization stores it at less. Think of it as rounding — a value kept to a lot of decimal places gets kept to fewer, and takes less room as a result.


Halve the bits and you roughly halve the weights. On that 16-gigabyte model, that's 8 gigabytes handed back.


And here's the part that isn't obvious. The space you get back doesn't go to the model. It goes to the KV cache. Which is the thing that decides how many people you can serve at the same time.

So quantization isn't really about making a model fit. It's about what's left over once it has.

It does cost you something. It's small, and it lands harder on some kinds of work than others. That's a whole video by itself.

When does this start to matter to you? Roughly as soon as you're serving anybody at all on hardware you'd actually want to pay for.

## Not Doing The Same Work Twice

Before a model can answer anything, it has to read the question. All of it. And the question is usually a great deal bigger than whatever the person actually typed.

There's a system prompt. There are tool definitions. In a chat, there's the entire conversation so far, resent from the beginning on every single turn.

So if you're running an agent, every request it sends opens with the same tokens. Thousands of them. And in a session that's been going for a while, hundreds of thousands — because all of it goes back, every time.


And the engine dutifully works through the whole lot again, from scratch, as though it had never seen any of it before in its life.

Prefix caching is the engine noticing.


It hangs on to the work it already did on that opening stretch, and when the next request turns up starting the same way, it skips ahead to the part that's genuinely new.

For a chatbot where everybody types something different, this does very little. For agents, where every request opens with the same enormous wall of context, it's the biggest single lever available.

When does it start to matter to you? The moment your requests have something in common at the front. Which, if you're building anything agentic, is every request you will ever send.

## Two Phases, Two Bottlenecks

When a model answers a question, it's really doing two separate jobs, and they stress the hardware in completely different ways.

The first is reading the input. And the input is everything that got sent — the system prompt, the tool definitions, the whole conversation so far — not just the sentence somebody typed. The engine works through all of it in one go, in parallel. That's **prefill**, and what limits it is raw arithmetic. The GPU is doing as much maths as it possibly can, as fast as it can.

The second job is writing the answer, and this one is stubbornly sequential. The model produces a token, looks at what it just wrote, produces the next one. It can't go faster by doing more at once, because token 5 doesn't exist until token 4 does.


That's **decode**, and it isn't limited by arithmetic at all. To produce each token it has to haul the entire model out of memory and back. 16 gigabytes, for one token. The GPU spends most of that time waiting on memory rather than calculating.

So one phase is limited by compute and the other by memory bandwidth. Same request, two completely different constraints. Which is why a long input and a long answer make your GPU struggle in different places, and why nearly everything clever in inference is aimed at one or the other.


**Speculative decoding** is aimed squarely at decode. If the trouble is that tokens only arrive one at a time, you bring in a second, much smaller model to guess the next few, then let the big model check all of those guesses in a single pass. When the guesses are right — and for predictable text they often are — you've had several tokens for the price of one.

When do these start to matter to you? Prefill and decode, as soon as your inputs get long or vary a lot in size. Speculative decoding, when a single request feels slow — which, notice, is a different question from everything else in this video. It's the one thing here that helps one person rather than fifty.

## Above The Engine


Everything so far has been inside one engine, serving one model, on one GPU. Which is a perfectly fine place to be, right up until it isn't.

Because sooner or later there's a second model. And a second replica of the first one. And a version you're testing before it goes anywhere near production. Now somebody has to decide what runs where, what gets updated when, and what happens when a node quietly disappears underneath you.

That somebody is a **control plane**. It deploys engines, it manages them, and it takes the Deployment YAML you've been hand-writing away from you.


And once more than one engine is running, something has to decide which of them any given request goes to. That's a **gateway**.

You'd be forgiven for thinking that one's solved. We've been load balancing HTTP for 30 years. It isn't solved, and plain round-robin is actively wrong for language models — requests aren't interchangeable, one replica may already have your conversation warm in its cache while another has never seen you, and landing on the wrong one costs real money and real time.

Why that is, and what to do about it, is a video of its own. For now, just the word: the gateway routes across models and replicas.

When do these start to matter to you? Later than most of this video, and then all at once. One model and one replica, and you need neither of them. The moment there's a second of anything, you need both.

## When One Isn't Enough

Two more, and then we're done.



The first is what happens when the model doesn't fit on one card at all. We've been talking about 8 billion parameters on a 23-gigabyte GPU. The models people actually want are a great deal larger than that — large enough to need a rack, not a card.

So you split the model across several GPUs. That's **sharding**, and there's more than one way to slice it. You can cut every layer across all the cards, or you can hand different cards different layers. Tensor parallelism and pipeline parallelism, if you want the words for them. They have very different consequences for how much those cards have to talk to each other, and that is a video of its own.

The second is that your traffic isn't constant. Nobody's is. And a GPU is far too expensive to leave sitting idle all night on the off chance.

So you scale, on signals particular to this work: how deep the queue is, how full the KV cache is. Not CPU usage, which tells you almost nothing here.


The catch is real, though. A GPU node that scales up starts completely empty. It has to pull a container image, then the weights, before it can answer anybody — and by the time it's ready, the spike that triggered it may be long over. That's the cold-start wall, and it's why scale-to-zero is harder than it sounds.

One last word to leave you with: **disaggregation**. Reading the input and writing the answer want different things from the hardware — so run them on different machines, each matched to what its phase actually needs. It's the most interesting idea in inference at the moment, and it needs just about everything in this video to make sense.

When do these start to matter to you? Sharding, when your model won't fit, which is something you work out before you deploy rather than after. Autoscaling, the first time you look at a GPU bill.

## Where To Go Next

That's the machine. 12 ideas, one pass.

If you keep one thing, keep the question rather than the list. For every one of these, what matters isn't what it is. It's when it starts to matter to you.


The weights, the moment you deploy. Batching, at your second user. Quantization, as soon as you're paying for the card. Prefix caching, if you're running agents. Sharding, when the model won't fit. Autoscaling, the first time you read a GPU bill. And one or two of them, disaggregation most likely, possibly never.

So don't go and implement all of this. Work out which two or three you're already past, deal with those, and leave the rest alone until you aren't.

Where you go next depends on which ones you landed on. How many people a single GPU can really serve is a video by itself. So is choosing between engines. So is the gateway, and scaling, and working out what to measure.

One thing I've left out entirely. Past a certain size this stops being one cluster and becomes many, across regions or providers, and that's a different problem with a different set of answers. Further away than most people think, and out of scope here.

Everything else in this video, though, you will hit. Probably in roughly that order.

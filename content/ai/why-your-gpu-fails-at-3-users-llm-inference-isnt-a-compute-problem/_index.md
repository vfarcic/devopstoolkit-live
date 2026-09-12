
+++
title = "Why Your GPU Fails at 3 Users (LLM Inference Isn't a Compute Problem)"
date = 2026-09-14T16:00:00+00:00
draft = false
+++

I put a model on a GPU. It fit, with room to spare. It loaded, it answered instantly, and for about ten minutes I looked like a genius.


Then the third person asked it something, and the answers just stopped coming.

The third. Not the three hundredth. Nothing else changed. Same GPU, same model, same prompt. The only difference was how many people were talking to it at once. Does the model fit is the wrong question. It's the check everybody runs before they deploy, and it tells you nothing at all about how many people you can serve.


And the size barely matters here. Whether you're running something small enough to sit on one cheap card, or something so large it needs a rack of them, the arithmetic is the same shape, and the thing that runs out runs out for the same reason.

Now, I've said before that self-hosting your own models is a bad idea, and I still think that. But plenty of you are doing it anyway. Air-gapped environments. Data residency rules. Models you fine-tuned yourself. Those are real reasons. So if you're going to do it, let's do it properly.


Four layers sit between a model and the person waiting for an answer. At the bottom, kernels, the attention implementations everyone benchmarks and nobody actually chooses. Above them, the engine, which loads the weights, manages GPU memory, batches requests, and serves an API. Then a control plane that deploys and manages engines, and a gateway that routes traffic across models and replicas.


You can stop at the engine. It works. On your laptop. Maybe on that Mac mini you convinced yourself was an investment. For real inference, serving real traffic, you need the other two.

Today we stay inside the engine, on a single question. How much can one GPU actually hold, and how many people does that let you serve? Which engine to run, and everything in the layers above it, I'll get to in other videos.

Here's how we'll do it. We'll work out the arithmetic on paper first. Then we'll put a real model on a real GPU, throw a hundred requests at it, and watch it grind to a halt on camera. And then we'll fix it.

By the end you'll know why **inference is a memory management problem** rather than a compute one. You'll know which line in your engine's startup logs tells you your real user ceiling, before a single request arrives. And you'll watch that ceiling move by roughly a factor of six, without touching the hardware or the model.

The mental model almost everyone starts with is simple. The model is smaller than the GPU. It fits. Done.

That's the mistake.

<!--more-->

{{< youtube g_5g1hBmAzA >}}






## Setup

One thing worth knowing before we start. The cluster this repo builds is managed by Flux, so you get a setup that behaves like a real one rather than a pile of shell commands. That isn't today's subject, so I'll let it work in the background. The inference deployments we'll be applying by hand, so you can see exactly what changes each time.

```sh
git clone https://github.com/vfarcic/inference-demo

cd inference-demo
```

> Watch [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell
```

> All outputs in this post come from Google Cloud. The instructions work the same on AWS, and only the provider changes.

[user]
```sh
export PROVIDER=google # `aws` is also supported
```

Two heads-ups before you run the next one. It's going to ask you to log in to whichever provider you picked, so don't be alarmed when a browser window opens on its own. And on Google it creates a brand new project with a timestamped name and links your billing account to that, so nothing you already had gets touched. The destroy at the end takes the whole project with it.

```sh
chmod +x dot.nu

./dot.nu setup inference $PROVIDER

source .env
```

The cluster comes up with two node pools. The general nodes run Flux and whatever else the system needs. The GPU pool is tainted, so the only things that can land on it are inference workloads. That matters later, because it means we can tear down and rebuild the expensive node on its own without disturbing anything else.

## Model Weights VRAM Math


Weights are the easy part. Parameters times bytes per parameter. At 16 bit precision that's two bytes each, so an 8 billion parameter model needs roughly 16 gigabytes. Everyone gets this far.
Those weights also have to reach the node. Sounds easy.

And surely you want a capable model, not a toy.


So let's say we want to run Kimi K3. It's 2.8 trillion parameters. The weights alone are 1.56 terabytes, and serving it wants somewhere around 1.68 terabytes of VRAM. No single GPU comes anywhere near that. The biggest one NVIDIA sells is a B300, at 288 gigabytes, so even the flagship needs six of them. Down at H100 size it's 24, across three nodes, plus a suitcase full of large denomination bills.

They run 25 to 33 thousand dollars each, assuming you can get an allocation at all. Lead times at channel resellers are 36 to 52 weeks.

On a single server you'd move those weights once and forget about it. Kubernetes doesn't work like that.


Pods get rescheduled, nodes get added and removed, and a GPU node that scaled to zero comes back completely empty. Every one of those events moves the whole thing again, while you pay GPU prices for machines doing nothing but copying files.

## What Is A KV Cache

Weights are only half of what sits in that memory. To produce the next token, the model looks at every token that came before it. Recomputing all of that for every single new word would be absurd, so the model keeps those intermediate values around and reuses them. That's the **KV cache**, and it lives in the same VRAM as the weights.

And it isn't a fixed cost. It grows with the length of the conversation, and it grows again with every person having one at the same time. Ten users with long contexts need ten times the cache of one. That's the number nobody puts in the spreadsheet, and it's the one that decides how many people a GPU can actually serve.

We're not running Kimi today, primarily because I don't have that suitcase to spend on this video. We're running an **8 billion** parameter model on a single **L4**, and the arithmetic from here on is for that one.

If you haven't met an L4, it's an NVIDIA card built for inference rather than training. **24 gigabytes**, low power, and about the cheapest datacenter GPU you can rent. Google attaches them to these machine types, Amazon sells them too, and you can buy the card outright if you'd rather. Nothing here is specific to one cloud.


So let's ask the card what we're actually working with.

```sh
kubectl --namespace inference logs gpu-info
```

The output is as follows.

```
Tue Aug 11 12:03:22 2026
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 580.159.04             Driver Version: 580.159.04     CUDA Version: 13.0     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA L4                      Off |   00000000:00:03.0 Off |                    0 |
| N/A   59C    P8             17W /   72W |       0MiB /  23034MiB |      0%      Default |
+-----------------------------------------+------------------------+----------------------+
```

**23 thousand megabytes**. Call it **23 gigabytes** to work with.

And to be clear about which memory we're talking about, because this trips people up. That's the GPU's own memory, on the card. The node it's plugged into has **31 gigabytes** of ordinary RAM as well, and none of that helps. The model has to sit on the card, next to the thing doing the maths. RAM is only where the weights pass through on the way there.


Which matters, because the two are not priced anything alike. This node without the GPU is **27 cents an hour**. With it, **84 cents**. Same number of processors, same 31 gigabytes of RAM. Adding one card roughly triples the bill, and all that money buys is 24 gigabytes of the right kind of memory. On the cheapest datacenter GPU you can rent.

And you can't buy the memory on its own. It arrives welded to a processor you pay for whether you need it or not, which is why those queues from earlier aren't really about chips any more. The shortage moved to the memory.

We already worked out the weights: 8 billion parameters at two bytes each, so **16 of those 23**. The engine wants roughly **one** more for itself. Whatever's left, and it's around **six gigabytes**, is all the KV cache gets.

And those six gigabytes are shared by every single person talking to the model at the same time.

All of that is arithmetic though. Let's find out whether the GPU agrees.

## Watching vLLM Break Under Load


We'll be using [vLLM](https://docs.vllm.ai) for this. It's an inference engine, it's the one most people reach for, and today it's nothing more than the vehicle. Whether it's the right engine for you is a whole other video.


Let's put a model on that GPU.

```sh
kubectl apply --filename demo/vllm-16bit.yaml

kubectl apply --filename demo/ingress.yaml
```

The output is as follows.

```
deployment.apps/vllm created
service/silly-model created
ingress.networking.k8s.io/silly-model created
```


Now we wait for it to come up, and this genuinely takes a while, so I'll fast forward. Assume every wait from here on is doing the same.

```sh
kubectl --namespace inference rollout status deployment vllm
```

```
deployment "vllm" successfully rolled out
```

Almost all of that wait was two downloads. An eight gigabyte container image, and then the weights themselves. And for every second of it, we were paying GPU prices for a machine doing nothing but copying files.


vLLM tells us what the weights actually cost.

```sh
kubectl --namespace inference logs --selector app=silly-model --tail -1 \
    | grep "loading took"
```

The output is as follows.

```
... Model loading took 15.27 GiB and 116.741063 seconds
```

**15.27 gigabytes.** We calculated 16, so the arithmetic holds. And it took nearly two minutes just to move them onto the card.


Let's ask it something.

```sh
curl --silent "http://silly-model.$INGRESS_HOST/v1/chat/completions" \
    --header "Content-Type: application/json" \
    --data '{
        "messages": [{"role": "user", "content": "What is a KV cache? One sentence."}]
    }' | jq --raw-output ".choices[0].message.content"
```

The output is as follows.

```
<think>
Okay, the user is asking about what a KV cache is, and they want a one-sentence
answer. Let me start by recalling what I know about KV caches. KV stands for
Key-Value, so it's a type of cache that stores data in key-value pairs.

...

Alright, putting it all together.
</think>

A KV cache is a data structure that stores key-value pairs to efficiently manage
and retrieve intermediate results, commonly used in machine learning models to
optimize attention mechanisms during inference.
```

Hooray. It works. Job done. Everyone in the company can start using it.

One thing worth noticing before we move on. It's a reasoning model, so it thought out loud before it answered. Every one of those thinking tokens sits in the KV cache exactly like the answer does.


Let's see what happens when a hundred of them do.

```sh
hey -n 100 -c 100 -t 0 -m POST \
    -H "Content-Type: application/json" \
    -d '{"messages": [{"role": "user", "content": "Explain what a KV cache is."}]}' \
    "http://silly-model.$INGRESS_HOST/v1/chat/completions"
```

The output is as follows.

```
Summary:
  Total:	620.4257 secs
  Slowest:	620.4255 secs
  Fastest:	99.6918 secs
  Average:	349.8610 secs
  Requests/sec:	0.1612

...

Latency distribution:
  10% in 142.1108 secs
  25% in 215.6874 secs
  50% in 344.3657 secs
  75% in 470.3372 secs
  90% in 557.5124 secs
  95% in 570.9585 secs
  99% in 620.4255 secs

...

Status code distribution:
  [200]	100 responses
```

Start at the bottom, because that's the part people get wrong. Every single request succeeded. Nothing errored, nothing timed out, nothing crashed. If you were watching error rates, you'd be looking at a perfectly healthy service.

Now compare it to the single request we sent a minute ago, which came back in seconds. Here, the *fastest* of the hundred took minutes. The slowest took several times longer again. Same prompt, same model, same hardware. The only thing that changed is how many people asked at once.

Look at the shape of that distribution. It isn't a cliff, it's a smear. An even spread all the way from the quickest response to the slowest, which is exactly what a queue looks like from the outside. And throughput fell to a fraction of a request per second, on hardware that felt perfectly quick a minute ago.

And before anyone says a hundred concurrent requests is a lot, it isn't. It isn't even a hundred people. I generate tens of concurrent requests on my own, running a swarm of agents. So this isn't a company of a hundred. It's more like a handful of developers, each with agents doing their work for them.


None of that should have been a surprise. vLLM told us exactly what would happen, before a single request arrived. We just didn't look.

```sh
kubectl --namespace inference logs --selector app=silly-model --tail -1 \
    | grep --extended-regexp "Available KV cache|KV cache size|Maximum concurrency"
```

The output is as follows.

```
... Available KV cache memory: 3.14 GiB
... GPU KV cache size: 22,880 tokens
... Maximum concurrency for 8,192 tokens per request: 2.79x
```

The KV cache got **about three gigabytes**. Earlier I estimated six, so let's be honest about why I was wrong, because both reasons matter. We told vLLM it may only use ninety percent of the card, so a couple of gigabytes sit untouched on our own instruction. And the engine's own overhead is bigger than I guessed: over a gigabyte for activations, and half a gigabyte again for CUDA graphs.

Which leaves room for **twenty-odd thousand tokens**. That's the entire conversational memory of this GPU, and everybody talking to it shares that one number.

How many people that works out to depends entirely on how long their conversations run. Short exchanges, a couple of thousand tokens each, and you fit around a dozen. Let them run to the full eight thousand tokens on that last line, and you fit **under three**. Not two thousand. Not two hundred. Under three.

And that was sitting in the logs before anybody sent a single request.

Keep in mind this is the small one. The model we actually wanted at the start of this, Kimi, is around a hundred times bigger than what we just deployed. Every number on this screen moves with it.

So what do we do about it? We could rent a bigger card, and we've seen the hourly rate. We could buy one, and we've seen both the price and the waiting list. The more interesting question is whether we can serve more people on the hardware we already have.

## How Quantization Frees VRAM

There's one lever that moves that ceiling, and it's **quantization**.

Every weight in a model is just a number, and every one of them is stored at whatever precision the model was trained at. Quantization stores them at less. Think of it as rounding: a value kept to many decimal places gets kept to fewer, and takes less room as a result.


Concretely, at 16 bits each weight can be one of about 65,000 distinct values. Drop to 8 bits and it can be one of 256. Drop to 4 and it's 16. The weights halve, then quarter, and every single one of them gets nudged to the nearest value it's still allowed to be.

Each of those nudges is tiny. There are **8 billion** of them. So this costs something, and we'll come back to exactly what once we've seen what it buys.


Here's the same deployment, with one thing changed.

```sh
diff demo/vllm-16bit.yaml demo/vllm-8bit.yaml
```

The output is as follows.

```
41,42c41,42
<             - --dtype
<             - bfloat16
---
>             - --quantization
>             - fp8
```


Two lines. Same model, same GPU, same everything else. We're asking for eight bits per weight instead of sixteen.

```sh
kubectl apply --filename demo/vllm-8bit.yaml

kubectl --namespace inference rollout status deployment vllm
```

The output is as follows.

```
deployment "vllm" successfully rolled out
```

Same wait as before, and for the same reason. Worth noting that we're downloading exactly the same weights we downloaded last time. vLLM is converting them to eight bits as it loads them, so the transfer doesn't get any smaller. Pre-quantized models exist and would fix that, but that's a different conversation.


```sh
kubectl --namespace inference logs --selector app=silly-model --tail -1 \
    | grep --extended-regexp "loading took|Available KV cache|KV cache size|Maximum concurrency"
```

The output is as follows.

```
... Model loading took 9.06 GiB and 113.161044 seconds
... Available KV cache memory: 9.26 GiB
... GPU KV cache size: 67,424 tokens
... Maximum concurrency for 8,192 tokens per request: 8.23x
```

The weights lost about a third of their size, which is roughly what we expected. Now look at where that went.

Every gigabyte we freed went to the KV cache and nowhere else. It was the smallest slice on the card, so adding to it has leverage that adding to anything else wouldn't. The cache roughly tripled. The number of tokens it holds roughly tripled. And the concurrency figure vLLM prints at startup went up by about the same factor.

Which is worth stopping on, because it inverts why people think they quantize. We didn't shrink this model to make it fit. It already fit, comfortably, with room to spare. We shrank it to buy conversation space.

And because the cache was the smallest slice on the card, cutting the weights by a third bought roughly three times the users. That's the leverage. It's also why "will it fit" is the wrong question to ask about a GPU. Fitting is table stakes. What's left over afterwards is the thing you're actually buying.


Same load as before. Nothing else changed.

```sh
hey -n 100 -c 100 -t 0 -m POST \
    -H "Content-Type: application/json" \
    -d '{"messages": [{"role": "user", "content": "Explain what a KV cache is."}]}' \
    "http://silly-model.$INGRESS_HOST/v1/chat/completions"
```

The output is as follows.

```
Summary:
  Total:	241.7013 secs
  Slowest:	241.6988 secs
  Fastest:	104.3059 secs
  Average:	173.0435 secs
  Requests/sec:	0.4137

...

Latency distribution:
  10% in 125.6067 secs
  25% in 138.0466 secs
  50% in 173.2755 secs
  75% in 208.6301 secs
  90% in 224.8841 secs
  95% in 229.8510 secs
  99% in 241.6988 secs
```

Same hundred requests, same prompt, same card, and the whole lot finished in well under half the time it took before.

But look at where that came from, because it isn't what people assume. Compare the fastest request to the fastest from the previous run. It improved a little. Nothing like the improvement in the total.

And that small gain isn't the model running faster. We didn't make the GPU better at arithmetic, and a single conversation is limited by exactly that. What we changed is how many conversations fit in memory at once, so even the requests served first are elbowing fewer competitors out of the way. More of them ran in parallel, the queue drained quicker, and the whole batch finished sooner. The work didn't speed up. The waiting did.

One honest note before we move on. The concurrency figure vLLM printed roughly tripled, but the throughput we actually measured went up by less than that. You never collect all of a ceiling. Past a certain point the GPU runs out of arithmetic rather than memory, and then more cache space stops helping. Memory was our constraint, so fixing it bought a lot. It didn't buy everything.

So we tripled our capacity by changing two lines. Which raises the obvious question. What did that cost us?

## What Quantization Costs You

Not much, and I want to be careful here rather than dramatic. At **8 bits**, quality loss on standard benchmarks is a fraction of a percent for big models, and a percent or two for models this size. For chat, summarising, and following instructions, that's below the noise floor. You would not find it without a rigorous evaluation.

Where it does show up is multi-step maths, scientific reasoning, and **code generation**. Anywhere small errors compound instead of washing out.

Which is worth stopping on, because if you're watching this, code and agents are probably exactly what you intend to run on it. That isn't the forgiving category. It's the sensitive one. And we're running a reasoning model on top of that, so we're on the wrong side of the line twice over.

Nothing in this demo would reveal that, and pretending otherwise would be dishonest. It's exactly why people quantize casually, see no difference, and get bitten months later on the work that actually mattered.

That doesn't mean don't do it. Tripling your capacity for a percent or two is a good trade almost every time. It means measure it against your own work rather than trusting somebody else's benchmark, because the tasks you care about are the ones most likely to notice.

Which leads to a fair question. If it's this cheap, why isn't it on by default?


Hardware, partly. **FP8** arithmetic only exists on Ada Lovelace and Hopper cards and anything newer. Go back one generation to an A100 and there are no FP8 units on the die at all. vLLM will still take the model. It just stores the weights at eight bits and unpacks them back to sixteen before every multiplication.

Which is worth being precise about, because it's better news than it sounds. You still get the memory back. The weights really are half the size on the card, so the cache still grows and the ceiling still moves. What you don't get is the faster arithmetic, and you pay a little for the unpacking.

But the bigger reason it isn't a default is simpler. It changes what your model outputs. Not by much, as we just went through, but it changes it. No engine should quietly do that to you without being asked.

There's still a nice irony in the hardware, though. This **L4**, about the cheapest datacenter GPU you can rent, handles FP8 natively. An A100, which costs several times more, does not.

Which brings us to the question you're probably already asking. If 8 bits nearly tripled our capacity, why stop there? Why not 4 bits, or 2?

4 bits is fine, and it's what a lot of production deployments actually run. With the better methods it recovers almost all of the original quality, and it quarters your weights instead of halving them.

2 bits is where it falls apart. Accuracy drops off a cliff, not by a fraction of a percent but by several, and no amount of clever encoding has fixed that yet. Below 4 bits, reasoning is the first thing to go, and there's a decent rule of thumb doing the rounds: if you're running agents or tool calling, don't go below 4.


So the useful range is narrower than it looks. Sixteen down to eight is close to free. Eight to four is a real trade you should measure. Below four, don't.

## Quantizing The KV Cache


And there's one more lever I've deliberately left until now, because it's the most on-the-nose of all of them. Everything we just did was about the **weights**. You can also quantize the **cache itself**. One more flag, and it roughly **doubles** the number of tokens the card can hold, for very little accuracy cost.


Which, given that the cache is the exact thing we've spent this entire video running out of, is worth more than a mention.

```sh
diff demo/vllm-8bit.yaml demo/vllm-8bit-kvcache.yaml
```

The output is as follows.

```
42a43,44
>             - --kv-cache-dtype
>             - fp8
```


Two more lines, and this time they're aimed directly at the thing we're short of.

```sh
kubectl apply --filename demo/vllm-8bit-kvcache.yaml

kubectl --namespace inference rollout status deployment vllm
```

```
deployment "vllm" successfully rolled out
```

```sh
kubectl --namespace inference logs --selector app=silly-model --tail -1 \
    | grep --extended-regexp "loading took|Available KV cache|KV cache size|Maximum concurrency"
```

The output is as follows.

```
... Model loading took 9.06 GiB and 110.276965 seconds
... Available KV cache memory: 9.26 GiB
... GPU KV cache size: 134,864 tokens
... Maximum concurrency for 8,192 tokens per request: 16.46x
```


Look carefully at what did and didn't change.

The weights are identical, because we didn't touch them this time. The cache is still exactly the same number of gigabytes, because we didn't give it any more room either. What changed is how much fits inside it. The token count doubled and the concurrency figure doubled, on the same memory, because every token now takes half the space it did before.

Add it all up and we've gone from under three concurrent conversations to over sixteen. Same card, same model, four lines of configuration.


That's what the engine claims. Let's put the same hundred requests through it one more time.

```sh
hey -n 100 -c 100 -t 0 -m POST \
    -H "Content-Type: application/json" \
    -d '{"messages": [{"role": "user", "content": "Explain what a KV cache is."}]}' \
    "http://silly-model.$INGRESS_HOST/v1/chat/completions"
```

The output is as follows.

```
Summary:
  Total:	164.1475 secs
  Slowest:	164.1467 secs
  Fastest:	85.5845 secs
  Average:	132.2238 secs
  Requests/sec:	0.6092

...

Latency distribution:
  10% in 112.0489 secs
  25% in 121.3806 secs
  50% in 133.0707 secs
  75% in 143.5025 secs
  90% in 156.0854 secs
```

Faster again. The whole batch finished quicker than the last run, which itself finished in under half the time of the first. And look at the spread compared to where we started: instead of a long smear from fast to painfully slow, most requests now land close together. That's a queue that's mostly gone.

But be careful reading this one, because the numbers don't line up the way you'd want. We doubled the ceiling and throughput went up by less than half.

That gap is the interesting part. We've stopped being short of memory. Somewhere between the last change and this one, the constraint moved, and now the GPU is running out of arithmetic rather than room. More cache space keeps buying less, because cache space isn't what's holding us back any more.

## Destroy

That's a GPU sitting there costing money, so let's not leave it running.

```sh
./dot.nu destroy inference $PROVIDER
```

That removes the cluster and everything setup created around it. Nothing lingers, and nothing you already had gets touched.


+++
title = 'Why Self-Hosting AI Models Is a Bad Idea'
date = 2026-03-04T20:00:00+00:00
draft = false
+++

Open weight LLM models are bullshit.

Now, before you write an angry comment telling me I'm just another person who hates open source, let me be clear. Open weight is NOT open source. The licenses are screwed and can change for worse at any time. You can't use the truly powerful models unless you have money to burn. And you definitely can't build a sustainable business on them.

That is, if you want models that actually compete with the best. Not the toys that are fun to play with but aren't the real deal.

Did those statements just put me on top of your hate list? Are you already typing a furious response? Good.

I get it though. You think using AI is expensive. You want to save money, so you have a brilliant idea: self-host free models. Right?

Stick around because I'm going to show you the actual math. And by the end of this video, you might realize that the "open" in open weight is costing you more than the "closed" alternatives ever would.

<!--more-->

{{< youtube pWtDTkfNaUU >}}

## Self-Hosting LLMs: The True Cost

Let's say you don't want to pay **Anthropic**, **Google**, **Microsoft**, and other "evil corporations" for their latest models. You know better. You're going to host an LLM yourself. Right?

Take **Kimi K2.5**. It's a **trillion-parameter** open weight model that claims to match or beat **Claude Opus**. Benchmarks can easily be gamed, so I won't discuss the differences today. The point is that it's a very capable model, and for this story it doesn't matter whether it's better or worse than the competition. It's great.

All you need is a bit of hardware and that's it. You can rent it from a hyperscaler or host it on your own hardware. Right?

Here's what "a bit of hardware" actually means for **Kimi K2.5**. For realistic performance, you need **four NVIDIA H100 GPUs** with quantization. For full performance, you need **sixteen H100s**. The model takes up **595 gigabytes** on disk and needs **300 to 400 gigabytes** of VRAM during inference.

What does that cost in the cloud? **AWS** and **Google Cloud** charge around **three to four dollars per GPU hour** for H100s. Specialized providers like **RunPod** or **Lambda Labs** offer them for around **two dollars per hour**. Sounds reasonable until you do the math. Running four H100s 24/7 costs you roughly **$8,600 per month**. That's over **$100,000 per year** just for the GPUs. Want full performance with sixteen H100s? That's **$35,000 per month** or **$420,000 per year**. And that's before storage, networking, and egress fees.

Will that give you performance comparable to what you'd get from vendor APIs? No. But let's say you're willing to trade performance for the satisfaction of "sticking it" to those evil corporations. Are you willing to pay that much for the statement?

And good luck actually getting those GPUs. Even if you have the money, enterprise orders still face **four to eight month lead times**. **AWS** and **Azure** throttle access during peak usage. Smaller GPU clouds often have queue times, especially when startup credits drop.

What about buying your own hardware? An **H100** costs **$25,000 to $40,000**. You need four of them for realistic performance. Plus a server with **512GB RAM**, storage, cooling, and power. You're looking at **$150,000 to $200,000 upfront**.

Someone always suggests **Mac Studios**. Two **M4 Ultras** with **512GB unified memory** each can technically run **Kimi K2.5**. Cost: around **$20,000**. Performance: roughly **100 times slower** than H100s. Thunderbolt bandwidth becomes a massive bottleneck. And if you're thinking about **Mac Minis**, their **64GB maximum memory** doesn't even come close to what these models need.

And that's if you can even buy them. **NVIDIA** doesn't sell retail. You go through partners and OEMs with allocation priorities. Enterprise pre-orders often face months of waiting.

But let's say you got the hardware. Now you need someone to maintain it. Someone who understands GPU clusters, model serving, load balancing, and inference optimization. That's not your average DevOps engineer. That's specialized talent commanding **$150,000 or more per year**. Add electricity, cooling, rack space, and the inevitable hardware failures. Your $150,000 upfront investment just became a **$300,000 to $350,000 first year operation**, with **$150,000 or more every year** after that.

Now here's the punchline. The **Kimi K2.5 API** costs **$0.60 per million input tokens** and **$3 per million output tokens**. For the same throughput your self-hosted 4x H100 setup could handle, you'd pay roughly **$300 to $800 per month** through the API. Compare that to **$8,600 per month** for cloud GPU rental. Or **$300,000** in the first year if you buy your own hardware.

The API is **10 to 30 times cheaper** than self-hosting in the cloud.

But wait. That was for a company serving a team or customers. What if you're just a single user? Surely self-hosting makes more sense at that scale?

No. The costs are essentially the same. You still need **four H100s** to run the model. Whether you're serving one person or a thousand, the hardware requirements don't change. The model doesn't get smaller because you're the only one using it. You'd be spending **$8,600 per month** in cloud fees, or **$300,000** in the first year for your own hardware, just to serve yourself.

And here's the kicker. GPU technology moves fast. The H100 you buy today will be outperformed by next year's chips at a fraction of the cost. By the time your investment even starts to pay off, your hardware is already obsolete. You're not buying an asset. You're buying a depreciating liability.

Okay, wait a minute. We don't need the biggest and meanest models. Smaller models work well. They're cheaper to run. They do what we need them to do.

Fair point. Let's talk about smaller models. Models like **Mistral 7B**, **Microsoft Phi-3**, **Qwen3**, or **SmolLM3**. These run on consumer hardware. A decent GPU with **8 to 16 gigabytes** of VRAM can handle them. You could even run some on a Mac with **32GB** of unified memory.

Let's do the same cost exercise. An **RTX 4090** costs around **$1,600**. It can run a **7B model** at thousands of tokens per second. Add a decent PC build and you're looking at maybe **$3,000 to $4,000** total. That's a lot more reasonable than $150,000.

What if you're an individual, not a company? A **Mac Mini M4 Pro** with **64GB unified memory** costs around **$1,400**. It can comfortably run **7B to 32B models**. Silent, low power, sits on your desk. For personal use, it's actually a decent setup.

But what about API costs for these smaller models? They're also cheap. **Haiku-class models** cost fractions of a cent per request. You could make thousands of API calls per day for less than **$50 per month**. Your **$4,000** hardware investment would take years to pay off in API savings. And by then, you'd need to upgrade anyway.

## Why AI APIs Are Unbeatable Right Now

But here's what most people don't realize. These API prices are artificially cheap. The companies providing them are losing money to capture you as a customer.

American companies like **OpenAI** and **Anthropic** are burning through billions in venture capital. **OpenAI** lost around **$9 billion in 2025**. **Anthropic** lost **$3 billion**. They've raised nearly **$100 billion** combined, and they're spending it hoping you'll get locked into their ecosystem.

Chinese companies have it even better. **Moonshot AI** has raised over **$1.8 billion** from **Alibaba**, **Tencent**, and others. **Alibaba** pledged **$53 billion** over three years for AI infrastructure. On top of private investment, the Chinese government is pouring in money. A **$70 billion** chip incentive package. Provincial subsidies. Billions in computing hubs.

Here's the thing though. The lock-in strategy isn't really working. If you avoid proprietary features like fine-tuning on their platforms or using platform-specific APIs, switching providers is easy. When **Claude 4** launched, users switched within weeks. Multi-model strategies are increasingly common. Use **Anthropic** for code, **OpenAI** for retrieval, **Gemini** for multimodal.

So take advantage of the subsidies. Use their cheap APIs. Just don't do anything silly that locks you in. When prices rise or a better option appears, you can switch.

## Open Weight Is Not Open Source

Now, what about those open weight licenses? Surely there's value in having access to the model weights?

There is. You can inspect the model. You can run it locally if you have the hardware. You can fine-tune it for your specific use case. That's real value.

But here's the thing. Open weight is not open source. The **Open Source Initiative** looked at **Meta's Llama** license and said it "fails in spectacular ways at granting basic rights." The **Free Software Foundation** classified it as non-free.

The restrictions are real. **Llama** blocks companies with over **700 million monthly active users**. **Llama 4**? Blocked entirely in the **European Union**. You can't use model outputs to train competing models. And here's the kicker: **Meta** can update the acceptable use policy whenever they want.

And here's the part that should worry you if you're building a business. These licenses can change. What's permissive today might not be tomorrow. You have no guarantee that the terms you agreed to will stay the same.

Now, if you absolutely must run inference yourself, **Kubernetes** is the way to go. But I don't think self-hosting makes sense unless you're special. And by special, I mean organizations like **CERN**, or companies with massive scale and existing GPU infrastructure, or situations where data absolutely cannot leave your premises.

So use those providers. Take advantage of the money that venture capitalists and governments are giving you through subsidized pricing. It won't last forever, but while it does, there's no reason not to benefit from it. Think about alternatives when the subsidies dry up.

Look, I'm not saying you shouldn't use open weight models like **Kimi** or **Qwen**. Use them. They're excellent. I'm saying you shouldn't waste your money hosting them yourself when you can access them through cheap APIs.

The situation might change in the future. Inference costs are dropping. We might eventually get truly open source models that don't suck. Models might enter foundations that protect users from license changes. Hardware might become cheap enough that self-hosting makes sense for smaller organizations.

Many things could change. But not today.

## The Bottom Line

The math is clear. Use the APIs.

Self-hosting a **trillion-parameter model** costs **$100,000 or more per year**. Using the API for the same model is **10 to 30 times cheaper**. Even for small models on a **$1,400 Mac Mini**, API savings won't pay off for years.

And if you really want to "stick it" to big corporations? Do it by taking the subsidies they're giving you. They're losing billions hoping to capture you as a forever customer. Take their cheap APIs. Don't lock yourself in. And leave the moment the situation changes and self-hosting starts making sense. If that moment ever comes.


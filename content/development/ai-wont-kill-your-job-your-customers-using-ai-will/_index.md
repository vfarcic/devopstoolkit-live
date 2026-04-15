
+++
title = "AI Won't Kill Your Job. Your Customers Using AI Will"
date = 2026-04-20T15:00:00+00:00
draft = false
+++

Over a trillion dollars in software market cap vanished in a single month. Software companies are laying off thousands. Analysts are calling it the SaaSpocalypse. And yet, software engineering jobs are at multi-year highs. What the hell is going on?


If you're a software engineer and you think your job is at risk because of AI, you're right. But probably not for the reasons you think. If you work at a company that *sells* software, yeah, you should be worried. But if you work at a bank, an insurance company, a retailer, a manufacturer, any company that *uses* software rather than selling it, AI might actually make your job more secure than ever. Your company is likely about to build a lot more software than it ever has before.

<!--more-->

{{< youtube Z-ZnXPlrZME >}}

The numbers back this up. JPMorgan Chase boosted its 2026 tech budget to **$19.8 billion**, with $1.2 billion of the increase specifically targeting AI. Walmart is offering tech roles paying up to **$370,000**. Software engineering roles are at multi-year highs globally. The Bureau of Labor Statistics projects software developer jobs growing **15% through 2034**, "much faster than average." Satya Nadella has referenced the Jevons Paradox to argue that as AI gets more efficient, demand for it skyrockets, not shrinks. The demand for engineers is increasingly coming from non-tech companies. Banks, retailers, manufacturers.

But to understand why that matters, we need to talk about why companies buy software in the first place. We, software engineers, don't buy software because we can't build it ourselves. Given enough time and people, everything can be done. We buy software because it's cheaper to buy than to build. That's a crucial distinction. It's very different from organizations that don't have software engineers. They buy software because they literally cannot build it themselves. We buy it because the math works out. And **that math is about to change**.

Here's how a typical vendor conversation has worked until now. The vendor says: "We have enterprise features on top of the open source project you're using. We know you need them. You'll pay us $100k a year." You say: "That's too expensive." The vendor asks: "How much would it cost you to build all this yourself?" You do the math and say: "$1 million." And the vendor smiles: "See? $100k a year doesn't sound that bad. Experts built those features. You need them. It would cost you ten times more to build it yourself. You get support when you need it. And if something breaks, you can blame us instead of being blamed yourself. It's a no-brainer."

And that deal has worked well enough, even though there are real downsides. When we buy software, we never get something tailor-made for our needs. We pick whatever is closest to what we need. That's one of the reasons open source became so popular. We try a project, it fits our needs, and we adopt it. It's free, but it's often missing key features, usually around security and scale. So we know we want it, we've already adopted the open source version, but we need more. We buy that "more" from vendors. Not because we can't build those missing features ourselves, but because it's cheaper to buy what we're missing.


Here's where it gets interesting. AI is making building software cheaper. Not a little cheaper. A lot cheaper. And it's getting cheaper fast. To show you just how fast, let me give you two data points. In mid-2025, the most rigorous study available, from METR, found that experienced developers using AI tools were actually 19% slower than without them, while believing they were 20% faster. That was less than a year ago. By early 2026, AT&T reported building an internal data product in 20 minutes using AI that would have taken 6 weeks without it. They deployed AI to over 100,000 employees, now processing billions of tokens per day, with active adopters reporting massive productivity gains. In under a year, we went from "AI actually slows you down" to "AI cuts weeks of work to minutes." According to Stack Overflow, 84% of developers now use or plan to use AI tools, with about half using them daily. Models are improving rapidly. Costs per token are collapsing, roughly 50x per year. That speed of improvement is the point. Whatever AI can do today is the worst it will ever be.


And we're already seeing the early signs of what's coming. The stock market is paying attention. In February 2026, over $1 trillion in software market cap was erased in a single month. The S&P North American Software Index had its worst monthly decline since 2008. Salesforce lost 26%. HubSpot fell 39%. Atlassian dropped 35%. Chegg, the education platform, went from a $100 stock to $0.44, with revenue down 49% year over year and a 79% probability of bankruptcy. For the first time in the modern era, software now trades at a discount to the S&P 500. Forrester declared "SaaS as we know it is dead." Palantir's CEO said many SaaS companies are "in danger of becoming irrelevant," and that single statement triggered a $300 billion sell-off in one day. These aren't hypotheticals. This is happening right now.


And it's not just stock prices. Companies are already acting on this. Retool's February 2026 survey of over 800 companies found that 35% have already replaced at least one SaaS tool with a custom build, and 78% expect to build more of their own tools this year. Klarna is the most high-profile example. They eliminated roughly 1,200 SaaS apps, ditched Salesforce and Workday, and built an internal AI-driven stack. Now, the reality turned out messier than the headlines. Their CEO later admitted "we didn't replace SaaS with an LLM." They actually replaced some SaaS with other SaaS. They even rehired humans after customer satisfaction dropped. But that's exactly the point. We're in the messy, early phase. The attempts are imperfect. The direction, however, is clear. And the economic logic is inescapable.

So here's the question that should keep every software vendor up at night. If software development is getting cheaper and faster, can they maintain the same prices? Not "can they today," but "can they in two or three years, when AI coding agents are dramatically better than they are now?"

Let's go back to that vendor conversation, but imagine it's a couple of years from now.

The vendor says: "We have enterprise features on top of the open source project you're using. We know you need them. You'll pay us $100k a year." You say: "That's too expensive." The vendor asks: "How much would it cost you to build all this yourself?" And this time, you say: "$100k to build it, and $50k a year to maintain it. We'll make it tailor-made for our needs. We'll include all the features we need, including the ones you don't have. They'll work exactly as we need them to work. And we'll skip the ones you're offering that we don't need. I'm not paying you $100k every year for that." The vendor pauses and says: "Thank you for this lovely conversation. We'll try to find someone else to sell to."


Now, some people will argue that the bottleneck was never writing code. It's knowing what to build. Andrew Ng has said the bottleneck in AI startups "isn't coding, it's product management." And that's true today.

Senior developers already spend less than a third of their time actually writing code. AI optimizes the minority of their time. But here's the thing. AI agents are getting better at understanding context, at maintaining codebases, at handling the integration and deployment work that currently eats up most of the effort. That bottleneck is shifting too. 65% of software costs happen after deployment, in maintenance. That's the next frontier. When AI gets good at that, and it will, the build-vs-buy math changes completely.


So what's the future for software vendors? What options do they have?


One option is to increase the feature set. Offer so much more value that the gap between what you can build yourself and what the vendor provides stays wide enough to justify the price. But is that realistic? Are there order of magnitude more features one might need for a given piece of software? Bain & Company identifies the most vulnerable category as what they call "Battlegrounds," products with high automation potential and high AI penetration. Think basic support tools like Intercom, invoice processing like Tipalti, time-entry approvals like ADP. The most resilient are what Bain calls "Core Strongholds," products with deep domain knowledge and regulated data flows. But most vendors aren't in that category.

Another option is to decrease the price. But most software companies are funded by VCs or are publicly traded. They need to show growth to their shareholders. Decreasing the price is not an option. And the seat-based model that most of them rely on is already collapsing. SaaStr put it bluntly: "If 10 AI agents do the work of 100 sales reps, you don't need 100 Salesforce seats." Companies are already replacing entire SDR teams with AI agents. Gartner's February 2026 forecast projects worldwide IT spending at $6.15 trillion, up 10.8% year over year. But the money is flowing to AI-native companies, not legacy SaaS. Enterprise software spending is growing 14.7% to over $1.4 trillion. And Gartner forecasts that 40% of enterprise SaaS contracts will include outcome-based elements by the end of 2026, up from single digits today. The per-seat pricing model is dying.


What about cutting headcount? Fire three-quarters of your people, decrease costs, and keep profits high even with lower prices. That's already happening. Block, the company behind Square and Cash App, cut 4,000 roles, 40% of its workforce, in early 2026. Atlassian cut 1,600 jobs. Workday announced 8.5% layoffs. CFOs privately admit AI layoffs will be 9x higher in 2026 than in 2025. But that's a short-term play at best. It doesn't solve the structural problem of needing ever-increasing profits to keep investors happy.

Or maybe there is no solution for many of them. Maybe a significant number of software vendors will simply go extinct, surviving only by selling to non-technical users who don't know how to build what they need, with or without AI.


Maybe only those with true moats will thrive. Think about it. Even if I build a competitor to YouTube, I still won't have a gazillion new videos uploaded every day. Even if I build something similar to AWS services, I still don't have the physical infrastructure to run them. Even if I build something equivalent to Kubernetes, the entire ecosystem of projects built on top of it wouldn't work in mine. Morningstar's analysis confirms this. Network effects are one of the most durable moat types in the AI era. Switching costs, on the other hand, are weakening because AI can automate migrations. There's even what analysts are calling an "Atoms over Bits" rotation. Physical infrastructure moats are becoming more valuable as software gets commoditized. In early 2026, over $1 trillion evaporated from the software sector while energy stocks surged 25%.

But here's the uncomfortable truth for most vendors. Most of them don't have moats like that. We might not need them anymore. Or if we do, the price we're willing to pay might change drastically.

Now, to be fair, this isn't a prediction that all software vendors will die tomorrow. Some are thriving. Palantir's revenue soared 56% year over year. CrowdStrike posted its first positive quarterly GAAP net income. The companies building AI infrastructure and tools are doing great. But they're the exception, not the rule. The broad trend is clear. AI is making software cheaper to build. The trajectory points to it getting dramatically cheaper still. And most vendors built their entire business on one assumption: that building would always be more expensive than buying. That assumption has an expiration date. The only question is how soon.

So, back to where we started. If you're a software engineer and you're worried about AI taking your job, ask yourself one question. Does your company *sell* software, or does it *use* software? If you work at a bank, a retailer, a manufacturer, you're probably fine. You might even be busier than ever. But if you work at a software vendor that sells tools other companies are learning to build for themselves, that's a different conversation. Your job isn't at risk because AI can write code. **Your job is at risk because AI can help your customers write code.**

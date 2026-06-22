+++
archetype = "home"
title = ""
+++

# Latest Posts

<a href="/infrastructure-as-code/infrastructure-with-ai-agents-for-dummies"><img src="/infrastructure-as-code/infrastructure-with-ai-agents-for-dummies/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Infrastructure with AI Agents for Dummies](/infrastructure-as-code/infrastructure-with-ai-agents-for-dummies)


AI agents are amplifiers. If you're good at your job, agents make you better. You do more great things, faster. But if you're bad at your job, agents amplify that too. Where you used to cause a slow trickle of shit, now you have the means to unleash a full-blown **shitstorm**, at scale, in minutes.

Now, AI is all the rage these days, and for good reason. So of course people are using agents to manage real resources: infrastructure, databases, applications, all of it. The question is what happens when they do. That's what we're looking at today: an agent managing actual cloud resources, what goes wrong, why it goes wrong, and what it takes to make it work properly.

**[Full article >>](/infrastructure-as-code/infrastructure-with-ai-agents-for-dummies)**


---

<a href="/development/why-one-ai-agent-is-never-enough"><img src="/development/why-one-ai-agent-is-never-enough/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Why One AI Agent Is Never Enough](/development/why-one-ai-agent-is-never-enough)


When I used to give an AI agent a task, it would finish in one go. Write the code, declare victory, done. Now, with th

e setup I'm about to show you, the same task can take tens of iterations before the work is considered finished. The output is **dramatically better**, and I'm spending **less time** on it, not more.

The reason is that there's no longer a single agent doing the work. There's a team. One agent writes the code. Another reviews it. A third audits it for security. A fourth ships it. They run on different models, with fresh context each time, and they push back on each other until the work actually holds up. I just play games until something genuinely needs me.

In this video, I'll walk through what that pipeline looks like, 

why each role exists, and how I run all of it end-to-end. By the end, you'll have a complete picture of how to set this up yourself, and a slightly uncomfortable realization about what your job becomes when the agents do the coding.

**[Full article >>](/development/why-one-ai-agent-is-never-enough)**

---


<a href="/ai/why-ai-code-review-goes-first-and-humans-go-second-feat-coderabbit"><img src="/ai/why-ai-code-review-goes-first-and-humans-go-second-feat-coderabbit/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [Why AI Code Review Goes First (And Humans Go Second) (feat: CodeRabbit)](/ai/why-ai-code-review-goes-first-and-humans-go-second-feat-coderabbit)



Code review was the safety net. The last check before something shipped. The place where bad ideas got caught, sloppy work got pushed back, and someone with fresh eyes made sure the change actually made sense.

On most teams, that net is breaking. Not because reviewers got worse. Not because standards dropped. Something fundamental about how code gets written changed, and the review process never caught up.


You can feel it if you've been paying attention. Pull requests sitting open for days. Approvals coming back so fast nobody could have read the diff. Small mistakes slipping through that would've been caught two years ago. The cracks are showing.

In this video, I'll show you what's actually breaking and why, the workflow that closes the gap, and the specific tool I use on every pull request to make it real.

**[Full article >>](/ai/why-ai-code-review-goes-first-and-humans-go-second-feat-coderabbit)**

---



<a href="/ai/how-i-access-every-ai-model-without-the-lock-in"><img src="/ai/how-i-access-every-ai-model-without-the-lock-in/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [How I Access Every AI Model Without the Lock-In](/ai/how-i-access-every-ai-model-without-the-lock-in)

New models keep dropping all the time, and I want to try them all. I want to see which one is better for which tasks, which one is cheaper, which one is faster. There's [OpenAI](https://openai.com), [Anthropic](https://anthropic.com), [Google Gemini](https://gemini.google.com), [Moonshot AI](https://moonshot.ai), [xAI](https://x.ai), and the list just keeps growing. I do not want to be locked into one model or one family of models forever. So I got subscriptions to some, I'm paying for API access to others, and then there are models that I would typically have to host myself. I don't want to do that. Even if I did, I neither have the hardware nor the patience for it.

**[Full article >>](/ai/how-i-access-every-ai-model-without-the-lock-in)**

---



<a href="/observability/i-stopped-staring-at-dashboards-ai-reads-my-grafana-metrics-now"><img src="/observability/i-stopped-staring-at-dashboards-ai-reads-my-grafana-metrics-now/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [I Stopped Staring at Dashboards. AI Reads My Grafana Metrics Now.](/observability/i-stopped-staring-at-dashboards-ai-reads-my-grafana-metrics-now)


Something just broke in production. An alert fired. You open Grafana, click through three dashboards trying to find one that matches what's actually happening, and twenty minutes later you're still squinting at panels and switching tabs to grep logs.

That entire workflow is about to disappear.


AI agents can now read your metrics, your logs, and your traces directly. They draw conclusions. They build custom dashboards on the spot. They tell you what's wrong while you're still typing the question. And the best part: **you don't have to leave your terminal to do any of it.**

In this video, I'll show you how. We'll start with Grafana Assistant inside the [Grafana](https://grafana.com) UI, then move to [Claude Code](https://claude.com/claude-code) wired up to the Grafana MCP server. By the end, you'll see how to query observability data from a chat prompt, build dashboards fitted to whatever you're investigating, and connect your own agents to live runtime data, all without touching a browser.

**[Full article >>](/observability/i-stopped-staring-at-dashboards-ai-reads-my-grafana-metrics-now)**

---




<a href="/ai/the-4-modes-of-ai-coding-and-why-your-tool-picks-itself"><img src="/ai/the-4-modes-of-ai-coding-and-why-your-tool-picks-itself/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [The 4 Modes of AI Coding (And Why Your Tool Picks Itself)](/ai/the-4-modes-of-ai-coding-and-why-your-tool-picks-itself)



Working with AI agents is management. Most people think the IDE versus TUI debate is about tools. It is not. It is about management style. And like any good manager, you adjust based on two things. How well can you specify the task? And how much does the situation warrant trust? The worse you are at specifying and the less trust is warranted, the more you need to intervene.


I think of it as four modes of working with AI agents. Four management styles. Mode 1: "I trust you only to complete what I started." Mode 2: "I need to review every single thing you do." Mode 3: "I observe what you are doing and intervene when needed." Mode 4: "I trust you with this task." These are not ranked from worst to best. They are different styles for different situations.

Both IDEs and TUIs can operate in all four modes. But each has modes where it is native and modes where it is fighting its own architecture. And that gap widens as autonomy increases. I used to be IDE-only. Then I combined IDE and TUI. Now I use TUI exclusively. That was not a tribal choice. It followed my shift through these four modes. As my center of gravity moved toward higher autonomy, the IDE stopped being the right tool for most of what I do.

So let me walk you through each mode, how IDE and TUI handle it differently, and where each paradigm hits its structural limits.

**[Full article >>](/ai/the-4-modes-of-ai-coding-and-why-your-tool-picks-itself)**

---





<a href="/ai/how-i-hooked-ai-video-generation-into-my-dev-workflow-with-higgsfield"><img src="/ai/how-i-hooked-ai-video-generation-into-my-dev-workflow-with-higgsfield/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [How I Hooked AI Video Generation Into My Dev Workflow (with Higgsfield)](/ai/how-i-hooked-ai-video-generation-into-my-dev-workflow-with-higgsfield)

Real productivity out of an AI agent starts with where it lives. Agents that run inside your terminal, your IDE, or your desktop beat agents that sit behind a web UI. [Codex](https://github.com/openai/codex), [Cursor](https://cursor.com), [Claude Desktop](https://claude.com). The surface around the model is what decides how much work you actually get out of it.

And it's not just code. Calendars, email, web research, summarizing long PDFs, drafting writing. All of it routinely happens inside the same agent session, through MCP connectors that wire the agent into Gmail, Google Calendar, Notion, Slack, and whatever else you've plugged in.

**[Full article >>](/ai/how-i-hooked-ai-video-generation-into-my-dev-workflow-with-higgsfield)**

---





<a href="/ai/i-built-a-tool-to-manage-multiple-ai-agents-at-once"><img src="/ai/i-built-a-tool-to-manage-multiple-ai-agents-at-once/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [I Built a Tool to Manage Multiple AI Agents at Once](/ai/i-built-a-tool-to-manage-multiple-ai-agents-at-once)

Running multiple AI agents in parallel sounds like the ultimate productivity hack. Two agents, five, a dozen, all grinding away on different features at the same time. But making that actually work changes more than you'd expect. Not just the tooling. The way we work changes too.

In this video, I'll walk through what that shift looks like, lay out the requirements for the kind of tool that can actually support it, and then give you a hands-on tour of the one I built after nothing else got it right.

**[Full article >>](/ai/i-built-a-tool-to-manage-multiple-ai-agents-at-once)**

---





<a href="/ai/mcp-is-burning-your-tokens-before-you-ask-a-single-question"><img src="/ai/mcp-is-burning-your-tokens-before-you-ask-a-single-question/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [MCP Is Burning Your Tokens Before You Ask a Single Question](/ai/mcp-is-burning-your-tokens-before-you-ask-a-single-question)

Every MCP tool you connect burns tokens before you even ask a question. Tool names, descriptions, parameter schemas, all of it gets stuffed into your context window on every single turn. Connect a few servers and you've lost thousands of tokens to tools you might never call. That's the reality of the MCP protocol. It's a tax on your agent's intelligence.

Now, don't get me wrong. MCP solves a real problem. It gives us standardized discovery and zero client installation. But there's an alternative that uses a fraction of the context, costs nothing extra to set up, and your agent might already know how to use. It's not perfect either, but I think it's the better choice for most real-world scenarios.

In this video, we're going to put MCP to the test, run some real operations through it, and then explore that alternative side by side. We'll look at where each one wins, where each one falls short, and by the end, you'll have a clear picture of which approach fits your situation.

**[Full article >>](/ai/mcp-is-burning-your-tokens-before-you-ask-a-single-question)**

---





<a href="/development/ai-wont-kill-your-job-your-customers-using-ai-will"><img src="/development/ai-wont-kill-your-job-your-customers-using-ai-will/thumbnail.jpg" style="width:50%; float:right; padding: 10px"></a>

## [AI Won't Kill Your Job. Your Customers Using AI Will](/development/ai-wont-kill-your-job-your-customers-using-ai-will)

Over a trillion dollars in software market cap vanished in a single month. Software companies are laying off thousands. Analysts are calling it the SaaSpocalypse. And yet, software engineering jobs are at multi-year highs. What the hell is going on?

If you're a software engineer and you think your job is at risk because of AI, you're right. But probably not for the reasons you think. If you work at a company that *sells* software, yeah, you should be worried. But if you work at a bank, an insurance company, a retailer, a manufacturer, any company that *uses* software rather than selling it, AI might actually make your job more secure than ever. Your company is likely about to build a lot more software than it ever has before

**[Full article >>](/development/ai-wont-kill-your-job-your-customers-using-ai-will)**

---


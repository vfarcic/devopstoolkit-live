
+++
title = "The 4 Modes of AI Coding (And Why Your Tool Picks Itself)"
date = 2026-05-18T15:00:00+00:00
draft = false
+++



Working with AI agents is management. Most people think the IDE versus TUI debate is about tools. It is not. It is about management style. And like any good manager, you adjust based on two things. How well can you specify the task? And how much does the situation warrant trust? The worse you are at specifying and the less trust is warranted, the more you need to intervene.


I think of it as four modes of working with AI agents. Four management styles. Mode 1: "I trust you only to complete what I started." Mode 2: "I need to review every single thing you do." Mode 3: "I observe what you are doing and intervene when needed." Mode 4: "I trust you with this task." These are not ranked from worst to best. They are different styles for different situations.

Both IDEs and TUIs can operate in all four modes. But each has modes where it is native and modes where it is fighting its own architecture. And that gap widens as autonomy increases. I used to be IDE-only. Then I combined IDE and TUI. Now I use TUI exclusively. That was not a tribal choice. It followed my shift through these four modes. As my center of gravity moved toward higher autonomy, the IDE stopped being the right tool for most of what I do.

So let me walk you through each mode, how IDE and TUI handle it differently, and where each paradigm hits its structural limits.

<!--more-->

{{< youtube 7ME4R__IlLg >}}

## Mode 1: AI Assists Your Writing

The first wave of AI coding tools was all about autocomplete. You type, the AI predicts what comes next, and you hit Tab to accept. That was it. And for that, IDEs were the perfect home.

An IDE agent lives inside your editor, hooked directly into the editing surface. It sees your cursor position, your open tabs, your file tree. It has access to LSP diagnostics, type information, hover data. All of that happens passively, in the background, without you asking for anything. The agent knows what you are looking at without being told.


So when you type a function signature, it already understands the types, the surrounding code, and the patterns in your project. It offers ghost text. You hit Tab. Inline suggestions, near-instant feedback loops, visual diffs right in the editor. Autocomplete on steroids.

This is what I call Mode 1. You write the code, the AI assists. And for this mode, IDEs are the clear winner.

It works because nothing about your workflow actually changes. You are still writing code. You are still driving. The AI just makes you faster. It is a **flow state** amplifier. The learning curve is basically zero. Install an extension, keep coding.

Now, what about TUI agents? TUI agents are a completely different thing. They run in the terminal as standalone processes. You give them a prompt, they read files, execute shell commands, and return results. They are built for conversation and delegation, not for completing the line you are halfway through typing. There is no TUI equivalent to tab-complete mid-keystroke. TUI agents simply do not compete in Mode 1. They start at Mode 2.

If what you need is to enhance your own coding, to write faster and with fewer typos, this is the mode for you. And for this mode, IDE is the only real option.


But Mode 1 has a ceiling. You are still the bottleneck. Every line of code goes through you. You are limited to micro-tasks. A function here, a test there. Remember the management framing? In Mode 1, you are not really a manager at all. You are the one doing the work, with a very eager assistant looking over your shoulder.

I stayed here for a while. IDE was the only tool that made sense for it. But eventually I hit that ceiling. I wanted the AI to do more than finish my sentences. The only way to remove that bottleneck is to stop being the coder and start being the manager. And if you are a coder transitioning to management, you know exactly how the work should be done, you have strong opinions about it, and you have a hard time letting go. So you are going to be a distrustful manager at first. You will want to review every single thing the AI does.

That is Mode 2.

## Mode 2: AI Executes, You Approve Everything


In Mode 2, you stop writing code yourself and start telling the AI what to write. You specify the task, the AI executes it, and you approve every step. You are the micromanager. And you micromanage for two reasons. You do not trust the AI to do it right. But you also do not trust yourself to explain what needs to be done well enough. You have been the one writing the code. You know how many edge cases and implicit decisions go into every function. Now you have to articulate all of that in a prompt, and you are not confident you can. So you compensate by reviewing everything, catching what your spec missed.

Both IDEs and TUIs work in Mode 2, but they work very differently.


In an IDE, you describe a task in a chat panel or an inline prompt. The agent proposes changes and presents them as visual inline diffs. Green and red highlighting, accept and reject buttons. You see exactly what will change before it happens. The editor's diff view is your approval interface.


In a TUI, you describe the same task in the terminal. The agent plans, then asks permission before each action. File edits, shell commands, everything stops and waits for your yes or no. The feedback is binary. Exit code zero means it worked. Anything else means it did not.

For this mode, IDEs have a genuine advantage. Visual diffs are excellent when you do not trust what is happening. You see the before and after side by side. Green lines added, red lines removed. And IDEs can show you changes across multiple files at once, so you get the full picture before you approve.

TUIs work differently. Everything is sequential. One file edit, approve. One shell command, approve. Then the next. You never see the full picture at once. That is slower and more tedious, but each approval is a deliberate decision. You cannot accidentally approve a batch without examining it. The sequential nature forces attention.

So it is not about which one is better. It is about what kind of micromanager you are. If you want the overview to verify the overall direction, IDE is better for that. If you distrust yourself to catch problems in a batch review, the forced sequentiality of TUI is actually a feature. In Mode 2, both paradigms hold up. Neither is a clear winner.


But here is the problem with the micromanager approach, regardless of which tool you use. You burn out. After enough approvals, you stop actually reviewing and start rubber-stamping. You become a confirmation service. Yes, yes, yes, approve, approve.

And the irony is brutal. An agent where a human approves every step carries similar risk to a fully autonomous agent when that human is just clicking through. You think you are being careful. You are not. **Supervision does not scale.** The micromanager burns out.

This was my second stage. I moved almost everything to Mode 2 with occasional Mode 1 for small edits. Still IDE-only at this point. It worked well until my tasks got bigger and the approval fatigue hit. I caught myself clicking approve without reading. And I started wondering, what if I just let it run and watched instead of approving every single step? What if I became a less controlling manager?

That is Mode 3.

## Mode 3: AI Works, You Watch

In Mode 3, you have learned to delegate. Not just because the approval fatigue wore you down, though that played a part. After all those reviewed tasks in Mode 2, you built a mental model of what the AI is good at, what it struggles with, and what it should never do unsupervised. At the same time, your prompts got better. You got better at specifying what you want. So Mode 3 is driven by three things. Approval fatigue pushing you away from Mode 2. A calibrated understanding of where the AI delivers and where it does not. And earned trust in yourself as a delegator.

And that understanding is what changes the permissions. In Mode 2, everything required approval. Now you start loosening selectively, based on what you have seen work. You can commit without asking me, but do not push without my approval. You can edit files freely, but ask before running destructive shell commands. Auto-accept for routine operations, manual review for anything that touches production. You are not approving every step any more. You are setting policy and watching for deviations.

Now, how do IDEs and TUIs handle this differently? Mode 3 is primarily a delegation and monitoring surface. You need a way to say "do this" and then watch what happens. IDEs are designed primarily as an editing surface. They can delegate and monitor, but those are secondary capabilities bolted onto an editing tool. The core strength of the IDE, the thing it was built around, is the thing you are barely using in Mode 3.

TUIs are naturally a delegation and monitoring surface. You type a command, that is delegation. You watch the output, that is monitoring. That is what a terminal has always been.



And because you are no longer bottlenecked on approvals, something else becomes practical. Running more than one agent at a time. A micromanager can only manage one person. A manager who delegates can oversee a small team. You check in on one agent, redirect if needed, move to the next. Each agent runs for a while before needing your attention, so you rotate between them.


TUI handles this more naturally. Multiple terminal windows or tmux panes, each running a different agent in its own process. Isolated worktrees keep parallel agents from stepping on each other. In an IDE, you are working within a single editor window. Running multiple agents in parallel means adding background features on top of a tool that was designed for one person editing one project.

But here is the honest problem with Mode 3, regardless of the tool. Watching is not reviewing. And the data backs this up.


Faros AI's analysis of over 10,000 developers found that AI adoption correlates with **9% more bugs**, 91% longer code review times, and 154% larger pull requests. So ask yourself honestly. Are you actually reviewing what your agents produce, or are you just watching tokens scroll by?

This is where TUI composability becomes a real advantage. You can build guardrails into the process rather than relying on your own attention. Lifecycle hooks trigger scripts before and after agent actions. Stdout pipes into other tools for automated checks. CI/CD integration verifies what the agent produced. You are not just watching any more. You are **building systems that watch for you**.

This is where I started using TUI. I still used IDE for some cases, but over time TUI became the right choice for most of what I did. Eventually I stopped switching back to IDE entirely. Not because TUI is better at everything. It is not. But it is more convenient to stick with one tool that is the right fit for 90% of your work and still works well for the remaining 10%, than to constantly switch back and forth.

And once you are comfortable building systems that watch for you, you start asking the next question. What if I did not have to watch at all?

That is Mode 4.

## Mode 4: High Autonomy, You Review Results

Mode 4 is not fire-and-forget. Let me be clear about that. The AI handles the entire implementation, but you still review the outcome. The difference between Mode 3 and Mode 4 is not watching versus not watching. It is about when you engage. In Mode 3, you watch during execution and redirect in real time. In Mode 4, you engage after execution. The agent runs, finishes, and presents you with a result. You evaluate the result, not the process.


In management terms, Mode 3 is the manager who walks the floor. Mode 4 is the manager who reads the reports. In some cases, like nightly CI jobs, scheduled scans, or automated test generation, it is truly autonomous with no human review of individual runs. But for most development work, you are the executive who reads the final report and decides what ships.

So what do IDEs and TUIs look like at this level? In an IDE, you assign an issue. The agent creates a pull request. You look at the PR when it is done. The IDE is the review surface. But the IDE itself adds nothing to the execution. It is not editing. It is not delegating in real time. It is not monitoring. It is just where you look at the result.

In a TUI, the agent runs headless. No human at a screen, no GUI required. You send a prompt, it executes, checks success criteria, and iterates until it is done. It can run overnight, in CI/CD, as a cron job, embedded in a pipeline. Unix composability is the foundation. Pipe output into other tools, chain agents together, script entire workflows. Lifecycle hooks intercept agent behavior at every stage.

At this level, both paradigms become task submission interfaces. The IDE advantage disappears. You are submitting a task and reviewing the result. That is the same workflow in both tools.

But here is the catch. High autonomy only works with precise specifications. **Context engineering** has displaced prompt engineering as the critical skill. It is not about how you phrase the prompt any more. It is about what information you give the agent to work with. And the gap between what agents can do on clean, well-scoped tasks and what they can do on messy real-world problems is still enormous. Ambiguous requirements multiply across parallel runs. The executive who gives vague direction gets vague results.

And this is where the IDE stops making sense. Not because it cannot do Mode 4. It can. But in Mode 4, most of what makes an IDE an IDE is irrelevant. You do not need the editor. You do not need the file tree. You do not need visual diffs or syntax highlighting. All of that becomes noise when your workflow is submit a task, wait, review the result. TUI does not win Mode 4 because it has features the IDE lacks. It wins because **TUI does not carry features you do not need**.

This is where I spend most of my time now. Modes 3 and 4, with occasional Mode 2 when I am working on something unfamiliar or high-risk.

## IDE and TUI Are Converging

IDEs and TUIs are converging. TUI agents now run inside VS Code and have desktop apps. Protocols like ACP and MCP are standardizing how agents connect to editors. Every major vendor ships both CLI and IDE agents. And IDEs are adopting TUI patterns. Cursor is a good example. It started as a VS Code fork built around autocomplete and chat. Over the next two years, it layered on multi-file editing, then agent mode, then a proprietary model with multi-agent support. And in version 3, it built an entirely new agent-first interface on top of the original editor. The old IDE view is still there, but agents are now the promoted surface. Cursor started in Mode 1 territory and had to keep rebuilding itself to support higher modes. Would it look the same if it had been built today, when most of the work is in Modes 3 and 4?

That convergence proves the point. Even as IDEs add agent capabilities, they are acknowledging that higher autonomy modes need the delegation paradigm. The IDE is becoming more like a TUI to serve Modes 3 and 4. When your editing tool has to reinvent itself as a delegation and monitoring surface, maybe the tool that was already a delegation and monitoring surface was the right fit all along.

So it is not about which tool you pick. It is about recognizing which mode you are in. And which mode you are in depends on two things. How well can you specify the task? And how much does the situation warrant trust?


My journey ended at TUI. Not because TUI is the answer for everyone. It ended there because my modes shifted. I went from writing code with AI assistance to managing AI agents that write code for me. The tool followed the management style, not the other way around. Yours might shift too.

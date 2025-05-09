
+++
title = 'Claude Code: AI Agent for DevOps, SRE, and Platform Engineering'
date = 2025-05-12T15:00:00+00:00
draft = false
+++

If you are a software engineer, you are probably already using an AI agent like [GitHub Copilot](https://github.com/features/copilot), [Cursor](https://cursor.com), [Windsurf](https://windsurf.com), [Cline](https://cline.bot/) or something similar. If you are, you probably have an opinion which one of those is the best one out there. Or you might have been dissapointed with the results AI agents provide and chose to use none of them.

Today I'll tell you which one is **the best AI agent** for any type of software engineers, especially for those focused on operations. If you call yourself DevOps, or SRE, or Platform Engineer, you'll find out which one you should use.

Ready?

<!--more-->

{{< youtube h-6LP133o6w >}}

It's **none of those**.

All those agents are based on extensions of VS Code, Jetbrains, or whichever IDE cool kids might be using these days. That's not the case with the one I'm talking about.

The AI agent in question is Claude Code from [Anthropic](https://anthropic.com).

With Claude Sonnet, Anthropic already established itself as provider of the best models for software engineers. Most of the AI agents are using it. Now, with the release of Claude Code, they have the best agent as well.

Let me show you how it works and why I think it's the best AI agent currently available when software engineering is concerned, especially for those focused on operations like DevOps, SREs, Platform Engineers, and others.

By the end of this video, I will either convince you that it is the one or, at least, to try it out. Also, I'll point to an issue that might be a deal-breaker, and give you a wish list of things that I feel are missing for it to completely take over our industry.

Let's see it in action.

## Setup

> Install [NodeJS](https://nodejs.org/en/download) if you don't have it already.

```sh
npm install -g @anthropic-ai/claude-code

git clone https://github.com/vfarcic/crossplane-sql

cd crossplane-sql

git fetch

git pull

git switch claude
```

> Make sure that Docker is up-and-running. We'll use it to create a KinD cluster.

> Watch [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell
```

## Claude Code AI Agent in Action

Unlike most other agents that are either separate desktop applications or extensions in VS Code and other IDEs, Claude Code is, at least for now, a **terminal app**. While that might not be ideal for application developers who might prefer working in an IDE, it is perfect for those focused more on operations. If you call yourself a platform engineer, SRE, DevOps engineer, or anything else, Claude Code is perfect since we use it inside a terminal which is where you probably spend most of your time.

That being said, I do not consider Claude Code the best because it works in a terminal, but because the results produced with its agent are, by far, better than anything I've seen no matter the form.

> There is no guarantee that Claude Code (or any other AI) will produce the same results twice. As a result, if you are following along, you might not see the same outputs and you might not be able to follow my instructions. You might need to adapt them.

Let's "fire it up".

```sh
claude
```

The first time we start it in a given project, we are instructed to execute `/init` which will create `CLAUDE.md` file with instructions Claude Code will use as the initial context for everything it will do.

```
/init
```

It will take a few moments for it to analyze all the files in that project. Once it's done, the *CLAUDE.md* is generated and I can say, right from the start, that it got everything right. It found all the commonly used commands and it figured out common guidelines used in the project.

> Select `1. Yes` to proceed.

> Unless specified otherwise, assume that we always say "yes" to its suggestions.

*CLAUDE.md* is critical. It serves as **initial instructions** AI will use and we should extend it with whichever other instructions we might want to add to Claude's memory.

For example, it is set to time out execution of commands after two minutes which might not be enough for this project, so we'll instruct it to change it. We can do that by modifying *CLAUDE.md* ourselves, or we can type `#` which is the shortcut we can use to put any instruction to its memory.

So, let's tell it to `set the timeout for executing Bash commands to 10 minutes`.

> Do not copy&paste `#` from prompts that follow. It needs to be typed.

> Execute the prompt that follows.

```
# Set the timeout for executing Bash commands to 10 minutes
```

It asks us where we want to save that memory which, in most cases, should be in `CLAUDE.md` in the current directory.

Similarly, let's instruct it to memorize to always `source` environment variables from `.env`.

> Execute the prompt that follows.

```
# Execute `source .env` before executing any command
```

Now we can verify whether the instructions we told it to memorize really works by telling it to `setup` everything we might need for this project. That will be a good test to see whether the information it discovered in the project is truly correct and whether it understood the custom instructions we told it to memorize.

> Execute the prompt that follows.

```
setup
```

Look at that!

It chose to `source` environment variables, just as we instructed it, and it found out that the cluster setup is done through the `dot.nu setup` command. 

It looks correct, so let's proceed.

Here's the first, and hopefully the last, thing I really dislike about Claude Code. It **does not stream output** of the commands it's running. If we executed that command ourselves, we would be able to see what it's doing and how far it got. With Claude Code, it's all hidden until it finishes the execution. That often leaves me wondering whether it's progressing correctly or is stuck in some silly loop or waiting for some prompt.

That script takes a few minutes to set up a local cluster, install everything needed to work on that project, configure whatever needs to be configured, and so on. Let's fast forward to the end.

Once it's finished running, it perfomed an extra step of explaining what happened, and it did that very well.

It listed all the *components* installed in that local cluster, informed us that it is a *Kind cluster**, and even told us that different Crossplane *providers and configurations were applied*.

That's awesome since it means that it not only figured out what to execute but also understood what the script it run does.

Let's confirm that everything is working as expected by asking it to check whether `there are any issues in the cluster`.

> Execute the prompt that follows.

```
Are there any issues in the cluster?
```

We should note here that it keeps using the instructions we told it to memorize. It is always executing `source .env` before commands. Without it it would not work since some of the commands are generating environment variables needed for the correct execution of other commands.

This is where the power of AI agents in general, and Claude Code specifically, becomes evident.

AIs we might have used before would probably try to deduce what's going on with a single command. Claude Code agent does much more than that.

It started by listing all the `pods` in `--all-namespaces`, then it retrieved all Crossplane `providers`, described `pods` in the `ack-system` Namespace, and continued following the bradcrumbs until it came to the conclusion that `ACK Controllers` are `in CrashLoopBackOff` state. It explained that the `Root Cause` is that we're using a Kind cluster. It figured out that all other components `are running properly` and suggested that we should `create and configure AWS credentials`. Finally, it offered to help us with that issue.

The part it does not know is that we don't need those ACK controllers to work properly for what we are about to do, so I'll politely tell it that there is no need for help on that one.

> Execute the prompt that follows.

```
No. I don't need it for testing.
```

Now, let's say that we would like to see `all AWS tests`.

> Execute the prompt that follows.

```
Show all AWS tests
```

It went through all the files, figured out what there are AWS tests in `tests/aws` and `tests/aws-ack` directories, and it even provided a few example `AWS configurations` that could come in handy if we would like to test it manually, which is not something I would do, yet you might.

It figured out that the project uses `Chainsaw` for testing and that `both types of AWS providers` do, more or less, the same.

Now, that is not really what I wanted. I hoped that it would open all the tests so that I can work on them, so let's give it one more instruction to memorize that `when asked to show or get files, I want` it `to open them in VS Code`.

> Execute the prompt that follows.

```
# When asked to show or get files, I want you to open them in VS Code using `code` command.
```

Let's see what will happen now if we repeat the same request to `show all AWS tests`.

> Execute the prompt that follows.

```
Show all AWS tests
```

After a bit of exploration and a few required confirmations from my side, it opened all the test files related to AWS in VS Code.

Next, let's say that I would also like it to `show` me all `AWS Composition code` in that project.

> Execute the prompt that follows.

```
Show AWS Composition code
```

That's a tricky one since AWS Compositions exist in two forms. There are KCL and YAML files. So, it suggested that all the *.k* files directly or indirectly related to AWS should be opened in VS Code. It also suggested that `compositions.yaml` should be opened, even though I don't need it. Finally, as always, it provided a description of what each of those files are and what they're used for.

Now we have all those in VS Code and we can start working on the project.

It's bloody **amazing!**

Still, sometimes we might not want Claude Code to run "stuff" for us.

We can see an example of that by instructing it to `run all tests in watch mode`.

> Execute the prompt that follows.

```
Run all tests in watch mode
```

It did figure out what the command is, but it did not figure out that it should not execute it due to its own limitations.

> Do not tell it to `proceed`. Choose `No` or press the `ecs` button.

Let's see whether we can "nudge" it to realize why running that command is not a good idea by asking it `how long will it take for that command to execute?`

> Execute the prompt that follows.

```
How long will it take for that command to execute?
```

Look at that!

It figured out that command would `continue` running `indefinitely`.

If we ignore the fact that we said that we'd like to run tests in the watch mode, many of us would not figure it out right away from looking at the code. Claude understood what will happen by going through all the relevant lines of that script and realized that it uses `watchexec` instruction.

As a cherry on top, it suggested that we can run tests once instead of in watch mode by executing `./dot.nu test full` instead.

Let's see whether it can `run commands indefinitely`.

> Execute the prompt that follows.

```
Can you run commands indefinitely?
```

As you can probably guess from the previous conversation about timeouts, it can't, and that's a pity.

I can imagine many scenarios where running an agent like Claude Code indefinitely would be amazing. I'd love to run tests watcher with it and instruct it to go through results and notify me when there's something wrong. It would be amazing if it could be watching my local cluster and notify me when I make a change that results in an unexpected behavior. I can imagine and number of agents running continuously and ensuring that what I'm doing is correct, helping me out when it detects I need help, and countless other cases.

Nevertheless, that would probably also result in me going bankrupt. We'll get to that one later.

For now, what matters, is that it is me who should run tests watcher, so let's ask it to `get the command`.

> Execute the prompt that follows.

```
Get the command that runs all tests in watch mode
```

It output the command we might need. Nevertheless, I don't want to select it and copy it so that I can paste it in a separate terminal session. Instead, let's instruct it to memorize to use `pbcopy` `when asked to get or fetch a command`, just as we instructed it to use *code* to show or open files.

> `pbcopy` works, as far as I know, only in Mac. If you're using a different operating system you might need to modify the instruction that follows.

> Execute the prompt that follows.

```
# When asked to get or fetch a command use `pbcopy` to copy the command to memory.
```

Let's see whether it understood that by asking it to `get the command` again.

> Execute the prompt that follows.

```
Get the command that runs all tests in watch mode
```

We can see that it used `echo` to output the command and then pipe it to `pbcopy`.

It works! Not only that it does what AI agents do and that it almost never makes a mistake, but it always implemens the instructions we told it to memorize.

From here on, I can open a separate terminal session, enter into `devbox shell`, since that's the Shell I always use, `source` the environment variables since they are needed to work with the local cluster and quite a few other things.

> Execute the commands that follow in the second terminal session.

```sh
devbox shell

source .env
```

Nevertheless, since, today, I plan to work only on AWS part of the project, let me go back to Claude and ask it to `get the command that runs only AWS tests in watch mode`. That will brush off a few seconds from tests execution.

> Execute the prompt that follows.

```
Get the command that runs only AWS tests in watch mode
```

There we go. It generated the correct command, it copied it to memory, and we can go to the second terminal session and just paste it.

> Paste the command copied by Claude Code into the second terminal session.

I will save you from watching me write new tests, seeing them fail, writing implementation, seeing tests succeed, and repeating that process until I'm finished. I don't want to bore you with a rant about Test-Driven Development, at least not today. All I'll say it that I rely on Claude to help me with all that, but I also do not trust it fully. Claude Code might be better than any other agent, but is still not perfect and my job is to fix its mistakes, put it into the right path when it goes astray, and convert often decent code it suggests into code I am proud of.

> Press `ctrl+c` to stop running tests in the watch mode.

You saw a glimpse of it and I don't want to bore you with more examples. Instead, let's talk about the pros and cons.

## Claude Code AI Agent Pros and Cons

Claude Code is, without doubt, **the best AI agent** for anything related to software engineering. It does not matter whether we work primarily with Java or Go or we are more focused on scripts or we spend most of our time executing commands. It does not matter whether we are focused on application development or operations. Claude Code puts other AI agents to shame. That does not mean that it is perfect, because it's not. It often need an experienced engineer to detect mistakes, to push it into the right direction, to distinguish good from bad, and so on. Still, when compared with other AI Agents, Claude Code is, right now, the best. That will likely change in the future since AI is moving very fast and it is common for one tool to fall behind only to pass the competition the next day. So, when I say that Claude Code is, without doubt, the best, I mean that it worked better than others in May 2025.

That, however, does not mean that it is the most comfortable AI agent out there. Even though it produces better results than others, some might still prefer AI agents that are used as an extension in their favorite IDE. That could be Cursor, Cline, GitHub Copilot, or something similar. For some, having worse results might be less important than the comformity of using AI agents inside an IDE. I can understand that.

Howerever, if you are not scared of terminals or, even better, if you find yourself having a terminal opened all the time, with or without an IDE beside it, Claude Code is the best option today.

Moreover, if your work is focused on operations and you call yourself DevOps engineer, or SRE, or a platform engineer, than Claude Code is without almost any competition.

That does not mean that we should use AI for everything. More often than not, it is easier to just execute a command than to ask AI to do it for us, especially if we already know the commands we want to execute. I don't have a need for AI to tell me how to create a local Kubernetes cluster if I already did it countless times before. There are times when it is better or faster to use AI agents, and times when it is just better to do it ourselves. More often than not, it is a combination of both. I tend to see myself moving more and more towards being AI supervisor in a similar way I would supervise a more junior person on my team.

I am not here to tell you whether you should or should not use AI agents. That would be a subject of a separate video. I am here to tell you that Claude Code is the one, especially if you're more on the ops side of software engineering. GitHub Copilot tends to get confused and give us wrong suggestions most of the time. Cursor and a few others are better, yet still make more mistakes than Claude. That, combined with Claude Code being a terminal app, is a winning combination, at least for some of us.

Still, there are a few negative points.

**Cons**
* The price
* 10 minutes max. timeout
* No auto-memory
* No output stream
* No background execution

Let's start with probably the biggest issue with Claude Code; **the price**. It's expensive when compared to other solutions like, for example, Cursor or GitHub Copilot. While others offer monthly or yearly subscriptions that tend to be around $20 to $30, Claude Code charges per tokens spent. I had cases when I burn through hundred dollars in a single day of AI heavy work. That's not common though and, more often than not, the figure it much lower, yet, at the same time, much higher than a typical monthly subscription of other agents. I do not necessarily think that's a problem. If it helps me do work that would take me a couple of days in a few hours, the price is actually low. I have a subscription to both GitHub Copilot and Cursor, yet I find myself using Claude Code more than those, even though I have to pay for it on those of those.

The default timeout is 2 minutes and it can be increased to **10 minutes max. timeout**. While that might be okay when generating changes to the code, it is often not not enough for operations.

The ability to memorize instructions that act as a context into CLAUDE.md is great, yet I don't think it's enough. I would want it to also memorize what it did automatically so that it builds a "real" knowledge how to work with me and for me. Over time, it gets annoying to treat it as a child by telling it to memorize every single important detail. I'd love if it would **keep its own memory up-to-date** without me telling it to do so.

I really dislike that it **does not stream the output** but shows what happened after it happened. That would be very important, especially early on while we're trying to establish trust between us. I want to know what is happening while it's happening and not to be surprised at the end.

Finally, it **cannot execute processes in the background** while still watching and evaluating the output. If it could do that, we could use it for many tasks that we either cannot use it for today, or would have to give it the same instruction over and over again. Actually... With the current pricing model, that would probably be very expensive, but only until we compare that price with a price of a person sitting in front of a monitor and watching a dashboard waiting for something to happen. So, it would be expensive, but not that expensive, and it would certainly be useful.

Here comes the important note. Most, if not all of those complaints are equally valid for other AI agents. CLaude Code is just as good or better than any other solution we have today. So, my complaints are more of a wish list than me finding problems that are solved elsewhere. The only real complaint is the price. It's more expensive than most other solution and you need to see whether the benefits of using Claude Code overweigh the increase in the cost.

As for pros... Everything is a pro. It is **the best AI agent** right now. It outperforms the competition. It understands better, it performs better, it is more reliable, and it produces better results than any other agent.

Try it. Use it. Let me know what you think.

## Destroy

> Execute the prompt that follows.

```
destroy
```

> Press `ctrl+c` twice to exit `claude`.

```sh
git stash

exit

exit

exit
```


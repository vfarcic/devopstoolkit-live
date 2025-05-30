
+++
title = 'From Shame to Fame: How I Fixed My Lazy Vibe Coding Habits with Taskmaster'
date = 2025-06-09T15:00:00+00:00
draft = false
+++

**AI does not work**, or, to be more precise, works poorly when trying to accomplish larger tasks that require many steps.

Imagine that we have a Product Requirements Document, or a PRD, that requires some major development, or a major refactoring. We might have spent hours or even days defining that PRD, and even more time defining all the tasks such a PRD should contain. Once we have it all set, we can start writing the code that implements that PRD, and that is likely to take even more time.

That situation presents one problem and one opportunity for improvement.

<!--more-->

{{< youtube 0WtCBbIHoKE >}}

First, we should be able to **speed up creation of the PRD and the tasks** behind it. We can do that part with an AI agent, as long as we are willing to create elaborated prompts. As such, that is not af problem but, rather, something we can improve.

The problem, however, is that AI would likely **fail to implement all those tasks** in one go. Since the context is limited, agents eventually hit those limits and start forgeting what they did. That's bad since that means that they would not be able to continue working on latter tasks effectively or, even worse, that they would start outputting random garbage that is unrelated to what they already did.

Another problem is that if we start a new session, everything is **forgotten** so it's hard to continue where we left.

There are other issues and they all boil down to inability of AI agents to work well on larger tasks.

There's a solution though. There is a project that we can plug into AI agents as an MCP server and that will take care of those issues. It can create PRDs and the tasks, orchestrate AI agents, keep track of what is done and provide relevant information about the tasks that are still pending.

That project is Taskmaster. It acts as a project manager, of sorts, by **organizing, researching, expanding, prioritizing,** and **shipping tasks**. It provides a **permanent context, it's free,** and it's **open source**.

It's awesome. If you adopt it, it will change the way you use AI.

## Setup

> Install [Go](https://go.dev/doc/install) if you don't have it already.

> Install [NodeJS](https://nodejs.org/en/download) if you don't have it already.

```sh
npm i -g task-master-ai

git clone https://github.com/vfarcic/youtube-automation

cd youtube-automation

git pull

git fetch

git switch taskmaster
```

> Watch [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell
```

> Get Anthropic API key and replace `[...]` with it in the command that follows.

```sh
export ANTHROPIC_API_KEY=[...]

yq --inplace \
    '.mcpServers."taskmaster-ai".env.ANTHROPIC_API_KEY = strenv(ANTHROPIC_API_KEY)' \
    .cursor/mcp.json
```

> Press `ctrl/cmd+shift+p`, type `Shell command: Install 'code' command`, and press the enter key to create `code` alias code Cursor if you do not have it already.

```sh
code .
```

> Enable MCP servers when asked to do so in Cursor.

## The Shame

Here's what we'll do.

We'll use one of my hobby projects and try to write tests for it. That's something I should have done from the start, but I didn't, because I was lazy, and now I'm ashamed. The only way I can remove the stink of that shame is to do it.

The problem is that even though it's not a big project, I expect those tests to be measured in thousands of lines of code and that's not something that can be done fast. I need to figure out which testing tools I'll use, I need to write mocks and stubs, I need to write helper functions, I need to write tests themselves, and probably do quite a few other things that are not self-evident from the start.

Actually, the problem is that I want to do all that, but I'm still lazy. I don't want to spend days on that alone.

I cannot cut corners since that would bring back the shame, but I don't want to spend too much time on it.

It's a pickle.

I tried doing all that in Cursor by instructing its agent to write all the missing tests, and that was a **disaster**. It started fine but soon after it started forgetting what it did and the outcome was not what I wanted. It reached the point where I got so dissapointed that I threw it all away.

This is my second attempt to accomplish the same outcome but, this time, I'll be using Taskmaster to help me out.

Taskmaster should work with GitHub Copilot, Windsurf, or any other AI agent you might be using. My current preference is Cursor, so that's what I will use today, and you are free to try it out with something else. The process and the results should be similar, if not the same.

I already added Taskmaster entry in Cursor's list of MCP server. So, Cursor now know that Taskmaster is there, but, before we start using it in this project, we need to instruct the AI agent to initialize it.

```
Can you please initialize taskmaster-ai into my project?
```

We can see that, the agent started `Calling MCP tool`. That's the proof that it is using Taskmaster MCP server when it needs to work PRDs.

That's it. That's all it takes to set it up.

Now we can create a PRD.

## Product Requirements Document (PRD) with Taskmaster

We'll instruct it to `learn what this project is about and write a PRD.` In this particular case, the goal is to write `tests for all the code in this project.` `The tests should be unit tests that do not make any interaction with external APIs.` `There is no need to mock file system operations.` Finally, we want the agent to `Create Taskmaster file scripts/tests.md.`

> Type the prompt that follows

```
Learn what this project is about and write a PRD. The PRD should be about writing tests for all the code in this project that is not already covered by tests. The tests should be unit tests that do not make any interaction with external APIs. There is no need to mock file system operations. Create Taskmaster file scripts/tests.md.
```

Consider that as a high level objective of what we want to accomplish.

Without Taskmaster, we would probably spend quite some time defining all the details we might take into the account before we start working on this task. I could easily see myself spending few hours thinking through it and a bit more writing it down. Taskmaster did that for us in a couple of seconds.

Now, you might think that I'm exagerating and that you would assemble the whole testing plan in a blink of an eye. I bet you do, and you might be right. We'll see whether that is the case later when we take a look at what Taskmaster did.

It examined the project structure, it looked at all the relevant source code, and it created the PRD.

> AI is non-deterministic. If you follow along, you will likely see different results and be asked different questions. As such, you might not be able to reproduce exactly the same results as what you'll see in this post.

There is a short `Overview` of the project, the description of the `Current State` of the project, and, more importantly, the `Core Requirements` we might want to implement. Those requirements are `Test Coverage Goals`, `Testing Approach`, `Mock Requirements`, and `Test Organization`. Further on, we got `Specific Test Requirements` that go into more details by listing `CLI and Configuration Testing`, `YouTube Functionality Testing`, `Email Notification Testing`, `Workflow and State Management`, and `Utility Functions`.

That's not all though. It also defined `Testing Tools and Libraries`, the `Implementation Strategy`, the `Success Criteria`, and what is `Out of Scope`.

That looks like a serious PRD, but we should not take it for granted. Like with most outcomes of AI, we should not take it for granted. We might want to review it and modify it, manually or with further help of AI, until we reach the state we are happy with. I won't do that today for the sake of brevity, but you should not take that as my recommendation to trust it blindly. Always review and, if needed, modify, anything AI produces.

We are only starting. There's much more coming.

After a couple of additional calls to Taskmaster MCP and a few commands, it created `tasks.json` file which contains the tasks we might need to perform to implement that PRD. There's one for setting up `Testing Infrastructure`, another one to `Create Mock Interfaces`, then one to `Implement Utility Function Tests`, and so on and so forth. There are fifteen tasks in total, each with the `title`, `description`, the `status`, `dependencies`, and other information we might need to implement that PRD.

If we would stop it here and switch to do everything else manually, the outcome would already be amazing. It would take us at least a day to do such detailed planning of what we should do. Taskmaster did it in no time.

There's more though, much more.

Now we can start the actual work of implementing all those tasks, and that's where Taskmaster really shines.

You see, we could have done all that without Taskmaster by writing prompts to whichever AI agent we might be using. It would take us more time, but we could still do it. What comes next is what makes Taskmaster truly amazing.

Right now, there is no AI agent that could implement all those tasks. Every one of those currently available would eventually get confused, run out of context space, or simply freak out. AIs are not yet good at performing large and complex tasks with many dependencies. They are great for relatively short tasks, but not so good and maintaining the context in cases when we work on bigger problems that take longer to solve and require large context.

That's where Taskmaster comes in. Think of it as an orchestrator of AI Agents. It keeps internal memory of what was done and it instructs agents what to do and when to do it. If we would compare AI agents with containers that do one or a few things very well, we could compare Taskmaster to Kubernetes which orchestrates those containers and all other resources we might need.

We can also see that a separate file with even more details is created in the `tasks` directory. That's the context that AI will need to perform each of the tasks.

## Working on Tasks with Taskmaster

We could initialize the "orchestration" of the work we need to do with the `task-master set-status` command suggested to us, but I find it tedious to start running commands directly when we are in the AI agent chat. So, let's continue "chatting" by asking it `what's the next task I should work on?`.

> Type the prompt that follows

```
What's the next task I should work on?
```

After a few calls to Taskmaster MCP, it figured out that we should start working on the first task. Duh! I could have figured that out myself. We should always start with the first tasks. Right? Well... It gets more complicated later with dependencies between tasks. Nevertheless, starting with the `Setup` of `Testing Infrastructure` does sound like a good first task. We can even see that there are `5 subtasks` involved, so Taskmaster broke it down into smaller pieces and is waiting for us to tell it to `start working on that task.`

> Type the prompt that follows

```
Start working on that task.
```

This is when Taskmaster starts the orchestration. It changed the `status` of the task to `in-progress` and instructed the agent what to do to implement it. After a while, it changed the first subtask to `in-progress`, gave additional instructions to the agent which, in turn, created a bunch of new files, and, finally, changed that subtask to `done`. That "dance" continued for a while until all the subtasks of the first task were completed and we got the summary of everything done so far instructing us that, if we are pleased with the progress so far, we can continue working on the next task.

The important note here is that you should not do what I just did. You should not trust it blindly as I did. You should inspect everything it did, and either accept the changes, provide additional instructions if needed, or, in some cases, intervene by making changes to the code manually. AI is great, but not perfect. Do not trust it, just yet.

Now we can, for example, get the status of the progress by, let's say, asking it `how many tasks are left to do?`

> Type the prompt that follows

```
How many tasks are left to do?
```

In this case, we can see that we `completed 1 out of 15 main tasks.`

We can also check `what's the next task I should work on?`

> Type the prompt that follows

```
What's the next task I should work on?
```

In this case, that would be the task to `Create Mock Interfaces`. It is a `High Priority` task, probably because those mocks are required for the actual tests that will be created later.

The logical next step is to instruct it to `start working on that task.`

> Type the prompt that follows

```
Start working on that task.
```

Now it is working on the next task following the same pattern.

I won't bore you going through all fifteen tasks. I'm sure you get the point. The only thing that I will say is that when I did the same exercise "for real", it took me a few hours to complete them all, with most of that time spend on reviewing the work, giving additional instructions, and making manual changes in cases it messes it up.

Feel free to explore `mcp.json` to see how Taskmaster is set up.

> Open `.cursor/mcp.json` in Cursor

## Taskmaster Pros and Cons

Taskmaster solves a known problem when working with AI agents.

When working on larger set of changes, we often run into limitations imposed with the size of the context. More importantly, when working on something bigger, we might not finish it in one go and agents can easily get confused if we start a new session.

Taskmaster allows us to **split** the **work into smaller tasks** and to **orchestate what agents do** when working on all those tasks. It is a must for anything but relatively simple and small changes we might be doing.

That is not to say that we could not have accomplished the same without Taskmaster. We could, but that would require a lot of additional work. Taskmaster makes tedious work easy.

All in all, Taskmaster is **awesome**. It quickly became one of my favorite tools and I do not have anything negative to say about it, except a word of warning that applies to averything related to AI. Do not trust it blindly. Review and, if needed, modify the PRD, and the tasks it creates. It is not perfect and it certainly cannot read our minds. It's a great tool but, like any other AI-related tool, we need to be on top of it. Do not run it blindly. Do not enable auto mode. As long as you are in control of it at all times, it is amazing and I stronly recommend it.

## Destroy

> Stop Cursor agent

> Close Cursor

```sh
git checkout -- .

git clean -fd

git switch main

exit
```


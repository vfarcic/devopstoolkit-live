
+++
title = 'AI vs Developer: Can GitHub Copilot or Claude Replace My Job?'
date = 2025-07-14T16:00:00+00:00
draft = false
+++

I just gave two AI agents the same complex coding task. One completely failed. The other... honestly shocked me.

But here's the twist: I'm not sure if I want the better one to succeed. Because if these autonomous coding agents can really implement full features from scratch, write tests, and handle edge cases like an experienced developer... well, we might all be looking for new careers sooner than we think.

Today, I'm putting GitHub Copilot's new coding agent head-to-head with Claude's autonomous agent. Same PRD, same requirements, zero hand-holding. And the results? Let's just say one of these agents just changed everything I thought I knew about AI coding.

<!--more-->

{{< youtube ahTkFqssZxM >}}

I have an application that, currently, works as a CLI. It has all the features I need, and now I would like to expose them through a REST API.

![](issue-description.png)

But here's where it gets interesting. I already have a GitHub issue that acts as a PRD for that. The issue is full of details. There is `introduction`, there are `Goals`, and there is `API Design`, with `Core Principles`, and `Endpoint Structure`. There is also information about `Testing`, `Non-Functional Requirements`, `Future Considerations` with the list of things that are out of the scope, and even answers to some `Questions & Clarifications`.

There's more than enough information there that any software engineer should be able to implement it and my hope, today, is that AI will manage to do it. Ideally, given that all the information is provided, it should be able to do everything in one go. If not, I'm here to babysit and give it a hand if it gets confused or something goes wrong.

Today, I'll give the task to implement that PRD to two fully autonomous agents, and they're about to battle it out.

![](issue-copilot.png)

In the red corner: GitHub `Copilot` - all I had to do is assign it to that issue. It cannot get easier than that.

![](issue-claude.png)

In the blue corner: Claude. Unlike Copilot, we cannot assign it to an issue, but we can mention it in a comment like `@claude implement the feature based on the issue description`.

If I would have to nit-pick, I'd say that Copilot's way to trigger work on an issue is more in-line with how we would do it with people and slightly easier. But which approach actually delivers better results? You're about to find out.

What matters is that both agents started working on that issue independently from each other and will not stop until everything described in it is implemented and, hopefully, tested.

If the end result fulfills my expectations, this is earth shattering. This will change the industry forever. If that works, you, and me, and every other software developer will be out of work. Hence, I'm not sure whether I want my expectations to be fulfilled or not. Ideally, it will not work. Ideally, we will get autonomous agents that meet my expectations ten or so years from now, since that's when I will retire and not care anymore.

But wait, there's more at stake here than just my retirement plan.

Speaking of my expectations, they are simple. I want those agents to fully implement that PRD in a way an experienced engineer would do. I want it to write the code that meets all the requirements, I want it to test it, and I want it to document it. If it gets stuck or something is not clear, I want it to ask me for guidance. In this scenario, I am acting as a product owner, and a developer only if it messes it up. Given that the PRD is very detailed and, essentially, provides all the steps and all the information, accomplishing those expectations should be easy. I believe that any developer could do it, even a junior one.

Both agents started working on it and created a PR. The results? One impressed me, one disappointed me, and one completely changed my mind about autonomous coding agents.

Let's see whether my expectations were met or my retirement plan is safe for now.

We'll start with the PR Copilot created.

## Copilot's Epic Fail: When AI Gets Overconfident

![](copilot-view-session.png)

We'll skip the description of the PR since it is, essentially, a summary of the issue, and move down where we can see that it started to work on it. Next to it is the `View session` button that leads us to all the steps it performed.

![](copilot-session.png)

That is a very nice Copilot feature that you will probably not use, unless you do not have Netflix subscription and choose to watch it progress instead of watching Squid Games. The point of autonomous agents is to work in the background and, most of the time, we are going to inspect the outcome, the code, rather than go through each and every one of the steps they performed.

This is very different from using agents in an IDE where we do want to see each of the steps, confirm whether the agent should perform them or correct it. The point of autonomous agents is to let them finish everything instead of working with them side by side. We'll talk whether that is a good thing later, spoiler alert: it's complicated.

For now, what matters, is that the view session option is nice, but likely useless most of the time.

![](copilot-code.png)

Once it finished, we can see the changes it made. That's the part that matters. We should review the changes in detail, at least initially until we gain trust that it indeed meets our expectations.

And here's where things get ugly.

I won't bore you with details behind the code review I did, except to say that I did review everything and concluded that it did create an API that mostly works, but that it failed to meet all the requirements of the PRD. The code it wrote works, more or less, but is s\*\*t. It's garbage.

My dreams were shattered, but my retirement is secured for now.

Now, to be clear, I did not expect it to do everything correctly from the first attempt. When working with an agent in an IDE, we need to correct it every once in a while if we want it to do the right thing so it would be silly to expect similar agents working autonomously to be much better. That's fine. If such agents can get us, let's say, **80%** where we need to be from the first attempt, we can step in to help them get where they need to be fully. That would be still great. Right?

But here's the thing: did it even reach that **80% threshold**?

![](copilot-manual-testing.png)

Here's the first red flag that should have warned me about what was coming.

One of the requirements of the PRD was to write instructions I could use to check whether the API works correctly manually. That is a very simple thing for an agent to do and I have no doubt that it wrote it, but, just to be on the safe side, I asked it whether it did that.

It didn't! I knew that since I reviewed the code it did and haven't seen such a file.

When it saw my question, it added the docs and they are now fine. I'm disappointed that it missed that one **completely** since it is such an obvious requirement that was explicitly stated in the PRD. One thing is to make a mistake, and another to completely skip doing it. The only conclusion I can make is that it did not create a list of tasks it needs to do but simply started working on the PRD, reached the limit of the size of the context, and missed it.

![](copilot-shared-functions.png)

But that was just the beginning. Here's where things get really problematic.

Here's a more complicated part of the PRD. I wanted it to `ensure` that `both the API and the CLI use the same` `functions for data operations`, so I asked Copilot whether it accomplished that goal.

The response is assuring, but only to those who are not familiar with that code. Unfortunately for Copilot, I know that code and I know that what it did is equivalent to a lazy developer doing absolute minimum in hopes that no one will notice that the actual non-functional requirement was not met.

I reviewed the code and I know that answer is bul\*\*hit. It duplicated bunch of code that performs the same calculations. If it said something like "oh my, I see that I haven't done that, let me correct it", I would be much happier. Making a mistake and, later on, correcting it is much better than claiming that you did everything **perfectly**.

![](copilot-shared-functions-followup.png)

Still, I explained why it did not do what it was supposed to do, and got an answer essentially saying "Viktor, you're an idiot. I did exactly what I was supposed to do."

This is where I started getting genuinely frustrated with this agent.

![](copilot-the-end.png)

After a few other failed attempts to reason with Copilot, I told it that I give up. The answer I got pissed me off even more. It essentially said that it understands my frustration and that it can actually refactor the code to remove duplication which, until that moment, it claimed does not exist. It gave me an answer only a teenager can give after claiming that everything is okay at school and, when final grades come in, change the tune by saying "I'll do better next year".

At this point, I was ready to write off autonomous coding agents entirely.

But then Claude stepped into the ring.

## Claude's Redemption: The Agent That Changed My Mind

I expect Claude to do just as bad as Copilot. I think that both are using the same Sonnet models. If, by some unlikely chance Claude does better work, it will be a proof that it's not only about models but that agents themselves are not all made equal.

Spoiler alert: I was about to be very, very wrong.

Let's see whether that's the case.

Claude also wrote a relatively short description of what should be done. There's nothing special about it.

![](claude-code.png)

If we take a look at the changes it made, two things become apparent immediately. One being harder to detect than the other, but both **absolutely crucial**.

First, the **overall quality** of changes is higher than what Copilot did. That's the hard part since I had to go through every line it added or removed or updated. That is also subjective since code quality is both objective and subjective. I might have different preferences than others. But the difference was significant enough that even you would probably notice it.

The second observation is that it succeeded in adding documentation that explains how to test changes manually. That's a simple one, but is something that Copilot missed, even though it is an **explicit part** of the PRD.

Right off the bat, Claude was already winning.

![](claude-allowed-tools.png)

On the negative side, when I tried to build the code myself, I noticed that it does not compile, so I wrote a comment to Claude which, in turn, fixed it or, at least, claimed it did. The real problem here is that Claude does not have Bash permissions. It did fix the issue which, to be honest, it wouldn't have made if it could compile it itself. Nevertheless, that's both good and bad news.

The bad news is that it is **unacceptable** to have an autonomous agent that cannot even compile or run tests. That, however, is not a problem anymore. After I wrote this post, Claude corrected that. GitHub Actions it created for that repo used incorrect syntax to define allowed tools. If you try it now, you should not have those issues. I suffered through it, mainly because I thought it is a good idea to try it out on the first day.

But here's the silver lining that blew my mind: it wrote **better code** than Copilot, even without being able to build it or test it. Now, if without being able to do that it does a better job, how much better is it now that the issue with allowed tools is fixed? **A lot better**.

I'll ignore the rest of nightmare I went through caused by inability to use any tools, mainly because that is not happening anymore.

![](claude-shared-functions.png)

Now comes the **moment of truth**. The same test that broke Copilot.

Claude code, just as Copilot, missed the requirement to avoid repetition by ensuring that `both the API and CLI use the same underlying functions for data operations`. So, I asked it to double check that.

That's another difference between the two. Both made mistakes, and that's to be expected. Agents and models are not perfect, and neither are we. They can make mistakes, and sometimes mistakes are on us for not having clear PRDs. However, Copilot was incapable of finding the mistake once I pointed it out. It refused to acknowledge that its work might not be perfect. Claude, on the other hand, evaluated the code and came back to me saying "You're right. That `goal` was `**NOT fully achieved**`" followed by `**architectural issues**` and `**recommended refactoring**`. When I asked it to `fix that`, it created a new job and started working on it.

This is the difference between an AI that **argues with you** and an AI that **collaborates with you**.

![](claude-action.png)

Now, if you are so foolish as to take a look at a Claude job in an attempt to understand what it's doing, you will discover that is close to impossible. While Copilot presents its work in a format that can be followed and understood, Claude shows us a GitHub Actions run in a way that no human can follow. I don't think that's a problem since you are unlikely to actually go through all the steps either of those two agents are performing. If that's what you want, you should avoid using unsupervised agents and go back to AI in whichever IDE you like using.

![](claude-refactor.png)

At the end, Claude did the right thing. It analyzed code, it found duplications, it created new functions and removed those that were in both places, and it refactored the code so that both the CLI and the API use those new functions. It did all that by creating a `task list` of what it needed to do, and then followed those. That resulted in `**code duplication**` being `**successfully eliminated**`, as I requested from the start. There is now `**60% reduction**` in duplicated business logic.

If we ignore the struggle caused by it not being allowed to use any tools (which is now fixed), there were very few additional instructions I needed to give it to make that PR be everything I wanted it to be.

But this raises a bigger question about autonomous coding agents in general.

## The Verdict: Should You Trust Autonomous Agents With Your Code?

Let me start by saying that both GitHub Copilot Coding Agent and Anthropic Claude Action are **amazing**. If you asked me few years ago whether we would have something like those so soon, I would answer with: "No way. Not that soon." Yet, here we are. Autonomous coding agents exist and they are impressive.

However, they are **not yet** good enough to be fully autonomous. Neither of those is capable of implementing anything but a very simple PRD without human intervention, and that poses a question.

If we do need to interact with agents to ensure that they do the right thing, does it make sense to trigger them through GitHub issues and to interact with them through comments in pull requests? The answer to that one is both yes and no. Let me explain the workflow that actually works.

Most of the time, such agents will do part of code changes correctly and we need to jump in to correct the rest. That is essentially the same as what is happening if we work with agents in an IDE like Cursor. We give them a task, they do something, we review what they did, and we give them instructions to correct parts of their work. The first iteration is the easiest one for us humans. We spend most of our time with what comes after.

Now, here's the key insight: interacting with agents is **much faster through an IDE than GitHub**. If we are inside VS Code or Cursor or some other IDE, it's easier to see what's going on, it is faster to chat with an agent, and agents are much faster since there is no need for them to spin up a new VM or a container. The process is much better and faster in an IDE than in GitHub, and that is to be expected since IDEs are designed to make us productive while coding.

So, does that mean that we should not use agents in GitHub? No. We should, but probably not in the way their creators intended us to use them.

The **most optimum workflow** is to combine both. When we create an issue, we should instruct agents in GitHub to create a pull request and implement whatever that issue is about. That can be a PRD or a bug or anything else. Most of the time, that initial implementation will not be what we want it to be. It will be **60%** or **70%** or **80%** of what it should be. More often than not, it will be significant part of the work. The advantage is that part of the work is done in the background. It does not require our attention and it can be done as soon as the issue is created, and wait for us to walk the last mile.

After that my process likely differs from what authors of those agents thought it should be.

When I'm ready to dedicate time to any of those issues, I switch to an IDE, check out the branch where the remote agent did the initial work, and, from there on, do the rest in that IDE. I review the changes it made and I iterate with instructions what should be corrected until it gets into the shape I believe it should be and create a new release.

From there on, I can jump to the next one which is also half baked and do the same.

What I'm trying to say is that those remote agents in GitHub are great, but not as tools that will do all the work and not tools with which I want to interact through GitHub. Instead, they are great for that **initial first iteration** after which I switch to IDE for the last mile.

Assuming that you agree with that approach, we are left with only one question unanswered. Which of the two should we use?

## The Clear Winner: Why Claude Dominates Autonomous Coding

When working locally, Claude Code is the **best agent** out there, at least today. The problem, for many, is that it is terminal-based thus missing many of the benefits we get with IDE-based agents like Cursor. However, being the best agent running in a terminal gives it an edge when working in GitHub. If it already performed better than any other agent in a terminal, it makes perfect sense that it works better than Copilot when running autonomously.

Hence, Claude Action is, **without doubt**, a better option than Copilot Coding agent. The initial work it did was better even without it being able to use tools to compile or run tests. With that issue fixed, it only got even better.

Right now, **Claude Action is the choice**. That might change in the future since AI space is moving very fast. Still, given the Anthropic direction and the consistent quality it is producing, I would be surprised if any other solution gets in front of it any time soon.

The only area where Copilot proved to be better is **presentation**. Inspecting what it did directly in GitHub is easier. Nevertheless, I don't think that matters, at least today. I don't think that we should do the whole process in GitHub. I don't think that interacting with agents and reviewing what they did should be done through GitHub. Instead, I believe that GitHub is a great place to do that **initial first iteration** and that we should switch to an IDE for the **last mile**. The first iteration is where most of the code changes are done, but that last mile is what takes most of our time.

All that might change and agents might be able to do everything we need them to do in a **single iteration**. That day, however, is not today and we still need the benefits of IDEs.

But based on what I've seen, that day is coming **faster than I expected**. And when it does, Claude will likely be leading the charge.


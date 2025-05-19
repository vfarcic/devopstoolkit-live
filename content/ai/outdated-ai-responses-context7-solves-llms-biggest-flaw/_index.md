
+++
title = "Outdated AI Responses? Context7 Solves LLMs' Biggest Flaw"
date = 2024-05-19T16:00:00+00:00
draft = false
+++

**LLMs are always behind**. They do not contain up to date information and examples for programming languages, libraries, tools, and whatever else we, software engineers, are using. Depending on the date an LLM was created, it might be days, weeks, or months behind. As such, examples will be using older libraries, outdated APIs, and deprecated versions of the tools.

Moreover, since LLMs are, in a way, databases of the whole Internet, they might give us code examples taken from places other than, for example, oficial documentation. They might give us generic answers that do not match versions we're working with.

We are going to fix that today in a very simple, yet effective way. We are going to **teach our agents how to get up to date information** they might need to come to the right conclusion and perform correct actions.

By the end of this post, the likelyhood of your AI agent doing the right thing will increase exponentially.

<!--more-->

{{< youtube DeZ-gw_aop0 >}}

## Setup

```sh
mkdir context7-demo

cd context7-demo
```

## Agents Using LLM Alone

I'm using Claude Code today for a simple reason. It is, in my opinion, the best AI agent out there right now! Still, that does not matter since you should be able to accomplish the same outcome with almost any other agent you might be using.

> Depending on the model you're using and, more importantly, the date when you're reproducing the steps that follow, the outcome might be very different. You might not experience the same issues I was experiencing when I wrote this post.

Now, let's say that I am interested in knowing `what are the most important new features and changes in Crossplane v2?`

```sh
claude "What are the most important new features and changes in Crossplane v2?"
```

After a few unsuccessful attempts, Claude Code realized that it could not find the correct information. That was to be expected since, Crossplane v2 did not exist at the time the model I'm using was created and it failed to find what it's looking for online.

The good news is that the agent responded saying that it does not `currently gave direct access to Crossplane v2 documentation.` At least we know that it cannot help us with that query, at least not yet.

Crossplane v2 is, at the time of this writing, still in the preview phase. Nevertheless, I want to use it because its awesome, so let's write a prompt that instructs it to `create a new Crossplane Composition and CompositeResourceDefinition that represent an application` and that it should `compose resources directly using Crossplane v2.`

> Type the text that follow in the `claude` prompt.

```
Create a new Crossplane Composition and CompositeResourceDefinition that represent an application. The Composition should compose a Kubernetes Deployment, Service, and Ingress. Do it in separate files. Compose resources directly using Crossplane v2.
```

The outcome is, this time, even worse. Unlike the response of the first prompt that admitted that it cannot find the information, this time it confidently created a `CompositeResourceDefinition` based on `v1` API. If I would not already know that it's wrong and that whatever follows next would not work, I would probably spend quite some time trying to figure out why the code it generated with absolute confidence is ultimately failing.

Let's get out of Cloud Code and fix that.

> Press `ctrl+c` twice to exit `claude`.

## Agents with Context7 MCP

We'll fix the problem we're facing by enabling the AI agent to use Context7 which will provide it with up-to-date documentation for any, and I repeat, any public project. We'll do that by adding an MCP server.

```sh
claude mcp add context7 --scope project \
    -- npx -y @upstash/context7-mcp@latest
```

I won't go into details about MCP servers since I'm already working on a post that will talk about them in more detail. Stay tuned. For now, what matters, is that we instructed `claude` to spin up a server that will give it the information it needs. That server contains up-to-date information about thousands of project and the agent will use it in tandem with whichever LLM it is set to use.

Let's go back to `claude`.

```sh
claude
```

Next, we'll ask it the same question about Crossplane v2 features again but, this time, we'll instruct it to use `Context7` server we just added.

> Type the text that follow in the `claude` prompt.

```
What are the most important new features and changes in Crossplane v2? Use Context7.
```

We can see that, this time, it asked us, a couple of times, whether we want to use `context7`. First time it asked for the permission to `resolve-library-id` which allows us to find the specific documentation we're looking for, and the second time to `get-library-docs` which will give us the information from the official Crossplane documentation.

The important note here is that by doing so it is fetching the latest Crossplane docs or, at least, the version that might be only a few days old. We'll see later how we can change that so that the documentation is lirelly the latest one.

After a bit more thinking, it spit out the answer that is indeed the correct one.

It figured out that v2 got `Namespaced Resources`, that it can create `Composition of Any Kubernetes Resources`, and that there is a `New Copmosite Resource Definition` `API Version`.

Before we started using Context7, the agent created for us an XRD based on v1, and, this time, it found out that there is `v2alpha1` API.

That's absolutely awesome since all that means that the agent now works with up-to-date data and we can ask it, again, to `Create a new Crossplane Composition and CompositeResourceDefinition` based on `Crossplane v2` spec.

> Type the text that follow in the `claude` prompt.

```
Create a new Crossplane Composition and CompositeResourceDefinition that represent an application. The Composition should compose a Kubernetes Deployment, Service, and Ingress. Do it in separate files. Compose resources directly using Crossplane v2.
```

After a bit of thinking, just as before, it created the first file for us. This time, however, it did the right thing by using the `v2alpha1` API.

We'll let it create the rest and you need to trust me when I say that, this time, everything it did is correct because it used not only the information it contains in the LLM it's using, but also up-to-date docs accessible through Context7.

Let's take a closer look at Context7.

## What Is Context7?

What is [Context7](https://context7.com)?

It is an **MCP server that pulls up-to-date, version-specific documentation** and code examples directly from the source.

Right now, it includes over five thousand projects, and that number is growing rapidly.

Let's say that we are interested in `crossplane`. We can check whether it is already included by searching for it.

Right now, there are four repositories that match that keyword. That was not the case few days ago when I realized that there is the Crossplane project itself, but that the repo with the docs is not included. So... I added it and we'll see in a moment how to do that.

We can see the `UPDATE` field that tells us when was each of those repos pulled into Context7.

Crossplane docs is `5 days` old, while I said earlier that we always get up-to-date information with Context7. That was not fully true. More precise wording would be that we might get outdated information but that we can update it ourselves.

If we go to, for example, Crossplane docs, we can click the `Refresh` button and, a few moments later, the data related to that project will be fully up-to-date.

Now, even though there are thousands of repos in there, we might still be working with something currently not available in Context7.

I, for example, tend to use Crossplane KCL function a lot and it is currently not available in Context7. We can fix that as well by simply typing the repo and pressing the `+ Add Docs` button. A few minutes later all the docs in that repo will be included for anyone using Context7.

> Do not waste your time trying to add the `https://github.com/crossplane-contrib/function-kcl` repo yourself. I already added it and you should see it if you search for `crossplane` or `kcl`.

That's about it. Context7 is a simple, yet very effective MCP server that makes a huge difference when using AI agents for software development or operations.

Try it. Use it. It's a must.

## Destroy

> Press `ctrl+c` twice to exit `claude`.


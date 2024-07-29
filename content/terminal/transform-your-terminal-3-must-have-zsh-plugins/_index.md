
+++
title = 'Transform Your Terminal: 3 Must-Have Zsh Plugins!'
date = 2024-07-29T15:00:00+00:00
draft = false
+++

I use Zsh as my shell. It is the default shell in macOS and available in any other operating system. There's **no way I'll ever go back to Bash** and the only other Shell comparable to Zsh is Fish.

Fish is great, maybe even better than Zsh, but it's **not POSIX compliant** meaning that some of the commands I would use in Fish might not work elsewhere. Zsh, on the other hand, is POSIX compliant meaning that whatever I do in it would work in any other Shell, except Fish. Compatibility wins.

All in all, Zsh is great and I love it. However, Zsh alone is... well, it's **just a shell**. We need to extend it to make it truly great.

Specifically, there are **three must-have plugins**. With those we'll explore today, Zsh gets transformed from "yet another shell" to "I can't live without it".

Here it goes. Here are the three must-have Zsh plugins that will transform the way you work.

<!--more-->

{{< youtube MT7lA2nN-Nc >}}

## Setup

> Install [zsh](https://zsh.org) if you don't have it already. It is the default shell in macOS (not sure about other operating systems).

> Install https://github.com/zsh-users/zsh-autosuggestions

> Install https://github.com/zsh-users/zsh-history-substring-search

```sh
echo "
bindkey '^[[A' history-substring-search-up
bindkey '^[[B' history-substring-search-down
zstyle ':completion:*' menu yes select
" | tee -a ~/.zshrc
```

> Install https://github.com/zsh-users/zsh-syntax-highlighting

> Make sure that Docker is up-and-running.

## Zsh Syntax Highlightning (zsh-syntax-highlighting)

The first must-have plugin is [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting). It highlights the commands we type in Zsh. This is a must-have plugin because it helps you catch typos and syntax errors before you hit enter. At the same time, it makes commands we type much more readable by applying colors to them.

If, for example, I type `eco`...

```sh
eco
```

...I can immediately see that there is no such command since it is in red. I made a typo. The correct command is `echo`.

> Unfortunately, I could not show colors in this post so I advice you to follow along in your terminal to see proper outputs.

Now, if I do type the `echo` command correctly, it turns into green while the text I type is yellow.

```sh
echo "What is this?"
```

We'll execute a few other commands, not only to show syntax highlighting but also to generate a bit of history for the plugin that follows.

```sh
docker container ls
```

The command is in green, and the arguments in white.

The same can be said for the following two commands.

```sh
docker container run --detach --name nginx nginx

docker container run --rm --detach --name alpine alpine sleep 1
```

Now, you might say that syntax highlightning is not that important, and you'd be right. It's not. But it's nice to have.

The next one is a life-changer.

## Zsh Autosuggestions (zsh-autosuggestions)

The [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) plugin is a must-have. It is a time-saver.

Take a look at this.

I can start typing a command like `docker` and the last command that starts with `docker` will be suggested to me.

```sh
docker
```

> Press `→` to autocomplete `docker container run --rm --detach --name alpine alpine sleep 1`

If that's what I want to run, I can just press the right arrow key to autocomplete the command.

In some other cases, I might not want to re-run the last command that starts with docker but some other command that also starts with docker.

In that case, I can just type `docker`, hold the `option` key and press the arrow right key twice to reach `docker container run`.

Now, that's not the command I want to run either. I do know that the next argument was `--detach` so I can now start typing `--de` and... That's the one I want to run. Now I can press the right arrow key to autocomplete the command.

```sh
docker
```

> Press `option` + `→` twice to reach `docker container run`, type `--de`, and press `→` to autocomplete `docker container run --detach --name nginx nginx`

That command should fail since I'm already running a container with the same name. That should not matter. What matters is that with the **zsh-autosuggestions** plugin, I can autocomplete commands in a way that I never could before and that saves me a lot of time and, more importantly, allows my to avoid remembering all the commands and arguments I would need to use.

However, sometimes I do not remember the first few arguments of a command I should run and I might want to go through history to find it. That's where the third plugin comes in.

## Zsh History Substring Search (zsh-history-substring-search)

Typically, I would execute the *history* command and pipe the output to grep to find the command I'm looking for.

With [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search) finding commands from the history becomes much easier and faster.

Let's say that I would like to list all the containers and that I bumped my head earlier this morning and forgot the command to do so. I can start typing `docker container` and press the up arrow key to go through history until I find `docker container ls` and just press the enter key to run it.

```sh
docker container
```

> Press ↑ and ↓ to select `docker container ls`.

That was a silly example. Save from having amnesia, I would never forget the `ls` argument, so let's try something that I would likely forget. Let's say that I'd like to run alpine container that sleeps for a second.

I can do that by typing `docker container run` and press arrow up or down keys until I find the command I'm looking for, and press the enter key.

```sh
docker container run
```

> Press ↑ and ↓ to select `docker container run --detach --name nginx nginx`.

That's it. Those are three **must-have Zsh plugins** that you can install directly on top of Zsh or as part of Oh My Zsh. I stopped using Oh My Zsh a while ago due to performance issues and, simply, because I don't need it any more so I tend to install those plugins directly on top of Zsh.

If you're interested, I can explain why I abandoned Oh My Zsh in another post. For now, all I will say is...

Thanks for watching.
See you in the next one.
Cheers.

## Destroy

```sh
docker container rm nginx --force
```


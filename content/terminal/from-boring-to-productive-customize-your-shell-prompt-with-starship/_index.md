
+++
title = 'From Boring to Productive: Customize Your Shell Prompt with Starship'
date = 2024-07-22T16:00:00+00:00
draft = false
+++

What I'm about to say might sound silly, but I'm going to say it anyway.

One of the easiest ways to improve productivity is by having a **good Shell prompt**.

I, for example, tend to have a minimal prompt that shows only the pending actions I need to perform.

> Do NOT try to execute the commands in this section. We'll go through the setup and details in the next sections. This is only the preview of what's comming.

```
dotfiles-demo 
➜ 
```

That means that, most of the time, the only information shown in my prompt is the current directory which, in this case, is `dotfiles-demo`. It is a boring prompt, by design.

However, let's see what happens if...

<!--more-->

{{< youtube VLzc1iSDe9A >}}

...I start a Devbox session,...

```sh
devbox shell
```

...switch to a branch,...

```sh
git checkout -b something
```

...and create a Kubernetes cluster.

```sh
kind create cluster
```

The output is as follows.

```
☸ kind-kind in dotfiles-demo on  something via ❄️  devbox underwent 15s
➜ 
```

My prompt changed and now I know that I'm working with a Kubernetes cluster (`☸ kind-kind`) which I might need to destroy when I'm done, that I am in the ` something` branch which I should merge to mainline of create a pull request, and that I'm in a Devbox session (`via ❄️  devbox`) which I should exit when I'm done.

That way, my prompt is a kind of a **reminder of the tasks I should perform**, sooner or later while, at the same time, it gives me the **information I might need**. It is minimalistic when there is nothing special for me to do, yet informative when there are pending actions.

Your goals will differ and I will not try to convince you to use the same approach. Instead, I want to show how you can customize your prompt to fit your needs.

We'll do that with Starship which is a customizable Shell prompt. Now, that is not new by itself and there are plenty other tools that can customize prompts. What makes it special is the way customization works and the performance it provides. It is probably **the easiest way to create custom prompts** and, at the same time, **it is fast**, lightning fast. Other similar tools I used in the past fail at at least one of those two aspects. They are either difficult to customize or slow. Starship is neither.

Let's see it in action.

## Setup

This setup assumes that you are using `zsh`. If you are not, you can still use Starship but you might need to adjust the instructions accordingly.

> Install one of [Nerd Fonts](https://www.nerdfonts.com/). My recommendation is [Fira Code](https://github.com/tonsky/FiraCode), but any other should do.

> [Install Starship](https://starship.rs/installing).

```sh
git clone https://github.com/vfarcic/dotfiles-demo

cd dotfiles-demo
```

> Make sure that Docker is up-and-running. We'll use it to create a Kubernetes cluster with KinD.

> Watch https://youtu.be/WiFLtcBvGMU if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell
```

## Starship Presets

This is my prompt before I started using Starship.

```
vfarcic@Viktors-MacBook-Pro-2 dotfiles-demo %
```

That's what I get from a "virgin" Mac that is not customized in any way. It is not very informative and, frankly, it is ugly.

All I have to do to convert it into something much more useful and much less ugly is to initialize Starship.

```sh
eval "$(starship init zsh)"
```

> The previous command should be added to `~/.zshrc` so that it is executed every time we start a new shell session.

The output is as follows.

```
dotfiles-demo on  main via ❄️  impure (nix-shell-env) on ☁️  viktor@farcic.com(us-east1)
```

> I could not reproduce colors and "special" symbols in this post so you'll have to use imagination to picture different segments in the prompt being painted in different colors.

Now I can see which directory I'm in (`dotfiles-demo`), what is the current branch (`main`), that I'm in a Nix shell (`impure (nix-shell-env)`), and that I'm connected to a Google Cloud in `us-east1`.

That's pretty and useful, yet, for me, that prompt contains some missinformation and is cluttered with information I do not need. I am using a Nix shell, but indirectly through Devbox. Seeing `impure` is a bit missleading, even though it is technically correct. I do not need to see the cloud region in my prompt, and I do not care to know that I'm working in the `main` branch.

Fortunately, we can customize our prompt by changing starship.toml file. We can do that manually, or by applying one of the presets.

We can, for example, switch to the `bracketed-segments` preset.

```sh
starship preset bracketed-segments \
    --output ~/.config/starship.toml
```

The output is as follows.

```
dotfiles-demo [ main][❄️  impure (nix-shell-env)][☁️  viktor@farcic.com(us-east1)]
❯
```

Having brackets in the prompt results in too much noise, so I'll switch to the `plain-text-symbols` preset.

```sh
starship preset plain-text-symbols \
    --output ~/.config/starship.toml
```

The output is as follows.

```
dotfiles-demo on git main via nix impure (nix-shell-env) on gcp viktor@farcic.com(us-east1)
```

We can also switch to the `pure-preset` which is a minimalistic prompt.

```sh
starship preset pure-preset --output ~/.config/starship.toml
```

The output is as follows.

```
dotfiles-demo main
❯
```

There are plenty of other prompts like `pastel-powerline`,...

```sh
starship preset pastel-powerline \
    --output ~/.config/starship.toml
```

...which makes it more colorful.

```
vfarcic  …/dotfiles-demo   main   ♥ 20:00 
```

Then there is `tokyo-night` which is a bit more minimalistic.

```sh
starship preset tokyo-night --output ~/.config/starship.toml
```

The output is as follows.

```
░▒▓   …/dotfiles-demo   main   20:03 
❯
```

For those of you who like information on the right side of a prompt, there is `jetpack`.


```sh
starship preset jetpack --output ~/.config/starship.toml
```

The output is as follows.

```
◉           ▦ dotfiles-demo ◬ main nix ⊛ impure nix-shell-envon ☁️  viktor@farcic.com(us-east1)  20:04
```

...and so on and so forth.

One of the presets might be just what you need. Unfortunately, none of them is what I need so I created my own.

## Starship Configuration

Let's take a look at the `starship.toml` file.

```sh
cat starship.toml
```

The output is as follows.

```toml
"$schema" = 'https://starship.rs/config-schema.json'

add_newline = true

[character]
success_symbol = '[➜](bold green)'
error_symbol = '[✗](bold red) '

[package]
disabled = true

[cmd_duration]
min_time = 5000
format = 'underwent [$duration](bold yellow)'

[aws]
disabled = true

[azure]
format = 'on [$symbol($subscription)]($style) '
disabled = true

[gcloud]
disabled = true

[directory]
truncate_to_repo = true

[docker_context]
disabled = true

[git_branch]
ignore_branches = ['master', 'main']

[git_status]
conflicted = '🏳'
ahead = '🏎💨'
behind = '😰'
diverged = '😵'
up_to_date = ''
untracked = '🤷'
stashed = ''
modified = '📝'
staged = '[++\($count\)](green)'
renamed = '👅'
deleted = '🗑'
disabled = false

[golang]
disabled = true

[kubernetes]
disabled = false

[nix_shell]
disabled = false
impure_msg = 'devbox'
format = 'via [$symbol$state](bold blue) '

[sudo]
disabled = false
```

The first thing you'll notice is that Starship configuration is **as simple as it can get**.

First I'm telling it to move the cursor to the new line (`add_newline = true`) after the prompt. I prefer having a full line to type whatever I'm typing so I prefer that any prompt information is above.

Further on, I'm redefining few characters. I prefer arrow (`➜`) instead of the default greater then sign for the prompt. I also prefer a red cross (`✗`) for errors.

The rest are configurations for specific tools or files.

I don't care about packages like NPM, Gradle, Helm, and others so I disabled them by setting `disabled = true` in the `package` section.

I want to know when commands take more than `5000` milliseconds to execute so I'm showing the `duration` in `yellow`.

By default, I don't need any information about `aws`, `azure`, `gcloud`, and `docker`, so I `disabled` them. As you will see later, it's easy to enable any module if the need arises.

I'm interested in `branch` information only when I'm not working on `master` or `main` branches, so I told it to ignore those.

I do want to see statuses (`git_status`) of what was changed in the Git repo I'm working on, but I'm not necessarily happy with the default icons, so I changed some of those.

I disabled `golang` (`disabled = true`) and enabled `kubernetes` (`disabled = false`).

Since I tend to use Devbox to create Nix shells, I changed the `impure_msg` to `devbox` and tweaked the `format`.

Finally, I want to know if `sudo` credentials are cached so I enabled that module (`disabled = false`).

That config might look overwhelming at first, but it is actually quite simple. We can disable or enable any module by setting *disabled* to *true* or *false*, we can *format* the output, and we can change the symbols used to represent different states.

That's all there is to it.

Now, to apply such a configuration, all we have to do is write it inside `~/.config/starship.toml` file or, since we already have it, simply copy the existing file.

```sh
cp starship.toml ~/.config/starship.toml
```

The output is as follows.

```
dotfiles-demo via ❄️  devbox
➜
```

We can see that the prompt changed right away.

There is only the information about the current directory (`dotfiles-demo`), that I'm using a Nix shell (`via ❄️  devbox`), and the arrow (`➜`) that indicates where I can type the next command and that the previous command was successful.

Everything else in that configuration depends on the context and it will show additional information only under specific conditions.

## Starship In Action

For example, the configuration defined that command duration should be displayed only if it takes more than five seconds, so let's run a command that takes more than that by executing `sleep`.

```sh
sleep 6
```

The output is as follows.

```
dotfiles-demo via ❄️  devbox underwent 6s
➜
```

There we go. The prompt was augmented with `underwent 6s` message since the last command took more than five seconds to execute.

Similarly, we defined that branch information should be displaed if we're not in the master or main branches, so let's checkout `something`.

```sh
git checkout -b something
```

The output is as follows.

```
dotfiles-demo on  something via ❄️  devbox
➜
```

There we go again. We can clearly see that we're inside the `something` branch and that we might need to merge it to mainline.

If we switch back to the `main` branch,...

```sh
git checkout main
```

The output is as follows.

```
dotfiles-demo via ❄️  devbox
➜
```

...the branch information is gone.

Right now, I'm not connected to any Kubernetes cluster, so there is no information about that. Let's create one.

```sh
export KUBECONFIG=$PWD/kubeconfig.yaml

kind create cluster
```

The output is as follows.

```
☸ kind-kind in dotfiles-demo via ❄️  devbox underwent 15s
➜
```

TODO: Fast-forward until the `kind create cluster` finishes executing.

There we go. We can clearly see that we're connected to the `kind-kind` cluster.

If we delete the cluster,...

```sh
kind delete cluster
```

The output is as follows.

```
dotfiles-demo via ❄️  devbox
➜
```

...the Kubernetes information is gone.

If we execute a `sudo` command,...

```sh
sudo ls -l
```

The output is as follows.

```
dotfiles-demo via ❄️  devbox as 🧙
➜
```

...we can see that there is a potential danger to mess up something since sudo credentials are now cached (`as 🧙`).

Sometimes, we might be confused with the meaning of some of the modules within the current context. When that happens, we can show the output of a specific `module` like, for example, `nix_shell`.

```sh
starship module nix_shell
```

The output is as follows.

```
via ❄️  devbox %

dotfiles-demo via ❄️  devbox as 🧙
➜
```

We can see that `via ❄️  devbox` part of the prompt belongs to the *nix_shell* module.

If we get confused what each part of the prompt means we can simply ask Starship to `explain` the whole current prompt.

```sh
starship explain
```

The output is as follows.

```
 Here's a breakdown of your prompt:
 "dotfiles-demo " (1ms)   -  The current working directory
 "via ❄️  devbox " (<1ms)  -  The nix-shell environment
 "as 🧙 " (20ms)          -  The sudo credentials are currently cached
 "➜ " (<1ms)              -  A character (usually an arrow) beside where the text is entered in your terminal
```

We can see that it starts with the current working directory (`dotfiles-demo`), followed by the Nix shell environment (`via ❄️  devbox`), then the cached sudo credentials (`as 🧙`), and finally the character (`➜`) that indicates where we can type the next command.

One potential issue with Startship promts can be performance. Even though Startship is faster than most prompts, it might still slow down the terminal if we have too many modules. To mitigate that, we can check how much it takes to assemble each part of the prompt by executing `timings`.

```sh
starship timings
```

The output is as follows.

```
 Here are the timings of modules in your prompt (>=1ms or output):
 sudo        -  19ms  -   "as 🧙 "
 git_status  -  12ms  -   ""
 directory   -   5ms  -   "dotfiles-demo "
 kubernetes  -   1ms  -   ""
 git_branch  -   1ms  -   ""
 nix_shell   -  <1ms  -   "via ❄️  devbox "
 line_break  -  <1ms  -   "\n"
 character   -  <1ms  -   "➜ "
```

We can see that, in this case, `sudo` and `git_status` parts of the prompt are the slowest. If we want to improve performance, we can disable those modules or tweak the configuration to make them faster.

Finally, we can toggle any module on or off.

For example, if we do not want to see the information about `nix_shell`, we can toggle it off.

```sh
starship toggle nix_shell
```

The output is as follows.

```
dotfiles-demo as 🧙
➜
```

Later on, if we do want to have that information back, we can toggle it on.

```sh
starship toggle nix_shell
```

The output is as follows.

```
dotfiles-demo via ❄️  devbox as 🧙
➜
```

The *toggle* command is a quick way to enable or disable any module without the need to edit the configuration file. We could accomplish the same outcome by changing disabled parameter inside starship.toml file, but *toggle* is a quicker way to do it.

That's all there is to it. Starship is a simple yet powerful tool that can help us create custom prompts that fit our needs. It is fast, easy to use, and can be customized to show only the information we need.

Startship is what generates my prompts. You haven't seen it in my previous videos mostly because I did not want to redirect your attention from subjects I was discussing. However, when I work on my projects, I always use Startship for my prompts. It's awesome and you should give it a try.

Thank you for watching.
See you in the next one.
Cheers.

## Destroy

```sh
git branch -d something

exit
```


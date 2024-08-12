
+++
title = 'Master Your New Laptop Setup: Tools, Configs, and Secrets!'
date = 2024-08-12T15:00:00+00:00
draft = false
+++

This is a new machine. I just got it. It has nothing on it beyond what's available out-of-the box.

Actually, that's not true. The only thing I did was to clone a Git repo. That's it. That's all I have on this machine, for now.

<!--more-->

{{< youtube FH083GOJoIM >}}

This is how my terminal looks like.

```
viktorfarcic@Viktors-Mac-Studio ~ %
```

That's **depressing**. I cannot work like that. I need my tools and I need them to be configured just the way I like it.

Here it goes.

> Do not run the commands just yet. This is only a preview. We'll go through all the commands starting with the **setup** later.

```sh
./install.sh
```

The output is as follows (truncated for brevity).

```
...
Downloading and Installing
✓ Downloading devbox binary... [DONE]
→ Installing in /usr/local/bin/devbox (requires sudo)...
✓ Installing in /usr/local/bin/devbox... [DONE]
✓ Successfully installed devbox 🚀

Next Steps
  1. Learn how to use devbox
     Run devbox help or read the docs at https://github.com/jetify-com/devbox
  2. Get help and give feedback
     Join our community at https://discord.gg/jetify
```

That was the easy part. Global apps that I need are installed, but my terminal is still depressing. The tools that were installed have not been configured the way I like to use them and I'm still missing the tools specific to the demo.

```sh
devbox shell
```

The output is as follows.

```
(devbox) viktorfarcic@Viktors-Mac-Studio dotfiles %
```

The tools are now there, but they are still not configured properly. My terminal prompt is still depressing. I don't like being depressed, so let's change that.

```sh
./sync.sh

source ~/.zshrc
```

The output is as follows.

```
dotfiles [📝] via ❄️  devbox as 🧙 
➜ 
```

> I could not reproduce colors in this post so you'll have to use imagination to picture different segments in the prompt being painted in different colors.

That's it. Now I'm ready to work on my new laptop. Everything is set up just the way I like it, and it took me no time to get here.

Getting a new laptop is amazing. There's that feeling of opening the box while knowing that what's inside is faster, better, and sleaker than the machine we were using so far. Yet, at least in my case, that joy evaporates once I realize that I need to spend a large amount of time making the new machine behave like the old one. 

If you're priviledged, or spoiled as I am, to use multiple machines, you might want all of them to be configured the same way so you can work on any of them in the same way and use the same tools configured in the same way. I, for example, tend to use a desktop computer when working from home and a laptop when traveling. Those two need to be the same, at least when software is concerned, or I'll go crazy.

Then there is the case of sharing configurations between people working in the same team. Some of that configuration might be in project's Git repo but there are others that might be global.

All in all, I need to install all the software I'm using on any machine, old or new. That's the least of my problems. A bigger issue is configuring everything I'm using to behave just the way I like it.

More often than not, the pain is in **setting up all the dot files**. Those are the hidden configuration files typically located in the home directory. They're called dot files since they are prefixed with dot.

To begin with, there is *zshrc* or *bashrc* file that, essentially, contains all the instructions required to start a Shell session and that one can often have hundreds of lines alone. Then there are application-specific configurations. For example, I spent quite some time setting up [Starship](https://youtu.be/VLzc1iSDe9A) prompt to be just the way I like it and the result of all that work is in the *.config/starship.toml*.

Essentially, I need most of the dot files located in the home directory to be the same on any machine I might be using. If I get a new computer, it needs to behave just as the old one and if I switch from one to another, things need to be the same as well.

Now, as you can probably guess, the solution is to keep all the installation scripts and configuration files in a shared location accessible from any machine. That shared location is Git or, to be more specific, in my case a GitHub repo. There are three problems though.

1. How do we keep the repo always up-to-date with any changes made on a computer we might be using?
2. How do we synchronize what's in that repo to whichever machine we might be using?
3. How do we ensure that secrets are not leaked to the Git repo, yet synchronized into a machine we're working on?

I'll try to answer all those questions today. Actually, I'll do better than that. I'll show how to implement solutions to those problems.

## Setup

```sh
cd ~/

git clone https://github.com/vfarcic/dotfiles

cd dotfiles

git pull

git fetch

git checkout dotfiles

chmod +x install.sh
```

> The commands that follow will copy a few dot files that you might have in your home directory. After the demo you should be able to restore them if you'd like to go back to the initial state.

> If one of the commands that follow throw an error, it's most likely because you do not have the corresponding dot file in your home directory. That's fine. Just ignore the error and move to the next command.

```sh
mv ~/.zshrc ~/.zshrc-orig

mv ~/.config/starship.toml ~/.config/starship.toml-orig

mv ~/.config/fabric ~/.config/fabric-orig
```

## How to Manage dot Files?

Let's start with a proof that I am not a liar, or, at least, that I am experienced at simulating conditions that are not real.

If I try to output `~/.zshrc`,...

```sh
cat ~/.zshrc
```

The output is as follows.

```
cat: /Users/viktorfarcic/.zshrc: No such file or directory
```

...we can see that ZSH configuration file does not exist. My ZSH is untouched.

We can confirm that other configurations are not there either. For example, if I try to output `~/.config/starship.toml`,...

```sh
cat ~/.config/starship.toml
```

The output is as follows.

```
cat: '/Users/vfarcic/.config/starship.toml': No such file or directory
```

...we can see that one is not there either. My dear new machine has not been touched by a human hand. It's coming straight from the factory (except that I cloned a repo with some files).

My current location is important, so let's take a look at where I am.

```sh
pwd
```

The output is as follows.

```
/Users/vfarcic/dotfiles
```

I am in a Git repo `dotfiles`. The important part is that I cloned that repo directly into the home directory. Remember that since what I'll show soon will be based on that fact.

The first thing I should do is install the apps I need.

I created a simple shell script for that so let's take a look at it.

```sh
cat install.sh
```

The output is as follows.

```sh
brew install font-fira-code-nerd-font

# https://starship.rs/guide/#%F0%9F%9A%80-installation
brew install starship

# https://github.com/zdharma-continuum/zinit?tab=readme-ov-file#install
bash -c "$(curl --fail --show-error --silent \
    --location https://raw.githubusercontent.com/zdharma-continuum/zinit/HEAD/scripts/install.sh)"

# https://github.com/nvbn/thefuck?tab=readme-ov-file#installation
brew install thefuck

# https://github.com/tonsky/FiraCode/wiki/Installing
brew install --cask font-fira-code

# https://github.com/eza-community/eza/blob/main/INSTALL.md
brew install eza

# https://github.com/junegunn/fzf?tab=readme-ov-file#installationc
brew install fzf

# https://github.com/ajeetdsouza/zoxide?tab=readme-ov-file#installation
brew install zoxide

# https://github.com/sharkdp/bat
brew install bat

# https://www.jetify.com/devbox/docs/installing_devbox/
curl -fsSL https://get.jetify.com/devbox | bash
```

> If you are NOT using macOS, you might need to adapt the commands in that script to your operating system.

There is nothing special about that script. Most of the commands are *brew install* this and *brew install* that. That's such a simple and boring script that I would not be offended if you stopped reading this post and never come back. However, before you do just that, I'd advice staying a while longer since the purpose of this post is not to show you how to install stuff, I assume you already know that, but how to do everything else.

With that in mind, let's move forward fast and simply execute that script.

```sh
./install.sh
```

The output is as follows (truncated for brevity).

```
...
Downloading and Installing
✓ Downloading devbox binary... [DONE]
→ Installing in /usr/local/bin/devbox (requires sudo)...
✓ Installing in /usr/local/bin/devbox... [DONE]
✓ Successfully installed devbox 🚀

Next Steps
  1. Learn how to use devbox
     Run devbox help or read the docs at https://github.com/jetify-com/devbox
  2. Get help and give feedback
     Join our community at https://discord.gg/jetify
```

A few moments later...

Everything was installed, and that was the boring part that you already know how to do. The only important thing to note is that when I say "everything is installed", I am lying. I'm a lying liar that lies. The truth is that only global apps or, to be more precise, global CLIs were installed. There aren't many of them, simply because most of the tools I need are project-specific, including the project to configure everything. Those project specific tools will be installed through Devbox.

Here's the definition.

```sh
cat devbox.json
```

The output is as follows.

```
{
  "$schema": "https://raw.githubusercontent.com/jetify-com/devbox/0.11.0/.schema/devbox.schema.json",
  "packages": [
    "kind@0.23.0",
    "google-cloud-sdk@478.0.0",
    "gum@0.14.1",
    "teller@1.5.6",
    "stow@2.4.0",
    "gh@2.50.0",
    "jq@1.7.1",
    "yq-go@4.44.1",
    "kubectl@1.30.1"
  ],
  "shell": {
    "init_hook": [
      "autoload -Uz compinit",
      "compinit",
      "source <(kubectl completion zsh)",
    ],
    "scripts": {}
  }
}
```

The first thing you'll notice is that the output is depressingly ugly. There is no syntax coloring. Even though we installed some tools, they are not yet configured so *cat* is still *cat* and not an alias to *bat*.

Apart from that, you should notice `teller` and `stow`. Those two tools are essential for what we're trying to do.

I won't go into Devbox and why you should use it since I already did that in the [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) video. Instead, we'll just start a new `shell` that will bring us the tools we need.

```sh
devbox shell
```

Now that we have the tools, both those installed globally and those we need in relataion to the repo we're working on, we can, finally, take a look at the script that does that.

```sh
cat sync.sh
```

The output is as follows.

```
gcloud auth login

teller env >.config/fabric/.env

rm ~/.zshrc

stow .

echo "## Follow the instructions at https://github.com/tonsky/FiraCode/wiki/VS-Code-Instructions to enable Fira Code in VS Code" \
    | gum format

echo '## Execute `source ~/.zshrc`.' | gum format
```

Two of those lines are critical.

First, since we cannot store credentials of any type in Git, we're using `teller` to retrieve them from whichever secrets storage we're using and generating configuration (`.config`) for `fabric`, one of the tools we'll use later.

Teller is awesome. It is one of my favorite tools and you should adopt it if you haven't already. Just as with Devbox, I won't go through it in more detail since I already did that in the [Secrets Made My Life Miserable - Consume Secrets Easily With Teller](https://youtu.be/Vcjz-YM3uLQ) video.

> If you're following along, the `teller env` command will not work as-is. You'll need to modify `.teller.yml` to point to your secrets storage.

The key is in the `stow .` line. That one is the star of today's show.

Stow is a **symlink farm manager**. It will take (almost) everything we have in this directory and create links in the parent directory.

I know... What I just said might sound confusing. Let's just execute that script and clarify what Stow does by observing the outcome.

```sh
chmod +x sync.sh

./sync.sh
```

The output is as follows.

```
Your browser has been opened to visit:

    https://accounts.google.com/o/oauth2/auth?response_type=code&client_id=32555940559.apps.googleusercontent.com&redirect_uri=http%3A%2F%2Flocalhost%3A8085%2F&scope=openid+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.email+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fcloud-platform+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fappengine.admin+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fsqlservice.login+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fcompute+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Faccounts.reauth&state=1ytC82p0kSBpyBt3jGPjEFqsm7tjXg&access_type=offline&code_challenge=Kl3lqOMWJqHWIBoTVFp3AEZXdXN_Fi7OgsrxEAFk9Q8&code_challenge_method=S256


You are now logged in as [viktor@farcic.com].
Your current project is [dot-20210822142533].  You can change this setting by running:
  $ gcloud config set project PROJECT_ID
  ▌ Follow the instructions at https://github.com/tonsky/FiraCode/wiki/VS-Code-Instructions to enable Fira Code in VS Code
  ▌ Execute  source ~/.zshrc .
```

Stow created the symbolic links so now *.zshrc* from this repo is available through the link in the home directory (`~/.zshrc`) as well and we can, for example, `source` it.

```sh
source ~/.zshrc
```

The output is as follows.

```
dotfiles [📝] via ❄️  devbox
➜ 
```

You'll notice that everything **automagically started working** just the way I want it to work. The prompt changed to the Starship configuration. Everything is colored and all the tools are working.

How did that "magic" happened.

The Git repository where we're in right now has all the dot files I need on any machine I'm using or will use in the future.

```sh
ls
```

The output is as follows.

```
.
..
.config
.devbox
.git
.gitignore
.stow-local-ignore
.teller.yml
.zshrc
devbox.json
devbox.lock
install.sh
sync.sh
uninstall.sh
```

Do you see that beautiful output of the *ls* command that distinguishes between directories and different types of files? That's *eza* aliased as *ls*. That's one of many outcomes of synchronizing dot files.

Among other files, we can see that there's `.zshrc` which is ZShell configuration file. There's also the `.config` directory with the configuration for Starship, Fabric, or any other tool we might be using.

I won't go into details of what's in the *.zshrc*, *.config/starship.toml* and other files. They will be different in your case. What's important is that you can have all the configuration you need in a single place and synchronize it across all your machines.

That synchronization is done with Stow which creates symbolic links in the parent directory to the files in this repository.

We can see that's truly the case by taking a look at the `.zshrc` file located in the home directory (`~/`).

```sh
ls ~/.zshrc
```

The output is as follows.

```
-- /Users/vfarcic/.zshrc -> dotfiles/.zshrc
```

We can see that `.zshrc` is a link to `dotfiles/.zshrc`. That's the important part. Since it is not a physical file but a link to the file in this repository, if we pull a new version of the repo, the changes will be immediately reflected in the configuration. Similarly, if we, or some application, edit the file in the home directory we will effectivelly edit the file in the repository and we can push those changes to the Git repo so that it can be synched to other machines.

The same is true for any other file in the repository. Stow converted all those **files to symbolic links** in the parent directory which just so happens to be the home directory where dot files are. The only exceptions are entries in the *.stow-local-ignore* which serves a similar purpose as the *.gitignore* file. It tells Stow which files to ignore.

Now that everything on this machine is set up to be the same as on any other machine I use, we can, for example, open a new terminal session and start working.

> Start a new terminal session.

The output is as follows.

```
~/code
➜
```

We can see that the prompt is just as it should be and the tools are configured properly. To demonstrate that, we can use Zoxide which is a replacement of *cd* command that allows us to navigate directories more easily. So, if we type `cd dot ` and press `tab`, we'll be taken to the `dotfiles` directory automatically.

```sh
cd dot 
```

> Ensure that there is `space` at the end of the previous command and press the `tab` key to go directly to the `dotfiles` using `zoxide`.

The output is as follows.

```sh
dotfiles 
➜ 
```

Thank you for watching.
See you in the next one.
Cheers.

## Destroy

> Ignore errors in the commands that follow.

```sh
mv ~/.zshrc-orig ~/.zshrc

mv ~/.config/starship.toml-orig ~/.config/starship.toml

mv ~/.config/fabric-orig ~/.config/fabric
```


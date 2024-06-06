+++
title = 'Mastering Kubernetes Testing Kyverno Chainsaw!'
date = 2024-06-03T16:00:00+00:00
draft = false
+++

When I work locally, if I need to create a cluster I just execute `cluster-create`, wait for a few moments, and a local cluster with everything I need is running.
<!--more-->

{{< youtube hgNN2wOE7lc >}}

```sh
just cluster-create
```

The output is as follows.

```
...
function.pkg.crossplane.io/crossplane-contrib-function-go-templating condition met
function.pkg.crossplane.io/crossplane-contrib-function-patch-and-transform condition met
```

If I need to run tests in the background while developing, I just execute `test-watch` and let my tests run every time I make a change to the source code.

```sh
just test-watch
```

The output is as follows.

```
....
PASS
Tests Summary...
- Passed  tests 3
- Failed  tests 0
- Skipped tests 0
Done.
[Command was successful]
```

When I'm finished, I just execute `cluster-destroy`, and move on.

```sh
just cluster-destroy
```

The tool I'm using to just execute something is... *just*.

It is a tool that allows us to **execute tasks** which, with *just*, are called recipes. That's what it does, and the question is whether it does it well. Should you use it? Should you replace your Makefile, Taskfile, or other formats to *just*?

It's no secret that **I don't like Makefile**. It served us well over decades, but it's too complex and too brittle for my taste. If I never see .PHONY recipes again, I will die a happy person. If I never again encounter an error because I used spaces instead of tabs, I will consider it a success. Tabs are evil and should never be used as indentation.

just solves many of the issues I have with GNU make and Makefile. It is a valid replacement for a tool and a format that should have been retired long time ago.

Now, I already talked about an alternative to Makefile in that video where I explored Taskfile. Many of you told me that its great, but quite a few of you sent me a message complaining about it being YAML. **"We do NOT want more YAML!"** **"YAML is evil!"** **"YAML ruined my marriage!"**.

I invented only one of those.

Others asked me what I think about *just* since it is very similar to Task, but without YAML. Instead of answering those directly, I decided to make a video.

I can summarize *just* as a Make and Makefile replacement, just as Task and Taskfile are also replacements. The major difference is that there is no YAML.

Let's take a look at it.

## Setup

```sh
git clone https://github.com/vfarcic/crossplane-kubernetes

cd crossplane-kubernetes

git pull

git checkout just
```

> Make sure that Docker is up-and-running. We'll use it to create a Kind cluster.

> Watch https://youtu.be/WiFLtcBvGMU if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell
```

## just Define Justfile and just Run It

As we can expect from the tools that enable us to run something, we can use just to execute any previously defined recipes.

In this project, I typically start my work by creating a local cluster, installing some third-party apps, deploying what I'm working on, and a few other things I need before I start working on it. I defined all that as a recipe called `cluster-create`, so let me execute it.

```sh
just cluster-create
```

The output is as follows.

```
...
kubectl wait --for=condition=healthy function.pkg.crossplane.io --all --timeout=300s
function.pkg.crossplane.io/crossplane-contrib-function-auto-ready condition met
function.pkg.crossplane.io/crossplane-contrib-function-go-templating condition met
function.pkg.crossplane.io/crossplane-contrib-function-patch-and-transform condition met
```

Now that everything I need is set up, I would normally execute `test-watch` recipe.

```sh
just test-watch
```

The output is as follows (truncated for brevity).

```
...
PASS
Tests Summary...
- Passed  tests 3
- Failed  tests 0
- Skipped tests 0
Done.
[Command was successful]
```

All my tests were executed and now it is watching for changes in my source code. The moment I modify any of the files that it is monitoring it will re-run the tests. That way I can focus on code knowing that tests are running in the background and will let me know if I mess it up.

> Press `ctrl+c` to stop the watcher.

Once I'm done working, I just execute `cluster-destroy` and wipe out everything.

```sh
just cluster-destroy
```

The output is as follows.

```
kind delete cluster
Deleting cluster "kind" ...
Deleted nodes: ["kind-control-plane"]
```

Now, if you watched my video about Taskfile, that one, you might have noticed that the recipes I just executed with just are the same as those I had with Taskfile. That's on purpose. This is the same project and I wanted to compare the two based on the same criteria. I wanted to see why would we choose one over the other.

Let's see what we can do with just apart from being able to execute recipes.

We can, for example, get a list of all the recipes by simply executing `just` without any additional argument.

```sh
just
```

The output is as follows.

```
just --list
Available recipes:
    cluster-create         # Creates a kind cluster, installs Crossplane, providers, and packages, waits until they are healthy, and runs tests.
    cluster-destroy        # Destroys the cluster
    default                # List tasks.
    package-apply          # Applies Compositions and Composite Resource Definition.
    package-generate       # Generates package files.
    package-generate-apply # Combines `package-generate` and `package-apply`.
    package-publish        # Builds and pushes the package.
    test                   # Combines `cluster-create`, `test-watch` (without the watcher), and `cluster-destroy` tasks.
    test-once              # Runs tests once assuming that the cluster is already created and everything is installed.
    test-watch             # Runs tests in the watch mode assuming that the cluster is already created and everything is installed.
```

There we go. That's the list of all the recipes we can execute with just in this project. There is a name of a recipe (`cluster-create`) and a description (`# Creates a kind cluster...`).

If we'd like a more detailed information about any of those, we can execute `just` with the instruction to `show` a specific recipe.

```sh
just --show cluster-create
```

The output is as follows.

```
# Creates a kind cluster, installs Crossplane, providers, and packages, waits until they are healthy, and runs tests.
cluster-create: package-generate _cluster-create-kind
    helm upgrade --install crossplane crossplane-stable/crossplane --namespace crossplane-system --create-namespace --wait
    for provider in `ls -1 providers | grep -v config`; do kubectl apply --filename providers/$provider; done
    just package-apply
    sleep 60
    kubectl wait --for=condition=healthy provider.pkg.crossplane.io --all --timeout={{ timeout }}
    kubectl wait --for=condition=healthy function.pkg.crossplane.io --all --timeout={{ timeout }}
```

There are quite a few other things we can do with the CLI. Nevertheless, what matters, at least when the CLI is concerned, is that we can discover which recipes are available, get information about any specific recipe, and execute it.

None of those things distinguish *just* from the competition. Where it differs is not that much in what it can, or cannot do, but, rather, in the syntax used to define recipes, so let's take a look at `Justfile`.

```sh
cat Justfile
```

The output is as follows.

```
timeout := "300s"

# List tasks.
default:
  just --list

# Generates package files.
package-generate:
  timoni build dot-kubernetes timoni > package/all.yaml
  head -n -1 package/all.yaml > package/all.yaml.tmp
  mv package/all.yaml.tmp package/all.yaml

# Applies Compositions and Composite Resource Definition.
package-apply:
  kubectl apply --filename package/definition.yaml && sleep 1
  kubectl apply --filename package/all.yaml

# Builds and pushes the package.
package-publish: package-generate
  up login --token $UP_TOKEN
  up xpkg build --package-root package --name kubernetes.xpkg
  up xpkg push --package package/kubernetes.xpkg xpkg.upbound.io/$UP_ACCOUNT/dot-kubernetes:$VERSION
  rm package/kubernetes.xpkg
  yq --inplace ".spec.package = \"xpkg.upbound.io/devops-toolkit/dot-kubernetes:$VERSION\"" config.yaml

# Combines `package-generate` and `package-apply`.
package-generate-apply: package-generate package-apply

# Combines `cluster-create`, `test-watch` (without the watcher), and `cluster-destroy` tasks.
test: cluster-create test-watch cluster-destroy

# Runs tests in the watch mode assuming that the cluster is already created and everything is installed.
test-watch: package-generate package-apply
  watchexec -w timoni -w tests "just package-generate-apply && chainsaw test"

# Runs tests once assuming that the cluster is already created and everything is installed.
test-once: package-generate package-apply
  chainsaw test

# Creates a kind cluster, installs Crossplane, providers, and packages, waits until they are healthy, and runs tests.
cluster-create: package-generate _cluster-create-kind
  helm upgrade --install crossplane crossplane-stable/crossplane --namespace crossplane-system --create-namespace --wait
  for provider in `ls -1 providers | grep -v config`; do kubectl apply --filename providers/$provider; done
  just package-apply
  sleep 60
  kubectl wait --for=condition=healthy provider.pkg.crossplane.io --all --timeout={{timeout}}
  kubectl wait --for=condition=healthy function.pkg.crossplane.io --all --timeout={{timeout}}

# Destroys the cluster
cluster-destroy:
  kind delete cluster

# Creates a kind cluster
_cluster-create-kind:
  -kind create cluster
  -helm repo add crossplane-stable https://charts.crossplane.io/stable
  -helm repo update
  helm upgrade --install crossplane crossplane-stable/crossplane --namespace crossplane-system --create-namespace --wait
  for provider in `ls -1 providers | grep -v config`; do kubectl apply --filename providers/$provider; done
```

At the top, I defined a variable `timeout`. That way I can easily change the value without trying to figure out where it is used. We'll see that one in action later.

The recipes are defined with a name that ends with a colon. The first recipe I defined is called `default` and it executes a single command `just --list`.

The first recipe is always the default one and, in this case, I chose that it should simply list all the available recipes. That way, I don't have to execute `just --list`. Instead, I can simply execute `just` as I did above.

Comments above recipe names (`# Generates package files.`) serve as descriptions. That's what was displayed when I listed all the recipes.

A recipe can execute any number of sequential commands. In the case of `package-generate`, I am executing `timoni`, followed by `head`, and `mv`. It does not matter what those commands do and why I have them. What matters is that any command can be executed sequentially as part of running a recipe.

Recipes can depend on other recipes.

In the case of `package-publish`, the `package-generate` will be executed before it starts running the commands (`up login`, `up xpkg`, ...).

Commands are not mandatory. We can have recipes that only depend on other recipes like in the case of `package-generate-apply` which would do nothing but run `package-generate` and `package-apply`.

Unlike Taskfile, Justfile does not have a built-in mechanism to watch for file changes and re-run a task or a recipe. That's not necessarily a bad thing since that feature in Taskfile does not work as intended. It is its weakest point so I'm glad that Justfile does not even attempt to do that. Instead, if we would like to watch for changes in some files and execute some commands we can use a separate CLI for that. In this case, I'm using `watchexec` to monitor `timoni` and `tests` directories and execute a `just` recipe `package-generate-apply` whenever a file in one of those directories changes.

Now, that's not ideal either since running just CLI from inside just can result in some silly issues. Nevertheless, there is no built-in watcher and that's still better than the one baked into Taskfile which does not work as it should.

One potential problem is that, unlike in Taskfile, we cannot instruct Justfile to run some dependencies (`package-generate`, `_cluster-create-kind`), then some commands (`helm upgrade...`), and then some other dependencies. The only way to accomplish that would be to treat those dependencies as exeternal just commands (`just package-apply`). Sometimes that works, while, at other times, that might result in issues. It would be much better if Justfile would have a mechanism to execute dependencies at any time and not only before we start running commands. That's one of the areas where Taskfile is much better.

We can also define "internal" recipes by prefixing them with `_` (`_cluster-create-kind`). Those recipes will not be available when executing just CLI but only internally. They can only be invoked by other recipes.

By default, execution of a recipe stops the moment it encounters an error. We can avoid that by prefixing a command with `-`. For example, if `kind create cluster` command fails, just recipe will continue running. It will ignore that error. So, whichever other recipe uses `_cluster-create-kind` as a dependency would continue running even if it fails to create a KinD cluster.

There are, ofcouse, many other features of just but I feel that what we explored so far is enough to give you a clue whether Justfile might be a good alternative to Makefile and Taskfile.

## just and Justfile Pros and Cons

Let me start by saying that Justfile is, without doubt, a better option than Makefile. I won't even bother listing all my grievances with Makefile. I don't want to see it ever again.

The real question is whether Justfile is better than Taskfile, so that's what I'll compare it with.

There are many things that work against Justfile, at least when compared with Taskfile, so let's start with those.

**Cons:**
* Smaller community
* VS Code addon
* No "special" features
* No dry-run
* No watcher

To begin with, just and Justfile have a **smaller community**. There are fewer forks and stars in the repo, fewer contributors, and less activity. Now, that's not necessarily a reason to choose one over the other, especially since just has a healthy and active community, just not as big as the one behind Taskfile. It's a minor con.

**VS Code** addon is not maintained any more and, frankly, it was never good. As a matter of fact, there is more than one of those and all but one are dead. The one that isn't is relatively new and not very good either.

just does not have **"special" features**. There are no for constructs, there is no option to defer execution of a task, and so on and so forth. I'd like to say that just is intentionally missing those things since it assumes that you'll use Shell scripts and commands to accomplish those but that's not the case since it did add some of those like the ability to ignore errors by prefixing a command with "-". That could also be done with a "special" syntax. Hence, I don't buy that is intentional but, rather, missing. Still, it's also a minor issue since you can indeed accomplish all those with a bit of scripting.

Next, there is no option to **mix execution of commands and dependencies**. We need to execute just recipe as an external command to run dependencies mixed with commands. This is also a minor issue.

What else... There is **no dry-run**. We can see which commands will be executed but we cannot get the exact commands after loops, variables, and other stuff is expanded. We cannot see what exactly will be executed. Taskfile does that very well. Now, to be honest, that's not something I use often, so it's a minor issue as well.

Finally, there is **no watcher**. Now, that's is not really a bad thing since the baked-in watcher in Taskfile does not work as it should be, so I use watchexec over there as well. Hence, that's not really a con.

Now, let's move into good things, into pros.

**Pros:**
* Simpler
* Not YAML

When compared with Makefile, the list of pros is huge, but that's not what we're doing today. We're comparing it with Taskfile and there are only two selling points.

To begin with, Justfile syntax is easier to learn than the syntax of Taskfile. Now, when I say "easier", both are easy. You will have no trouble learning either of the two. You'll be master in either in a matter of hours. So, it's just that you'll learn Justfile a bit faster. If it takes half a day to master Taksfile, it takes even less to master Justfile. So, this one is a minor advantage and I'm listing it only so that the one that comes next is not the only one.

The major difference is that it is not YAML. That's the only reason I can imagine one would choose Justfile over Taskfile.

Now, that might sound like an insignificant reason to use Justfile, especially since there are a few cons. They are minor, but cons nevertheless.

Still, YAML is not everyone's cup of tee and it might introduce unnecessary verbosity. Justfile feels more elegant. There is no need to specify keys and values. It's just a recipe followed with dependencies and commands. It's easier to write and easier to read.

All in all, both tools are great and the choice will likely depend on how much you like or dislike YAML.

## Destroy

```sh
exit

git checkout main
```

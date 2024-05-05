+++
title = 'Say Goodbye to Makefile - Use Taskfile to Manage Tasks in CI/CD Pipelines and Locally'
date = 2024-03-04T16:00:00+00:00
draft = false
+++

When I work with pipelines, what you might call CI, or CI/CD, I have "special" requirements that might not be the same as what others might have, or, more likely, what others think of.
<!--more-->

{{< youtube Z7EnwBaJzCk >}}

I need to be able to execute a set of tasks every time I push changes to a Git repo. That's, hopefully, everyone's requirement, so there's nothing new there.

I also need to be able to run tasks both **locally** while developing **and remotely** triggered by an event like a push to a Git repo. That's a tough one since most, if not all, pipeline solutions work only remotely. I tend to solve that issue by using a tool that us independant of pipelines and than simply invoke that tool from my laptop or from pipelines. That's why I advocate for Dagger which I explored in [Dagger: The Missing Ingredient for Your Disastrous CI/CD Pipeline](https://youtu.be/oosQ3z_9UEM).

Finally, I prefer using containers as an easy way to get all the tools needed for each of the steps (tasks) of a pipeline. That's also one of the things that makes **Dagger** special. It allows us to work with containers easily and in a similar way we'd work with them in remote pipelines.

Nevertheless, while some of you liked the idea of writing pipelines in a programming language of choice, many of you complained. Quite a few of you prefer a declarative format for defining pipelines.

This post is my way of saying "**I hear you**, I'll find something else."

So, what can we use to define all the tasks we should run, that would work well with containers, that would work both locally and remotely, and that would be using a declarative format to define all the steps?

## GNU Make (Makefile) to the Rescue?

My first reaction was to use **Makefile**. It's been around forever and many people know it. I could define all my tasks in Makefile and just run them from my laptop or from inside a Pipelines like GitHub Actions, Jenkins, Tekton, or whatever I'm using.

Now, unlike Dagger, Makefile does not provide any help when working with Containers. Sure, I can execute `docker` this and `docker` that commands, but that's cumbersome when we need to move files from one to another, share the state, and do whatever else we're used to doing in pipelines.

That's, however, not a problem since I'm moving **away from containers** when executing tasks locally is concerned. If you watched that [Say Goodbye to Containers - Ephemeral Environments with Nix Shell](https://youtu.be/0ulldVwZiKA), you know that I'm going back to Nix. It's awesome and I don't need containers to manage tooling I need. I am not dropping containers. All my apps are running and will continue running as containers, at least until WASM becames a thing. What I'm saying is that I don't need containers to run specific tasks that require specific tools. I believe that Nix is a better option.

So, lack of meaningful support for containers in Makefile is not an issue.

Nevertheless, **I don't like Makefile**. It's there. I use it, but I don't like it. There are quite a few issues, often not big ones, that make me look for something else. I won't waste your time with long rambling about all the things I dislike, so here's a short version.

Using **tabs** for indentations should be illegal, yet it is mandatory in Makefile. It's often **hard to read** it and deduce what it's supposed to do. Making it **list targets** and their descriptions is possible, but painful. It first tries to **read files** instead of executing commands resulting in some silly situations. There are quite a few other annoyances with Makefile and, as I already mentioned, I won't go into all of them nor explain in more depth why I don't like Makefile.

The point is that I want Makefile-like solution that is simple, that can do what I need it to do, that I can execute both locally and inside remote pipelines, and I don't care much about any "special" features related to containers.

The moment I crystalized those requirements in my head, the solution self-revealed itself. There is a tool that is very similar to Make and Makefile, yet one that does not make me miserable. That tool is Task.

## Setup

> Make sure that Docker is up-and-running

```sh
git clone https://github.com/vfarcic/crossplane-kubernetes

cd crossplane-kubernetes

git pull

git checkout -t origin/task
```

> Please watch [Say Goodbye to Containers - Ephemeral Environments with Nix Shell](https://youtu.be/0ulldVwZiKA) if you are not familiar with Nix.

```sh
nix-shell --run $SHELL
```

> Open a second terminal sessions in the same directory.

## Task and Taskfile (Replacements for Make and Makefile)

**[Task](https://taskfile.dev)** is, as you can guess from the name, a tool that helps us run tasks, like those related to testing, building, packaging, deploying, or whichever other tasks you might be executing, especially while developing. It's simple. It's easy to write and read, and, since it's a single command, it can run on your laptop, in a remote pipeline, or anywhere else. It's awesome for those that do want to be able to run tasks anywhere and not only as remote pipelines. It's great for those who us but do not like Make and Makefile. It is for those that do not want to use a programming language like Go or Python.

Let's take a look at a Taskfile I'm using in one of my projects.

```sh
cat Taskfile.yaml
```

The output is as follows.

```yaml
version: '3'

tasks:

  # Package
  package-generate:
    desc: Generates package files.
    cmds:
      - timoni build dot-kubernetes timoni > package/all.yaml
      - head -n -1 package/all.yaml > package/all.yaml.tmp
      - mv package/all.yaml.tmp package/all.yaml
  package-apply:
    desc: Applies Compositions and Composite Resource Definition.
    cmds:
      - for: ["definition.yaml", "all.yaml"]
        cmd: kubectl apply --filename package/{{ .ITEM }}
  package-publish:
    desc: Builds and pushes the package.
    deps:
      - package-generate
    cmds:
      - up login --token $UP_TOKEN
      - up xpkg build --package-root package --name kubernetes.xpkg
      - up xpkg push --package package/kubernetes.xpkg xpkg.upbound.io/$UP_ACCOUNT/dot-kubernetes:$VERSION
      - rm package/kubernetes.xpkg
      - yq --inplace ".spec.package = \"xpkg.upbound.io/devops-toolkit/dot-kubernetes:$VERSION\"" config.yaml

  # Test
  test:
    desc: Combines `cluster-create`, `test-watch` (without the watcher), and `cluster-destroy` tasks.
    cmds:
      - task: cluster-create
      - task: test-watch
      - defer: { task: cluster-destroy }
  test-watch:
    desc: Runs tests assuming that the cluster is already created and everything is installed.
    deps:
      - package-generate
    cmds:
      - task: package-apply
      - kubectl kuttl test
    watch: true
    sources:
      - timoni/**/*.cue
      - tests/**/*.yaml
    generates:
      - package/all.yaml

  # Cluster
  cluster-create:
    desc: Creates a kind cluster, installs Crossplane, providers, and packages, waits until they are healthy, and runs tests.
    vars:
      TIMEOUT: 300s
      PROVIDERS:
        sh: ls -1 providers | grep -v config
    deps:
      - package-generate
      - cluster-create-kind
      - helm-repo
    cmds:
      - helm upgrade --install crossplane crossplane-stable/crossplane --namespace crossplane-system --create-namespace --wait
      - for: { var: PROVIDERS }
        cmd: kubectl apply --filename providers/{{ .ITEM }}
      - task: package-apply
      - sleep 60
      - kubectl wait --for=condition=healthy provider.pkg.crossplane.io --all --timeout={{.TIMEOUT}}
      - kubectl wait --for=condition=healthy function.pkg.crossplane.io --all --timeout={{.TIMEOUT}}
  cluster-destroy:
    desc: Destroys the cluster
    cmds:
      - kind delete cluster
  cluster-create-kind:
    desc: Creates a kind cluster
    cmds:
      - cmd: kind create cluster
        ignore_error: true
    internal: true
  helm-repo:
    cmds:
      - helm repo add crossplane-stable https://charts.crossplane.io/stable
      - helm repo update
    internal: true
```

That's probably too much to digest in one go, so I'll go back to snippets from that file as we progress.

To begin with, we can easily discover what are the available tasks.

```sh
task --list
```

The output is as follows.

```
task: Available tasks for this project:
* cluster-create:         Creates a kind cluster, installs Crossplane, providers, and packages, waits until they are healthy, and runs tests.
* cluster-destroy:        Destroys the cluster
* package-apply:          Applies Compositions and Composite Resource Definition.
* package-generate:       Generates package files.
* package-publish:        Builds and pushes the package.
* test:                   Combines `cluster-create`, `test-watch` (without the watcher), and `cluster-destroy` tasks.
* test-watch:             Runs tests assuming that the cluster is already created and everything is installed.
```

Having a clear and easy to access list of all the tasks we can execute is a crucial part of the user-experience. I might not need it since I wrote that Taskfile but anyone else using it will appreciate it. That output is pretty similar to what we'd expect from a good CLI.

From there on, we can choose which task to execute.

We can also get a summary of a specific task.

```sh
task --summary package-generate
```

The output is as follows.

```
task: package-generate

Generates package files.

commands:
 - timoni build dot-kubernetes timoni > package/all.yaml
 - head -n -1 package/all.yaml > package/all.yaml.tmp
 - mv package/all.yaml.tmp package/all.yaml
```

Besides the description, we can see which commands will be executed if we run it. In this case, I'm executing Timoni to generate YAML manifests and removing the last line of the output. The details of what I'm doing in this task or any other are not important, especially since this project might be doing stuff that you will likely not need. What matters is that I'm using Taskfile to define all the tasks I need to execute and that, so far, it's been very helpful telling me what I can do with each of them.

Let's take a look at the definition of that task.

```sh
cat Taskfile.yaml | yq ".tasks.package-generate"
```

The output is as follows.

```yaml
desc: Generates package files.
cmds:
  - timoni build dot-kubernetes timoni > package/all.yaml
  - head -n -1 package/all.yaml > package/all.yaml.tmp
  - mv package/all.yaml.tmp package/all.yaml
```

As you can see, it's a simple list of commands (`cmds`) which we can execute with `task package-generate`.

```sh
task package-generate
```

The output is as follows.

```
task: [package-generate] timoni build dot-kubernetes timoni > package/all.yaml
task: [package-generate] head -n -1 package/all.yaml > package/all.yaml.tmp
task: [package-generate] mv package/all.yaml.tmp package/all.yaml
```

That's it. Discover all the available tasks and execute whichever task you'd like to run. It should be simple, and it is. The best part is that, as long as you know what to execute, there is almost nothing to learn, at least not for such a simple scenario.

But there's more, much more.

Let's take a look a slightly more complicated example.

```sh
cat Taskfile.yaml | yq ".tasks.cluster-create"
```

The output is as follows.

```yaml
desc: Creates a kind cluster, installs Crossplane, providers, and packages, waits until they are healthy, and runs tests.
vars:
  TIMEOUT: 300s
  PROVIDERS:
    sh: ls -1 providers | grep -v config
deps:
  - package-generate
  - cluster-create-kind
  - helm-repo
cmds:
  - helm upgrade --install crossplane crossplane-stable/crossplane --namespace crossplane-system --create-namespace --wait
  - for: {var: PROVIDERS}
    cmd: kubectl apply --filename providers/{{ .ITEM }}
  - task: package-apply
  - sleep 60
  - kubectl wait --for=condition=healthy provider.pkg.crossplane.io --all --timeout={{.TIMEOUT}}
  - kubectl wait --for=condition=healthy function.pkg.crossplane.io --all --timeout={{.TIMEOUT}}
```

That task adds a few additional features beyond a simple list of commands that should be executed.

To begin with, we can define variables like, in this case, `TIMEOUT` and `PROVIDERS`. While the `TIMEOUT` variable is straightfoward with a hard-coded value, `PROVIDERS` will execute a Shell command (`sh`) and convert the output into the value of that variable. In this case, I'm retrieving a list of all files in a directory with `ls`, and then excluding those that contain `config` with `grep`. Task does not care what those commands are. All it cares is that the output, in this case the list of files, will be stored in a variable.

Then we have dependencies (`deps`). Those are other tasks that will be executed before this one. That removes duplication. In this case, I need to generate a package both during execution of the `cluster-create` task, but also as a separate task or a part of some other task. That's the task we saw and executed earlier.

Dependencies are executed in parallel to speed up execution. It's almost always better to be fast. No one likes waiting for results. However, it is very important to keep that in mind when writing Taskfile. I got burned more than once by adding tasks to dependencies and forgetting that they run in parallel. As a result, I spend hours trying to figure out why the results are not as expected. So, when you're thinking whether to put a task as a depenency or into the list of commands, the question you should be asking is whether it can run in parallel with others. If it can't, it's not a dependency.

Anyways... Once all the dependant tasks are executed, it will execute the commands (`cmds`) of this task. Some of them, like `helm upgrade`, are straightforward commands while others are a bit more complicated. For example, I had to execute `kubectl apply` for each file in a directory except if that file contains `config` in the name. That's the list of files I stored as the value of the `PROVIDERS` variable. So, one of the commands will iterate over each value, each file, in the `PROVIDERS` variable, and execute `kubectl apply`.

There's also a possibility to execute another task (`package-apply`), not as a dependencies but as one of the task commands.

Now, as a user who just wants to understand what that task does without trying to figure out what the author wanted to accomplish, can simply execute it in the dry-run mode.

```sh
task cluster-create --dry
```

The output is as follows.

```
task: [helm-repo] helm repo add crossplane-stable https://charts.crossplane.io/stable
task: [helm-repo] helm repo update
task: [cluster-create-kind] kind create cluster
task: [package-generate] timoni build dot-kubernetes timoni > package/all.yaml
task: [package-generate] head -n -1 package/all.yaml > package/all.yaml.tmp
task: [package-generate] mv package/all.yaml.tmp package/all.yaml
task: [cluster-create] helm upgrade --install crossplane crossplane-stable/crossplane --namespace crossplane-system --create-namespace --wait
task: [cluster-create] kubectl apply --filename providers/aws.yaml
task: [cluster-create] kubectl apply --filename providers/azure.yaml
task: [cluster-create] kubectl apply --filename providers/civo.yaml
task: [cluster-create] kubectl apply --filename providers/function-auto-ready.yaml
task: [cluster-create] kubectl apply --filename providers/function-loop.yaml
task: [cluster-create] kubectl apply --filename providers/function-patch-and-transform.yaml
task: [cluster-create] kubectl apply --filename providers/google.yaml
task: [cluster-create] kubectl apply --filename providers/provider-helm-incluster.yaml
task: [cluster-create] kubectl apply --filename providers/provider-kubernetes-incluster.yaml
task: [package-apply] kubectl apply --filename package/definition.yaml
task: [package-apply] kubectl apply --filename package/all.yaml
task: [cluster-create] sleep 60
task: [cluster-create] kubectl wait --for=condition=healthy provider.pkg.crossplane.io --all --timeout=300s
task: [cluster-create] kubectl wait --for=condition=healthy function.pkg.crossplane.io --all --timeout=300s
```

That output is not a simple list of all the dependencies and commands specified in that task. It is truly a dry-run. It converted all that logic into the actual commands without running them. So, for example, we can the `kubectl apply` command that should be executed for each file in a directory, results in nine distinct commands and that two commands coming afterwards are inherited from the `package-apply` task.

Now that I saw what would happen if I run the `cluster-create` task, I can just run it.

```sh
task cluster-create
```

> That task will take a while to execute. Be patient...

The output is as follows (truncated for brevity).

```
task: [helm-repo] helm repo add crossplane-stable https://charts.crossplane.io/stable
task: [cluster-create-kind] kind create cluster
task: [package-generate] timoni build dot-kubernetes timoni > package/all.yaml
"crossplane-stable" already exists with the same configuration, skipping
task: [helm-repo] helm repo update
Hang tight while we grab the latest from your chart repositories...
Creating cluster "kind" ...
...Successfully got an update from the "paralus" chart repository
task: [package-generate] head -n -1 package/all.yaml > package/all.yaml.tmp
 ✓ Ensuring node image (kindest/node:v1.27.3) 🖼
task: [package-generate] mv package/all.yaml.tmp package/all.yaml
⠈⠑ Preparing nodes 📦  ...Successfully got an update from the "cilium" chart repository
⢄⡱ Preparing nodes 📦  ...Successfully got an update from the "crossplane-stable" chart repository
Update Complete. ⎈Happy Helming!⎈
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/
task: [cluster-create] helm upgrade --install crossplane crossplane-stable/crossplane --namespace crossplane-system --create-namespace --wait
Release "crossplane" does not exist. Installing it now.
NAME: crossplane
LAST DEPLOYED: Fri Jan 12 00:54:20 2024
NAMESPACE: crossplane-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Release: crossplane

Chart Name: crossplane
Chart Description: Crossplane is an open source Kubernetes add-on that enables platform teams to assemble infrastructure from multiple vendors, and expose higher level self-service APIs for application teams to consume.
Chart Version: 1.14.5
Chart Application Version: 1.14.5

Kube Version: v1.27.3
task: [cluster-create] kubectl apply --filename providers/aws.yaml
provider.pkg.crossplane.io/provider-aws-eks created
provider.pkg.crossplane.io/provider-aws-iam created
...
```

There's more, much more than we can cover in this post, so let me show you only one more feature. It's something I cannot live without, yet something that does not work as well as I'd hope it to work. That feature is the watcher.

```sh
cat Taskfile.yaml | yq ".tasks.test-watch"
```

The output is as follows.

```yaml
desc: Runs tests assuming that the cluster is already created and everything is installed.
deps:
  - package-generate
  - package-apply
cmds:
  - kubectl kuttl test
watch: true
sources:
  - timoni/**/*.cue
  - tests/**/*.yaml
generates:
  - package/all.yaml
```

In this case, I instructed it to `watch` for changes in files with `sources` and `generates`. Those should be self-explanatory. `sources` is the location of the source code it watches, and `generates` is the list of files generated from the source code.

So, the `test-watch` task monitors my source code and, whenever it changes, it will execute the dependencies (`deps`) and commands (`cmds`).

Let's see it in action.

```sh
task test-watch
```

The output is as follows (truncated for brevity).

```
task: Started watching for tasks: test-watch
task: [package-apply] kubectl apply --filename package/definition.yaml
task: [package-generate] timoni build dot-kubernetes timoni > package/all.yaml
task: [package-generate] head -n -1 package/all.yaml > package/all.yaml.tmp
task: [package-generate] mv package/all.yaml.tmp package/all.yaml
compositeresourcedefinition.apiextensions.crossplane.io/compositeclusters.devopstoolkitseries.com unchanged
task: [package-apply] kubectl apply --filename package/all.yaml
composition.apiextensions.crossplane.io/cluster-civo unchanged
composition.apiextensions.crossplane.io/cluster-aws-official unchanged
composition.apiextensions.crossplane.io/cluster-google-official unchanged
composition.apiextensions.crossplane.io/cluster-azure-official unchanged
task: [test-watch] kubectl kuttl test
=== RUN   kuttl
    harness.go:462: starting setup
    harness.go:252: running tests using configured kubeconfig.
    harness.go:275: Successful connection to cluster at: https://127.0.0.1:57230
    harness.go:360: running tests
    harness.go:73: going to run test suite with timeout of 30 seconds for each step
    harness.go:372: testsuite: ./tests/ has 4 tests
=== RUN   kuttl/harness
=== RUN   kuttl/harness/aws
...
=== CONT  kuttl
    harness.go:405: run tests finished
    harness.go:513: cleaning up
    harness.go:570: removing temp folder: ""
--- PASS: kuttl (12.78s)
    --- PASS: kuttl/harness (0.00s)
        --- PASS: kuttl/harness/civo (6.22s)
        --- PASS: kuttl/harness/azure (9.47s)
        --- PASS: kuttl/harness/google (9.82s)
        --- PASS: kuttl/harness/aws (12.75s)
PASS
```

After a while, it executed all the commands specified in that task, and now it is waiting patiently for me to change the source code. So, let me, for example, modify one of the tests, and...

> Modify `tests/aws/00-assert.yaml` in the second terminal session.

The output is as follows.

```
...
=== CONT  kuttl/harness/aws
    logger.go:42: 01:20:42 | aws/0-install | test step failed 0-install
    case.go:364: failed in step 0-install
    case.go:366: --- ClusterClaim:kuttl-test-dear-lobster/a-team-eks
        +++ ClusterClaim:kuttl-test-dear-lobster/a-team-eks
        @@ -1,20 +1,117 @@
         apiVersion: devopstoolkitseries.com/v1alpha1
         kind: ClusterClaim
         metadata:
        ...
        -      cluster: eksxxx
        +      cluster: eks
               provider: aws-official
        ...
--- FAIL: kuttl (35.53s)
    --- FAIL: kuttl/harness (0.00s)
        --- PASS: kuttl/harness/civo (6.27s)
        --- PASS: kuttl/harness/azure (7.32s)
        --- PASS: kuttl/harness/google (7.37s)
        --- FAIL: kuttl/harness/aws (35.50s)
FAIL
task: Failed to run task "test-watch": exit status 1
```

...it detected the change and started executing all the commands again. As a result, I tend to keep it running in one window while I write code in another. I don't have to run commands every time I want to perform some actions like testing, but just glimpse at the result and, depending on the outcome, either continue writing code or fix the issue I just caused. If you write tests first, you get **test-driven development**.

Let's do another change by, for example, undoing the previous one and,...

> Undo the change to `tests/aws/00-assert.yaml`.

The output is as follows (truncated for brevity).

```
task: [package-apply] kubectl apply --filename package/definition.yaml
task: [package-generate] timoni build dot-kubernetes timoni > package/all.yaml
compositeresourcedefinition.apiextensions.crossplane.io/compositeclusters.devopstoolkitseries.com unchanged
task: [package-apply] kubectl apply --filename package/all.yaml
task: [package-generate] head -n -1 package/all.yaml > package/all.yaml.tmp
task: [package-generate] mv package/all.yaml.tmp package/all.yaml
composition.apiextensions.crossplane.io/cluster-civo unchanged
composition.apiextensions.crossplane.io/cluster-aws-official unchanged
composition.apiextensions.crossplane.io/cluster-google-official unchanged
composition.apiextensions.crossplane.io/cluster-azure-official unchanged
task: [test-watch] kubectl kuttl test
=== RUN   kuttl
    harness.go:462: starting setup
    harness.go:252: running tests using configured kubeconfig.
    harness.go:275: Successful connection to cluster at: https://127.0.0.1:57230
    harness.go:360: running tests
    harness.go:73: going to run test suite with timeout of 30 seconds for each step
    harness.go:372: testsuite: ./tests/ has 4 tests
=== RUN   kuttl/harness
=== RUN   kuttl/harness/aws
...
=== CONT  kuttl
    harness.go:405: run tests finished
    harness.go:513: cleaning up
    harness.go:570: removing temp folder: ""
--- PASS: kuttl (12.86s)
    --- PASS: kuttl/harness (0.00s)
        --- PASS: kuttl/harness/civo (6.20s)
        --- PASS: kuttl/harness/azure (9.41s)
        --- PASS: kuttl/harness/google (9.81s)
        --- PASS: kuttl/harness/aws (12.83s)
PASS
```

...there we go. It run again.

Once I'm finished working, all I have to do is press `ctrl+c` to stop the watcher, and execute the `cluster-destroy` task that will destroy everything and leave my machine in the state it was before I started.

> Press `ctrl+c` to stop the process

```sh
task cluster-destroy
```

There are many other features **Task** offers, which I won't explore today. Instead, I want to talk about pros and cons using it but, before I do that, let me show you how the pipeline of that project looks like.

```sh
cat .github/workflows/build11.yaml
```

The output is as follows.

```yaml
name: build11
on:
  push:
    branches:
      - main
jobs:
  build-package:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          persist-credentials: false
          fetch-depth: 0
      - name: Setup Timoni
        uses: stefanprodan/timoni/actions/setup@main
      - name: Setup Task
        uses: arduino/setup-task@v1
      - name: Setup kuttl
        run: |
          (
            set -x; cd "$(mktemp -d)" &&
            OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
            ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
            KREW="krew-${OS}_${ARCH}" &&
            curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
            tar zxvf "${KREW}.tar.gz" &&
            ./"${KREW}" install krew
          )
          export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
          kubectl krew install kuttl
          echo "PATH=$PATH" >> "$GITHUB_ENV"
      - name: Setup up
        run: |
          curl -sL https://cli.upbound.io | sh
          mv up /usr/local/bin/
      - name: Test
        run: task test
      - name: Publish
        run: task package-publish
        env:
          UP_ACCOUNT: ${{ secrets.UP_ACCOUNT }}
          UP_TOKEN: ${{ secrets.UP_TOKEN }}
          VERSION: v0.11.${{ github.run_number }}
      - name: Commit changes
        run: |
          git config --local user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git config --local user.name "github-actions[bot]"
          git add .
          git commit -m "Config update [skip ci]"
      - name: Push changes
        uses: ad-m/github-push-action@master
        with:
          github_token: ${{ secrets.CROSSPLANE_TOKEN }}
          branch: ${{ github.ref }}
```

This is GitHub Actions pipeline and the logic would be the same no matter what your choice is.

If we ignore code `checkout` and the `Setup` steps that are installing all the tools that pipeline needs, all the other commands I had in that pipeline are now replaced with simple `task test` and `task package-publish` commands. Actually, there is the part that commits and pushes changes back to the repo. Since I have no plans to run those commands from Task, I left them as-is.

As a result, all the tasks that I might run locally or remotely are defined in one place, in Taskfile, and the pipeline just executes it. There is no duplication and, excluding the parts that are executed only from the pipeline, everything works everywhere.

I do have one pending task though. I'm planning to replace those `Setup` steps with Nix so that all the tools are also managed in one place, in `shell.nix`, but I was too lazy to do it, so it's in my TODO list.

Finally, there is also a VS Code extension which I haven't tried so I can't comment on it. I prefer terminals for everything but writing code and observability.

## Task (Taksfile) Pros and Cons

I could not find a single thing I don't like, assuming that I prefer a declarative way to define tasks.

Actually... There is one thing that annoys me with **Task**.

**Watcher is silly**. It will execute the dependencies and the tasks every time we change the source code. That might sound like the expected behavior at first, but it's not. That might result in a new iteration being executed before the previous ends completely messing up with whatever you're doing. That can be somewhat mitigated by changing the default watch interval from the default value of five seconds to whatever you expected to be the duration of the task execution, multiplied by two for good taste. Still, that's only a workaround and I think that the correct behavior should be for the watcher to wait until the execution of the current iteration is finished before starting a new one. I had to develop muscle memory to never save the file I'm working on if less than 10 seconds passed since the last time I saved. It's not the end-of-the-world type of an issue, but it is an annoying issue nevertheless. As a matter of fact, it could be the only negative thing to say about Task.

So, let's move onto cons.

**Cons**
* Watcher
* Containers support (maybe)

The first negative thing I might say is that Task was not designed to work with containers like, for example, Dagger is. You can certainly run it inside containers and you can certainly use it to execute `docker` this and `docker` that commands. However, it does not have anything baked in. Now, for me, that is not a real con for two reasons. When executing it locally, I switched to Nix so I don't need containers to run tasks. As a matter of fact, I stopped using containers when running development tasks. Second, when tasks run remotely, pipelines already support containers and we normally need Task to specify what will be executed inside them rather than it spinning them up. So, for me, no "special" container-related features is not a negative thing but I can understand that for some it is.

The second, and more important con thing is the watcher. It works, but it does not work well. It should not execute the task whenever source code changes but whenever source code changes unless a task is already running. Otherwise, we might end up with multiple tasks running in parallel or a new task stopping the old one or whichever random permutation might happen.

As for pros... There are many and, in the interest of brevity, I'll limit myself to only a few.

**Pros**
* Easy
* Feature complete
* Excellent docs
* Mature

To begin with, Taskfile is easy to write and easy to read. Anyone can learn it in no time.

Next, Task probably provides all the features we might need. It has dependencies, it can run them in parallel, it can run in the watch mode (when it works), and so on and so forth.

What else... Oh yeah. Documentation is excellent. I never had an issue finding an example or details for what I'm trying to accomplish. The documentation is not huge. It does not contain everything anyone might ever need. Yet, it is not too short either. It is not missing anything, yet it's not overwhelming either. It's just right.

Finally, it's a mature project with large and active community that's been around for some time now. It's not too old so it's based on the experience we have from older ones like Make, yet it was not born a few weeks ago. It's age is just right.

That's it. There is only one (or two) negative things I can say about it, assuming that we're looking for a declarative task or pipeline or steps or commands executor. If you're trying to write it in Go, or Python, or some other "real" language, Dagger is still a better option. But if you are looking for a solution that allows us to use a declarative format and if YAML is the one you prefer, Task is a great choice, especially for those who are using Make, but are not happy with it.

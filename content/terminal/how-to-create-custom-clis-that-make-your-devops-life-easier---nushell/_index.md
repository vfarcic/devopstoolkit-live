
+++
title = 'How to Create Custom CLIs That Make Your DevOps Life Easier - Nushell'
date = 2025-01-13T14:00:00+00:00
draft = false
+++

When building user interfaces for developer platforms or, frankly, for anything else, we tend to **focus mostly on Web UIs**. That's why there is a surge in popularity of portals like Backstage, Port, and many others. Still, user interfaces can have many other forms and, more often than not, a single one is not enough. The truth is that some forms are more appropriate for some tasks while others for other tasks.

<!--more-->

{{< youtube TgQZz2kGysk >}}

## Intro

For example, observability, in any form, tends to work the best with Web UIs or desktop applications. We need that "rich" experience to visualize charts and graphs. While that can be done from, let's say, a terminal, we often need more. On the other hand, while some prefer to do operations also from a Web UI, some, me included, might prefer CLIs. Similarly, if we want to automate something, there isn't much choice beyond writing scripts and executing them from workflows like GitHub Actions, Jenkins, Argo Workflows, or whatever else we might be using.

Then there are IDEs like VSCode and JetBrains which are yet another type of a user interface that tends to be better suited for certain type of tasks like, for example, coding than Web UIs or a terminal.

What I'm trying to say is that there are many types of user interfaces and we **should not be focusing only on Web UIs**.

In that spirit, today we'll explore CLIs. To be more specific, we'll explore how we can create our own CLIs that will do exactly what we need them to do. Today we'll build a "proper" *platform* CLI.

We can see a glimpse of what it can do through help.

```sh
platform --help
```

The output is as follows.

```
[32mUsage[0m:
  > platform 

[32mSubcommands[0m:
  [36mplatform apply argocd [32m(custom)[0m - 
  [36mplatform apply crossplane [32m(custom)[0m - 
  [36mplatform apply ingress [32m(custom)[0m - 
  [36mplatform apply kyverno [32m(custom)[0m - 
  [36mplatform apply port [32m(custom)[0m - 
  [36mplatform build image [32m(custom)[0m - Builds a container image
  [36mplatform create kubernetes [32m(custom)[0m - 
  [36mplatform delete crossplane [32m(custom)[0m - 
  [36mplatform delete port [32m(custom)[0m - 
  [36mplatform destroy all [32m(custom)[0m - Destroys the complete demo
  [36mplatform destroy clis [32m(custom)[0m - Destroys the CLIs demo
  [36mplatform destroy kubernetes [32m(custom)[0m - 
  [36mplatform get github [32m(custom)[0m - 
  [36mplatform get hyperscaler [32m(custom)[0m - 
  [36mplatform get ingress [32m(custom)[0m - 
  [36mplatform run ci [32m(custom)[0m - Runs all CI tasks
  [36mplatform run unit-tests [32m(custom)[0m - Executes tests
  [36mplatform setup all [32m(custom)[0m - Sets up the complete demo
  [36mplatform setup clis [32m(custom)[0m - Sets up the CLIs demo
  [36mplatform update gitops [32m(custom)[0m - Executes tests

[32mFlags[0m:
  [36m-h[0m[39m,[0m [36m--help[0m: Display the help message for this command

[32mInput/output types[0m:
  ╭───┬───────┬────────╮
  │ # │ input │ output │
  ├───┼───────┼────────┤
  │ 0 │ any   │ any    │
  ╰───┴───────┴────────╯
```

It will be a CLI that will enable us to perform any of the tasks specified through that auto-generated help.

More importantly, we'll make it in a very easy way.

It will be a "proper" CLI that anyone can create easily.

## Setup

> Watch the [GitHub CLI (gh) - How to manage repositories more efficiently](https://youtu.be/BII6ZY2Rnlc) video if you are not familiar with GitHub CLI.

```sh
git clone https://github.com/vfarcic/idp-full-demo

cd idp-full-demo

git fetch

git checkout clis
```

> Make sure that Docker is up-and-running. We'll use it to create a Kubernetes KinD cluster.

> Watch [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell
```

> Watch [The Future of Shells with Nushell! Shell + Data + Programming Language](https://youtu.be/zoX_S6d-XU4) if you are not familiar with Nushell. Alternatively, you can inspect the `setup/kubernetes.nu` script and transform the instructions in it to Bash or ZShell if you prefer not to use that Nushell script.

```sh
chmod +x platform

platform setup clis

source .env
```

## Why Build Our Own CLIs?

Let's start with **why?** Why we might want to build our own CLIs?

After all, we can use existing CLIs like, for example, `curl` that can sent HTTP requests to any API.

```sh
curl "http://silly-demo.127.0.0.1.nip.io"
```

The output is as follows.

```
This is a silly demo
```

The problem with `curl` and other similar CLIs is that they are **low level**. We could, for example, use it to communicate with Kubernetes API but that would be painful, complicated, and not very user-friendly. That's why we have higher-level CLIs like, for example, `kubectl`.

> I'm using APIs as examples. CLIs come in many flavors and are certainly not limited only to APIs. I'm using them only as examples.

As an example, we could use it to retrieve all resources based on definitions that are baked into Kubernetes.

```sh
kubectl --namespace a-team get all
```

The output is as follows.

```
NAME                              READY   STATUS    RESTARTS   AGE
pod/silly-demo-5ff4469d75-tgn5z   1/1     Running   0          2m6s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/silly-demo   ClusterIP   10.96.114.132   <none>        8080/TCP   2m6s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/silly-demo   1/1     1            1           2m6s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/silly-demo-5ff4469d75   1         1         1       2m6s
```

`kubectl` is still very generic and we can use it to work even with custom resources even though it has no idea what they are.

```sh
kubectl --namespace a-team get appclaims
```

The output is as follows.

```
NAME         HOST   SYNCED   READY   CONNECTION-SECRET   AGE
silly-demo          True     True                        2m26s
```

`kubectl` is great, in part because it's dynamic.

Yet, it can also be complicated for the same reason. For example, if we would like retrieve all the Crossplane rsources related to the claim that I applied earlier, we would need to execute `kubectl get` followed with the list of resources types that might be a part of it, and then making the selection (`--selector`) that filters it by the `claim-name` label set to `silly-demo`.

```sh
kubectl get \
    managed,providerconfigs.kubernetes.crossplane.io,providerconfigs.helm.crossplane.io \
    --selector crossplane.io/claim-name=silly-demo
```

The output is as follows.

```
NAME                                                    KIND         PROVIDERCONFIG   SYNCED   READY   AGE
object.kubernetes.crossplane.io/silly-demo-deployment   Deployment   silly-demo-app   True     True    6m53s
object.kubernetes.crossplane.io/silly-demo-ingress      Ingress      silly-demo-app   True     True    6m52s
object.kubernetes.crossplane.io/silly-demo-service      Service      silly-demo-app   True     True    6m52s

NAME                                                     AGE
providerconfig.kubernetes.crossplane.io/silly-demo-app   6m52s
```

That was painful, in big part because `kubectl` is dynamic, but also generic. It's supposed to work with anything available in Kubernetes and, as is often the case, that makes it not excell at anything specific. As a result, we might want to use a more specific CLI like, in this case, would be `crossplane` which, among other things, allows us to `trace` all the resources related to a claim.

```sh
crossplane beta trace --namespace a-team appclaim silly-demo
```

The output is as follows.

```
NAME                                  SYNCED   READY   STATUS
AppClaim/silly-demo (a-team)          True     True    Available
└─ App/silly-demo-bxpsz               True     True    Available
   ├─ Object/silly-demo-deployment    True     True    Available
   ├─ Object/silly-demo-ingress       True     True    Available
   ├─ Object/silly-demo-service       True     True    Available
   └─ ProviderConfig/silly-demo-app   -        -
```

All the examples we saw "suffer" from the same issue. They are generic. *curl* was designed to work with any API but, arguably, it is the worst for any specific API. *kubectl* is more specific since it works only with the Kubernetes API, yet it is also the most generic one we can use if the scope is limited to Kubernetes. Finally, we saw the *crossplane* CLI which is more specific. It is arguably the best one to work with Crossplane resources in Kubernetes clusters, yet, at the same time, the most limited since it cannot do anything else but work with Crossplane resources.

Why am I explaining all that?

Well... I'm trying to make a point that we need both generic but also specialized CLIs that are focused on a specific task or a specific platform. Even though we could operate AWS, Azure, or Google Cloud with `curl`, we are much better off using `aws`, `az`, or `gcloud` CLIs. Even though we could use `curl` with Kubernetes, we are better off with `kubectl`. Within the Kubernetes ecosystem, if, for example, we would like to work with Istio, we can do it with `kubectl`, but we might be better off with `istioctl`. More **specific CLIs tend to be better at more specific tasks**, but they do not exclude the usage of more generic CLIs. We need both.

Here comes the question though.

If we are building an Internal Developer Platform or an IDP, is it enough to operate it using only more generic CLIs like `kubectl`, `curl`, and others, or should we have a CLI that is specifically designed for that platform?

I think that the answer is yes, to both. Some of us, if not all of us, should continue using more generic CLIs, but we should provide a **platform-specific CLI** as well.

Here comes the problem though.

If we want to have a platform-specific CLI, and that platform was built by us, that CLI needs to be built by us as well. It does not have to be the only way to interact with the platform, but it should certainly be one of the available interfaces.

Here comes another, potentially more important question.

How do we build such a CLI?

## How to Build a Platform-Specific CLI?

When building CLIs or any other type of executables that should, essentially, run tasks, people tend to choose either Bash or a programming language like Go or Python. Both of those choices come with their own set of problems.

Scripting languages like Bash are great for internal-use CLIs, mainly because they are a very easy way to wrap other CLIs. More often than not, when we are building a CLI that will be used internally, much of the functionality can be accomplished by wrapping other CLIs. It might need to be sending requests with *curl*, or it might be doing some "funky" stuff with *kubectl*, or it might be modifying YAML with *yq*. There is an infinite number of things that we might need that could be accomplished by wrapping other CLIs. 

There are a couple of problems with Bash and other scripting languages though.

To begin with, they are not "real" programming languages. Adding logic can be cumbersome. There is no compilation so its hard to find out issues until we hit them at runtime. Everything is treated as text so its hard to pass data from one instruction and process it in another. Maintaining anything but simple scripts can be a burden.

The alternative is to use a "real" language like Go, or Python, or whatever one might be comfortable with.

That is certainly a great choice when working on an external CLI since we can, in some, but not all case, compile them into a single executable self-sufficient binary making it easy to distribute and install. We get code-completion and other "nice" things available in IDEs.

However, as I already mentioned, internal CLIs tend to rely heavily on other CLIs which, essentially, means that a significant part of our CLI is about execution of other binaries. That tends to be cumbersome in "real" languages. They were not designed to do what scripting languages do. Also, they are much harder to iterate fast. Typically, a single line of a Bash script often results in multiple lines of code in other languages.

So, neither scripting languages nor "real" programming languages are a good choise. Each has its own pros and cons and I ended up going from one to another frequently. I would often start with a Bash script, get annoyed when it gets a bit larger or more complex, then switch to Go, only to get annoyed again with its verbosity and annoyances that make it relatively complicated to do things that are simpler in Bash. As a result, I've been switching back and forth, until I discovered... Nushell.

Nushell gives me the best of both worlds. It is a scripting language so it's just as easy to execute other binaries as when working with, let's say, Bash. On the other hand, it is a **compiled language** so I get to detect many of the issues before scripts are executed. Finally, output of its instructions is **structured data** so processing outputs is very easy and intuitive.

As a result of all that, and quite a few other reasons, Nushell became my go-to language when building internal CLIs. I still think that it is not necessarily a good choice when working with those that are distributed to external users. I would not rewrite *kubectl*, *crossplane*, or any other similar CLI to Nushell but, for internal usage, it is, in my opinion, the best choice, especially for those organization who adopted Nix packages.

Let me show you an example I wrote for a group of videos I'm working on.

*Please note that this post focuses on reasons why Nushell might be a good choice for building internal CLIs. If you're new to Nushell, you might want to watch [The Future of Shells with Nushell! Shell + Data + Programming Language](https://youtu.be/zoX_S6d-XU4) first.

## Requirements for Internal CLIs

There are a few things I feel are important when building internal CLIs.

It needs to be a "proper" CLI that allows us to execute different commands with mandatory or optional **parameters**. I need it to be able to show **help*** that explains how to use any of the commands it provides. It need to have the option to load **shared code** in case there are reusable pieces across multiple CLIs or as a way to organize the code. It needs to be able to **execute other binaries**, other CLIs easily without much, if any, boiler-plate code. It needs to be able to produce outputs from other binaries or libraries as **data** so that it can be processed easily. Finally, it should be a **single executable** so that the distribution is easy.

Let's see whether Nushell can give us all those features and, if it can, how do we get them. We'll use a CLI I built for a series of videos.

## Help with Nushell

Here's the output of my `platform` CLI with the `--help` argument.

```sh
platform --help
```

The output is as follows.

```
[32mUsage[0m:
  > platform 

[32mSubcommands[0m:
  [36mplatform apply argocd [32m(custom)[0m - 
  [36mplatform apply crossplane [32m(custom)[0m - 
  [36mplatform apply ingress [32m(custom)[0m - 
  [36mplatform apply kyverno [32m(custom)[0m - 
  [36mplatform apply port [32m(custom)[0m - 
  [36mplatform build image [32m(custom)[0m - Builds a container image
  [36mplatform create kubernetes [32m(custom)[0m - 
  [36mplatform delete crossplane [32m(custom)[0m - 
  [36mplatform delete port [32m(custom)[0m - 
  [36mplatform destroy all [32m(custom)[0m - Destroys the complete demo
  [36mplatform destroy clis [32m(custom)[0m - Destroys the CLIs demo
  [36mplatform destroy kubernetes [32m(custom)[0m - 
  [36mplatform get github [32m(custom)[0m - 
  [36mplatform get hyperscaler [32m(custom)[0m - 
  [36mplatform get ingress [32m(custom)[0m - 
  [36mplatform run ci [32m(custom)[0m - Runs all CI tasks
  [36mplatform run unit-tests [32m(custom)[0m - Executes tests
  [36mplatform setup all [32m(custom)[0m - Sets up the complete demo
  [36mplatform setup clis [32m(custom)[0m - Sets up the CLIs demo
  [36mplatform update gitops [32m(custom)[0m - Executes tests

[32mFlags[0m:
  [36m-h[0m[39m,[0m [36m--help[0m: Display the help message for this command

[32mInput/output types[0m:
  ╭───┬───────┬────────╮
  │ # │ input │ output │
  ├───┼───────┼────────┤
  │ 0 │ any   │ any    │
  ╰───┴───────┴────────╯
```

That looks like a "proper" CLI. Whomever is unsure what is available can simply execute *--help* and get the list of the commands with the description what each of them does. I even removed the *.nu* extension since it is irrelevant for the users (developers) in which language its written.

If we would like to get more information about one of those commands we can execute `platform`, type the command we're interested in, like `build image`, and finish with `--help`.

```sh
platform build image --help
```

The output is as follows.

```
Builds a container image

[32mUsage[0m:
  > platform build image {flags} <tag> 

[32mFlags[0m:
  [36m--registry[0m <[94mstring[0m>: Image registry (default: [32m'ghcr.io/vfarcic'[0m)
  [36m--image[0m <[94mstring[0m>: Image name (default: [32m'idp-full-demo'[0m)
  [36m--push[0m <[94mbool[0m>: Whether to push the image to the registry (default: [96mtrue[0m)
  [36m-h[0m[39m,[0m [36m--help[0m: Display the help message for this command

[32mParameters[0m:
  [36mtag[0m <[94mstring[0m>: The tag of the image (e.g., 0.0.1)

[32mInput/output types[0m:
  ╭───┬───────┬────────╮
  │ # │ input │ output │
  ├───┼───────┼────────┤
  │ 0 │ any   │ any    │
  ╰───┴───────┴────────╯
```

That's everything one might need to know how to build and push images. In this specific case, the `tag` is mandatory, and there are optional `--registry`, `--image`, and `--push` flags, each with the description and, if available, default values.

Here's how I added help to all the commands in that CLI.

```sh
cat platform
```

The output is as follows (truncated for brevity).

```nu
...
# Builds a container image
def "main build image" [
    tag: string                    # The tag of the image (e.g., 0.0.1)
    --registry = "ghcr.io/vfarcic" # Image registry
    --image = "idp-full-demo"      # Image name
    --push = true                  # Whether to push the image to the registry
] {

    docker image build --tag $"($registry)/($image):latest" .

    docker image tag $"($registry)/($image):latest" $"($registry)/($image):($tag)"

    if $push {

        docker image push $"($registry)/($image):latest"

        docker image push $"($registry)/($image):($tag)"

    }

}

# Executes tests
def "main run unit-tests" [] {

    print "Faking execution of unit-tests..."

}
...
```

To begin with, all the definitions (`def`) that start with `main` are automatically converted into commands and, more importantly for the current discussion, automatically output when we add the *--help* argument.

The comment on top of each, as, for example, the one above `run unit-tests`, automatically becomes the description (`Executes tests`) of that command.

Finally, each parameter and flag can have a description which is a comment on the right side of it (e.g. `The tag of the image (e.g., 0.0.1)`).

That's it. It's as easy as it can get and all we have to do is adopt a few simple conventions.

Now that we mentioned parameters and flags, let's see them in action and see how they are defined.

## Parameters and Arguments with Nushell

As we saw with the help output, commands can optionally have arguments or, as Nushell calls them, parameters and flags.

For example, if we would like to build *latest* image that is also tagged as *1.2.3*, but we would not like to push them to the registry, we can execute `platform build image` with the tag `1.2.3`, and set the `--push` flag to `false`.

```sh
platform build image 1.2.3 --push false
```

The output is as follows (truncated for brevity).

```
[+] Building 16.8s (15/15) FINISHED                                                                                     docker:desktop-linux
 => [internal] load build definition from Dockerfile                                                                                    0.0s
 => => transferring dockerfile: 431B                                                                                                    0.0s
 => [internal] load metadata for docker.io/library/golang:1.23.3-alpine                                                                 1.6s
 => [internal] load .dockerignore                                                                                                       0.0s
 => => transferring context: 2B                                                                                                         0.0s
 => [build 1/9] FROM docker.io/library/golang:1.23.3-alpine@sha256:c694a4d291a13a9f9d94933395673494fc2cc9d4777b85df3a7e70b3492d3574     6.0s
 => => resolve docker.io/library/golang:1.23.3-alpine@sha256:c694a4d291a13a9f9d94933395673494fc2cc9d4777b85df3a7e70b3492d3574           0.0s
 => => sha256:4152418b1c7ced56e47197c3aaf822a218d3f5be12de867e912c3f2fc8e5a0b5 1.92kB / 1.92kB                                          0.0s
 ...
 => [build 9/9] RUN chmod +x silly-demo                                                                                                 0.1s 
 => [stage-1 1/1] COPY --from=build /src/silly-demo /usr/local/bin/silly-demo                                                           0.0s 
 => exporting to image                                                                                                                  0.0s 
 => => exporting layers                                                                                                                 0.0s 
 => => writing image sha256:74b677c55425cdf14a6da1c5a0fa8d6a1b0cc69b2c518c6904b5976319bc3f76                                            0.0s 
 => => naming to ghcr.io/vfarcic/idp-full-demo:latest                                                                                   0.0s 

View build details: docker-desktop://dashboard/build/desktop-linux/desktop-linux/rtye97612s78ylbk2qe40rpxe

What's next:
    View a summary of image vulnerabilities and recommendations → docker scout quickview 
```

That command is a replacement for a few *docker* commands. It built the image with the specified tag, then it created the *latest* tag of that image, and it skipped pushing both to the registry because we told it to do that.

As we saw through the *help*, we could have specified the *registry* and the *image* name but we didn't so the command used the default values for those.

Here's how that was done.

```sh
cat platform
```

The output is as follows (truncated for brevity).

```nu
...
# Builds a container image
def "main build image" [
    tag: string                    # The tag of the image (e.g., 0.0.1)
    --registry = "ghcr.io/vfarcic" # Image registry
    --image = "idp-full-demo"      # Image name
    --push = true                  # Whether to push the image to the registry
] {

    docker image build --tag $"($registry)/($image):latest" .

    docker image tag $"($registry)/($image):latest" $"($registry)/($image):($tag)"

    if $push {

        docker image push $"($registry)/($image):latest"

        docker image push $"($registry)/($image):($tag)"

    }

}

# Executes tests
def "main run unit-tests" [] {

    print "Faking execution of unit-tests..."

}
...
```

Each `main` command can be without any parameters and flags by specifying an empty array (`[]`), as is the case of `run unit-tests`.

The `build image` command, on the other hand, has both a parameter and a few flags.

Parameters, like `tag`, are not named. When we executed *platform build image*, we added *1.2.3* without specifying what it is. Parameters must be set in a specific order simply because we set only their values

Arguments, on the other hand, are named. If, like we saw when we executed the command, we want to tell it not to push images, we have to set `--push` before the value, in this case *true* or *false*.

Both parameters and flags can have default values like, for example, is the case of `--registry` which, if not specified, will be set to `ghcr.io/vfarcic`. Default values are specified with the `=` sign followed by some value. In those cases, we don't have to specify the type. Nushell will automatically deduce, depending on the value itself, whether it is a string, boolean, integer, or whichever other type is available.

Bear in mind that, in the case of flags, we are using the variables without *--*, as, for example, is the case with `$registry`. *--* is there only to tell Nushell that we want to have a named flag.

Let's move to the next requirement and see how we can organize the code and have reusable pieces.

## Source Code with Nushell

Unless we are working with a relatively small amount of code, we often want to separate it into logical units. That's what I did with the files in the `scripts` directory.

```sh
ls scripts/
```

The output is as follows.

```
-- argocd.nu
-- crossplane.nu
-- get-hyperscaler.nu
-- github.nu
-- ingress.nu
-- kubernetes.nu
-- kyverno.nu
-- port.nu
```

By having the code in separate files, we can accomplish two distinct objectives.

First, it is often easier to keep the code in different files. In this case, there is `argocd.nu` with the code related to Argo CD, `crossplane.nu` for operations related to Crossplane, `kubernetes.nu` that creates and destroys different types of Kubernetes clusters, and so on and so forth.

That way, the main file, in this case *platform* is relatively short or, as is my case, only contains the parts that are unique to that CLI. That brings me to the second, even though probably unique to me, possibility.

I have a library of functionalities in a separate repository. That way, when I'm building a CLI that is used in a specific project, I can simply copy those that I need, and leave out those that I don't, thus keeping it minimal and not encumbered with commands that project does not need.

Let's, for example, take a look at `kubernetes.nu`.

```sh
cat scripts/kubernetes.nu
```

The output is as follows (truncated for brevity).

```nu
#!/usr/bin/env nu

def --env "main create kubernetes" [provider: string, name = "dot", min_nodes = 2, max_nodes = 4, auth = true] {

    $env.KUBECONFIG = $"($env.PWD)/kubeconfig-($name).yaml"
    $"export KUBECONFIG=($env.KUBECONFIG)\n" | save --append .env

    if $provider == "google" {

        if $auth {
            gcloud auth login
        }

        ...

}

def "main destroy kubernetes" [provider: string, name = "dot", delete_project = true] {

    if $provider == "google" {

        rm --force kubeconfig.yaml

        (
            gcloud container clusters delete $name
                --project $env.PROJECT_ID --zone us-east1-b --quiet
        )

        if $delete_project {
            gcloud projects delete $env.PROJECT_ID --quiet
        }
    
    ...
```

There is no functional difference between the code in that file when compared with the *platform* we executed and saw earlier.

It contains two definitions, `main create kubernetes` and... `main destroy kubernetes`, each of them having some parameters. The only thing that makes them "special" is that I was foo lazy to write comments so you won't see a fancy output with *help*. Still, even in that case, help with the list of parameters is available automatically.

Now, let's take another look at `platform`. 

```sh
cat platform
```

The output is as follows (truncated for brevity).

```nu
#!/usr/bin/env nu

source scripts/kubernetes.nu
source scripts/crossplane.nu
...
# Sets up the complete demo
def "main setup all" [] {
    
    rm --force .env

    let hyperscaler = main get hyperscaler

    let github_data = main get github

    main create kubernetes kind
    ...
```

Code is imported by sourcing it (`source`) as, for example, in the case of `kubernetes.nu`. From there on, the commands, those prefixed with *main*, are automatically included by executing that CLI. We can invoke it from some other definition like, for example, `main create kubernetes kind` executed through `setup all`.

At the same time, those sourced definitions that are prefixed with *main*, are automatically available as separate commands in the CLI. We can see that by, for example, executing `platform create kubernetes --help`.

```sh
platform create kubernetes --help
```

The output is as follows.

```
[32mUsage[0m:
  > platform create kubernetes <provider> (name) (min_nodes) (max_nodes) (auth) 

[32mFlags[0m:
  [36m-h[0m[39m,[0m [36m--help[0m: Display the help message for this command

[32mParameters[0m:
  [36mprovider[0m <[94mstring[0m>
  [36mname[0m <[94mstring[0m>:  (optional, default: [32m'dot'[0m)
  [36mmin_nodes[0m <[94mint[0m>:  (optional, default: [1;35m2[0m)
  [36mmax_nodes[0m <[94mint[0m>:  (optional, default: [1;35m4[0m)
  [36mauth[0m <[94mbool[0m>:  (optional, default: [96mtrue[0m)

[32mInput/output types[0m:
  ╭───┬───────┬────────╮
  │ # │ input │ output │
  ├───┼───────┼────────┤
  │ 0 │ any   │ any    │
  ╰───┴───────┴────────╯
```

There we go. It works. The code is included. Any definition can be executed from inside the main one, the one that sourced it. On top of that, those prefixed with *main* are automatically available as separate commands. Awesome!

Next, let's talk about execution of binaries.

## Binaries Execution with Nushell

Some more generic languages are, frankly, painful to work with when trying to execute Shell commands. With Go, for example, we have to use *exec.Command*, pass it the command and all the arguments as an array, then capture *stdout* and *stderr* as variables of the output, do some error verification, and whatever else needs to be done. Shell scripts do not need any of that. We just specify what we want to execute and that's it, most of the time.

Nushell is a Shell programming language. As such, just as with Bash, there is nothing special we have to do to execute other Shell commands. We can simply execute any command that is available in the path.

Here's an example.

```sh
cat platform
```

The output is as follows (truncated for brevity).

```nu
...
# Builds a container image
def "main build image" [
    tag: string                    # The tag of the image (e.g., 0.0.1)
    --registry = "ghcr.io/vfarcic" # Image registry
    --image = "idp-full-demo"      # Image name
    --push = true                  # Whether to push the image to the registry
] {

    docker image build --tag $"($registry)/($image):latest" .

    docker image tag $"($registry)/($image):latest" $"($registry)/($image):($tag)"

    if $push {

        docker image push $"($registry)/($image):latest"

        docker image push $"($registry)/($image):($tag)"

    }

}
...
```

Over there, we simply execute `docker image build`, followed with `docker image tag`, and, finally, `docker image push` twice. It's the same as with Bash or any other Shell scripting language.

It just works, and we can move to the talk about data.

## Data Management with Nushell

Most of the time with internal CLIs, we execute some commands, parse the output, and use it as input of other commands. Nushell, just as Bash, does that by piping data from one to another command. What makes is special is that outputs are not text blobs but structured data. As such, we know what that data is and we can do some "funky" stuff to filter or manipule it.

Here's an example.

```sh
cat platform
```

The output is as follows (truncated for brevity).

```nu
...
# Executes tests
def "main update gitops" [
    tag: string                    # The tag of the image (e.g., 0.0.1)
    --registry = "ghcr.io/vfarcic" # Image registry
    --image = "idp-full-demo"      # Image name
] {

    open apps/silly-demo.yaml |
        | upsert spec.parameters.image $"($registry)/($image):($tag)"
        | save apps/silly-demo.yaml --force

}
...
```

Inside the `update gitops` definition, we are using `open` to read contents of a YAML file. Since it knows that it is YAML, it knows how to transform the text in that file into its internal data format. That structured data is then passed to `upsert` instruction that changes the value of `spec.parameters.image` entry in that data to the combination of `registry`, `image`, and `tag` variables. From there on, that modified data is sent, still as data, not as text, to the `save` command that transforms it to YAML and saves it back into that same file.

That block of code is similar to what we would do with, let's say, *yq*, except that, we do not need any external binaries. Instead, all is done using Nushell's internal instructions. Every Nushell instruction or command outputs data and each is capable of taking data as input. It's not text, it's data, and that makes it very special. Even if we try to execute Shell commands that are not available in Nushell, we can use it to transform almost any text blob into data. Here's an example.

```sh
cat scripts/ingress.nu
```

```nu
...
def "main get ingress" [provider: string, type = "traefik", env_prefix = ""] {

    sleep 30sec
    
    mut ingress_ip = ""
  
    if $provider == "aws" {

        let ingress_hostname = (
            kubectl --namespace traefik
                get service traefik --output yaml
                | from yaml
                | get status.loadBalancer.ingress.0.hostname
        )
        ...
```

The `get ingress` definition, among other things, executes `kubectl` to retrieve YAML representation of a `service`. The output of that command is not YAML or any other data structure. It looks like YAML, but is actually a text blob. If that text would be in a file, Nushell would know, from the file extension, how to transform it from YAML to its own internal data format. But, in this case, it is a text output from an external command. So, we're "helping" Nushell know what the text format is by specifying that it comes `from yaml`. From there on, it is pure data passed to the `get` instruction that retrieves `status.loadBalancer.ingress.0.hostname` value and stores it into the `ingress_hostname` variable.

There's only one requirement left. Can Nushell scripts run as single self-sufficient executables that can be distributed as a binary?

## Creating Single Self-Sufficient Executable with Nushell

If, for example, we would be building a CLI with Go, we would probably produce executable binaries for each operating system. From there on, users would not need to have Go compiler or any other prerequisite to run it. That's how, for example, *kubectl* works. We just download it and run it. There are no dependencies of any kind.

Can we do that with Nushell?

The short answer is "no". A slightly longer answer is "no, and even if we could, we shouldn't".

One big advantage of scripting languages like Bash is that they are both executable but also in clear text. Anyone can modify it and run it right away without compilation. Nushell is similar, but, unlike Bash that runs anywhere, we need to have Nushell installed. From that perspective, it is closer to, let's say Python, except that we can run Python code as-is, as long as we have Python installed, or we can package it into a self-executable binary that does not require separate installation of Python runtime. With Nushell, the latter is not, as far as I know, possible. Still, as I mentioned, I don't think we should be compiling internal tooling and distributing executables since that would complicate fast and easy modifications. If, for example, we have a CLI related to a project, or, let's say, a platform, it is very convenient to be able to quickly modify it to either fix a bug, or add a new feature, or whichever other reason there might be.

So, it does not matter that it cannot be compiled to a binary, except that everyone using our CLI made with Nushell would need to have Nushell runtime installed. That's a potential problem for some. In my case, that is not really an issue since I think that everyone should be using Nix packages, in my case through Devbox. In such a case, Nushell is yet another package that is installed when working on a project or with an internal platform. We need packages anyway since those internal CLIs often depend on other third-party CLIs like *kubectl*, *helm*, or whatever you're using and wrapping into your internal CLI.

That's it.
Try it out.
I'm convinced you'll find Nushell very useful if you're trying to write internal CLIs or for any other type of scripts.

Please let me know in the comments what you think of Nushell and what is your favorite language to write internal CLIs.

## Destroy

```sh
platform destroy clis

exit

git checkout main
```


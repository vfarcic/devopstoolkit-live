
+++
title = 'Say Goodbye to Tedious Docker Commands: Embrace Docker to Bake Images'
date = 2025-02-17T15:00:00+00:00
draft = false
+++

Building and pushing container image with Docker is **easy**. Right? We define a Dockerfile and we execute a command like `docker image build ...`. Docker file is easy to define and the rest is just a CLI command. How hard can it be?

Well... It can be hard or, at least, **tedious**.

Imagine that we have to build images for **multiple platforms**, that each of those images should be released both as a **specific version** but also as **latest**. Then add to that the situation that we need to build **more than one image**, let's say a backend and a frontend.

How many commands do we need to execute and how many arguments should each of those commands have? Can we remember all those arguments and are we willing to execute a bunch of commands?

That simple example already shows that building and pushing container images can be hard and tedious. The good news is that there is a better way. There is a declarative way to do all that.

<!--more-->

{{< youtube 3Fc7YuTWptw >}}

## Setup

```sh
git clone https://github.com/vfarcic/silly-demo

cd silly-demo

git pull

git fetch

git switch docker-bake
```

> Watch [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell
```

> Watch [The Future of Shells with Nushell! Shell + Data + Programming Language](https://youtu.be/zoX_S6d-XU4) if you are not familiar with Nushell. Alternatively, you can inspect the `dot.nu` script and transform the instructions in it to Bash or ZShell if you prefer not to use that Nushell script.

```sh
chmod +x dot.nu

./dot.nu setup bake

source .env
```

## Building and Pushing Docker Images Without Bake

Let's try to reproduce the scenario I mentioned earlier. We are going to build and push images for both the backend and the frontend application. We'll make sure that both specific and the latest tags are used. Finally, we'll do all that both for AMD and ARM architectures.

Here's how we would do all that using the commands most of us are very familiar with.

We'll `build` a `tag` for whatever is the name of the image stored in the environment variable `IMAGE`. It will be version `v0.0.1` and it will be built both for `amd64` and `arm64` platforms. We want to `push` that image to the registry and we'll set the context to be the current directory (`.`). There is already *Dockerfile* in there and, since Docker assumes it by default, there is no need to specify `--file`.

```sh
docker image build --tag ${IMAGE}:v0.0.1 \
    --platform linux/amd64,linux/arm64 --push .
```

A few moments later, the image was built and pushed to the registry.

That was easy. Right?

It would be even easier if we wouldn't need to specify the platforms and the context directory since they are always the same for those images. Executing that command runs a risk of forgetting or misstyping some of those arguments. Still, it wasn't that bad. Right?

Now, let's make sure that same image is available as the *latest* version as well. Even though I don't think we should be encouraging people to use the latest, that's what many are doing so we'll do it as well.

To publish that image as latest, we can repeat the same command as before but, this time, with `latest` as the `tag` version.

```sh
docker image build --tag ${IMAGE}:latest \
    --platform linux/amd64,linux/arm64 --push .
```

That was a bit more annoying, especially since we are likely to do that every single time. Normally, I would put both into a script but, as we'll see soon, there is a better way.

We're not done yet.

Next, let's say that we would like to do the same for the frontend application with code that happens to reside in the same Git repo.

We'll repeat the first command but, this time, the tag should be different. We'll add `-frontend` to it. Also, the context is different, so we need to change it to `./frontend`.

```sh
docker image build --tag ${IMAGE}-frontend:v0.0.1 \
    --platform linux/amd64,linux/arm64 --push ./frontend
```

After a while, we got our third image, and my annoyance of having to run similar, yet sufficiently different commands is converting itself from "I don't mind" to "this is silly".

There's more though.

Now we need the latest tag of frontend image.

So, let's re-run the previous command but, before we do that, change `v0.0.1` to `latest`.

```sh
docker image build --tag ${IMAGE}-frontend:latest \
    --platform linux/amd64,linux/arm64 --push ./frontend
```

Let's confirm that all the images were indeed built.

```sh
docker image ls
```

```
REPOSITORY                            TAG       IMAGE ID       CREATED        SIZE
ghcr.io/vfarcic/silly-demo            latest    2c17e2d577c6   45 years ago   74.4MB
ghcr.io/vfarcic/silly-demo            v0.0.1    0925e74ebccd   45 years ago   74.4MB
ghcr.io/vfarcic/silly-demo-frontend   v0.0.1    f728edc98c37   45 years ago   1.41GB
ghcr.io/vfarcic/silly-demo-frontend   latest    9309fecc251a   45 years ago   1.41GB
```

They are all there. There are two tags of `silly-demo` and two tags of `silly-demo-frontend`. Each of those were built and pushed 

That's it. We did it, and I'm annoyed by having to run similar commands over and over again while, at the same time, risking making a typo or forgetting some of the arguments.

Here's what I want.

I want to specify all those variations somewhere and just tell Docker: "**Build whatever is defined there. Don't ask me anything but the specific version I'm building since that is the only thing that changes over time. Do it all at once, unless I tell you otherwise.**"

Is that too much to ask?

I can easily make my own script that does all that but I would prefer if there is a standard and out-of-the-box way to do something that is fairly common. I'm surely not the only one.

Luckily, Docker recently released GA version of the feature that does just that. That feature is called *Docker Bake*.

## Building and Pushing Docker Images With Bake

Here's how we would accomplish the same outcome with Docker Bake.

We'll `export` environment variable `TAG` to version we want to build and push,...

```sh
export TAG=v0.0.2
```

...and we'll execute `docker buildx bake` with the `--push` argument that should be self-explanatory.

```sh
docker buildx bake --push
```

The output is as follows (truncated for brevity).

```
[+] Building 15.2s (52/52) FINISHED                                                                                              docker:desktop-linux
 => [internal] load local bake definitions                                                                                                       0.0s
 => => reading docker-bake.hcl 628B / 628B                                                                                                       0.0s
 => [frontend internal] load build definition from Dockerfile                                                                                    0.0s
 => => transferring dockerfile: 321B                                                                                                             0.0s
 => [backend internal] load build definition from Dockerfile                                                                                     0.0s
 => => transferring dockerfile: 394B                                                                                                             0.0s
 => [frontend linux/arm64 internal] load metadata for docker.io/library/node:22-alpine                                                           0.5s
 => [frontend linux/amd64 internal] load metadata for docker.io/library/node:22-alpine                                                           0.5s
 => [backend linux/arm64 internal] load metadata for docker.io/library/golang:1.23.6-alpine                                                      1.0s
 => [backend linux/amd64 internal] load metadata for docker.io/library/golang:1.23.6-alpine                                                      0.9s
 ...
 => CACHED [frontend linux/arm64 2/6] WORKDIR /app                                                                                               0.0s
 => CACHED [frontend linux/arm64 3/6] COPY package.json .                                                                                        0.0s
...
 => [frontend] pushing ghcr.io/vfarcic/silly-demo-frontend:v0.0.2 with docker                                                                    2.1s
 ...
 => [backend] pushing ghcr.io/vfarcic/silly-demo:v0.0.2 with docker                                                                              2.1s
...
 => [frontend] pushing ghcr.io/vfarcic/silly-demo-frontend:latest with docker                                                                    2.0s
...
 => [backend] pushing ghcr.io/vfarcic/silly-demo:latest with docker                                                                              2.0s
...
View build details:
  frontend: docker-desktop://dashboard/build/desktop-linux/desktop-linux/7b9ya9a8z5kjfcbxkctcvadil
  backend: docker-desktop://dashboard/build/desktop-linux/desktop-linux/mb5fn006ywhpduwbeqkqs1syb
```

That's it. We built and pushed both backend and frontend images with specific versions as well as latest. We built those for both ARM and AMD platforms. It found correct contexts and Dockerfiles, and whatever else was needed. To make it even sweater, all that was happening in parallel so the whole process was not only easier and less error-prone but also faster than before.

I love easy, especially when easy also means with less chance to produce human-caused issues. I love this.

To be safe, let's confirm that it really did what it's supposed to do by listing all the images.

```sh
docker image ls
```

The output is as follows.

```
REPOSITORY                            TAG       IMAGE ID       CREATED        SIZE
ghcr.io/vfarcic/silly-demo            latest    257a3e47e31e   45 years ago   74.4MB
ghcr.io/vfarcic/silly-demo            v0.0.2    257a3e47e31e   45 years ago   74.4MB
ghcr.io/vfarcic/silly-demo            v0.0.1    0925e74ebccd   45 years ago   74.4MB
ghcr.io/vfarcic/silly-demo-frontend   v0.0.1    f728edc98c37   45 years ago   1.41GB
ghcr.io/vfarcic/silly-demo-frontend   latest    993e69a5397c   45 years ago   1.41GB
ghcr.io/vfarcic/silly-demo-frontend   v0.0.2    993e69a5397c   45 years ago   1.41GB
```

We can see that `silly-demo` and `silly-demo-frontend` were built with both `v0.0.2` and  `latest` tags. They were also pushed to the registry, and all it took is a single command with no arguments except for the environment variable with the version we need.

Now, let's say that we are interested in building only the frontend, without backend instead of building it all.

We can do that by specifying the new `TAG`,...

```sh
export TAG=v0.0.3
```

...and executing the same command as before but, this time, with `frontend` as the target.

```sh
docker buildx bake frontend
```

That's it. Only frontend was built and, since we did not specify the `push` argument, it was not pushed to the registry.

I think that Docker Bake is awesome. It's just what we needed, especially when working with multiple images, platforms, tags, contexts, or anything else that results in more than a simple `docker image build ...` command.

Now, let's see how it works.

## How Docker Bake Works?

All we need to make Docker Bake work is a manifest that contains all the information it needs to do the "magic".

Here's an example.

```sh
docker buildx bake --print | jq .
```

The output is as follows.

```json
{
  "group": {
    "default": {
      "targets": [
        "backend",
        "frontend"
      ]
    }
  },
  "target": {
    "backend": {
      "context": ".",
      "dockerfile": "Dockerfile",
      "args": {
        "SOURCE_DATE_EPOCH": "315532800",
        "VERSION": "v0.0.3"
      },
      "tags": [
        "ghcr.io/vfarcic/silly-demo:v0.0.3",
        "ghcr.io/vfarcic/silly-demo:latest"
      ],
      "platforms": [
        "linux/amd64",
        "linux/arm64"
      ]
    },
    "frontend": {
      "context": "frontend",
      "dockerfile": "Dockerfile",
      "args": {
        "SOURCE_DATE_EPOCH": "315532800",
        "VERSION": "v0.0.3"
      },
      "tags": [
        "ghcr.io/vfarcic/silly-demo-frontend:v0.0.3",
        "ghcr.io/vfarcic/silly-demo-frontend:latest"
      ],
      "platforms": [
        "linux/amd64",
        "linux/arm64"
      ]
    }
  }
}
```

That JSON has two targets, `backend` and `frontend`. Each of them specifies the `context`, `dockerfile`, `args`, `tags`, and `platforms`, which are all, essentially, what we would normally pass to *docker image build* as arguments, except in the case of *tags* which we had to build and push separately.

As a result, we can bake either the `backend` or the `frontend`.

On top of those is the `group` called `default`. As a naming convention, if we do not specify a target, Docker Bake will assume it is the `default` one. But, in this case, we are not talking about a target but a group which allows us to group targets together. That's why when we executed *docker buildx bake* without any arguments, it baked both `backend` and `frontend`.

All in all, if we define JSON like that one, we can bake either *backend* or *frontend* or both.

There are problems with that one though.

Quite a few things are repeated, and I hate repetition. The `dockerfile`, `VERSION` argument, and `platforms` are the same for both targets. Moreover, the tags follow the same pattern that is image with version and latest.

We can improve on that, but before we do, it might be important to understand that the JSON we just saw is the format Docker Bake expects and we have a few options how to generate it. We could certainly write it as-is but, that would be a pain. Instead, we can use HCL, JSON, or YAML to generate it.

HCL seem to be the format Docker Bake put most effort into, so let's use that.

Here's an example.

```sh
cat docker-bake.hcl
```

The output is as follows.

```hcl
variable "IMAGE" {
    default = "ghcr.io/vfarcic/silly-demo"
}
variable "TAG" {
    default = "dev"
}
target "default" {
    name = item.name
    matrix = {
        item = [{
            name = "backend"
            context = "."
            tags = ["${IMAGE}:${TAG}", "${IMAGE}:latest"]
        }, {
            name = "frontend"
            context = "./frontend"
            tags = ["${IMAGE}-frontend:${TAG}", "${IMAGE}-frontend:latest"]
        }]
    }
    tags = item.tags
    dockerfile = "Dockerfile"
    context = item.context
    platforms = ["linux/amd64", "linux/arm64"]
    args = {
        VERSION = TAG
    }
}
```

We have the `target` called `default` which uses a `matrix` to dynamically create the targets we saw in final Json we explored earlier.

That matrix is essentially an array of items with the `name`, `context`, and `tags`.

A separate target will be generated for each item in that matrix and we are using the values of those items to populate the final `name`, `tags`, and `context`. On the other hand, the `dockerfile`, `platforms`, and `args` are the same for all targets.

Moreover, we have variables `IMAGE` and `TAG`. Each of those have default values that can be overwritten by environment variables. That's how we built the specific tag.

We are using those variables to construct the `tags` in the matrix as well as to overwrite the argument `VERSION` inside Dockerfile itself.

Without entering into the discussion whether you should be building images with Docker or something else, I must say that I love it. **It's awesome** and if Docker is your container image builder of choice, you should definitely give it a try.

## Destroy

```sh
exit
```


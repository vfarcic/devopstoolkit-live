
+++
title = 'If Dockerfile and Makefile Had a Baby... Earthly and Earthfile'
date = 2024-07-08T16:00:00+00:00
draft = false
+++

**Makefile** walks into a bar and notices **Dockerfile** sitting alone. She joins him, they talk, they flirt...

> The rest of that story has been censored. You'll have to fill in the gaps using your imagination.

...a baby was born, and that baby was named **Earthfile**.

<!--more-->

{{< youtube YP9N82-5TAg >}}

We all know and love Dockerfile. It's great for building container images and, through Multi-stage builds we can do even more like, for example, run tests before an image was built. On the other hand, Makefile is old and grumpy one that not many are still in love with. Yet, it is, in a way, the de-facto standard for executing tasks. We need both types of tools and I've been advocating for replacing **Makefile** with **Taskfile** or **Justfile** and combining them with Dockerfile. So, we still need both types of tools, no matter whether tasks are run with Makefile, or Justfile, or Taskfile, or anything else.

Eartly recognized the need for both Dockerfile and Makefile and created a new format Earthfile. It combines both the ability to build images and to execute tasks into a single format very similar to Dockerfile. With it, we can do something like the following.

> Do NOT run the command that follows. We'll run it again later after we set up everything we'll need.

```sh
earthly --push \
    --secret cosignpassword=$COSIGN_PASSWORD \
    --secret cosignkey=$COSIGN_PRIVATE_KEY \
    --secret password=IWillNeverTell \
    +all --tag 9.9.9 \
    --registry ttl.sh --image $IMAGE
```

That single command built two different images with four different tags and it pushed those images to a registry. It signed those images and pushed signatures to the registry as well. It also created a Timoni package, pushed it to a registry, and it updated the values file. Finally, it generated Helm package as well, pushed them to the registry, and updated Chart's values file. If I wasn't too lazy, I could have added so much more like, for example, execution of unit and integration tests, or anything else.

The best part is that if you are familiar with **Dockerfile**, you should be able to master Earthfile in no time since it's very similar. On the other hand, if you're still struggling to understand Dockerfile, you should change your profession for something easier like being a doctor or a lawyer.

So, I'm assuming that you did not choose the easier path, so let's dive into Earthly and Earthfile.

## Setup

```sh
git clone https://github.com/vfarcic/silly-demo

cd silly-demo

git pull

git checkout earthly
```

> Watch https://youtu.be/WiFLtcBvGMU if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell

export COSIGN_PASSWORD=IWillNeverTell

cosign generate-key-pair
```

> Press the `enter` key when asked for a password (twice).

```sh
mkdir -p cosign

mv cosign.pub cosign.key cosign/.

export COSIGN_PRIVATE_KEY="$(cat cosign/cosign.key)"

export IMAGE=silly-demo-$(date +%Y%m%d%H%M%S)
```

## Earthly and Earthfile in Action

Let's take a look at an Earthfile.

```sh
cat Earthfile
```

The output is as follows (truncated for brevity).

```
VERSION 0.8
FROM ghcr.io/vfarcic/silly-demo-earthly:0.0.5
ARG --global registry=ghcr.io/vfarcic
ARG --global user=vfarcic
ARG --global image=silly-demo
WORKDIR /go-workdir

binary:
    COPY go.mod go.sum vendor .
    COPY *.go .
    RUN go mod vendor
    RUN GOOS=linux GOARCH=amd64 go build --mod vendor -o silly-demo
    SAVE ARTIFACT silly-demo

image:
    BUILD +binary
    ARG tag='latest'
    ARG taglatest='latest'
    ARG base='scratch'
    FROM $base
    ...
```

The `VERSION` entry specifies which Earthly specification we're using. Then there are targets like `binary` and `image` which are similar to targets in other task executors, including Makefile.

Most of the rest of that file is, more or less, the same as what we would normally do in Dockerfile.

We can, for example, specify `FROM` that, in Dockerfile, defines the base image but, in case of Earthfile it can be overwritten in a target. In this case, I'm using `ghcr.io/vfarcic/silly-demo-earthly:0.0.5` image that contains all the tools I need. You can see it by exploring Dockerfile-earthly in the repo. It's a "normal" container image which, in the context of Earthfile is used for all the tasks. However, since I don't want the images I'm producing to be biggger than necessary, I'm overwriting it in the `image` target with `$base` which is a variable set to be the `scratch` image by default.

Further on we see instructions that are the same or very similar to those in Dockerfile. We're setting a few variables with `ARG` but, in this case, we're setting them as `--global` so that they can be used in any of the targets. Finally, the last instruction in the general section is `WORKDIR` that, as you can guess, specifies the working directory.

Let's move into the first target.

```sh
cat Earthfile
```

The output is as follows (truncated for brevity).

```
...
binary:
    COPY go.mod go.sum vendor .
    COPY *.go .
    RUN go mod vendor
    RUN GOOS=linux GOARCH=amd64 go build --mod vendor -o silly-demo
    SAVE ARTIFACT silly-demo
...
timoni:
    ...
    SAVE ARTIFACT timoni/values.cue.tmp AS LOCAL timoni/values.cue
    ...
```

The `binary` target copies files from the local file system to the container where it will be executed. Since, behind the scenes, Earthly is using Docker, the typical optimization techniques apply. So, the first `COPY` is copying files that are not likely to change often (`go.mod go.sum vendor`) while the second copies all `*.go` files which are likely changing all the time. That way, we can benefit from caching image layers.

Further on, we're executing `RUN` instruction which, just as in Dockerfile, executes commands.

The last instruction in the binary target is `SAVE`. That one is specific to Earthfile and can be used to save a file generated in a container image either to be used in other targets or to be copied back into the local file system. The `SAVE` instruction can be applied to artifacts (`ARTIFACT`) or images. In this example, `silly-demo` binary will be saved to be used in other targets.

If we switch out attention to the `timoni` target, we can see the `SAVE` instruction as well but, in that specific case, it is storing it `AS LOCAL` meaning that it will be copied as `timoni/values.cue` on the local file system.

Let's go back to the binary target and execute it.

Targets that should be executed need to be prefixed with the `+` sign so we can run it by executing `earhly +binary`.

```sh
earthly +binary
```

The output is as follows.

```
 Init 🚀
————————————————————————————————————————————————————————————————————————————————

           buildkitd | Found buildkit daemon as docker container (earthly-buildkitd)

 Build 🔧
————————————————————————————————————————————————————————————————————————————————

              logbus | Setting organization "" and project ""
             +binary | --> FROM +base
g/v/silly-demo-earthly:0.0.5 | --> Load metadata ghcr.io/vfarcic/silly-demo-earthly:0.0.5 linux/arm64
               +base | --> FROM ghcr.io/vfarcic/silly-demo-earthly:0.0.5
               +base | [----------] 100% FROM ghcr.io/vfarcic/silly-demo-earthly:0.0.5
               +base | *cached* --> WORKDIR /go-workdir
             +binary | *cached* --> COPY go.mod go.sum vendor .
             +binary | *cached* --> COPY *.go .
             +binary | *cached* --> RUN go mod vendor
             +binary | *cached* --> RUN GOOS=linux GOARCH=amd64 go build --mod vendor -o silly-demo
              output | --> exporting outputs

 Push Summary ⏫ (disabled)
————————————————————————————————————————————————————————————————————————————————

To enable pushing use earthly --push

 Local Output Summary 🎁
————————————————————————————————————————————————————————————————————————————————



========================== 🌍 Earthly Build  ✅ SUCCESS ==========================

🛰️ Reuse cache between CI runs with Earthly Satellites! 2-20X faster than without cache. Generous free tier https://cloud.earthly.dev
```

You'll notice that, in my case, some of the layers are `cached`. That means that, just as with Docker, Earthly did not have to execute that step but can use cache. If you're following along, you will not see that since it is likely the first time you're building the binary.

Another interesting thing to note is that message saying that `to enable pushing` we should `use earthly --push`. We'll see that option later. For now, what matters is that artifacts and images are not pushed to registries without us explicitly saying so.

All in all, the files were copied, the binary was built, and the artifact can be used in other tasks.

Let's see such a task that can use an artifact created elsewhere.

```sh
cat Earthfile
```

The output is as follows (truncated for brevity).

```
...
image:
    BUILD +binary
    ARG tag='latest'
    ARG taglatest='latest'
    ARG base='scratch'
    FROM $base
    ENV DB_PORT=5432 DB_USERNAME=postgres DB_NAME=silly-demo
    EXPOSE 8080
    CMD ["silly-demo"]
    ENV VERSION=$tag
    COPY +binary/silly-demo /usr/local/bin/silly-demo
    SAVE IMAGE --push \
        $registry/$image:$tag \
        $registry/$image:$taglatest
...
```

On the first look that target looks the same as what we would normally define as Dockerfile. But, on a closer look, there are a few distinct differences.

To begin with, we're calling the `+binary` target that, as we saw earlier, builds the binary. The objective is similar to what we would do with [Docker Multi-Stage Builds](https://youtu.be/zpkqNPwEzac) except that it is now called explicitly.

Then there are some arguments (`ARG`) that serve the same purpose as in Dockerfile. One of the big differences is that we can use them for almost anything, including specifying the base image (`FROM`) which is set to `scratch` by default.

Another difference is the way we copy artifacts. In case of Earthfile, we specify the target (`+binary`), the artifact (`silly-demo`), and where we'd like to copy it (`/usr/local/bin/silly-demo`).

Finally, unlike Dockerfile, Earthfile allows us also to push an image or any other artifact. It does that with the `SAVE` instruction we already saw in the binary target. This time, however, it is not saving an artifact but two images (`IMAGE`). The `registry`, the `image`, and the tags (`tag`, `taglatest`) are all defined as arguments.

The important part of that example with the `SAVE` instruction is the `--push` argument. That one indicates that the image might be pushed to the registry.

I said "might" because, as you'll see soon, we're still in control whether it will be pushed or not.

Let's build an image.

```sh
earthly +image --tag 0.0.1
```

The output is as follows.

```
 Init 🚀
————————————————————————————————————————————————————————————————————————————————

           buildkitd | Found buildkit daemon as docker container (earthly-buildkitd)

 Build 🔧
————————————————————————————————————————————————————————————————————————————————

              logbus | Setting organization "" and project ""
              +image | tag=0.0.1
              +image | --> FROM +base
g/v/silly-demo-earthly:0.0.5 | --> Load metadata ghcr.io/vfarcic/silly-demo-earthly:0.0.5 linux/arm64
              +image | tag=0.0.1
              +image | --> BUILD +binary
             +binary | tag=0.0.1
             +binary | --> FROM +base
              +image | tag=0.0.1
              +image | --> SAVE IMAGE ghcr.io/vfarcic/silly-demo:0.0.1 ghcr.io/vfarcic/silly-demo:latest
               +base | tag=0.0.1
               +base | --> FROM ghcr.io/vfarcic/silly-demo-earthly:0.0.5
               +base | [----------] 100% FROM ghcr.io/vfarcic/silly-demo-earthly:0.0.5
               +base | tag=0.0.1
               +base | *cached* --> WORKDIR /go-workdir
             +binary | tag=0.0.1
             +binary | *cached* --> COPY go.mod go.sum vendor .
             +binary | tag=0.0.1
             +binary | *cached* --> COPY *.go .
             +binary | tag=0.0.1
             +binary | *cached* --> RUN go mod vendor
             +binary | tag=0.0.1
             +binary | *cached* --> RUN GOOS=linux GOARCH=amd64 go build --mod vendor -o silly-demo
              +image | tag=0.0.1
              +image | *cached* --> COPY +binary/silly-demo /usr/local/bin/silly-demo
              output | --> exporting outputs
              output | [----------] 100% exporting outputs
              output | --> exporting outputs

 Push Summary ⏫ (disabled)
————————————————————————————————————————————————————————————————————————————————

To enable pushing use earthly --push
Did not push image ghcr.io/vfarcic/silly-demo:0.0.1
Did not push image ghcr.io/vfarcic/silly-demo:latest

 Local Output Summary 🎁
————————————————————————————————————————————————————————————————————————————————

Image +image output as ghcr.io/vfarcic/silly-demo:0.0.1
Image +image output as ghcr.io/vfarcic/silly-demo:latest


========================== 🌍 Earthly Build  ✅ SUCCESS ==========================

🛰️ Reuse cache between CI runs with Earthly Satellites! 2-20X faster than without cache. Generous free tier https://cloud.earthly.dev
```

The important thing to note in that output is that it `did not push image`. We did not use the `--push` argument with the CLI so it built images without pushing them.

If we would like to build and push an image, we need to add the `--push` argument. Since this is a demo, we'll also overwrite `registry` to use [ttl.sh](ttl.sh) since it is ephemeral and does not need credentials.

```sh
earthly --push +image \
    --tag 0.0.1 --registry ttl.sh --image $IMAGE
```

The output is as follows.

```
 Init 🚀
————————————————————————————————————————————————————————————————————————————————

           buildkitd | Found buildkit daemon as docker container (earthly-buildkitd)

 Build 🔧
————————————————————————————————————————————————————————————————————————————————

              logbus | Setting organization "" and project ""
              +image | image=silly-demo-20240526015336 registry=ttl.sh tag=0.0.1
              +image | --> FROM +base
g/v/silly-demo-earthly:0.0.5 | --> Load metadata ghcr.io/vfarcic/silly-demo-earthly:0.0.5 linux/arm64
              +image | image=silly-demo-20240526015336 registry=ttl.sh tag=0.0.1
              +image | --> BUILD +binary
             +binary | image=silly-demo-20240526015336 registry=ttl.sh tag=0.0.1
             +binary | --> FROM +base
              +image | image=silly-demo-20240526015336 registry=ttl.sh tag=0.0.1
              +image | --> SAVE IMAGE ttl.sh/silly-demo-20240526015336:0.0.1 ttl.sh/silly-demo-20240526015336:latest
               +base | image=silly-demo-20240526015336 registry=ttl.sh tag=0.0.1
               +base | --> FROM ghcr.io/vfarcic/silly-demo-earthly:0.0.5
               +base | [----------] 100% FROM ghcr.io/vfarcic/silly-demo-earthly:0.0.5
               +base | image=silly-demo-20240526015336 registry=ttl.sh tag=0.0.1
               +base | *cached* --> WORKDIR /go-workdir
             +binary | image=silly-demo-20240526015336 registry=ttl.sh tag=0.0.1
             +binary | *cached* --> COPY go.mod go.sum vendor .
             +binary | image=silly-demo-20240526015336 registry=ttl.sh tag=0.0.1
             +binary | *cached* --> COPY *.go .
             +binary | image=silly-demo-20240526015336 registry=ttl.sh tag=0.0.1
             +binary | --> RUN go mod vendor
             +binary | go: downloading github.com/bytedance/sonic v1.9.1
             +binary | go: downloading github.com/chenzhuoyu/base64x v0.0.0-20221115062448-fe3a3abad311
             +binary | go: downloading github.com/twitchyliquid64/golang-asm v0.15.1
             +binary | go: downloading github.com/klauspost/cpuid/v2 v2.2.4
             +binary | go: downloading github.com/gin-gonic/gin v1.9.1
             +binary | go: downloading github.com/go-pg/pg/v10 v10.11.2
             +binary | go: downloading github.com/go-resty/resty/v2 v2.10.0
             +binary | go: downloading github.com/stretchr/testify v1.8.3
             +binary | go: downloading golang.org/x/arch v0.3.0
             +binary | go: downloading github.com/gabriel-vasile/mimetype v1.4.2
             +binary | go: downloading golang.org/x/net v0.17.0
             +binary | go: downloading github.com/goccy/go-json v0.10.2
             +binary | go: downloading github.com/go-playground/validator/v10 v10.14.0
             +binary | go: downloading github.com/pelletier/go-toml/v2 v2.0.8
             +binary | go: downloading github.com/ugorji/go/codec v1.2.11
             +binary | go: downloading google.golang.org/protobuf v1.30.0
             +binary | go: downloading github.com/gin-contrib/sse v0.1.0
             +binary | go: downloading github.com/mattn/go-isatty v0.0.19
             +binary | go: downloading github.com/go-pg/zerochecker v0.2.0
             +binary | go: downloading github.com/jinzhu/inflection v1.0.0
             +binary | go: downloading github.com/vmihailenco/msgpack/v5 v5.3.4
             +binary | go: downloading github.com/vmihailenco/tagparser v0.1.2
             +binary | go: downloading mellium.im/sasl v0.3.1
             +binary | go: downloading github.com/tmthrgd/go-hex v0.0.0-20190904060850-447a3041c3bc
             +binary | go: downloading github.com/vmihailenco/bufpool v0.1.11
             +binary | go: downloading github.com/go-playground/locales v0.14.1
             +binary | go: downloading golang.org/x/sys v0.13.0
             +binary | go: downloading github.com/pmezard/go-difflib v1.0.0
             +binary | go: downloading github.com/go-playground/universal-translator v0.18.1
             +binary | go: downloading github.com/leodido/go-urn v1.2.4
             +binary | go: downloading golang.org/x/crypto v0.14.0
             +binary | go: downloading golang.org/x/text v0.13.0
             +binary | go: downloading github.com/vmihailenco/tagparser/v2 v2.0.0
             +binary | image=silly-demo-20240526015336 registry=ttl.sh tag=0.0.1
             +binary | --> RUN GOOS=linux GOARCH=amd64 go build --mod vendor -o silly-demo
              +image | image=silly-demo-20240526015336 registry=ttl.sh tag=0.0.1
              +image | *cached* --> COPY +binary/silly-demo /usr/local/bin/silly-demo
              output | --> exporting outputs
              output | [----------] 100% exporting outputs
              output | --> exporting outputs

 Push Summary ⏫
————————————————————————————————————————————————————————————————————————————————

Pushed image github.com/vfarcic/silly-demo:earthly+image as ttl.sh/silly-demo-20240526015336:0.0.1
Pushed image github.com/vfarcic/silly-demo:earthly+image as ttl.sh/silly-demo-20240526015336:latest

 Local Output Summary 🎁
————————————————————————————————————————————————————————————————————————————————

Image +image output as ttl.sh/silly-demo-20240526015336:0.0.1
Image +image output as ttl.sh/silly-demo-20240526015336:latest


========================== 🌍 Earthly Build  ✅ SUCCESS ==========================

🛰️ Reuse cache between CI runs with Earthly Satellites! 2-20X faster than without cache. Generous free tier https://cloud.earthly.dev
```

This time, the images were pushed to the [ttl.sh](ttl.sh) registry.

Let's turn our attention to the `all` target.

```sh
cat Earthfile
```

The output is as follows (truncated for brevity).

```
...
cosign:
    ARG --required tag
    RUN --push \
        --secret COSIGN_PASSWORD=cosignpassword \
        --secret cosignkey \
        --secret password \
        cosign sign --yes --key env://cosignkey \
        --registry-username $user \
        --registry-password $password \
        $registry/$image:$tag
...
all:
    ARG tag
    WAIT
        BUILD +image --tag $tag --taglatest latest
        BUILD +image --tag $tag-alpine \
            --taglatest latest-alpine --base alpine:3.18.4
    END
    BUILD +cosign --tag latest --tag $tag \
        --tag latest-alpine --tag $tag-alpine
    BUILD +timoni --tag $tag
    BUILD +helm --tag $tag
```

The `all` target combines most of the other targets thus allowing me to execute all the tasks at once. It is, in a way, more similar to what Makefile does than Dockerfile.

First, we're building scratch and alpine images. Since the image target has base argument set to scratch we only need to specify the base image for `alpine`.

Since Earthly tends to run as many tasks in parallel as it can, we have `WAIT` that ensures that images are built before the other tasks are executed. Otherwise, `cosign` might start signing images before they are built.

Under different circumstances, we would not need to wait for anything since Earhly would figure out the dependency tree as, for example, it knows that it needs to build the binary before it copies it to the image. But, in case of `cosign`, there is no explicit dependency so waiting for images to be built is a must.

*By the way, if you are not familiar with Cosign, please watch [Signing and Verifying Container Images With Sigstore Cosign and Kyverno](https://youtu.be/HLb1Q086u6M).*

Another interesting thing to note is how Earthly uses arguments. In case of the `cosign` target we are specifying the `tag` argument four times. As a result, Earthly will, effectively, execute the cosign target four time. That's very uselful since it saves us from adding loops and similar constructs.

Finally, it builds Timoni and Helm packages, pushes them to the registry, and makes changes to values stored locally.

Now, if we move back to the `cosign` target, we can see that it has the `--required` argument `tag` meaning that it will fail if we do not specify the tag of the image we want to sign.

More importantly, cosign needs a few secrets to be able to sign and push signatures. They are specified with `--secret` argument which can be in the key/value format or as only the value which becomes the key as well.

Once secrets are specified, we can use them as any other environment variable (`$password`).

I'll let you explore `timoni` and `helm` targets yourself and jump right into the execution of the `all` target.

To execute the `all` target, we need a few additional arguments.

We want to `--push` images and artifacts and we need to specify the `--secrets`. From there on, we can specify the target (`+all`) and the arguments.

```sh
earthly --push \
    --secret cosignpassword=$COSIGN_PASSWORD \
    --secret cosignkey=$COSIGN_PRIVATE_KEY \
    --secret password=IWillNeverTell \
    +all --tag 9.9.9 \
    --registry ttl.sh --image $IMAGE
```

Before we execute that command, there is one important thing to note. The order of arguments matters. The `--push`, `--secret` and other arguments available in earthly need to be specified before the target (`+all`) but the arguments (`--tag`, `--registry`, `--image`) that represent ARG key and values must be set after the target. It's silly and confusing. I wasted a few hours trying to figure out what I did wrong only to discover that the order matters. I understand that Earthly did that because that's how Docker expects arguments, but it's silly and bad nevertheless.

Let's press the enter key and see the end result.

The output is as follows (truncated for brevity).

```
 Init 🚀
————————————————————————————————————————————————————————————————————————————————

           buildkitd | Found buildkit daemon as docker container (earthly-buildkitd)

 Build 🔧
————————————————————————————————————————————————————————————————————————————————

              logbus | Setting organization "" and project ""
                +all | image=silly-demo-20240526015336 registry=ttl.sh tag=9.9.9
...
              output | --> exporting outputs

 Push Summary ⏫
————————————————————————————————————————————————————————————————————————————————

Pushed image github.com/vfarcic/silly-demo:earthly+image as ttl.sh/silly-demo-20240526015336:9.9.9
Pushed image github.com/vfarcic/silly-demo:earthly+image as ttl.sh/silly-demo-20240526015336:latest
Pushed image github.com/vfarcic/silly-demo:earthly+image as ttl.sh/silly-demo-20240526015336:9.9.9-alpine
Pushed image github.com/vfarcic/silly-demo:earthly+image as ttl.sh/silly-demo-20240526015336:latest-alpine

 Local Output Summary 🎁
————————————————————————————————————————————————————————————————————————————————

Artifact github.com/vfarcic/silly-demo:earthly+helm/Chart.yaml output as helm/app/Chart.yaml
Artifact github.com/vfarcic/silly-demo:earthly+helm/values.yaml output as helm/app/values.yaml
Artifact github.com/vfarcic/silly-demo:earthly+timoni/values.cue.tmp output as timoni/values.cue
Image +image output as ttl.sh/silly-demo-20240526015336:9.9.9
Image +image output as ttl.sh/silly-demo-20240526015336:latest
Image +image output as ttl.sh/silly-demo-20240526015336:9.9.9-alpine
Image +image output as ttl.sh/silly-demo-20240526015336:latest-alpine


========================== 🌍 Earthly Build  ✅ SUCCESS ==========================

🛰️ Reuse cache between CI runs with Earthly Satellites! 2-20X faster than without cache. Generous free tier https://cloud.earthly.dev
```

That's it. That single target did evertything. It compiled the binary, created and pushed four images, signed them all, and created Helm and Timoni packages. Awesome!

We're almost done. There are only two things I wanted to show before we jump into pros and cons and try to figure out whether you should use Earthly.

One of those is the GitHub Actions job that uses that Earthfile.

```sh
cat .github/workflows/ci.yaml
```

The output is as follows.

```yaml
name: ci
run-name: ci
on:
  push:
    branches:
      - main
      - master
jobs:
  build-container-image:
    runs-on: ubuntu-latest
    env:
      TAG: 1.4.${{ github.run_number }}
      FORCE_COLOR: 1
    steps:
      - uses: earthly/actions-setup@v1
        with:
          version: v0.8.0
      - name: Checkout
        uses: actions/checkout@v4
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3
      - name: Login to ghcr
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: vfarcic
          password: ${{ secrets.REGISTRY_PASSWORD }}
      - name: Build and push
        run: |
          earthly --push \
            --secret cosignpassword="$COSIGN_PASSWORD" \
            --secret cosignkey="$COSIGN_PRIVATE_KEY" \
            --secret password="$REGISTRY_PASSWORD" \
            +all --tag $TAG
        env:
          COSIGN_PRIVATE_KEY: ${{ secrets.COSIGN_PRIVATE_KEY }}
          COSIGN_PASSWORD: ${{ secrets.COSIGN_PASSWORD }}
          REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
      - name: Commit changes
        run: |
          git config --local user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git config --local user.name "github-actions[bot]"
          git add .
          git commit -m "Release ${{ env.TAG }} [skip ci]"
      - name: Push changes
        uses: ad-m/github-push-action@master
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          branch: ${{ github.ref }}
```

If we ignore the initial code `Checkout` and `Login to ghcr` (GitHub registry) and the steps that commit (`Commit changes`) and pushe (`Push changes`) changes back to the repo, all the tasks are executed as a single `earthly` command in the same way we'd execute it locally or from anywhere else.

There's not much more to it. If we want to run Earthly from CI/CD pipelines, we just need to execute the CLI.

Finally, there is [Earthly Cloud](https://earthly.dev/earthly-cloud) service that helps with caching and a few other things.

The only thing left to explore is to go through pros and cons and see whether you should adopt Earthly.

## Earthly and Earthfile Pros and Cons

Earthly is an interesting project.

Many have been trying to use Dockerfile as a task executor that goes beyond building container images, especially since the emergence of Multi-Stage Builds. However, the truth is that, as far as I know, all those attempts failed simply because Dockerfile alone was not designed to do much more than to act as a specification used to build container images.

Earthly changes that. It adopted Docker as the engine and combined Dockerfile with Makefile both as formats and as functionalities those two offer when combined. The end result is, more or less, a familiar syntax that can build artifacts and execute tasks.

Yet, trying to make Dockerfile do what Makefile does comes at a cost and can easily result in frustration. Through a familiar syntax, one can aadvance fast only to be blocked by some silly issue later.

Personally, I'm not sure that Earthly is good enough for me to switch away from my current favorites. I think that [Devbox](https://youtu.be/WiFLtcBvGMU) is still a superior way to manage Nix dependencies instead of using container images. Similarly, [Task with Taskfile](https://youtu.be/Z7EnwBaJzCk) or [Just with Justfile](https://youtu.be/hgNN2wOE7lc) are better task executors than Earthly with Earthfile. On top of those, you'll likely need Dockerfile that defines container images which are created by executing tasks through those tools.

The advantage Earthly has is that it combines all those. Instead of using Nix or Devbox for dependencies and Task or Just for executing tasks, and Dockerfile for specifying container images, we define everything as Earthfile. That's, in my opinion, its main strenght.

Let's go through pros and cons.

**Cons**
* Order matters
* Secrets
* Silly issues
* Images saved last

CLI arguments, at least as perceived by users, must be placed in specific order. Some like `--push` and `--secret` must be placed before the target while others, specifically those defined inside Earthfile as `ARG` must be placed after the target. I understand that the reason for that lies in some being baked-in arguments while others are passed to Docker containers as arguments. However, when executing the CLI, they all look like "normal" arguments. As a result, one can easily place an argument in a wrong place, not receive any warning, yet receive unexpected results which might lead to hard to debug yet silly issues. Anyway, beware that **order matters**. You need to know what goes before and what goes after the target.

Next, **secrets** can easily leak. Even though Earthly masks them under certain conditions, it is very easy to make an unintentional (or intentional) mistake that will leak them. It's enough to have an instruction like `RUN echo $secret` and it'll appear in the output.

Then there are some quirks like, for example, `IF` instruction not working until we place it inside a busybox image. Earthly is full of such **silly issues** which leads me to conclude that it is immature.

I'm not sure whether everything runs in parallel or **images are saved last**. In any case, not being able to control what happens when often results in having to perform some silly workaround like explicitly specifying that Earthly should wait for images to be saved before, for example, trying to sign them.

As for pros...

**Pros**
* Familiar syntax
* All in one

There are two big positive reasons why Earthly is very interesting.

To begin with, it uses a **familiar syntax**. For the most part, it is Dockerfile split into recipes like those in Makefile. If you know Dockerfile, you already know most of what you'll need to write Earthfile. Assuming that most of us, by now, know Dockerfile, that means that the learning curve for Earthfile is almost non-existent.

The second big one is that Earthfile combines multiple files and formats into one. Instead of specifying `devbox.json` or whichever other way we're using to defined dependencies, and Taskfile or Justfile or Makefile for tasks, and Dockerfile for images, we can simply define a single Earthfile. It's all in one solution and **all in one** format for dependencies, tasks, and image specs.

All in all Earthfile combines functionality of different tools and formats into one. That's awesome. However, it's not necessarily better than any of them individually. That leads to a question. **Does it make sense to switch to slightly worse solution for the sake of simplification?** More often than not, the answer to that question is yes. However, I'm not sure that I am ready and willing to abandon Devbox and Just. I'll keep using those and switch only a few projects to Earthly. Who knows. It might grow on me enough to consider moving away from my current favorites.

Until then, do not take my conservative approach as a negative thing. Earthly is a very interesting project that is worth exploring further. I truly like the idea behind it. I'm just not yet willing to adopt it exclusively.

## Destroy

```sh
git stash

git checkout master

exit
```


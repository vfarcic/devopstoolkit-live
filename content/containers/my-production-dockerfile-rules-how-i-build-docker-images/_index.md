
+++
title = 'My Production Dockerfile Rules: How I Build Docker Images'
date = 2026-01-22T13:30:00+00:00
draft = false
+++

Most Dockerfiles I see in production are security nightmares waiting to happen. Running as root. Using `:latest` tags. Copying entire directories including secrets. And the images? Bloated with debugging tools that attackers love.

Here's the thing. Writing a good Dockerfile isn't hard. It's just that nobody taught you the rules. Today, I'm going to show you every best practice you need to build production-ready containers. We'll cover image selection, build optimization, security hardening, and maintainability. And at the end, I'll show you how AI can apply all these rules automatically.

Let's start with the foundation: choosing the right base image.

<!--more-->

{{< youtube ueTe-VQaD7c >}}

## Setup

> This demo is using Claude Code as the coding agent. With a few modification, it should work with any other coding agents like Cursor, GitHub Copilot, etc. The major change you might need to make is to change `.mcp-docker-minimal.json` to whichever format and location for MCP config your agent expects.

```sh
npm install -g @anthropic-ai/claude-code

git clone https://github.com/vfarcic/dot-ai-website

cd dot-ai-website

git pull

git fetch

git switch demo/dockerfile

export DOT_AI_IMAGE=ghcr.io/vfarcic/dot-ai:0.159.0
```

## Dockerfile Base Images

The first rule is simple: **use minimal base images**. Containers are not virtual machines. They already run on top of the host operating system, sharing its kernel. You don't need Ubuntu or Debian inside your container. All you need are the runtime dependencies your application requires, nothing more.

So forget about full distribution images. Instead, reach for Alpine, slim variants, distroless, or even scratch images. Every package you don't include is a package that can't have vulnerabilities. Smaller images mean a smaller attack surface, faster pulls, and less storage.

Here's a general rule of thumb. If you're working with compiled languages like Go or Rust, distroless or scratch images are your best bet since you can compile everything into a static binary with no runtime dependencies. For interpreted languages like Node.js or Python, you'll need the runtime, so Alpine or slim variants make more sense.

Let's take a look at the Dockerfile I'm using.

```sh
cat Dockerfile
```

The output is as follows (truncated for brevity). We'll see other parts of Dockerfile later.

```Dockerfile
FROM node:20-alpine AS builder
...
```

I'm using `node:20-alpine` as my base. There are arguably better options like Chainguard images, but the free version fails with one of the rules we'll cover later, so Alpine it is.

The second rule: **use multi-stage builds**. The idea is to separate your build dependencies from your runtime. Why does this matter? Because the tools you need to compile or build your application are not the tools you need to run it. Node modules for building, compilers, dev dependencies... none of that belongs in your production image. It bloats the image and increases the attack surface for no good reason.

```Dockerfile
FROM node:20-alpine AS builder
...
COPY docusaurus.config.ts ./
COPY tsconfig.json ./
COPY sidebars ./sidebars
COPY src ./src
COPY static ./static
COPY docs ./docs
...
RUN npm run build
...
FROM nginx:1.29-alpine
...
COPY --from=builder /app/build /usr/share/nginx/html
...
CMD ["nginx", "-g", "daemon off;"]
```

Here's how it works. The first stage uses `node:20-alpine` to install dependencies (`COPY ...`) and build the application (`RUN npm run build`). The second stage is just `nginx:1.29-alpine`. It copies only the built files from the first stage (`COPY --from=builder /app/build /usr/share/nginx/html`) and serves them (`CMD ["nginx", "-g", "daemon off;"]`). All those node_modules, all those build tools, even the source code... they're gone. The final image contains only what's needed to run.

The third rule: **derive the version from your project**. Don't just pick a random version for your base image. Your project already defines what version it needs, so use that as your source of truth. Why? Because you want your container to run the same version your project was designed for. Mismatched versions lead to subtle bugs that are a pain to debug.

```sh
cat package.json
```

```json
{
  ...
  "engines": {
    "node": ">=20.0"
  }
}
```

The *package.json* specifies `node` `20` or higher in the engines field.

```sh
cat Dockerfile
```

```Dockerfile
FROM node:20-alpine AS builder
...
```

And the Dockerfile uses `node:20-alpine`. The version matches. For Go projects, you'd check *go.mod*. For Python, *pyproject.toml* or *.python-version*. The point is: don't guess, derive.

Now let's talk about how to optimize the build process itself.

## Dockerfile Layer Caching

The first optimization rule is about **layer caching**. Docker caches each layer, and when something changes, it invalidates that layer and everything after it. Why does this matter? Because if you copy your source code before installing dependencies, every code change forces a full dependency reinstall. That's slow and wasteful.

The fix is simple: put the things that change rarely at the top, and the things that change often at the bottom. Base image first, then dependency manifests, then configuration, and finally source code.

```Dockerfile
FROM node:20-alpine AS builder
...
COPY package.json package-lock.json ./
...
COPY docusaurus.config.ts ./
COPY tsconfig.json ./
COPY sidebars ./sidebars
COPY src ./src
COPY static ./static
COPY docs ./docs
```

Notice the order. The `package.json` and `package-lock.json` come first. These define dependencies and rarely change. Configuration files come next (`COPY package.json package-lock.json ./`). Source code comes last (`COPY ...`) because that's what changes most frequently. Now when you change your code, Docker reuses the cached layers for everything above it.

Next: **combine RUN commands**. Each RUN instruction creates a new layer. Fewer layers generally means a smaller image.

```Dockerfile
...
RUN chown -R appuser:appgroup /usr/share/nginx/html && \
    chown -R appuser:appgroup /var/cache/nginx && \
    chown -R appuser:appgroup /var/log/nginx && \
    touch /var/run/nginx.pid && \
    chown -R appuser:appgroup /var/run/nginx.pid
...
```

This example shows multiple `chown` commands chained together. Not the most exciting demonstration, but this image doesn't need package installations.

Here's where combining commands actually matters: **cleaning caches**. If you install packages in one RUN command and clean the cache in another, the cache still exists in the earlier layer. You've saved nothing. The cleanup must happen in the same layer as the installation.

```Dockerfile
...
RUN npm ci --ignore-scripts && \
    npm cache clean --force
...
```

See how `npm ci` and `npm cache clean` are in the same RUN command? That's the pattern. Install and clean in one shot, so the cache never makes it into any layer.

Next: **explicit COPY**. Never use `COPY . .`. The consequences depend on where you use it. In the final stage, it copies everything into your image, including secrets, `.git` directories, and files you don't need. That's a security and size problem. In earlier stages, those files won't end up in the final image, but you're still slowing down your build and hurting layer caching. Any file change invalidates that layer, even files you don't care about.

```Dockerfile
...
COPY src ./src
COPY static ./static
COPY docs ./docs
...
```

Be explicit about what you copy (`COPY src ./src`, `COPY static ./static`, `COPY docs ./docs`). Yes, it's more verbose. But you know exactly what's going into each stage.

Finally: **production dependencies only**. If your runtime image needs dependencies, install only what's required to run, not what's required to build or test. For Node.js, that means *npm ci --omit=dev*. For Python, separate your requirements files. DevDependencies have no business in a production container.

Now let's talk about security.

## Dockerfile Security Hardening

First rule: **non-root user**. Never run containers as root. Why? Because if an attacker breaks into your container, they shouldn't get root privileges on top of it. Create a dedicated user and run your application as that user.

```Dockerfile
RUN addgroup -g 10001 -S appgroup && \
    adduser -u 10001 -S appuser -G appgroup
```

We create a group (`RUN addgroup -g 10001 -S appgroup`) and user (`adduser -u 10001 -S appuser -G appgroup`) with UID 10001. Why that number? UIDs below 10000 are often reserved for system users. Starting at 10001 avoids conflicts.

Next: **pin image versions**. Never use *:latest*. Why? Because *:latest* changes without warning. Your build might work today and break tomorrow because someone pushed a new version. Worse, you can't reproduce builds reliably.

```Dockerfile
FROM node:20-alpine AS builder
...
```

Ideally, you'd pin to an exact version like *node:20.11.1-alpine3.19*. But that increases maintenance since you need to review, test, and merge updates constantly. Using `node:20-alpine` is a middle ground: you get the major version locked but still receive patches automatically.

Remember when I mentioned Chainguard images earlier? This is the rule they fail. Their free tier only offers the `:latest` tag. I'm too cheap to pay for it in a demo, but you should consider them for production.

Next: **official images**. Use Docker Official Images or Verified Publishers. Why? Because random images from unknown publishers could contain anything: malware, cryptominers, backdoors. Official images are maintained, scanned, and trusted.

```Dockerfile
...
FROM nginx:1.29-alpine
...
FROM nginx:1.29-alpine
...
```

In this case, both `node` and `nginx` are Docker Official Images, so we're good.

A few more rules that all boil down to the same principle: **keep it minimal**.

**No secrets in image.** Never embed credentials, API keys, or passwords in your Dockerfile or ENV instructions. Images get pushed to registries, cached on build servers, and pulled by who knows how many systems. Secrets in images are secrets exposed.

**No sudo.** If you need root access for something, use a separate build stage or switch USER explicitly. Installing sudo in a container is asking for trouble.

**Minimal packages.** Only install what your application actually needs to run. Every extra package is extra attack surface and extra bytes. If you do end up using apt-get, always use *--no-install-recommends*. Otherwise you'll get a bunch of optional packages you never asked for.

**COPY over ADD.** Always use COPY unless you specifically need ADD's tar extraction feature. And never use ADD with URLs. It's unpredictable and can introduce security risks.

Finally: **no debugging tools**. Don't install curl, wget, vim, or netcat in production images. Why? Because attackers love these tools. If they get into your container, you've just handed them everything they need to explore your network, download more malware, or exfiltrate data.

But what if you need to debug something in production? Use ephemeral debug containers.

> We did not set up the cluster so the next few `kubectl` commands won't work if you're following along.

Let's say I need to run *dig* to debug a DNS issue in my container.

```sh
kubectl --namespace dot-ai exec --stdin --tty [...] -- dig -h
```

```text
error: Internal error occurred: Internal error occurred: error executing command in container: failed to exec in container: failed to start exec "f9c3a72369ebd00e1def6e28dcae43510bad878e346f0b49efae36242ad3abe3": OCI runtime exec failed: exec failed: unable to start container process: exec: "dig": executable file not found in $PATH.
```

The tool isn't there. Good. That's exactly what we want. Now let's use an ephemeral debug container instead.

```sh
kubectl --namespace dot-ai debug --stdin --tty [...] \
    --image nicolaka/netshoot -- dig -h
```

```
Defaulting debug container name to debugger-ssb6c.
Usage:  dig [@global-server] [domain] [q-type] [q-class] {q-opt}
            {global-d-opt} host [@local-server] {local-d-opt}
            [ host [@local-server] {local-d-opt} [...]]
Where:  domain	  is in the Domain Name System
        q-class  is one of (in,hs,ch,...) [default: in]
        q-type   is one of (a,any,mx,ns,soa,hinfo,axfr,txt,...) [default:a]
                 (Use ixfr=version for type ixfr)
        ...
```

Now it works. The `netshoot` image has all the networking tools you'd ever need. It attaches temporarily to your pod, you do your debugging, and when you're done, it's gone. Your production image stays clean.

One more security rule: **executables owned by root**. Application binaries should be owned by root but executed by a non-root user. Why? If an attacker compromises your application, they can't modify the binaries to persist their access. The files are read-only to them.

Now let's wrap up with some maintainability best practices.

## Dockerfile Maintainability

First: **sort arguments**. When you have multi-line package lists, alphabetize them. Why? Because it makes diffs cleaner, reviews easier, and merge conflicts less likely. It's a small thing, but it adds up.

Next: **use WORKDIR**. Never use `RUN cd` to change directories.

```sh
cat Dockerfile
```

```Dockerfile
...
WORKDIR /app
...
```

Why not *cd*? Because it only affects that single RUN command. The next instruction starts back at the default directory. `WORKDIR` sets the directory for all subsequent instructions and makes your Dockerfile easier to read.

Next: **exec form for CMD**.

```Dockerfile
...
CMD ["nginx", "-g", "daemon off;"]
```

Always use the JSON array format for your CMD instruction (`CMD ["nginx", "-g", "daemon off;"]`).

Why does this matter? The exec form runs your process directly, so it receives signals properly. If you use the shell form like *CMD nginx -g "daemon off;"*, your process runs as a child of */bin/sh*. When Kubernetes or Docker sends a SIGTERM to stop your container, the shell receives it, not your application. Your app never gets the chance to shut down gracefully.

Finally, two rules about documentation. **Comment non-obvious decisions.** Explain why you made certain choices, not what the command does. Anyone can read that you're installing a package. What they can't figure out is why you pinned it to that specific version or why you're using a particular flag.

And **add OCI labels**. Labels like *org.opencontainers.image.source*, *org.opencontainers.image.version*, and *org.opencontainers.image.description* help with image management. When you're staring at a list of images trying to figure out what they are and where they came from, labels save you.

That covers the best practices. But remembering all these rules and applying them consistently is tedious. Let's see how AI can help.

## AI Dockerfile Generator

All these best practices are baked into a prompt available at [DevOps AI Toolkit](https://devopstoolkit.ai). You give it a project, and it generates a production-ready Dockerfile following every rule we just covered. Let's see it in action.

First, we'll remove Dockerfile so that we start from scratch.

```sh
rm Dockerfile .dockerignore
```

> The instructions below are for Claude Code. Minor adaptations might be needed to reproduce the same steps in other coding agents.

I'll open Claude Code with the MCP configuration that includes the Dockerfile generation prompt.

```sh
claude --mcp-config .mcp-docker-minimal.json
```

> Execute slash command (prompt) available as `/generate-dockerfile`.

```text
> /dot-ai:generate-dockerfile (MCP) is running…

⏺ Let me check for existing Dockerfile and .dockerignore files, and analyze the project structure.

⏺ Search(pattern: "Dockerfile*")
  ⎿  Found 1 file (ctrl+o to expand)

⏺ Search(pattern: ".dockerignore")
  ⎿  Found 0 files

⏺ Search(pattern: "package.json")
  ⎿  Found 100 files (ctrl+o to expand)

⏺ Search(pattern: "**/*.{js,ts,jsx,tsx}")
  ⎿  Found 100 files (ctrl+o to expand)

⏺ I see the project has no Dockerfile or .dockerignore at the root level (the Dockerfile in node_modules is from a dependency). Let me read the root package.json and explore the project structure.

⏺ Read(package.json)
  ⎿  Read 50 lines

⏺ Search(pattern: "*", path: "~/code/dot-ai-website")
  ⎿  Found 100 files (ctrl+o to expand)

⏺ Now let me explore the project structure more thoroughly to understand the Docusaurus configuration and build output.

⏺ Bash(ls -la /Users/viktorfarcic/code/dot-ai-website/ | head -30)
  ⎿  -D .
     -D ..
     -- .claude
     … +27 lines (ctrl+o to expand)

⏺ Read(docusaurus.config.ts)
  ⎿  Read 177 lines

⏺ Search(pattern: ".nvmrc", path: "~/code/dot-ai-website")
  ⎿  Found 2 files (ctrl+o to expand)

⏺ Search(pattern: ".node-version", path: "~/code/dot-ai-website")
  ⎿  Found 0 files

⏺ Now let me check the source directory structure, build output, and any static assets to understand what files need to be copied.

⏺ Bash(ls -la /Users/viktorfarcic/code/dot-ai-website/src/)
  ⎿  -- .
     -D ..
     -- components
     … +2 lines (ctrl+o to expand)

⏺ Bash(ls -la /Users/viktorfarcic/code/dot-ai-website/docs/)
  ⎿  -- .
     -D ..
     -I .DS_Store
     … +2 lines (ctrl+o to expand)

⏺ Bash(ls -la /Users/viktorfarcic/code/dot-ai-website/static/)
  ⎿  -- .
     -D ..
     -- .nojekyll
     -- img

⏺ Bash(ls -la /Users/viktorfarcic/code/dot-ai-website/sidebars/)
  ⎿  -- .
     -D ..
     -- controller.ts
     -- mcp.ts

⏺ Let me check if there's an existing build output and look at the .gitignore to understand what should be excluded.

⏺ Read(.gitignore)
  ⎿  Read 34 lines

⏺ Bash(ls -la /Users/viktorfarcic/code/dot-ai-website/build/ 2>/dev/null || echo "No build directory exists")
  ⎿  -I .
     -D ..
     -I .nojekyll
     … +9 lines (ctrl+o to expand)

⏺ Web Search("Node.js 20 LTS current stable version December 2025")
  ⎿  Did 1 search in 14s

⏺ Now I have a clear understanding of the project. Let me update my todo and generate the Dockerfile.

  Project Analysis Summary:
  - Type: Docusaurus static site (documentation portal)
  - Language: Node.js/TypeScript
  - Node Version: >=20.0 (specified in package.json engines)
  - Framework: Docusaurus 3.9.2
  - Build Command: npm run build (produces static files in /build)
  - Runtime: Static files only - needs nginx to serve
  - No external binaries needed at runtime
  - Build artifacts: Static HTML/CSS/JS in /build directory
```

The AI analyzed all the relevant files in the project: *package.json* for dependencies and Node version, *docusaurus.config.ts* for the framework, source directories, and everything else that might provide information needed to generate the Dockerfile. It figured out this is a static site that needs Nginx to serve the built files. Now it generates the Dockerfile.

```text
⏺ Write(Dockerfile)

──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 Create file
╭────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮│ Dockerfile                                                                                                                                         ││                                                                                                                                                    │
│ # Build stage - use Node.js LTS for building Docusaurus                                                                                            │
│ FROM node:20-alpine AS builder                                                                                                                     │
│                                                                                                                                                    │
│ WORKDIR /app                                                                                                                                       │
│                                                                                                                                                    │
│ # Copy dependency manifests first for layer caching                                                                                                │
│ COPY package.json package-lock.json ./                                                                                                             │
│                                                                                                                                                    │
│ # Install dependencies and clean npm cache in same layer                                                                                           │
│ RUN npm ci --ignore-scripts && \                                                                                                                   │
│     npm cache clean --force                                                                                                                        │
│                                                                                                                                                    │
│ # Copy source files needed for build                                                                                                               │
│ COPY docusaurus.config.ts tsconfig.json ./                                                                                                         │
│ COPY src/ ./src/                                                                                                                                   │
│ COPY static/ ./static/                                                                                                                             │
│ COPY sidebars/ ./sidebars/                                                                                                                         │
│ COPY docs/ ./docs/                                                                                                                                 │
│                                                                                                                                                    │
│ # Build static site                                                                                                                                │
│ RUN npm run build                                                                                                                                  │
│                                                                                                                                                    │
│ # Runtime stage - use nginx-unprivileged for serving static files                                                                                  │
│ FROM nginxinc/nginx-unprivileged:1.27-alpine                                                                                                       │
│                                                                                                                                                    │
│ # Copy built static files from builder stage                                                                                                       │
│ COPY --from=builder /app/build /usr/share/nginx/html                                                                                               │
│                                                                                                                                                    │
│ # nginx-unprivileged already runs as non-root user (uid 101)                                                                                       │
│ # No need to create user or change ownership                                                                                                       │
│                                                                                                                                                    │
│ # nginx-unprivileged listens on port 8080 by default                                                                                               │
│ EXPOSE 8080                                                                                                                                        │
│                                                                                                                                                    │
│ # Use default nginx command (already in exec form in base image)                                                                                   │
│ CMD ["nginx", "-g", "daemon off;"]                                                                                                                 │
│                                                                                                                                                    │
╰────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
 Do you want to create Dockerfile?
 ❯ 1. Yes
   2. Yes, allow all edits during this session (shift+tab)
   3. Type here to tell Claude what to do differently

⏺ Write(Dockerfile)
  ⎿  Wrote 37 lines to Dockerfile
     # Build stage - use Node.js LTS for building Docusaurus
     FROM node:20-alpine AS builder
     WORKDIR /app                                                                                                                                          # Copy dependency manifests first for layer caching                                                                                                   COPY package.json package-lock.json ./
     # Install dependencies and clean npm cache in same layer
     RUN npm ci --ignore-scripts && \
     … +27 lines (ctrl+o to expand)
```

Multi-stage build, Alpine base, layer caching, non-root user, pinned versions. All the best practices, applied automatically. It also generates a *.dockerignore*.

```text
⏺ Write(.dockerignore)
  ⎿  Wrote 16 lines to .dockerignore
     # Dependencies - large directory not needed (npm ci installs fresh)
     node_modules/
     # Build artifacts - regenerated during build
     build/
     .docusaurus/
     # Secret files that could accidentally exist in copied directories
     .env*
     *.pem
     … +6 lines (ctrl+o to expand)
```

It excludes *node_modules*, build artifacts, secrets, and git files. Now it builds the image and validates that everything works.

```text
⏺ Now let me build and test the Docker image.

⏺ Bash(docker build -t dot-ai-website-validation . 2>&1) timeout: 5m 0s
  ⎿  #0 building with "desktop-linux" instance using docker driver

     #1 [internal] load build definition from Dockerfile
     … +141 lines (ctrl+o to expand)

⏺ The build succeeded. Now let me run the container and validate it.

⏺ Bash(docker run -d --name dot-ai-website-test -p 8081:8080 dot-ai-website-validation && sleep 5)
  ⎿  607cd951c5b5f2c581d2e1b3f3fd836794d1afc347cd702e4c092fdfed298952

⏺ Bash(docker inspect --format='{{.State.Status}}' dot-ai-website-test)
  ⎿  running

⏺ Bash(docker inspect --format='{{.State.ExitCode}}' dot-ai-website-test)
  ⎿  0

⏺ Bash(docker logs dot-ai-website-test 2>&1)
  ⎿  /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
     /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
     /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
     … +22 lines (ctrl+o to expand)

⏺ Container is running successfully. Let me also run linting and security scan if available, then clean up.

⏺ Bash(hadolint /Users/viktorfarcic/code/dot-ai-website/Dockerfile 2>&1 || echo "hadolint not installed or completed")
  ⎿  (eval):1: command not found: hadolint
     hadolint not installed or completed

⏺ Bash(trivy image --severity HIGH,CRITICAL dot-ai-website-validation 2>&1 || echo "trivy not installed or completed")
  ⎿  (eval):1: command not found: trivy
     trivy not installed or completed

⏺ Linting and security scan tools are not installed (skipping). Let me check the image size and clean up.

⏺ Bash(docker images dot-ai-website-validation --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}")
  ⎿  REPOSITORY                  TAG       SIZE
     dot-ai-website-validation   latest    106MB

⏺ Bash(docker stop dot-ai-website-test && docker rm dot-ai-website-test && docker rmi dot-ai-website-validation)
  ⎿  dot-ai-website-test
     dot-ai-website-test
     Untagged: dot-ai-website-validation:latest
     Deleted: sha256:71734a6f9174e2aa66f81f208bd96dc5dac81037b42a9f4f1801f86a2dd6b3c4
```

It built the image, ran a container to verify everything works, checked the logs, and cleaned up after itself. 106 megabytes for the final image. Not bad for a Node.js application.

```
⏺ I've generated a production-ready Dockerfile and .dockerignore for your Docusaurus documentation portal. Here's a summary:

  Generated Files

  Dockerfile

  Multi-stage build with two stages:

  1. Builder stage (node:20-alpine): Installs dependencies and builds the static site
  2. Runtime stage (nginxinc/nginx-unprivileged:1.27-alpine): Serves the static files

  Security features:
  - Runs as non-root user (nginx-unprivileged uses uid 101)
  - Uses pinned image versions
  - Minimal alpine-based images
  - No debugging tools or secrets

  Optimizations:
  - Layer caching via dependency manifest copying first
  - Only production build artifacts copied to runtime
  - Final image size: ~106MB

  .dockerignore

  Minimal exclusions (~15 lines) to:
  - Exclude node_modules/ (npm ci installs fresh)
  - Exclude build artifacts that get regenerated
  - Exclude secrets and git files

  Commands

  Build:
  docker build -t dot-ai-website .

  Run:
  docker run -p 8080:8080 dot-ai-website

  The site will be available at http://localhost:8080.

  Validation Results

  - ✅ Image builds successfully
  - ✅ Container starts and runs (nginx serving static files)
  - ✅ No errors in logs
  - ⏭️ Hadolint/Trivy not installed (skipped)
```

That's it. From project analysis to a production-ready Dockerfile in seconds, following all the best practices we covered.

And those practices are worth remembering. For base images: **use minimal images** like Alpine or distroless, **use multi-stage builds**, and **derive versions from your project files**. For builds: **order layers** so rarely-changing content comes first, **combine RUN commands**, be **explicit about what you COPY**, and **only install production dependencies**. For security: **never run as root**, **pin versions**, **use official images**, **keep secrets out**, and **skip debugging tools**. For maintainability: **use WORKDIR**, **use exec form for CMD**, and **add OCI labels**.

That's a lot to keep track of. But now you don't have to. Use the AI prompt at [DevOps AI Toolkit](https://devopstoolkit.ai) and let it handle the details. Your containers will be smaller, faster, and more secure.


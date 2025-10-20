
+++
title = 'MCP Server Deployment Guide: From Local To Production'
date = 2025-10-20T16:00:00+00:00
draft = false
+++

Everyone's using MCP servers these days. They're connecting AI agents to databases, Kubernetes clusters, GitHub repos, cloud resources, you name it. MCP is becoming the standard way to give AI tools access to external systems. But here's the question: how should you actually run these MCP servers?

The documentation typically shows you one way: running it locally with NPX. But is that secure? Is it scalable? Can your team share it? What about production? There are actually multiple deployment options, each with different trade-offs.

So here's what we're going to do: I'll show you **four different ways** to deploy MCP servers, from the dead simple to the enterprise-ready. We'll look at local execution with NPX, Docker containers, Kubernetes deployments, and operator-managed resources. Plus, I'll cover a few notable cloud platforms like Fly.io, Cloudflare Workers, and AWS Lambda. For each approach, I'll show you exactly how it works, what problems it solves, and what new problems it creates.

<!--more-->

{{< youtube MHf-M8qOogY >}}

We'll explore all of this through practical examples. I'll use my [DevOps AI Toolkit](https://github.com/vfarcic/dot-ai) as the MCP server we're deploying. Not to sell you on the project, but because I need a real MCP server to demonstrate these patterns, and this one works with Kubernetes, vector databases, and all the complexity we need to see. Everything you'll learn applies to any MCP server you want to run.

## Setup

> This demo is using Claude Code as the coding agent. With a few modification, it should work with any other coding agents like Cursor, GitHub Copilot, etc.

> The project we'll explore (MCP) currently supports only Anthropic Sonnet models. Please open an [issue in the dot-ai](https://github.com/vfarcic/dot-ai/issues) repo if you'd like the support for other models or if you have any other feature request or a bug to report.

> Install [NodeJS](https://nodejs.org/en/download) if you don't have it already.

```sh
npm install -g @anthropic-ai/claude-code

git clone https://github.com/vfarcic/dot-ai

cd dot-ai

git pull

git fetch

git switch mcp-run
```

> Make sure that Docker is up-and-running. We'll use it to create a KinD cluster.

> Watch [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell

./dot.nu setup --dot-ai-tag 0.85.0 \
    --qdrant-run true --qdrant-tag 0.8.0

source .env
```

## MCP Local NPX Deployment

Let's start with the simplest way to run MCP servers: executing them directly on your local machine. This is typically the default approach you'll find in most MCP documentation. Whether it's NPX for JavaScript servers, Python for Python-based ones, or whatever runtime the server needs, you just run the command directly. It's what's usually documented, often because authors assume you'll figure out how to transform it into something better, or because they were too lazy to show you the alternatives.

Let me show you what this looks like. Here's the configuration file that tells Claude how to run my dot-ai MCP server locally. I'm using my own MCP server for this demo since that's the one I'm working on independently of this video, but the same principles apply to any MCP server.

```sh
cat .mcp-npx.json
```

```json
{
  "mcpServers": {
    "dot-ai": {
      "command": "npx",
      "args": ["-y", "--package=@vfarcic/dot-ai@latest", "dot-ai-mcp"],
      "env": {
        "KUBECONFIG": "./kubeconfig.yaml",
        "DOT_AI_SESSION_DIR": "./tmp/sessions",
        "QDRANT_URL": "http://localhost:6333"
      }
    }
  }
}
```

Pretty straightforward. We're telling Claude to use `npx` to run the MCP server, passing in the package name and the environment variables it needs. Notice that Qdrant URL there? That's our vector database dependency that needs to be running separately. We already started it during setup, but in a real-world scenario, you'd need to manage that yourself.

Now let's fire up Claude with this configuration and see it in action.

```sh
claude --mcp-config .mcp-npx.json
```

Once Claude starts up, let's test if the MCP server is working by asking it to list the patterns available in that MCP server.

[user]
```md
List patterns
```

> AI is non-deterministic. Your experience will differ and you might need to adapt examples to what you see on the screen.

[agent]
```md
⏺ You have 3 organizational patterns:

  1. AWS public services (ID: 70645609...)
    - 27 triggers, 1 resource
    - Created: 2025-08-30
  2. DevOps Toolkit DB (ID: 9cdb8e4f...)
    - 11 triggers, 1 resource
    - Created: 2025-08-25
  3. Azure ResourceGroup (ID: dedcafc2...)
    - 1 trigger, 1 resource
    - Created: 2025-08-15
```

Perfect! The MCP server is running locally and Claude can communicate with it. We can see the patterns that are stored in Qdrant. Everything works as expected.

Now, this local approach might seem great at first, but there are some serious trade-offs you need to consider. This is typically the default, and sometimes the only documented way to run an MCP server. But that doesn't mean it's the best approach for your situation.

First off, adding dependencies is a pain in the ass. See how we need Qdrant running separately? In a more complex setup, you might need multiple services, databases, or other dependencies. Now, agents typically start the MCP server when they launch and shut it down when you exit them. But if you wrap everything in a script to handle those dependencies, you might end up with zombie processes. The agent might kill the script but leave all the child processes running. Before you know it, you've got a bunch of orphaned services eating up your resources.

Then there's the installation requirements. You need Node and NPX installed on your machine for this JavaScript-based MCP. If you're using a Python MCP, you need Python. Ruby MCP? You need Ruby. Your machine starts to become a mess of different runtimes and package managers.

But here's the biggest issue: there's absolutely **no isolation**. This MCP server has direct access to your entire laptop. It can read your files, access your network, and do whatever the hell it wants. Sure, you might trust the MCP server code, but do you trust all its dependencies? All their dependencies? It's a security nightmare waiting to happen.

Look, this might be the easiest approach, aside from needing the right runtime installed. It might be what's documented everywhere. But let's be honest: it's potentially the worst way to run MCP servers. You've got dependency management issues, process lifecycle problems, and zero isolation. There has to be a better way, right?

Let me show you some alternatives that address these problems.

> If using Claude Code, press `ctrl+c` twice to exit it. Otherwise, if using a different agent, exist through whichever means that agent should be shut down.

```sh
docker container rm qdrant --force
```

## MCP Docker Container Deployment

Now let's try a better approach: running MCP servers in Docker containers. Here's the thing: at the end of the day, an MCP server isn't that different from any other server. Sure, it uses stdio instead of HTTP for communication, but it's still just a server that needs to run somewhere. And how do we typically run servers locally these days? Docker! It gives us isolation, better dependency management, and we don't need to pollute our machines with various runtimes.

Let me show you how this works. The configuration is slightly different now. Instead of running NPX directly, we're telling Claude to use Docker Compose.

```sh
cat .mcp-docker.json
```

```json
{
  "mcpServers": {
    "dot-ai": {
      "command": "docker",
      "args": [
        "compose", 
        "-f",
        "docker-compose-dot-ai.yaml",
        "run", 
        "--rm",
        "--remove-orphans",
        "dot-ai"
      ]
    }
  }
}
```

See the difference? We're using `docker compose run` as the command. The Docker Compose file handles all the complexity: starting Qdrant, setting up networking between containers, managing volumes, all that stuff. The `--rm` flag ensures containers are cleaned up when we're done, and `--remove-orphans` takes care of any leftover containers from previous runs.

Now let's fire up Claude with this Docker-based configuration.

```sh
claude --mcp-config .mcp-docker.json
```

This time, let's test it with a different command. Instead of listing patterns, let's ask for the Kubernetes capabilities that the MCP server has discovered.

[user]
```md
List capabilities
```

[agent]
```md
⏺ Your Kubernetes cluster has 344 discovered capabilities. Here are the first 10:

  1. ID: 014e4600-f3c1-c91d-b0b0-04242927cbfd
    - Resource: clusterusages.protection.crossplane.io
    - Capabilities: resource protection, deletion blocking, dependency management
    - Description: Crossplane resource for deletion blocking relationships
  2. ID: 0188da80-6f2e-ff3f-273e-004bc50c9eed
    - Resource: subscriptions.azure.m.upbound.io
    - Capabilities: Azure subscription management, billing, tenant management
    - Description: Manages Azure subscriptions with billing scope configuration
  3. ID: 03a6109b-a52e-19f3-13ab-633f36b9c1d4
    - Resource: providerconfigs.aws.m.upbound.io
    - Capabilities: AWS provider, credential management, IAM roles, authentication
    - Description: Crossplane AWS provider configuration for authentication
  4. ID: 03f61362-8904-f64c-6110-e0acf759278e
    - Resource: pullrequests.repo.github.upbound.io
    - Capabilities: Pull request management, GitHub, code review
    - Description: Manages GitHub pull requests through Crossplane
  5. ID: 04dc1a16-dd38-061e-b7c0-383c168bd6be
    - Resource: networks.network.upcloud.com
    - Capabilities: Private networking, SDN, VPC, subnet, DHCP, routing
    - Description: UpCloud SDN private network resource
  6. ID: 054993e1-e0fd-96ad-6b66-077f7c1468e3
    - Resource: ReplicaSet
    - Capabilities: Pod replication, scaling, high availability
    - Description: Ensures specified number of pod replicas are running
  7. ID: 06eb2cc6-bd3f-6b04-7dc8-7e00a1a8289b
    - Resource: teammemberships.team.github.upbound.io
    - Capabilities: GitHub team membership, user management, access control
    - Description: Manages GitHub team membership with role assignments
  8. ID: 06f2c5b3-3c62-8e40-ad2b-8f0b9c114cdf
    - Resource: defaultroutetables.ec2.aws.m.upbound.io
    - Capabilities: Route table, VPC routing, AWS EC2 networking
    - Description: Manages default route table of AWS VPC
  9. ID: 0710fc19-88b2-f489-583f-5a6a3441ab7d
    - Resource: runnergroups.actions.github.upbound.io
    - Capabilities: GitHub Actions, runner groups, CI/CD, workflow management
    - Description: Manages GitHub Actions Runner Groups
  10. ID: 0a40069f-b501-5786-6cf1-12393bed31c3
    - Resource: managedobjectstorageuseraccesskeys.objectstorage.upcloud.com
    - Capabilities: Object storage, access keys, S3 compatible, credentials
    - Description: Manages access keys for UpCloud Object Storage

  To view more capabilities or get details on a specific one, let me know!
```

Excellent! The Docker-based MCP server is working perfectly. It discovered 344 capabilities from our Kubernetes cluster. Notice how everything just works without us having to manage Qdrant separately or worry about process lifecycles. Docker Compose handles all of that for us.

So what have we gained with this Docker approach? First and foremost, we've got proper isolation. The MCP server runs in its container with controlled access to resources. Unless you do something silly like mounting your entire filesystem or running containers in privileged mode, you're much safer than with the direct local approach.

Everything runs in containers, which means you don't need to install Node, NPX, Python, or whatever runtime the MCP needs. Just Docker, and you're good to go. All the dependencies are bundled together in the compose file. Qdrant, the MCP server, networking between them: it's all defined in one place and managed as a unit.

But here's the thing: it's still running locally. This is still a single-user setup on your machine. Now, you might be thinking, "Can't I just run Docker on a remote server?" Sure, you could expose Docker's API over the network, but that opens up a whole can of security worms. You'd need to manage TLS certificates, authentication, network access. It gets complicated fast, and you're basically reinventing infrastructure that already exists.

There's a better way to go truly remote, with proper multi-user support, high availability, and all the enterprise features you might need. Let me show you what that looks like.

> If using Claude Code, press `ctrl+c` twice to exit it. Otherwise, if using a different agent, exist through whichever means that agent should be shut down.

```sh
docker container rm qdrant --force

docker volume rm qdrant-data
```

## MCP Kubernetes Production Deployment

Time to go truly remote. Here's where we make a fundamental shift. Think about it: running MCP servers locally is like local development. Everyone spins up their own instance, manages their own dependencies, deals with their own problems. But what if we could run MCP servers like production services? Deploy them once, properly, and let the entire team or company connect to them. That's exactly what Kubernetes gives us.

This isn't just about containerization anymore. It's about turning MCP servers into shared organizational infrastructure. Instead of every developer running their own instances of various MCP servers, we deploy them once to Kubernetes and everyone connects to the same, properly managed services. Whether it's my dot-ai server, or your custom MCP for internal tools, or that third-party MCP for cloud resources, the approach is the same.

Let's deploy an MCP server to Kubernetes using Helm. I'm using my dot-ai server as the example, but everything you see here applies to any MCP server you want to run in production.

```sh
helm install dot-ai-mcp oci://ghcr.io/vfarcic/dot-ai/charts/dot-ai:0.85.0 \
  --set secrets.anthropic.apiKey="$ANTHROPIC_API_KEY" \
  --set secrets.openai.apiKey="$OPENAI_API_KEY" \
  --set ingress.enabled=true \
  --set ingress.host="dot-ai.127.0.0.1.nip.io" \
  --create-namespace \
  --namespace dot-ai \
  --wait
```

> We are using `--set` to create the Secret with API credentials. Don't do that in production. Manage secrets with [External Secrets Operator (ESO)](https://external-secrets.io/) or whichever other secrets manager you prefer.

```sh
kubectl --namespace dot-ai get all,ingresses
```

Notice something important here? I'm running these commands, not Claude. This is a fundamental shift. In the previous examples, the agent was responsible for spinning up MCP servers when it started and shutting them down when it stopped. The configuration told the agent how to launch the server.

Now, we've separated those responsibilities. A human, or GitOps, or your CI/CD pipeline deploys the MCP servers to Kubernetes, just like any other production service. The agents just connect to them. They don't manage their lifecycle anymore.

Let's see what Kubernetes created for us.

```
NAME                            READY STATUS  RESTARTS AGE
pod/dot-ai-mcp-6555cd65f6-x75ph 1/1   Running 0        19s
pod/dot-ai-mcp-qdrant-0         1/1   Running 0        19s

NAME                               TYPE      CLUSTER-IP    EXTERNAL-IP PORT(S)                      AGE
service/dot-ai-mcp                 ClusterIP 10.96.17.101  <none>      3456/TCP                     19s
service/dot-ai-mcp-qdrant          ClusterIP 10.96.187.200 <none>      6333/TCP,6334/TCP,6335/TCP   19s
service/dot-ai-mcp-qdrant-headless ClusterIP None          <none>      6333/TCP,6334/TCP,6335/TCP   19s

NAME                       READY UP-TO-DATE AVAILABLE AGE
deployment.apps/dot-ai-mcp 1/1   1          1         19s

NAME                                  DESIRED CURRENT READY AGE
replicaset.apps/dot-ai-mcp-6555cd65f6 1       1       1     19s

NAME                               READY AGE
statefulset.apps/dot-ai-mcp-qdrant 1/1   19s

NAME                                 CLASS HOSTS                   ADDRESS PORTS AGE
ingress.networking.k8s.io/dot-ai-mcp nginx dot-ai.127.0.0.1.nip.io         80    19s
```

Perfect! We've got the MCP server running as a deployment, Qdrant as a statefulset for persistent storage, services for internal communication, and an ingress exposing it all at `dot-ai.127.0.0.1.nip.io`. This is proper production infrastructure, running independently of any agent.

Now let's look at how agents connect to this remote MCP server.

```sh
cat .mcp-kubernetes.json
```

```json
{
  "mcpServers": {
    "dot-ai": {
      "type": "http",
      "url": "http://dot-ai.127.0.0.1.nip.io"
    }
  }
}
```

Look at this configuration carefully. We're not telling Claude to run the MCP server anymore. We're telling it to connect directly to the already-running MCP server at that URL using HTTP transport.

The `type: "http"` tells Claude to use HTTP transport to communicate with the remote MCP server, and the `url` points to our Kubernetes ingress endpoint. This allows Claude to communicate directly with the MCP server running in Kubernetes over HTTP.

This is a clean, direct connection without any local bridge processes or protocol translation. The agent speaks HTTP directly to the remote MCP server.

> Use HTTPS with certificates in production instead of HTTP.

Let's connect Claude to our remote MCP server and check its status to make sure everything's working.

```sh
claude --mcp-config .mcp-kubernetes.json
```

[user]
```md
Show dot-ai status
```

[agent]
```md
⏺ Dot-AI System Status ✅

  Version: 0.85.0Overall Status: Healthy

  Core Services
                                                                                                                                                                    - Kubernetes: Connected (v1.33.1)
  - Vector DB (Qdrant): Connected at cluster endpoint
  - AI Services: Anthropic API connected
  - Embeddings: OpenAI text-embedding-3-small available
  - Kyverno: v1.15.1 installed and ready

  Data Collections

  - Patterns: Active (0 documents)
  - Policies: Not initialized
  - Capabilities: Not initialized

  Available Features

  ✓ Pattern management✓ Capability scanning✓ Semantic search✓ AI recommendations✓ Kubernetes integration✓ Policy generation

  The system is fully operational and ready to use.
```

Excellent! We're connected to the remote MCP server running in Kubernetes. Notice it shows the Kubernetes version, Qdrant is connected, all the services are healthy. This is the same MCP server we deployed, but now multiple users can connect to it simultaneously.

So what have we achieved with this Kubernetes approach? We've got **real isolation** through Kubernetes namespaces, RBAC, and network policies. The MCP server can't access your laptop anymore. It's confined to its namespace with only the permissions you explicitly grant it.

Everything still runs in containers, so there's nothing special to install on developer machines. But now we also get all the Kubernetes goodies: high availability if you want it, auto-scaling, security policies, audit logs, the whole enterprise package.

Most importantly, this is truly **remote and multi-user**. You deploy the MCP server once, and your entire team connects to it. Everyone shares the same patterns, the same capabilities, the same configuration. It's like the difference between everyone running their own database locally versus connecting to a shared production database.

Yes, the setup is more complex than local deployment, but that's the nature of production infrastructure. You need a Kubernetes cluster, Helm charts, ingress controllers. But here's the thing: you do this setup once, and everyone benefits. It's infrastructure, not something each developer needs to figure out.

Still, there's another approach using Kubernetes operators that claims to simplify MCP server management. Let me show you what happens when you add ToolHive to the mix.

> If using Claude Code, press `ctrl+c` twice to exit it. Otherwise, if using a different agent, exist through whichever means that agent should be shut down.

```sh
helm delete dot-ai-mcp --namespace dot-ai
```

## MCP ToolHive Kubernetes Operator

StackLok created an operator called ToolHive that promises to simplify MCP server management. The idea is that ToolHive manages MCP servers as Kubernetes custom resources with additional features like permission profiles and resource management. Let's see if it actually delivers on that promise.

ToolHive treats MCP servers as first-class Kubernetes citizens. Instead of deploying standard deployments and services, you create an MCPServer resource and the operator takes care of the rest. Or at least, that's the theory.

Let's deploy the same MCP server using ToolHive instead of standard Kubernetes resources. Notice the `deployment.method=toolhive` parameter in the Helm command.

```sh
helm install dot-ai-mcp oci://ghcr.io/vfarcic/dot-ai/charts/dot-ai:0.116.0 \
  --set deployment.method=toolhive \
  --set secrets.anthropic.apiKey="$ANTHROPIC_API_KEY" \
  --set secrets.openai.apiKey="$OPENAI_API_KEY" \
  --set ingress.enabled=true \
  --set ingress.host="dot-ai.127.0.0.1.nip.io" \
  --create-namespace \
  --namespace dot-ai \
  --wait

kubectl --namespace dot-ai get mcpserver dot-ai-mcp
```

```
NAME       STATUS  URL                                                       AGE
dot-ai-mcp Running http://mcp-dot-ai-mcp-proxy.dot-ai.svc.cluster.local:3456 4m13s
```

There's our MCPServer custom resource instead of a regular deployment. Let's dig deeper and see what this custom resource actually contains.

```sh
kubectl --namespace dot-ai get mcpserver dot-ai-mcp --output yaml
```

```yaml
apiVersion: toolhive.stacklok.dev/v1alpha1
kind: MCPServer
metadata:
  annotations:
    meta.helm.sh/release-name: dot-ai-mcp
    meta.helm.sh/release-namespace: dot-ai
  creationTimestamp: "2025-09-12T21:19:34Z"
  finalizers:
  - mcpserver.toolhive.stacklok.dev/finalizer
  generation: 2
  labels:
    app.kubernetes.io/instance: dot-ai-mcp
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: dot-ai
    app.kubernetes.io/version: 0.83.0
    helm.sh/chart: dot-ai-0.83.0
  name: dot-ai-mcp
  namespace: dot-ai
  resourceVersion: "22307"
  uid: 0f072076-82a3-4338-a245-ab9b4dd04a55
spec:
  image: ghcr.io/vfarcic/dot-ai:0.83.0
  permissionProfile:
    name: network
    type: builtin
  podTemplateSpec:
    metadata: {}
    spec:
      containers:
      - env:
        - name: TRANSPORT_TYPE
          value: http
        - name: PORT
          value: "3456"
        - name: HOST
          value: 0.0.0.0
        - name: SESSION_MODE
          value: stateless
        - name: DOT_AI_SESSION_DIR
          value: /tmp/sessions
        - name: ANTHROPIC_API_KEY
          valueFrom:
            secretKeyRef:
              key: anthropic-api-key
              name: dot-ai-secrets
        - name: OPENAI_API_KEY
          valueFrom:
            secretKeyRef:
              key: openai-api-key
              name: dot-ai-secrets
        - name: QDRANT_URL
          value: http://dot-ai-mcp-qdrant:6333
        image: ghcr.io/vfarcic/dot-ai:0.83.0
        imagePullPolicy: IfNotPresent
        name: mcp
        resources:
          limits:
            cpu: "1"
            memory: 2Gi
          requests:
            cpu: 200m
            memory: 512Mi
        volumeMounts:
        - mountPath: /tmp/sessions
          name: sessions
      serviceAccountName: dot-ai-mcp
      volumes:
      - emptyDir: {}
        name: sessions
  port: 3456
  proxyMode: sse
  resources:
    limits:
      cpu: 200m
      memory: 256Mi
    requests:
      cpu: 100m
      memory: 128Mi
  secrets:
  - key: anthropic-api-key
    name: dot-ai-secrets
  - key: openai-api-key
    name: dot-ai-secrets
  targetPort: 3456
  transport: streamable-http
status:
  message: MCP server is running
  phase: Running
  url: http://mcp-dot-ai-mcp-proxy.dot-ai.svc.cluster.local:3456
```

That's a lot of YAML. The key things to notice: it's got the container spec embedded in `podTemplateSpec`, it specifies `transport: streamable-http`, and in the status you can see that proxy URL. ToolHive took our MCPServer definition and created the necessary pods and services. 

But wait, notice the `proxyMode: sse`? That's Server-Sent Events, and here's the problem: SSE transport is deprecated in the MCP protocol as of November 2024. MCP moved to Streamable HTTP instead. So ToolHive is using a deprecated transport mode. Not exactly filling me with confidence about this operator's future.

Now let's see all the resources that were created, both by ToolHive and by our Helm chart.

```sh
kubectl --namespace dot-ai get all,ingresses
```

```
NAME                            READY STATUS  RESTARTS AGE
pod/dot-ai-mcp-0                1/1   Running 0        75s
pod/dot-ai-mcp-6bc8d5df55-k6qmm 1/1   Running 0        88s
pod/dot-ai-mcp-qdrant-0         1/1   Running 0        88s

NAME                               TYPE      CLUSTER-IP    EXTERNAL-IP PORT(S)                    AGE
service/dot-ai-mcp-qdrant          ClusterIP 10.96.17.38   <none>      6333/TCP,6334/TCP,6335/TCP 88s
service/dot-ai-mcp-qdrant-headless ClusterIP None          <none>      6333/TCP,6334/TCP,6335/TCP 88s
service/mcp-dot-ai-mcp-headless    ClusterIP None          <none>      3456/TCP                   84s
service/mcp-dot-ai-mcp-proxy       ClusterIP 10.96.203.236 <none>      3456/TCP                   88s

NAME                       READY UP-TO-DATE AVAILABLE AGE
deployment.apps/dot-ai-mcp 1/1   1          1         88s

NAME                                  DESIRED CURRENT READY AGE
replicaset.apps/dot-ai-mcp-6bc8d5df55 1       1       1     88s
replicaset.apps/dot-ai-mcp-888c64f68  0       0       0     87s

NAME                               READY AGE
statefulset.apps/dot-ai-mcp        1/1   84s
statefulset.apps/dot-ai-mcp-qdrant 1/1   88s

NAME                                 CLASS HOSTS                   ADDRESS   PORTS AGE
ingress.networking.k8s.io/dot-ai-mcp nginx dot-ai.127.0.0.1.nip.io localhost 80    88s
```

Look at all these resources. The Helm chart created the MCPServer custom resource, Qdrant, the ingress, and other supporting resources. Then ToolHive operator saw that MCPServer resource and created additional pods and services like dot-ai-mcp-0 and mcp-dot-ai-mcp-proxy. So we've got resources created by Helm, and resources created by ToolHive based on what Helm created. It's layers upon layers of abstraction.

> We are using `--set` to create the Secret with API credentials. Don't do that in production. Manage secrets with [External Secrets Operator (ESO)](https://external-secrets.io/) or whichever other secrets manager you prefer.

Now here's the interesting part. Let's see how the client connects to this ToolHive-managed MCP server and whether it's any different from our standard Kubernetes approach.

```sh
cat .mcp-kubernetes.json
```

```json
{
  "mcpServers": {
    "dot-ai": {
      "type": "http",
      "url": "http://dot-ai.127.0.0.1.nip.io"
    }
  }
}
```

Notice that the configuration is identical to our standard Kubernetes deployment. We're using HTTP transport to connect directly to the MCP server.

ToolHive's proxy supports both stdio and HTTP transports, but HTTP is clearly the better choice. According to [ToolHive's own performance testing](https://dev.to/stacklok/performance-testing-mcp-servers-in-kubernetes-transport-choice-is-the-make-or-break-decision-for-1ffb), stdio transport has severe performance limitations. In their tests with 50 concurrent requests, **only 2 succeeded**! The stdio implementation is unsuitable for production use.

By using HTTP transport directly, we get better performance and eliminate the complexity of stdio-to-HTTP translation that would otherwise be needed.

> Use HTTPS with certificates in production instead of HTTP.

Let's connect to the MCP server anyway, just to verify it works.

```sh
claude --mcp-config .mcp-kubernetes.json
```

[user]
```md
Show dot-ai status
```

[agent]
```md
⏺ dot-ai Status ✅

  System Health: Healthy

  Core Components

  - Version: 0.83.0
  - Platform: Linux arm64 (Node v20.19.5)
  - Kubernetes: Connected to cluster (v1.33.1)
  - Vector DB: Connected to Qdrant at http://dot-ai-mcp-qdrant:6333
  - AI Services:
    - Anthropic API: Connected ✓
    - OpenAI Embeddings: Available (text-embedding-3-small)
  - Kyverno: v1.15.1 installed and ready

  Collections Status

  - Patterns: Empty (0 documents)
  - Capabilities: Empty (0 documents)
  - Policies: Not initialized

  Available Features

  - Pattern management (semantic + keyword search)
  - Capability scanning
  - AI recommendations
  - Kubernetes integration
  - Policy generation

  All systems operational and ready for use.
```

Okay, it works. The MCP server is running and accessible. But let's be honest about what we actually achieved here.

The end result is almost the same as when using standard Kubernetes resources, except now some of them are created by the operator. We still need the ingress, we still need to manage secrets, we still need supporting resources. The ToolHive custom resource didn't eliminate complexity; it just moved it around. That's why I had to wrap everything in a Helm chart anyway.

So what's the theoretical advantage of ToolHive? The main selling point was supposed to be better MCP server lifecycle management through Kubernetes operators. But in practice, it's just another layer of abstraction that doesn't solve the fundamental deployment challenges. The stdio transport it supports has catastrophic performance issues, which is why we're using HTTP transport anyway.

At the end of the day, running MCP servers is just like running any other HTTP server. You deploy them, expose them through an ingress, and connect to them over HTTP. ToolHive adds a custom resource abstraction on top, but I'm honestly not seeing the value. It's using deprecated SSE mode, and it adds another layer of complexity without clear benefits.

If you prefer using operators and custom resources, sure, ToolHive is an option. But given its limitations and the fact that it doesn't actually deliver on its main promise, I'd stick with standard Kubernetes deployments. At least those are straightforward and don't pretend to solve problems they can't actually solve.

Let me show you a few other deployment options before we wrap this up.

> If using Claude Code, press `ctrl+c` twice to exit it. Otherwise, if using a different agent, exist through whichever means that agent should be shut down.

```sh
helm delete dot-ai-mcp --namespace dot-ai
```

## Alternative MCP Deployment Options

Before we wrap up, let me quickly run through some other ways to deploy MCP servers. There's a whole ecosystem emerging around MCP deployment, and while I've focused on Kubernetes because it's vendor-agnostic and production-ready, you should know what else is out there.

[Fly.io](https://fly.io/docs/blueprints/remote-mcp-servers/) takes an interesting approach. They run MCP servers as tightly isolated VMs, which they call Fly Machines. You can deploy with a simple `fly mcp launch` command, and they handle authentication and routing for you. They support both single-tenant (each user gets their own app) and multi-tenant patterns. It's pretty slick if you're already in their ecosystem.

[Cloudflare Workers](https://blog.cloudflare.com/remote-model-context-protocol-servers-mcp/) went all-in on MCP. They provide OAuth authentication out of the box, zero egress fees, and CPU-based billing that's perfect for streaming connections. You can deploy MCP servers as edge functions that run close to your users. Their `workers-mcp` tooling handles the protocol translation for you. If you're looking for truly serverless MCP, this is probably your best bet.

[AWS Lambda](https://awslabs.github.io/mcp/servers/aws-serverless-mcp-server/) offers the AWS Serverless MCP Server. It works, technically, but the developer experience is rough. Cold starts are painful, and the stdio transport has serious performance issues on Lambda.

[Vercel](https://vercel.com/docs/mcp/vercel-mcp) lets you add MCP endpoints directly to your Next.js apps using the `mcp-handler` package. If you already have a Next.js application on Vercel, this is the path of least resistance. But watch out for their egress charges and memory-based billing for idle connections.

[Railway](https://railway.com) keeps things simple. It's a deployment platform that just works, without needing platform engineers. Deploy your MCP server like any other app. Nothing fancy, but sometimes that's exactly what you need.

[Podman MCP Server](https://github.com/manusa/podman-mcp-server) deserves a mention, though it's not really a deployment solution. It's an MCP server that lets AI agents manage Podman containers on your local machine. Think of it like the Docker MCP server but for Podman users. Still local, still has all the same limitations we discussed earlier.

Here's the thing about all these alternatives: they each have their niche. Fly.io is great for multi-tenant isolation. Cloudflare excels at edge deployment with minimal latency. AWS Lambda... well, it exists. Vercel makes sense if you're already there.

But they all have one problem: vendor lock-in. You pick Cloudflare, you're stuck with Cloudflare. You pick AWS, you're stuck with AWS. And most of them are still figuring out MCP. The implementations are evolving, the performance varies wildly, and the developer experience ranges from decent to painful.

That's why I keep coming back to Kubernetes. It's vendor-agnostic. You can run it anywhere: AWS, Google Cloud, Azure, on-premises, or that server under your desk. The deployment patterns are mature, the tooling is solid, and you're not betting your infrastructure on a single vendor's interpretation of MCP.

Unless you're a small company with just a few apps, or you're already deeply committed to a specific cloud vendor, Kubernetes remains the most flexible option for production MCP servers. But hey, at least now you know what's out there. Choose what works for your situation, not what I tell you to use.

## Choosing the Right MCP Deployment

Alright, let's take a step back and look at what we've covered. We've gone through a journey of MCP server deployment options, from the simplest to the most complex, from local to remote, from vendor-specific to vendor-agnostic.

We started with **local NPX execution**. The simplest approach, sure, but it comes with zero isolation, dependency hell, and security nightmares. Your MCP server has full access to your machine, and managing dependencies becomes your personal problem. It's what most documentation shows you, but that doesn't make it right for production.

Then we moved to **Docker locally**. Better isolation, everything in containers, dependencies bundled together. But it's still a single-user setup on your machine. Fine for development, not so much for team collaboration.

Next came **Kubernetes with standard deployments**. This is where things got serious. Proper production infrastructure, multi-user access, high availability, all the enterprise features you'd expect. Deploy once, everyone connects. It's like the difference between running a database on your laptop versus having a proper database server.

We also tried **Kubernetes with the ToolHive operator**, which promised to simplify MCP server management but ended up adding complexity without clear benefits. It uses deprecated SSE mode and adds another layer of abstraction without solving fundamental deployment challenges.

Finally, we looked at **cloud platform options**. 

So which one should you actually use?

If you're **developing an MCP server itself**, Docker makes sense. You need to test locally, iterate quickly, and see immediate results. But let's be clear: this is for MCP server developers, not MCP server users.

For **any team or company setting**, the local options are ridiculous. Why would every developer spin up their own instance of the same MCP servers? That's like asking everyone to run their own Jira or Slack instance. You want shared services that everyone connects to. That means Kubernetes or one of the cloud platforms.

For **production workloads**, Kubernetes is the answer unless you have a damn good reason to avoid it. It's vendor-agnostic, battle-tested, and gives you all the operational capabilities you need. Deploy each MCP server once, and your entire organization can use it. That's how infrastructure should work.

If you're **already committed to a specific cloud vendor**, their native solutions might make sense. Got everything on Cloudflare? Use Workers. Deep in AWS? Maybe Lambda will work for you eventually. But understand you're trading flexibility for convenience.

The key insight here is that **MCP servers are just servers**. They're not special snowflakes. They need to be deployed, exposed, and accessed just like any other service. And just like you don't run your own copy of every microservice in your company, you shouldn't run your own copy of every MCP server. Share the infrastructure, share the costs, share the maintenance burden.

If you want to experiment with these approaches, especially the Kubernetes deployments I've been showing, check out my [DevOps AI Toolkit](https://github.com/vfarcic/dot-ai). It's the MCP server I've been using throughout these examples. Star the repo if you find it useful, open issues if something's not working, submit PRs if you want to contribute. 

The MCP ecosystem is still young, and we're all figuring this out together. The more we share what works and what doesn't, the better these deployment patterns will become. So don't just consume; contribute. Help make MCP deployment less painful for the next person who comes along.

## Destroy

```sh
./dot.nu destroy
```


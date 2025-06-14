
+++
title = 'Kubernetes AI: The Good, The Bad, and The Disappointing (kubectl-ai)'
date = 2025-06-16T15:00:00+00:00
draft = false
+++

I will make an assumption by saying that you work, in some capacity or another, with **Kubernetes** and that you are interested in making management of your clusters much easier, better, and faster with **AI**. If that's the case, I have a treat for you. We'll explore how to do just that. We'll take a look at an AI agent specialized in management of Kubernetes clusters. An agent that comes from the company that made Kubernetes. An agent that is open source. An agent that has the potential to be one of the most important tools in your toolbelt.

<!--more-->

{{< youtube gNpCDKH0BTA >}}

Today we'll see how you can use AI to manage resources in your Kubernetes clusters through an agent which should do a better job than any other agent mainly because it is specialized. Specialized tools tend to be better than generic tools within the area they are specialized at. Otherwise, we wouldn't be making them. Right?

Let's see.

## Setup

```sh
git clone https://github.com/vfarcic/crossplane-sql

cd crossplane-sql

git pull

git fetch

git switch kubectl-ai
```

> Make sure that Docker is up-and-running. We'll use it to create a KinD cluster.

> Watch [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell
```

> Watch [The Future of Shells with Nushell! Shell + Data + Programming Language](https://youtu.be/zoX_S6d-XU4) if you are not familiar with Nushell. Alternatively, you can inspect the `dot.nu` script and transform the instructions in it to Bash or ZShell if you prefer not to use that Nushell script.

```sh
chmod +x dot.nu

./dot.nu setup kubectl_ai

source .env

kubectl-ai
```

## Finding and Fixing Issues with Kubernetes AI

Right now we are in a `kubectl-ai` terminal session and can ask it anything about the Kuberentes cluster we are connected to.

We can, for example, check what the state of the cluster is.

> Execute the following prompt.

```
What is the state of the cluster?
```

As a result of that prompt, kubectl-ai retrieved all the nodes, and all the Pods in the `kube-system` Namesapce, and all the Pods in all Namespaces that are `running`. 

That last operation is silly. Why would it retrieve only `Ready` Pods if we asked him what the state of the cluster is. If anything, it should have returned everything but ready Pods in hope that it would found those that are failing or pending or in any other non-ready state.

Anyways... We'll ignore that, for now, and focus on the conclusion.

The output is as follows (truncated for brevity).

```
...
  Okay, I've checked the status of your Kubernetes cluster. Here's a summary:

  • Nodes: There is one node named  dot-control-plane  which is in the  Ready
  state. ✅
  • Control Plane: All essential control plane components in the  kube-system
  namespace (like  etcd ,  kube-apiserver ,  kube-controller-manager ,  kube-
  scheduler ,  coredns , and  kube-proxy ) are  Running  correctly. ✅
  • Pods: There are no pods in a failed or pending state. Two pods related to
  ingress-nginx  setup are marked as  Completed , which is normal for job-type
  pods. ✅

  Overall, your Kubernetes cluster appears to be healthy and operational. 👍
```

> AIs are non-determinstic; even if we execute the same prompt twice, the results are likely to differ. As such, if you follow along, you are likely to experience different results and might need to adapt instead of following my instructions blindly.

If found out that there is a single node called `dot-control-plane` and that it is `Ready`.

It figured out that the "stuff" that composes the `Control Plane` is `Running correctly`.

Hurray! Right? We'll... No!

If we did not pay attention to what it executed, we might be delighted to see that `there are no pods in a failed or pending state`, congratulate ourselves on yet another day without any incidents, and open a bottle of champagne to celebrate.

The problem with that statement is that it is a conclusion based on a wrong observation and that I know that there are issue in that cluster since I set it up with a few issues intentionally. I'm a mean guy who set a trap and kubectl-ai fell into it right away. I was hoping that would happen (because I'm evil), but I did not expect it to happen so soon.

Nevertheless, anyone working with AI knows that mistakes are common and that we should always verify claims, so let's do just that by asking it to `check the whole cluster`, in hope that `all` will make it work a bit harder to determine what's going on.

> Execute the following prompt.

```
Check the whole cluster.
```

The output is as follows (truncated for brevity).

```
...
  Okay, I've performed a deeper check of the cluster:

  1. Component Statuses:
    •  controller-manager : Healthy ✅
    •  scheduler : Healthy ✅
    •  etcd-0 : Healthy ✅
    (Note: The  kubectl get componentstatuses  command is deprecated and might
    be removed in future Kubernetes versions).
  2. Cluster Events Summary:
    • The cluster successfully initialized its node and control plane components
    around an hour ago.
    • Numerous applications and operators were deployed shortly after, including
    cert-manager ,  ingress-nginx ,  cnpg  (CloudNativePG),  atlas-operator ,
    crossplane  (with many providers and functions),  external-secrets ,  ack
    (AWS Controllers for Kubernetes), and  azureserviceoperator-system .
    • There were many warnings during the Crossplane package installation phase
    (related to package health, synchronization, and dependencies). While many
    seem to have resolved with "Successfully installed/configured" messages, the
    volume of initial warnings might indicate potential configuration
    sensitivities or transient issues during setup.
    • Some initial  FailedMount  warnings occurred for components like  ingress-
    nginx ,  external-secrets , and  azureserviceoperator , likely due to
    dependent resources (like secrets) not being ready immediately. These seem
    to have resolved as components came online.
    • 🚨 Key Issue: The most recent significant events (within the last few
    minutes) show  BackOff  warnings for the  ack-ec2-controller  and  ack-rds-
    controller  pods in the  ack-system  namespace. This indicates these
    controllers are repeatedly crashing and entering a crash loop (
    CrashLoopBackOff ).


  In summary: While the core Kubernetes components are healthy and most
  applications seem to have deployed, there's a critical issue with the AWS
  Controllers (ACK) for EC2 and RDS, which are currently unstable and crashing.
```

After a bit of work, it managed to get out of the silly trap I set for it. It figured out that there's something wrong and identified the `Key Issue`. There's something wrong with the `ack-ec2-controller` running in the `ack-system  namespace`.

That was useful. It's not ground breaking, but useful never the less. That's what was to be expected from an AI agent dedicated to Kubernetes.

Let's see whether it can `fix it`.

> Execute the following prompt.

```
Can you fix it for me?
```

After checking logs of different Pods, and retrieving a few Deployments, it came up with the instructions what we might need to do.

```
...
  To fix this, we need to ensure:

  1. An IAM OIDC provider exists for the cluster.
  2. An IAM role exists with the necessary permissions for RDS and EC2 operations.
  3. The trust policy of the IAM role allows the  ack-rds-controller  and  ack-ec2-
  controller  service accounts to assume it.
  4. The  ack-rds-controller  and  ack-ec2-controller  service accounts in the
  ack-system  namespace are annotated with the ARN of the IAM role.

  Since I don't have the capability to interact with AWS IAM or configure IRSA
  directly, I cannot fully fix this for you.

  However, I can guide you on the Kubernetes side:

  You need to annotate the respective service accounts ( ack-rds-controller  and
  ack-ec2-controller ) in the  ack-system  namespace with the ARN of the correctly
  configured IAM role.

  Example Annotation Command:

    kubectl annotate serviceaccount ack-rds-controller \\
        -n ack-system \\
        eks.amazonaws.com/role-arn=arn:aws:iam::<ACCOUNT_ID>:role/<IAM_ROLE_NAME> --
  overwrite

    kubectl annotate serviceaccount ack-ec2-controller \\
        -n ack-system \\
        eks.amazonaws.com/role-arn=arn:aws:iam::<ACCOUNT_ID>:role/<IAM_ROLE_NAME> --
  overwrite

  Replace  <ACCOUNT_ID>  and  <IAM_ROLE_NAME>  with your actual AWS Account ID and
  the name of the IAM role you've created for ACK.

  Once you have configured IRSA correctly and applied these annotations, the pods
  should be able to obtain credentials and start successfully.

  If you have already configured IRSA, please double-check that the OIDC provider,
  IAM role, trust policy, and service account annotations are all correct.
```

This is good and bad news wrapped situation.

The good news is that it did provide instructions how to fix the issue. As a matter of fact, I am pleasantly surprised that it figured out that the problem is not directly related only to Kubernetes. It understood that we need to set up AWS IRSA to fix it. It probably got that from logs, but I'll still give it credit.

The bad news is that it is an agent focused only to Kubernetes and, as such, incapable of performing operations outside a cluster. It does not have `the capability to interact with AWS`.

I guess that's okay. I did not expect a Kubernetes agent to do anything outside a cluster. Whether that's a good or a bad thing is something we'll talk about later.

Now, I still expect AI agents to clean up my s**t, instead of me copying and pasting commands, so let's see whether it can do what needs to be done if we give it the information it needs.

> Execute the following prompt.

```
Can I give you the account ID and IAM role name so that you fix it?
```

That should work. It claims that if we give it the `Account ID` and the `Role Name`, it will do what needs to be done.

That's awesome!

Let's try it out.

> Execute the following prompt.

```
AWS Account ID is 1234567890 and IAM role name is this-is-am-role.
```

Now we're talking!

It suggests that we should annotate the `serviceaccount` and waits for the confirmation to do just that.

The AWS account I provided is fake so it would ultimately fail. That's okay though. The important part is that it correctly identified how to fix the issue and is now working on fix it it. It's awesome.

> Press `ctrl+c` to stop fixing the issue.

## Observability with Kubernetes AI

Let's see whether it can correctly identify `what is running in that cluster?`

> Execute the following prompt.

```
What is running in that cluster?
```

While it's working, let me stress that I am not looking for a simple list of Pods, or Deployments, or Custom Resource, or something similar. I want it to explain what is really running in the cluster. I want to get information beyond what I would get by manually executing *kubectl get deployments*.

The output is as follows (truncated for brevity).

```
...
...
  Ingress:

  •  ingress-nginx : Ingress Nginx Controller (manages external access to
  services)

  Certificate Management:

  •  cert-manager : Cert-Manager (for automated certificate management),
  Cainjector, Webhook

  Secrets Management:

  •  external-secrets : External Secrets Operator, Cert Controller, Webhook (for
  syncing secrets from external stores)

  Database & DB Operators:

  •  cnpg-system : CloudNativePG Operator (PostgreSQL operator)
  •  atlas-operator : Atlas Operator (database schema management)

  Cloud Provider Controllers / Operators:
...
```

That's nice, even though it was to be expected. The agent executed a few queries, sent the outputs to Gemini, which responded with descriptions of each of the components running in the cluster.

How about creating resources?

## Create Resources with Kubernetes AI

What we saw so far is relatively primitive for an agent. We ask it a question, it goes to Gemini to get the commands what to do, it executes those commands, it goes back to Gemini to interpret the outputs of those commands, and it shows us the response.

I'd like to know whether it can do more than that. I'd like to know whether it can be a bit more proactive and, for example, ask questions when it needs more information from us.

Let's check that by telling it what we would like to `create a manifest with resources required to run an application.`

> Execute the following prompt.

```
Create a manifest with resources required to run an application.
```

The output is as follows (truncated for brevity).

```
...
  You can adapt this template by changing the image, ports, replicas, resource
  requests/limits, and adding other configurations like ConfigMaps, Secrets,
  Ingress, etc., as needed for your specific application.

    ---
    # Namespace: Provides a scope for names and isolates resources.
    apiVersion: v1
    kind: Namespace
    metadata:
      name: my-app-ns # Change this to your desired namespace name

    ---
    # Deployment: Manages stateless application Pods.
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-app-deployment      # Name of the deployment
      namespace: my-app-ns        # Deploy into the created namespace
      labels:
        app: my-app             # Label for grouping resources related to this app
    ...

  Explanation:

  • Namespace ( my-app-ns ): Creates a dedicated space for your application's
  resources.
  • Deployment ( my-app-deployment ): Ensures that 2 replicas (Pods) of your
  application container (using the  nginx:1.25-alpine  image as an example) are
  always running. It handles rolling updates if you change the image or
  configuration.
  • Service ( my-app-service ): Creates a stable internal IP address and DNS name
  ( my-app-service.my-app-ns.svc.cluster.local ) that other services within your
  cluster can use to access the application Pods on port 80.

  To use this:

  1. Replace placeholder values (like the  image ,  containerPort ,  namespace ,
  names, labels, replicas) with your actual application details.
  2. Add any other necessary resources (ConfigMaps, Secrets,
  PersistentVolumeClaims, Ingress, etc.).
  3. Save the content as a  .yaml  file (e.g.,  my-app-manifest.yaml ).
  4. You could then apply it using  kubectl apply -f my-app-manifest.yaml .

  Would you like me to try and create these resources in your cluster using this
  manifest as a base (with the example nginx image)? Or would you like to refine
  it further?
```

There is nothing wrong with that answer. We got a template we could use to create resources that compose an application, we got an explanation of what is what, and we are asked whether we would like to `create these resources` or we prefer to `refine it further`. The first option is obviously silly. Proposed resources are so generic that it would be pointless to create them as they are while the second is too vague.

That was a bit dissapointing. I expected it to ask us questions from the start. I expected it to ask us wich type of an application we would like to run, whether it should be exposed outside of the cluster, which image we would like to use, and so on and so forth. I expected it to know better than to output a generic example.

Let's fix that by instructing it to `create that manifest` and also to `ask me for the info` it might `need`. Also, we'll tell it `NOT` to `apply it` just yet.

> Execute the following prompt.

```
Let's create that manifest. Ask me for the info you need, assemble it, and save it as a file. Do NOT apply it.
```

The output is as follows.

```
...
  Okay, let's gather the details needed to create your manifest file.

  1. Namespace: What name would you like for the Kubernetes Namespace? (e.g.,
  production ,  staging ,  my-cool-app-ns )
  2. Application Name: What is the name of your application? This will be used for
  naming the Deployment and Service, and for labels. (e.g.,  frontend ,  user-
  service ,  data-processor )
  3. Container Image: What is the full name of the container image you want to
  deploy, including the tag? (e.g.,  nginx:latest ,  my-dockerhub-user/my-app:v1.0.
  1 ,  gcr.io/my-project/my-service:stable )
  4. Container Port: Which port does your application listen on inside the
  container? (e.g.,  80 ,  8080 ,  3000 )
  5. Replicas: How many replicas (instances) of your application Pod do you want
  to run? (e.g.,  1 ,  2 ,  3 )
  6. Service Port: Which port should the Service expose within the cluster? (This
  is often the same as the container port, e.g.,  80 )

  Once I have these details, I will assemble the manifest and save it to a file
  named  <your-app-name>-manifest.yaml .
```

What I expected is slightly different.

I expected it to ask each of those questions separately. Instead of outputting them all, it would be so much better if it started with the question `what name would you like for the Kubernetes Namespace?`, wait for input, then ask for a second question, wait for input, and so on and so forth. That would be much better.

If, on top of that, it offered a list of available Namespaces and asked us to pick one or create a new one, that would be amazing.

That was also okay. Nothing it did so far is great or specially impressive. None of the things we saw is bad either. It's all okay, including that output.

We'll try to make this and other situations much better later. For now, let's respond to the questions it asked.

> Execute the following prompt.

```
1. a-team, 2. my-app, 3. ghcr.io/vfarcic/silly-demo:v1.5.87, 4. 8080, 5. 3, 6. 8080
```

The output is as follows (truncated for brevity).

```
...
  Now, saving this content to  my-app-manifest.yaml :


  Running:
cat <<EOF > my-app-manifest.yaml
---
# Namespace: Provides a scope for names and isolates resources.
apiVersion: v1
kind: Namespace
metadata:
  name: a-team

---
# Deployment: Manages stateless application Pods.
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment      # Name of the deployment
  namespace: a-team        # Deploy into the created namespace
  labels:
    app: my-app             # Label for grouping resources related to this app
spec:
  replicas: 3                 # Number of desired Pod replicas
  ...
EOF


  Do you want to proceed ?
  1) Yes
  2) Yes, and don't ask me again
  3) No
  Enter your choice (number):
```

That was a bit better. Due to previous dissapointments, I expected it just output the manifest on the screen so, due to low expectations, I am now pleasantly surprised that it offers to save it to a file. It would be even better if it asked for the name of the file, but this is okay as well.

Let's confirm that we want it to proceed.

> Execute the following prompt.

```
1
```

The output is as follows (truncated for brevity).

```
...
  Done! The manifest containing the Namespace, Deployment, and Service for  my-app
  in the  a-team  namespace, using the image  ghcr.io/vfarcic/silly-demo:v1.5.87
  with 3 replicas listening on port 8080, has been saved to the file  my-app-
  manifest.yaml .

  You can inspect it using  cat my-app-manifest.yaml  or apply it to your cluster
  using  kubectl apply -f my-app-manifest.yaml  when you are ready.
```

Let's see whether it can do something slightly more complicated by writing a bit more elaborated prompt.

We'll ask it to `create a manifest based on one of the Crossplane compositions.` We'll instruct it to let us `choose which one to use as a numbered list`, to ask us `questions to collect data` it `might need`,  and to `save it as a file.`

> Execute the following prompt.

```
Create a manifest based on one of the Crossplane compositions. Let me choose which one to use as a numbered list. Once I choose the composition, ask me questions to collect data you might need to assemble the manifest that represents a Crossplane claim. Once you're finished, save it as a file.
```

The output is as follows.

```
...
  Let's list the compositions again, this time referencing the  SQL  kind:

  1.  aws-ack-postgresql  (Provider: AWS ACK)
  2.  aws-postgresql  (Provider: AWS Crossplane Provider)
  3.  azure-aso-postgresql  (Provider: Azure ASO)
  4.  azure-postgresql  (Provider: Azure Crossplane Provider)
  5.  google-postgresql  (Provider: GCP Crossplane Provider)
  6.  local-postgresql  (Provider: Local Kind Cluster)
  7.  upcloud-postgresql  (Provider: UpCloud Crossplane Provider)

  Please choose the number corresponding to the Composition you want to use.

  Once you've chosen, I'll need to inspect the corresponding Composite Resource
  Definition (XRD), which is  SQL.devopstoolkit.live , to understand the specific
  parameters (like  size ,  version , etc.) required for the claim manifest.
```

That makes sense. It did a great job by discovering the available compositions and present the choices.

Let's pick any of those.

> Execute the following prompt.

```
2
```

The output is as follows.

```
...
  Okay, you've selected Composition #2:  aws-postgresql .

  This Composition will create a PostgreSQL database using AWS RDS via the
  standard Crossplane AWS provider. To create the claim manifest (which is a  SQL
  resource, version  devopstoolkit.live/v1beta1 ), I need a few details from you.

  Please answer the following questions:

  1. Claim Name: What name do you want for this PostgreSQL database claim? (e.g.,
  my-prod-db ,  billing-db-claim )
  2. Namespace: In which namespace should this claim resource be created? (e.g.,
  default ,  a-team )
  3. Size: What size do you need:  small ,  medium , or  large ? (The default is
  small  if you skip this)
  4. Version: Which PostgreSQL engine version do you want? (e.g.,  16.2 ,  15.6 ).
  Please provide a version supported by AWS RDS in your target region.
  5. Region: (Optional) In which AWS Region should the database be created? (e.g.,
```

kubectl-ai continuously results in amotional cycles that range from this is okay to this is underwhelming.

It went to the old pattern of presenting all the questions and asking us to answer all of them at once, just as it did with the application. It would be so much better if it presented each question separately and, when applicable, if it provided choices like a numbered list.

Now, to be clear, many other agents would be making the same mistakes and would often not react in the way we want them to. Fortunately, we can fix that in most of the other agents, so I'm sure that we can fix that with this one, so let's give it a try.

## Memory and Model Context Protocol (MCP) with Kubernetes AI

There are two important things we can do to improve the performance of almost any AI agent. One is to make them memorize instructions and the other is usage of Model Context Protocol (MCP).

The ability to have some form of memory is important. It allows us to provide permanent instructions that an agent can follow so that we do not need to repeat ourselves over and over again. With memory, we could say something like "always ask me individual questions that will provide information you might need to create resources" or "never create resources directly; always save them as a file."

Given that most other agents can do it, I have no reason to suspect that this one can't, but let's ask it just to be on the safe side.

> Execute the following prompt.

```
Can you memorize instructions?
```

The output is as follows.

```
...
  Yes, I can remember instructions you give me within our current conversation
  session. My context window allows me to recall previous parts of our chat,
  including the details and steps you've asked me to follow.

  For example, I remember that:

  1. You want to create a manifest for a Crossplane claim.
  2. You chose Composition #2 ( aws-postgresql ).
  3. I need specific details from you to build that manifest (claim name,
  namespace, size, version, etc.).
  4. I should save the final manifest to a file and not apply it to the cluster.

  However, this memory is limited to our current active session. I won't remember
  the specifics of this conversation once it ends or after a significant amount of
  time has passed.

  Shall we continue with gathering the details for your  aws-postgresql  claim
  manifest? Just let me know the answers to the questions I asked previously!
```

The gist of the answer is that it can remember, but only `within our current conversation session`. That is just silly, and semi useless. We would have to never close the session or to save all the instructions in a file and copy&paste them into the prompt every time we start a new session.

Fortunately, there is no need for an agent to have a built-in mechanism for permanent memory. As a matter of fact, I do not like using built-in memory. It is, in my opinion, a good idea to use an MCP for that since we can hook multiple agents to the same memory MCP or we can share memorized instructions with other people in our organization by running the server remotely.

That answer also makes the next choice easy. I wasn't sure whether to try combining kubectl-ai with Context7 or with Taskmaster or with some other MCP. Given that it does not have built-in memory, the choice is easy. We'll try to hook it into an MCP that will provide it with permanent memory.

Let's see whether it `can` `use MCP servers`.

> Execute the following prompt.

```
Can you (kubectl-ai) use MCP servers?
```

The output is as follows.

```
...
  That's an interesting question! "MCP" can stand for a few things. Could you
  clarify what you mean by "MCP servers"?

  Common interpretations could be:

  1. Managed Control Planes: Like those provided by cloud providers (e.g., EKS,
  GKE, AKS). In this case, yes, absolutely. I interact with your Kubernetes
  cluster via  kubectl , which communicates with the Kubernetes API server.
  Whether that API server is part of a managed control plane or self-hosted
  doesn't change how  kubectl  (and therefore I) interact with it, as long as the
  API is standard Kubernetes.
  2. Multi-Cluster Platforms: Like Rancher, Anthos, OpenShift, etc. These
  platforms often manage multiple underlying Kubernetes clusters. I can interact
  with any individual Kubernetes cluster managed by such a platform, provided my
  kubectl  context is configured to point to that specific cluster's API server. I
  don't directly interact with the platform's management layer APIs themselves,
  but rather with the standard Kubernetes API of the clusters under their control.
  3. Something Else: If MCP refers to a different specific technology or product,
  please let me know!

  In short: As long as the "MCP server" exposes a standard Kubernetes API endpoint
  and my  kubectl  is configured to communicate with it, I can interact with the
  cluster it manages.
```

You dummy! You're an AI agent and given that most AI agents use MCP servers, they are aware that MCP does not stand for `Managed Control Plane` nor `Multi-Cluster Platforms`.

I should not be that harsh though. Those are terms that might have MCP as abbreviations and are used in the context of Kubernetes. Let's clarify.

> Execute the following prompt.

```
By MCP, I mean Model Context Protocol.
```

The output is as follows (truncated for brevity).

```
...
  Ah, thank you for the clarification! You're asking about the Model Context
  Protocol (MCP), which is a concept or proposed standard for how large language
  models like me could interact more effectively with external tools, APIs, and
  context management systems.

  While the goal of MCP is to standardize and improve how models interact with
  their environment and tools (like  kubectl  in my case), I don't directly "use"
  the Model Context Protocol as a specific, named protocol myself.

  Instead, I operate within the framework and constraints provided by Google's
  infrastructure. This framework handles:

  1. Receiving your prompts.
  2. Allowing me to generate text and decide when to use a tool.
  3. Providing me with a specific, predefined set of tools ( kubectl ,  bash ,
  scan_image_with_trivy ) accessible via specific function calls (like the
  default_api.kubectl(...)  I use).
  4. Executing those tool calls in a secure environment.
  5. Returning the results of those tool calls back to me so I can formulate a
  response.

  This interaction mechanism achieves some of the goals of what MCP aims to
  standardize (tool use, context handling), but it's implemented using Google's
  internal systems and APIs, not necessarily the specific "Model Context Protocol"
  specification itself (if one is formally defined and published).

  So, while I leverage a system for context management and tool interaction, I
  don't explicitly use something called the "Model Context Protocol". My
  interaction pattern is dictated by the platform I run on and the tools
  specifically made available to me.
```

Long story short... The answer is no, and I'm very dissapointed.

Let's talk about it. Let's discuss the pros and cons.

## Kubernetes AI Pros and Cons

kubectl-ai is great and dissapointing at the same time. I'll explain why I think so in a moment. First, let's go through pros and cons, starting with the parts I don't like.

**Cons:**
* Very basic
* No persistent memory
* No MCP support
* Only Gemini models

To begin with, the agent is **very basic**. It might not look like that since it is talking to Gemini model which is one of the most capable models right now. If we judge it only by the answers, it might be easy to conclude that it is great. However, we are not evaluating the model, but *kubectl-ai* which is an agent. As such, it could be better. It could be more proactive. It could be asking follow-up questions to get better context from us. It could be collecting and presenting more data. It could be doing many other things that more generic agents like the one in Cursor or Claude Code are doing. That, however, could be fixed if we could provide it additional instructions how to behave, which we can't; and that brings me to the second negative point.

There is **no persistent memory**. As such, we can make the agent do what we want it to do if we provide it with enough instructions. That part is not a problem. The problem is that those instructions would be gone the moment we close a session and we would need to go through the same exercise every time we start a new one. All agents I've been using lately have persistent memory so it would be hard to justify why this one doesn't. Still, that could be fixed as well with an MCP, which brings me to yet another negative point.

kubectl-ai **does not support MCP severs** which, in my opinion, are one of the most important improvements we got lately. They are crucial for any serious usage of AI and, just as persistent memory, all the agents I used lately support it; except kubectl-ai. That's a shame.

Finally, kubectl-ai **only supports Gemini models**, at least today. Now, to be clear, I see nothing wrong in Google releasing a Kubernetes agent that works only with Google models; if it would be close source. That's not the case though. Kubectl-ai is open source and given how easy is to hook different models to an agent, I expected it to support at least one other model from the get-go. That being said, I do believe that Gemini models are one of the best, if not the best models out there. That is not my complaint. Even if other models would be supported, I would still probably use Gemini. However, not everyone has a subscription to those models and it would be great if those would be able to benefit from kubectl-ai with whichever model they are using instead of being forced to use Gemini.

Let's switch to pros.

You might be dissapointed to discover that there is only one truly positive thing I can say about it, until you see that it is a big one.

**Pros:**
* Works as MCP

kubectl-ai works as **an MCP server**. That is huge. I already stated that MCP servers are very important piece of any agent and while this one does not support them, it can act as one. It can easily be the best MCP server for interacting with Kubernetes clusters, and that is great. The only problem with that is that it needs a model, and models cost money which you would be paying on top of whichever model you would be using when interacting with whichever other agent you might hook to kubectl-ai acting as an MCP server. So, it's great as an MCP server, but that greatness might not be great enough when compared with other MCPs that interact with Kubernetes cluster. As such, I wasn't sure whether to put that one as a pro or a con. Nevertheless, I chose to ignore the cost and say that it is great as an MCP server.

I truly believe that kubectl-ai would be great and that I would not have any serious complaints about it if it was released last year. That's not the case though. It was released in April 2025. Last year, our expectations from AI agents were very different. They were much less capable then they are today.

Today, we can get just as good or better results with other agents. If, for example, we did the same exercises with Claude Code, which is also a terminal-based agent, we would get better results even without the usage of persistent memory or MCPs. Moreover, not only that the outcomes would be better when working with Kubernetes alone, but we would also have the benefit of being able to use the same agent for other tasks that are not strictly related to Kubernetes.

So, the question is: "**Why choose kubectl-ai over Claude Code, or Cursor, or Windsurf**, or any other more generic agent **?**"

Normally, I would say that *kubectl-ai* is more specialized. It is focused on Kubernetes and, as such, it performs better when Kubernetes-related tasks are concerned. That would be an important advantage. I would be more than happy to use multiple agents, one of them being kubectl-ai when working with Kubernetes. However, as it stands now, it makes more sense to use Claude Code for everything, including Kubernetes-related tasks.

That is not to say that it is bad. If you work only with Kubernetes and you don't need MCP servers and you don't need persistent memory, kubectl-ai is a good choice. It's better than other agents focused on Kubernetes. The problem is that there are too many ifs.

On the bright side, kubrectl-ai is a very young project. I'm sure that all of the cons I mentioned will be fixed soon and when that happens, kubectl-ai will be a good choice, but only for those working only with Kubernetes. At some later date, the community behind it might come up with features that make it a better choice than more generic agents. If that happens, I will surely adopt it. Until then...

## Destroy

> Execute the commands that follow in the first terminal session.

```sh
quit

./dot.nu destroy

exit
```


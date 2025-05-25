
+++
title = 'Forget CLIs and GUIs: AI is the New Interface for Developer Platforms'
date = 2025-06-02T15:00:00+00:00
draft = false
+++

Most of the time, we, software engineers, are using AI to write code or get commands we might need to performs some operations. Today I want to do something different. I want to use AI to create user interface for an Internal Developer Platform.

Instead of telling users which button to click or which YAML to write or which command to execute, I want them to **interact with the platform through AI**.

<!--more-->

{{< youtube ApjnCa-a2xI >}}

Now, to be clear, I don't want them to ask common questions like "how to create an application in Kubernetes?" or "what the status of something is" or "how to delete some resources?". I want them to interact with the platform I built. I want AI to help them while being limited to the capabilities of that platform. I want AI to be very specific while still being very easy for anyone to use it.

In other words, I want to enable developers to **create, observe, and delete instances of services** available in the platform and I want to do that with AI. Moreover, I don't want them to have to write large prompts. I want to make it easy. It should be a no-brainer wizard-like experience.

To get there, I created APIs AI can interact with. I did that with Crossplane, but you should be able to accomplish a similar result using other tools.

It does not matter how you built the platform. What matters is that it is accessible through APIs. That part is important since, as you will see soon, that will enable the API to interact with the platform.

All in all, I already have an Internal Developer Platform accessible through different API endpoints and I want AI to be the interface developers use to interact with it.

We'll see how we can use it to create services, to observe the status and gather information about services running inside the platform, how to fix issues, and, finally, how to delete them.

We'll do all that from the perspective of a person that does not know anything about those services. From developer's perspective, all the details of the platform are irrelevant. They can, but do not need to know what's below the hood. All they have to know is that **AI will do everything** for them.

How does that sound?

## Setup

> Install [NodeJS](https://nodejs.org/en/download) if you don't have it already.

```sh
npm install -g @anthropic-ai/claude-code

git clone https://github.com/vfarcic/idp-full-demo

cd idp-full-demo

git fetch

git pull

git switch ai
```

> Watch [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell

chmod +x platform
```

> Make sure that Docker is up-and-running. We'll use it to create a KinD cluster.

> Watch [The Future of Shells with Nushell! Shell + Data + Programming Language](https://youtu.be/zoX_S6d-XU4) if you are not familiar with Nushell. Alternatively, you can inspect the `platform` script and transform the instructions in it to Bash or ZShell if you prefer not to use that Nushell script.

```sh
./platform setup ai

source .env
```

## AI to Create Platform Services

I will use Claude Code today and I will not go into details how it works for two reasons.

First, I already explored it in the [Claude Code: AI Agent for DevOps, SRE, and Platform Engineering](https://youtu.be/h-6LP133o6w) video, and I don't want to repeat myself.

Second, everything I'll show can be done with almost any AI agent. The results will differ and the process implementing what I did might vary. Still, the logic is not specific to Claude Code and should be applicable to any agent.

With that note out of the way, let's fire up Claude Code.

```sh
claude
```

Now, let's say that I am a developer who would like to create an instance of a service available in the platform.

One nice thing we can do in Claude Code are custom commands accesible through `/project`.

> Type the prompt that follows

```
/project:create-service
```

The moment we type `/project` we can see all the available custom commands. Given that the current task is to create a service, we'll select just that (`create-service`).

Now the AI gets to work and, while it's doing whatever its doing, let me say that I did not tell it which services are available in the platform, how they work, what their schema is, nor any other information specific to a service. I wanted it to work for any service available in the platform, and I did not want to spent time giving it detailed instructions. I want AI to **discover** what can be done, and that's what it's doing.

> AI is non-deterministic. If you are following along, you will likely see different results than those presented here. As a result, you might need to adapt and make choices that match what you see on the screen instead of blindly following the my instructions.

The output is as follows (truncated for brevity).

```
⏺ Now I'll present the available Custom Resources to the user:

  1. AppClaim - Application deployment
  2. SQLClaim - Database service (PostgreSQL)
  3. GitHubClaim - GitHub repository

  Please select the type of resource you want to create (enter the number):
```

After a while, AI discovered that in this specific platform, users can create an `Application`, a `Database`, and a `GitHub` repo and presented the user with numbered options through which they can select to create any of those.

Let's say that we would like to create a database, so all we have to do is type `2`.

As a result, AI gets to work to discover all the questions we might need to answer to get a database.

The output is as follows (truncated for brevity).

```
...
  📊 Please select a database provider for PostgreSQL:

  1. 🔶 AWS - Amazon Web Services PostgreSQL
  2. 💙 Azure - Microsoft Azure PostgreSQL
  3. 🟢 Google - Google Cloud PostgreSQL
  4. ⛵ Local Kubernetes - Local in-cluster PostgreSQL
  5. 🔵 UpCloud - UpCloud PostgreSQL

  Please select a provider by entering the corresponding number:
```

It figured out that a database can be created in `AWS`, `Azure`, `Google`, and `UpCloud` or inside the `Local Kubernetes` cluster.

If, for example, we would like to run it in `AWS`, all we have to do is type `1`, and AI will try to figure out the next question.

The output is as follows (truncated for brevity).

```
...
  🔍 Select a namespace for your database:

  1. a-team
  2. b-team

  Please select a namespace by entering the corresponding number:
...
  Now let's set a name for your database instance:

  🏷️ What name would you like to give to your PostgreSQL database?

  This will be used as the resource name in Kubernetes and will be part of the
  database connection details.
```

Next, we are asked to pick the Namespace where resources of the service will be created and the name we would like to give to the DB.

The next question is interesting.

The output is as follows (truncated for brevity).

```
...
  🌎 Select an AWS region for your PostgreSQL database:

  1. us-east-1 (N. Virginia) - Default
  2. us-east-2 (Ohio)
  3. us-west-1 (N. California)
  4. us-west-2 (Oregon)
  5. eu-west-1 (Ireland)
  6. eu-central-1 (Frankfurt)
  7. ap-northeast-1 (Tokyo)
  8. ap-southeast-1 (Singapore)
  9. skip (Will use default: us-east-1)

  Please select a region by entering the corresponding number:
```

Since we chose to use AWS, it retrieved the available regions from AWS itself, or from the documentation, or from somewhere else. It does not matter where it got the info it needs. What matters is that we are presented with the list of regions where that database server could run, and that list is specific to the choice we made earlier.

We can choose, for example, to run it in North Virginia by typing the option `1`.

The rest of the questions follows the same pattern. We are asked `specify the databases` we would like to `create inside` that `PostgreSQL` server. That is followed with the question whether we `would` `like to apply a database schema` and, if we say yes to that, `which database should the schema be applied to`, and the `SQL schema` itself. Finally, there is a question whether we `want to set up secrets management` to which we should answer with `no` since I was too lazy to configure secrets manager in AWS.

That's it when data gathering is concerned. AI now has all the information it needs to generate the manifest.

The output is as follows (truncated for brevity).

```
...
  Now, let's generate the manifest for your AWS PostgreSQL database.

  📄 Where would you like to save the manifest file?

  Please provide the path where you'd like to save the YAML file (e.g.,
  "./my-db.yaml"):
```

We are asked for `where would` we `like to save the manifest file?`. The example path `./my-db.yaml` sounds correct so we can type just that.

The output is as follows (truncated for brevity).

```
...
╭─────────────────────────────────────────────────────────────────────────────────────╮
│ Create file                                                                         │
│ ╭─────────────────────────────────────────────────────────────────────────────────╮ │
│ │ my-db.yaml                                                                      │ │
│ │                                                                                 │ │
│ │ apiVersion: devopstoolkit.live/v1alpha1                                         │ │
│ │ kind: SQLClaim                                                                  │ │
│ │ metadata:                                                                       │ │
│ │   name: my-db                                                                   │ │
│ │   namespace: a-team                                                             │ │
│ │ spec:                                                                           │ │
│ │   compositionSelector:                                                          │ │
│ │     matchLabels:                                                                │ │
│ │       db: postgresql                                                            │ │
│ │       provider: aws                                                             │ │
│ │   parameters:                                                                   │ │
│ │     region: us-east-1                                                           │ │
│ │     databases:                                                                  │ │
│ │       - db-01                                                                   │ │
│ │       - db-02                                                                   │ │
│ │     schemas:                                                                    │ │
│ │       - database: db-01                                                         │ │
│ │         sql: |                                                                  │ │
│ │           create table videos (                                                 │ │
│ │             id varchar(50) not null,                                            │ │
│ │             description text,                                                   │ │
│ │             primary key (id)                                                    │ │
│ │           );                                                                    │ │
│ │           create table comments (                                               │ │
│ │             id serial,                                                          │ │
│ │             video_id varchar(50) not null,                                      │ │
│ │             description text not null,                                          │ │
│ │             primary key (id),                                                   │ │
│ │             CONSTRAINT fk_videos FOREIGN KEY(video_id) REFERENCES videos(id)    │ │
│ │           );                                                                    │ │
│ ╰─────────────────────────────────────────────────────────────────────────────────╯ │
│ Do you want to create my-db.yaml?                                                   │
│ ❯ 1. Yes                                                                            │
│   2. Yes, and don't ask again this session (shift+tab)                              │
│   3. No, and tell Claude what to do differently (esc)                               │
╰─────────────────────────────────────────────────────────────────────────────────────╯
```

We can see the manifest it generated.

AI figured out what which services are available in the cluster. In this case, those services were created as Crossplane Compositions. After that it provided all the necessary information we might need to answer its questions, gathered all the necessary info, constructed the manifest, asked us how we want to name the file, and now it is asking us to confirm that we want to create it. Ofcourse we do.

Here comes the interesting part. It figured out that we might need a password for that database server, so it asks us `what password` we `would you like to set` and for the permission to `create` the file.

That's it. Right? It `created two files`, one that `contains the SQLClaim for your AWS PostgreSQL database` and the other with `the secret with` the `database password`.

There's more though. There's one more step left.

The output is as follows (truncated for brevity).

```
...
  🚀 Would you like to:

  1. Apply the manifests directly to the cluster
  2. Create a pull request with these manifests
  3. Just keep the files without applying or creating a PR

  Please select an option by entering the corresponding number:
```

We can choose to `apply the manifests directly to the cluster`, or to `create a pull request with these manifests`, or to `just keep the files` without doing anything else.

Not doing anything would be silly. Creating a PR would be the best option since that would allow Argo CD or Flux to synchronize it into the platform, but I was lazy with that one as well and did not set it up. So, the only sensible option is to choose to `apply the manifests directly`. 

Wasn't that amazing?

It think it was, but that's not all.

## AI to Observe Platform Services

Creating services is not enough. There are many other types of operations we might want to perform. We might, for example, want to see the status of the service. We can do that with yet another custom command.

> Type the prompt that follows

```
/project:observe-service
```

The output is as follows (truncated for brevity).

```
...
⏺ I found the following services in your cluster:

  1. 🔶 my-db - SQL Database (PostgreSQL on AWS) in namespace a-team

  Which service would you like to observe? Please enter the number:
```

After a bit of work, AI discovered that there is only one instance of our services running. That was to be expected since we created only one so far. So, the choice is easy.

The output is as follows (truncated for brevity).

```
⏺ 📊 Service Overview: my-db (PostgreSQL on AWS)

  Status: 🚧 Provisioning in progress

  Basic Information:
  - Name: my-db
  - Namespace: a-team
  - ID: my-db-id
  - Databases: db-01, db-02
  - Region: us-east-1
  - Status: Not Ready (Provisioning in progress)

  Infrastructure being provisioned:
  - ✅ VPC: Successfully created
  - ✅ Subnets: 3 subnets created
  - ✅ Internet Gateway: Successfully created
  - ❌ RDS Instance: Creation in progress
  - ❌ RDS Subnet Group: Waiting for subnet references
  - ❌ Database connections: Not established yet

  Issues detected:
  - The RDS subnet group is waiting for subnet references to be properly
  established
  - The RDS instance creation is failing because it can't find the DB subnet group
  - Database services are unable to connect to the infrastructure yet

  This is a typical provisioning state for a new AWS RDS instance. AWS
  infrastructure components are being created in the correct order, but the full
  process takes several minutes to complete.

  Would you like to see more detailed information about any specific aspect of
  this service?

  1. Infrastructure details - AWS resources being created
  2. Database schema - Details about the schema that will be applied
  3. Connection information - How applications will connect to the database
  4. Event timeline - Detailed sequence of events in the provisioning process
```

After a bit more work, AI came up with high-level information about the service. We can see that `provisioning` of the database server is `in progress`, we can see some `basic Information`, and which `infrastructure` is `being provisioned`.

Forther on, we can see that there are a few `issues` it `detected`, but we are assured that `is a typical provisioning state for a new AWS RDS instance`. It takes time until everything is up-and-running. There is no reason to panic, just yet.

We are even asked whether we `would you like to see more detailed information about any specific aspect of` that `service`.

We got both high-level information that is user-friendly and should not be confusing to developers, but also the ability to go deeper into one of the aspects of the service.

It's awesome!

Let's create an application while waiting for the database server to finish cooking. 

> Type the prompt that follows

```
/project:create-service
```

The output is as follows (truncated for brevity).

```
⏺ Now I'll present the available Custom Resources to the user:

  1. AppClaim - Application deployment
  2. SQLClaim - Database service (PostgreSQL)
  3. GitHubClaim - GitHub repository

  Please select the type of resource you want to create (enter the number):
```

We are again presented with the choice which service instance we want to create. This time it will be `AppClaim`.

I won't go into details of the process since it follows the same pattern, but with different outcome.

It asks us to `select a namespace`, type the `name`, and the `ID`, set `container image`, define the `port`, `enable auto-scaling`, set the `minimum` and the `maximum number of replicas`, type the `name of the database to connect to`, `specify the host address`, and whether we want to `enable a Git repository`.

At the end, just as we saw with the database, we get to choose where it will `save the manifest file`, we are shown the final manifest, and we can choose to `apply` it `directly to the cluster` or `create a pull request` so that Argo CD or Flux can sync it into the cluster.

## AI to Fix Issues with Platform Services

By now, the database might might be running. Let's check whether that is indeed the case.

> Type the prompt that follows

```
/project:observe-service
```

The output is as follows (truncated for brevity).

```
...
⏺ I found the following services in your cluster:

  1. 🔶 my-db - SQL Database (PostgreSQL on AWS) in namespace a-team
  2. 🚀 my-app - Application in namespace a-team

  Which service would you like to observe? Please enter the number:
```

Just as we did before, we'll select `1` to observe the status and gather information about the database we created earlier.

The output is as follows (truncated for brevity).

```
...
⏺ 📊 Service Overview: my-db (PostgreSQL on AWS)

  Status: 🚧 Provisioning in progress with issues

  Basic Information:
  - Name: my-db
  - Namespace: a-team
  - ID: my-db-id
  - Databases: db-01, db-02
  - Region: us-east-1
  - Status: Not Ready (Provisioning has encountered errors)

  Infrastructure status:
  - ✅ VPC: Successfully created
  - ✅ Subnets: 3 subnets created
  - ✅ Internet Gateway: Successfully created
  - ❌ RDS Instance: Failed to create (Password issues)
  - ❌ Database connections: Not established yet

  Critical Issues Detected:
  - ⚠️ Password Error: The RDS instance is failing to create with error "Invalid
  master password". This suggests the password format doesn't meet AWS RDS
  requirements.
  - ⚠️ Secret Naming Issue: The system is looking for a secret named "my-db-id"
  but you created "my-db-password".
  - ⚠️ Application Impact: Your new application "my-app" is trying to use this
  database but failing because of these issues.

  Possible Solutions:
  The AWS RDS PostgreSQL instance creation is failing due to password format
  requirements and secret naming mismatch.

  Would you like to see more detailed information about:

  1. Error Details - Specific error messages and logs
  2. Required Fixes - Steps to resolve the issues
  3. AWS RDS Requirements - Details about AWS password requirements
  4. Application Impact - How this is affecting your application
```

We can see that, this time, it discovered a "real" issue. It cannot find the `secret` with the database password. We can see what the `possible solutions` are, and we can get `more detailed information about` the `error details`, `required fixes`, `AWS RDS requirements`, and the `application impact`.

We can choose one of those options, or we can just tell it to `fix the DB issues`.

I won't bore by forcing you to watch me fixing the issue. I'm sure you get the point. We observed the status of a service that composed quite a few resources, detected that there is an issue, and instructed AI to fix it.

## AI to Fix Delete Platform Services

All that's left from the demo is to remove the services we created. We can do that with yet another custom command.

> Type the prompt that follows

```
/project:delete-service
```

The output is as follows (truncated for brevity).

```
...
⏺ Here are the custom resources you can delete:

  1. 🔶 my-db (SQLClaim) - PostgreSQL database in namespace a-team
  2. 🚀 my-app (AppClaim) - Application in namespace a-team

  Please select the resources you want to delete by entering their numbers (e.g.,
  1, 2, or 1,2 for multiple):
```

We can select which `custom resources` we want to delete. That can be the `database`, or the `application`, or both. Let's remove them all by typing `1,2`.

The output is as follows (truncated for brevity).

```
Would you like to see all the resources that will be removed when these custom
  resources are deleted?

  1. Yes - Show me the resources that will be removed
  2. No - Skip and proceed to deletion

  Please enter your choice (1 or 2):
```

Next, we are asked whether we want to `see all the resources that will be removed when these custom resources are deleted?` Why not?

The output is as follows (truncated for brevity).

```
...
⏺ Here are the resources that will be removed:

  For my-db (PostgreSQL database):
  - 🔶 SQLClaim: my-db in namespace a-team
  - 🔶 SQL: my-db-h7fqk
  - 📊 Databases: my-db-id-db-01, my-db-id-db-02
  - 🔌 Provider config: my-db-id
  - 🌐 AWS VPC: my-db-id
  - 🌐 AWS Subnets: my-db-id-a, my-db-id-b, my-db-id-c
  - 🌐 AWS Internet Gateway: my-db-id
  - 🌐 AWS RDS Subnet Group: my-db-id
  - 🔒 AWS Security Group: my-db-id
  - Various Provider Config Usages

  For my-app (Application):
  - 🚀 AppClaim: my-app in namespace a-team
  - 🚀 App: my-app-m4xsg
  - 📦 Deployment: my-app (with 2 replica pods)
  - 🔌 Service: my-app
  - 🌐 Ingress: my-app (pointing to silly-demo.127.0.0.1.nip.io)

  How would you like to delete these resources?

  1. Delete the manifests and create a pull request
  2. Delete directly using kubectl

  Please enter your choice (1 or 2):
```

At the very end, just as with creation of the resources, we can choose to `delete the manifests and create a pull request` or `delete directly using kubectl`. Let's go with the latter option.

The output is as follows (truncated for brevity).

```
...
  The deletion process is cascading through the Kubernetes resources. Crossplane
  will automatically clean up all the AWS resources that were created as part of
  these claims.

  To summarize what was deleted:
  - ✅ The PostgreSQL database (my-db) with all its AWS infrastructure
  - ✅ The application (my-app) with its deployment, service, and ingress

  Is there anything else you'd like me to help you with?
```

That's about it. Everything we did so far is gone!

Since we are done, for now, we can stop `claude` and, among other things, see how much it cost us so far.

> Press `ctrl+c` twice to stop `claude`

The output is as follows (truncated for brevity).

```
Total cost:            $0.2859
Total duration (API):  9m 45.1s
Total duration (wall): 41m 30.9s
Total code changes:    112 lines added, 0 lines removed
Tokens:                52.6k base input, 0 cache write, 0 cache read, 18.2k base output
```

We spent less than $0.30 and I can safely say that's a small price for what we accomplished. Moreover, today we are using Claude which is, arguably the most expensive AI available right now. If that's too high of a price, we can switch to a more budget-friendly option like one of those that come with monthly subscription.

Now comes the moment of truth. We'll take a quick look at how I did all that.

## How Did It All Happen

Making all that is easier than it seems. The first iteration, the one we saw today, took around one hour in total. All I had to do is create a few custom commands and store them in `.claude/commands` directory. Whatever is there is automatically accessible through `/project:` followed with the name of the file.

Here's one of those.

```sh
cat .claude/commands/create-service.md
```

The output is as follows.

```md
Create Kubernetes manifests:

Follow these general guidelines:

- Show on the screen only information related to user input (the questions and the options).
- When asking user question, ask the user one question at a time.
- Ask only questions based on the CRD schema and limit them to `metadata.name`, `metadata.namespace`, and `spec`.
- Take into the account previous choices when presenting questions.
- Include any additional information you think the user might find useful (e.g., the available regions of the provider, PostgreSQL versions available in the selected provider, etc.). Use your internal knowledge base for that (do not try to discover it from the cluster).
- If the input is not mandatory, instruct the user to type `skip` if it is text input or, if they are presented with options, to select the `skip` option.
- Check the Composition to see whether values for some field are restricted and need to be specific.
- When you ask for the Namespace, give the user the option to select one of the existing Namespaces. Include only Namespaces that contain the word `team`.
- Always use `spec.crossplane.compositionSelector.matchLabels` to select the Composition in the manifest you are generating.
- Do not include parts of the manifest that are optional and the user did not choose any value.
- Use Context7 MCP to find Crossplane information you might need.
- Make a plan how you are going to execute the steps.
- Ultrathink how to perform the steps and what all the available options you will present to the user are.
- Use colors and icons to make outputs easier to read.
- If asked to provide Ingress Class, detect available Ingress Classes and present a list. If there is only one Ingress Class, do not ask the user which one to use but populate the value automatically.
- Always present multiple choices as numbered lists.
- Always use full API to retrieve, get, or describe resources.
- Explain each choice given to the user in detail.

Follow these steps:

1. Discover all the Custom Resources a user can create in that Kubernetes cluster.
2. Limit it to CRDs with the API `devopstoolkit.live` and include only Claims. Those CRDs were created by Crossplane Compositions.
3. Output numbered list of Composite Resources a user can create and ask them to select one of them. Ensure that all Compositions for a given CRD (Composite Resource Definition) are presented.
4. Based on the selected Composite Resource, ask the user for information you might need to generate YAML manifest that can be used to create that resource. Ensure that the user can select any of the Compositions within the selected Configuration. Present all parameters (including sub-parameters) to the user.
5. After you gather all the information you might need, generate the manifest and ask the user for the path where to save it.
6. Generate the combination of labels that match those available in Compositions so that the correct one is selected.
7. If the user selected to work with `SQL`, after creating the manifest, ask the user for the password they would like to assign to that database. The password should be stored in the Secret manifest with the key `password`. If the user selected UpCloud provider, there is no need to create the secret.
8. If the user selected to work with `SQL`, create the Kubernetes secret with that password in the same Namespace. The name of the secret should be the same as the name of the Composite resource with the addition of `-password` suffix.
9. Ask the user whether they would like to create a pull request with manifests or apply those manifests directly to the cluster.
```

It might look complicated or overwhelming, only until we try to calculate how much code it would be required to create something like that.

It is actually straightforward. There are `general guidelines` like `Always present multiple choices as numbered lists` that apply to any of the steps, and then there are the `steps` themselves. It should `discover all the Custom Resources a user can create`, `limit` those to `CRDs with the API devopstoolkit.live`, `Output numbered list of Composite Resources`, and so on and so forth.

What makes it "special" is that it is fully dynamic. It would work with anything we put into that platform, as long as that something is discoverable through Kube API.

We have a similar custom command to observe services.

```sh
cat .claude/commands/observe-service.md
```

The output is as follows.

```md
Observe the status of Kubernetes resources:

Follow these general guidelines:

- When asking user question, ask the user one question at a time.
- Use Context7 MCP to find Crossplane information you might need.
- Make a plan how you are going to execute the steps.
- Ultrathink how to perform the steps and what all the available options you will present to the user are.
- Assume that the user is not aware of all the details of the resources managed by the service they want to observe but would like to know the overal status and potential issues of that service.
- Use colors and icons to make outputs easier to read.
- Always present multiple choices as numbered lists.
- Always use full API to retrieve, get, or describe resources.

Follow these steps:

1. Discover all the Custom Resources created in that Kubernetes cluster. Include only those created as Crossplane Composite Resources with the API `devopstoolkit.live` and only Claims. They respresent services users consume.
2. Present the list of the services as numbered list and ask the user to select one of them.
3. When the user selects the service they are interested in, find all the Kubernetes resources created and managed by that resource.
4. Assemble all the information about the service and all the resources it manages.
5. Show only the service overview and ask the user whether they would like to see more detailed information.
```

And another one to delete them.

```sh
cat .claude/commands/delete-service.md
```

The output is as follows.

```md
Delete Kubernetes services:

Follow these general guidelines:

- Show on the screen only information related to user input (the questions and the options).
- When asking user question, ask the user one question at a time.
- Take into the account previous choices when presenting questions.
- Use Context7 MCP to find Crossplane information you might need.
- Make a plan how you are going to execute the steps.
- Ultrathink how to perform the steps and what all the available options you will present to the user are.
- Use colors and icons to make outputs easier to read.
- Always present multiple choices as numbered lists.
- Always use full API to retrieve, get, or describe resources.

Follow these steps:

1. Discover all the Custom Resources currently running in Kubernetes cluster in any of the Namespaces.
2. Limit it to Custom Resources with the API `devopstoolkit.live` and include only Claims. Those CRDs were created by Crossplane Compositions.
3. Find manifests in the project that match those Custom Resources.
4. Offer the user numbered list of Custom Resources they can delete through those manifests. Allow the user to select multiple resources.
5. Offer the user to see all the resources that will be removed if those Custom Resources will be deleted. The user can choose to skip seeing those resources.
9. Ask the user whether they would like to delete the manifest and create a pull request or to delete it directly through `kubectl delete`.
```

Now, to be clear, those commands are far from being final. Those are only the first iteration that I did. This video is the first time I run any of them.

Just as with code, we need to iterate over custom commands and perfect them over time. Nevertheless, even after one hour of work, and without proper testing, the results are amazing. Or, at least, I think they are.

Still, nothing is perfect and we might want to spend a bit of time going through pros and cons.

## Pros and Cons of AI as Platform Interface

The biggest potential problem with using AI in general, and especially for user interface is the **non-deterministic nature of AI**. There is no guarantee that the same input will produce the same results. AI is not like automation which we expect to behave always in the same way. If you try to reproduce what I just did, you will likely not be able to follow along by repeating my instructions. Your outcome will be different.

On the other hand, if we used a script, there would be no flexibility. We could not say, in the middle of the process, to stop execution, modify something, deviate from the plan, tell it to correct some outcome, or many other things we can do with AI.

AI does not guarantee reproducible results, but it does provide a lot of flexibility. Which one of those is better will depend on your preferences.

If you do choose AI, the next question we should ask ourselves is whether the results will be satisfactory.

Is what I did perfect? It certainly isn't. It will never be perfect. Nothing is. However, with a few more iterations I am confident that it could be great. What I did can easily be clasified as not even good enough, but I am confident that it can reach the level of being great. Not perfect; just great.

There's room for improvement though.

Maybe all that should be wrapped into a CLI? Maybe it should be available through a Web UI? Maybe it should be a Backstage plugin? Maybe Scott will take it over and create one.

I feel that there should be an option to hide the output genarated by the AI executing the commands while trying to figure out what to do. I think that I should use MCP Memory server so that it remembers what it did in the past so that it can avoid discovering what's in the platform every single time. That should make it significantly faster.

There are many things missing for something like this to be great. For now, all I can say is that using AI to create interfaces users can use to interact with an Internal Developer Platform has **a lot of potential**, especially since we can choose when to follow the "plan" and when to deviate from it.

Right now, I need to get back and work on it so...

## Destroy

```sh
kubectl get managed
```

> Wait until all Managed resources are removed.

```sh
./platform destroy ai

git stash

git switch main
```


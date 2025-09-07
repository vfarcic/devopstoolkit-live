
+++
title = 'Why Kubernetes Discovery Sucks for AI (And How Vector DBs Fix It)'
date = 2025-09-08T15:00:00+00:00
draft = false
+++

Kubernetes API is **brilliant for execution**. Once you know exactly which resources to deploy and how they work together, it handles everything flawlessly. Controllers reconcile state, resources get created, and your applications and infrastructure run perfectly.

But it **sucks big time for discovery**. When you need to figure out what's actually possible in your cluster, the API becomes a complete nightmare. You're stuck sifting through hundreds of cryptically named resource types, hoping to stumble across something that might solve your problem.

Even AI struggles with this. Ask it to "create a PostgreSQL database with schema management in AWS," and it either gives you a complex multi-resource approach or completely misses the simple solution that actually exists. The problem isn't missing information. **The problem is that the information is impossible to find**.

Today, I'll show you how **semantic search with vector databases** can finally solve this discovery nightmare. By the end of this video, you'll see how AI can instantly find the perfect resources using natural language, even when the exact keywords don't match.

<!--more-->

{{< youtube MSNstHj4rmk >}}

## Setup

> This demo is using Claude Code as the coding agent. Please open an [issue in the dot-ai-demo](https://github.com/vfarcic/dot-ai-demo/issues) repo if you'd like me to extend it to other agents (e.g., Cursor, VS Copilot, etc.).

> The project we'll explore currently supports only Anthropic Sonnet models. Please open an [issue in the dot-ai](https://github.com/vfarcic/dot-ai/issues) repo if you'd like the support for other models or if you have any other feature request or a bug to report.

> Install [NodeJS](https://nodejs.org/en/download) if you don't have it already.

```sh
npm install -g @anthropic-ai/claude-code

git clone https://github.com/vfarcic/dot-ai-demo

cd dot-ai-demo

git pull

git fetch

git switch vector-db
```

> Make sure that Docker is up-and-running. We'll use it to create a KinD cluster.

> Watch [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell

./dot.nu setup

source .env
```

## Kubernetes API Discovery Nightmare

The Kubernetes API is brilliant at what it was designed to do. Its primary purpose is storing desired state so controllers can reconcile resources and maintain system health. Think of it as a sophisticated state storage system that excels at managing declarative configurations. However, this strength becomes a limitation when we need to discover what's possible rather than execute what we already know.

The Kubernetes API works perfectly when you already know what you need to do, but it's terrible at helping you discover what can be done. In our AI-driven era, this reveals a critical gap. The API lacks semantic search capabilities, making it nearly impossible to translate human intent into actionable resources.

The discovery problem becomes obvious when you try to figure out what's actually possible in your cluster. The API doesn't provide intuitive ways to explore capabilities, leaving you to guess and fumble through resource lists without context or guidance.

You might be thinking this is complete nonsense. After all, you can list all resource definitions and figure out what you need, right? Wrong. Completely wrong. The reality is far more brutal than most people realize.

Let's prove this with a real scenario. Imagine someone walks up to you demanding to "create a postgresql database instance with schema management in AWS". Think you can handle it without memorizing every single resource definition and schema in that cluster? Let's put that confidence to the test.

We'll start by listing all available resources to see what we're working with.

```sh
kubectl api-resources
```

The output is as follows (truncated for brevity).

```
NAME                                        SHORTNAMES APIVERSION                               NAMESPACED KIND
...
pods                                        po         v1                                       true       Pod
podtemplates                                           v1                                       true       PodTemplate
replicationcontrollers                      rc         v1                                       true       ReplicationController
resourcequotas                              quota      v1                                       true       ResourceQuota
secrets                                                v1                                       true       Secret
serviceaccounts                             sa         v1                                       true       ServiceAccount
services                                    svc        v1                                       true       Service
...
databases                                              dbforpostgresql.azure.upbound.io/v1beta1 false      Database
firewallrules                                          dbforpostgresql.azure.upbound.io/v1beta1 false      FirewallRule
flexibleserveractivedirectoryadministrators            dbforpostgresql.azure.upbound.io/v1beta1 false      FlexibleServerActiveDirectoryAdministrator
flexibleserverconfigurations                           dbforpostgresql.azure.upbound.io/v1beta1 false      FlexibleServerConfiguration
flexibleserverdatabases                                dbforpostgresql.azure.upbound.io/v1beta1 false      FlexibleServerDatabase
flexibleserverfirewallrules                            dbforpostgresql.azure.upbound.io/v1beta1 false      FlexibleServerFirewallRule
flexibleservers                                        dbforpostgresql.azure.upbound.io/v1beta2 false      FlexibleServer
...
```

The output reveals the scope of our problem. We're looking at hundreds of different resource types, each with cryptic names that give little indication of their actual purpose. Among these are standard Kubernetes resources like `pods` and `services`, but buried within are specialized resources like `flexibleserverdatabases` and `manageddatabasepostgresqls`. How the hell are we supposed to know which ones can help with PostgreSQL in AWS?

Let's get a count of exactly how many resources we're dealing with.

```sh
kubectl api-resources | wc -l
```

The output is as follows.

```
443
```

Four hundred and forty-three resources! That's not just overwhelming - it's completely impractical. No human can reasonably sift through hundreds of resource definitions to find what they need. This number could easily reach thousands in larger, more complex clusters.

Going through schemas for all these resources is completely impractical. That would take hours or even days of tedious documentation reading.

Memory isn't an option either. Nobody can remember hundreds of resource types, their purposes, and their relationships to each other.

Even if we somehow memorized every resource name, that still wouldn't tell us what each one actually does or how they work together.

So what's left? Search and filtering seem like the obvious solution. Let's see if we can narrow down the list by searching for database-related resources.

```sh
kubectl api-resources | grep database
```

The output is as follows.

```
manageddatabaselogicaldatabases                            database.upcloud.com/v1alpha1                          false        ManagedDatabaseLogicalDatabase
manageddatabasemysqls                                      database.upcloud.com/v1alpha1                          false        ManagedDatabaseMysql
manageddatabaseopensearches                                database.upcloud.com/v1alpha1                          false        ManagedDatabaseOpensearch
manageddatabasepostgresqls                                 database.upcloud.com/v1alpha1                          false        ManagedDatabasePostgresql
manageddatabaseredis                                       database.upcloud.com/v1alpha1                          false        ManagedDatabaseRedis
manageddatabaseusers                                       database.upcloud.com/v1alpha1                          false        ManagedDatabaseUser
databases                                                  dbforpostgresql.azure.upbound.io/v1beta1               false        Database
flexibleserverdatabases                                    dbforpostgresql.azure.upbound.io/v1beta1               false        FlexibleServerDatabase
databases                                                  mssql.sql.crossplane.io/v1alpha1                       false        Database
databases                                                  mysql.sql.crossplane.io/v1alpha1                       false        Database
databases                                                  postgresql.sql.crossplane.io/v1alpha1                  false        Database
databaseinstances                                          sql.gcp.upbound.io/v1beta2                             false        DatabaseInstance
databases                                                  sql.gcp.upbound.io/v1beta1                             false        Database
```

Here's where things get interesting. We can see several database-related resources, but none of them scream "PostgreSQL server in AWS." Some might be useful pieces of a larger puzzle, but none represent the main component we need. This gets worse when you realize my assessment is based on prior experience. Without that knowledge, you'd have to examine schemas and documentation for each resource only to discover that none of them solve your primary need.

Even more concerning: none of these are related to AWS. Does this mean there are no resource definitions that can help us with AWS? That seems unlikely in a cluster that should have AWS capabilities.

Maybe we're filtering wrong. The request is "create a postgresql database instance with schema management in AWS", and so far we've only used `database` to filter the API. Let's try a more specific search term.

```sh
kubectl api-resources | grep postgresql
```

The output is as follows.

```
manageddatabasepostgresqls                  database.upcloud.com/v1alpha1            false ManagedDatabasePostgresql
activedirectoryadministrators               dbforpostgresql.azure.upbound.io/v1beta1 false ActiveDirectoryAdministrator
configurations                              dbforpostgresql.azure.upbound.io/v1beta1 false Configuration
databases                                   dbforpostgresql.azure.upbound.io/v1beta1 false Database
firewallrules                               dbforpostgresql.azure.upbound.io/v1beta1 false FirewallRule
flexibleserveractivedirectoryadministrators dbforpostgresql.azure.upbound.io/v1beta1 false FlexibleServerActiveDirectoryAdministrator
flexibleserverconfigurations                dbforpostgresql.azure.upbound.io/v1beta1 false FlexibleServerConfiguration
flexibleserverdatabases                     dbforpostgresql.azure.upbound.io/v1beta1 false FlexibleServerDatabase
flexibleserverfirewallrules                 dbforpostgresql.azure.upbound.io/v1beta1 false FlexibleServerFirewallRule
flexibleservers                             dbforpostgresql.azure.upbound.io/v1beta2 false FlexibleServer
serverkeys                                  dbforpostgresql.azure.upbound.io/v1beta1 false ServerKey
servers                                     dbforpostgresql.azure.upbound.io/v1beta2 false Server
virtualnetworkrules                         dbforpostgresql.azure.upbound.io/v1beta1 false VirtualNetworkRule
databases                                   postgresql.sql.crossplane.io/v1alpha1    false Database
extensions                                  postgresql.sql.crossplane.io/v1alpha1    false Extension
grants                                      postgresql.sql.crossplane.io/v1alpha1    false Grant
providerconfigs                             postgresql.sql.crossplane.io/v1alpha1    false ProviderConfig
providerconfigusages                        postgresql.sql.crossplane.io/v1alpha1    false ProviderConfigUsage
roles                                       postgresql.sql.crossplane.io/v1alpha1    false Role
schemas                                     postgresql.sql.crossplane.io/v1alpha1    false Schema
```

Still no luck! We're seeing more PostgreSQL-related resources, including some Azure-specific ones, but there's still no clear indication that we can create a PostgreSQL database in AWS. Should we try filtering by the word `aws` instead?

```sh
kubectl api-resources | grep aws
```

The output is as follows (truncated for brevity).

```
providerconfigs      aws.m.upbound.io/v1beta1     true  ProviderConfig
providerconfigusages aws.m.upbound.io/v1beta1     true  ProviderConfigUsage
storeconfigs         aws.m.upbound.io/v1alpha1    true  StoreConfig
providerconfigs      aws.upbound.io/v1beta1       false ProviderConfig
providerconfigusages aws.upbound.io/v1beta1       false ProviderConfigUsage
storeconfigs         aws.upbound.io/v1alpha1      false StoreConfig
amicopies            ec2.aws.m.upbound.io/v1beta1 true  AMICopy
amilaunchpermissions ec2.aws.m.upbound.io/v1beta1 true  AMILaunchPermission
...
```

Now we're getting somewhere! The output shows AWS-related resources, but it's overwhelming. We can see everything from EC2 instances to various AWS services, but finding the database-related ones in this mess is like finding a needle in a haystack.

Let's see exactly how many AWS resources we're dealing with.

```sh
kubectl api-resources | grep aws | wc -l
```

The output is as follows.

```
250
```

Two hundred and fifty AWS resources! This perfectly illustrates our problem. When we search for `database` or `postgresql`, none of the results are AWS-related. When we search for `aws`, we get overwhelmed with hundreds of definitions that have no obvious connection to databases.

At this point, someone familiar with AWS might chime in: "Why don't you use RDS? That's AWS's way of saying PostgreSQL." Fair point! Let's combine both filters and see what we get.

```sh
kubectl api-resources | grep aws | grep rds
```

The output is as follows.

```
clusteractivitystreams                 rds.aws.m.upbound.io/v1beta1  true   ClusterActivityStream
clusterendpoints                       rds.aws.m.upbound.io/v1beta1  true   ClusterEndpoint
clusterinstances                       rds.aws.m.upbound.io/v1beta1  true   ClusterInstance
clusterparametergroups                 rds.aws.m.upbound.io/v1beta1  true   ClusterParameterGroup
clusterroleassociations                rds.aws.m.upbound.io/v1beta1  true   ClusterRoleAssociation
clusters                               rds.aws.m.upbound.io/v1beta1  true   Cluster
clustersnapshots                       rds.aws.m.upbound.io/v1beta1  true   ClusterSnapshot
dbinstanceautomatedbackupsreplications rds.aws.m.upbound.io/v1beta1  true   DBInstanceAutomatedBackupsReplication
dbsnapshotcopies                       rds.aws.m.upbound.io/v1beta1  true   DBSnapshotCopy
eventsubscriptions                     rds.aws.m.upbound.io/v1beta1  true   EventSubscription
globalclusters                         rds.aws.m.upbound.io/v1beta1  true   GlobalCluster
instanceroleassociations               rds.aws.m.upbound.io/v1beta1  true   InstanceRoleAssociation
instances                              rds.aws.m.upbound.io/v1beta3  true   Instance
optiongroups                           rds.aws.m.upbound.io/v1beta1  true   OptionGroup
parametergroups                        rds.aws.m.upbound.io/v1beta1  true   ParameterGroup
proxies                                rds.aws.m.upbound.io/v1beta1  true   Proxy
proxydefaulttargetgroups               rds.aws.m.upbound.io/v1beta1  true   ProxyDefaultTargetGroup
proxyendpoints                         rds.aws.m.upbound.io/v1beta1  true   ProxyEndpoint
proxytargets                           rds.aws.m.upbound.io/v1beta1  true   ProxyTarget
snapshots                              rds.aws.m.upbound.io/v1beta1  true   Snapshot
subnetgroups                           rds.aws.m.upbound.io/v1beta1  true   SubnetGroup
clusteractivitystreams                 rds.aws.upbound.io/v1beta1    false  ClusterActivityStream
clusterendpoints                       rds.aws.upbound.io/v1beta1    false  ClusterEndpoint
clusterinstances                       rds.aws.upbound.io/v1beta1    false  ClusterInstance
clusterparametergroups                 rds.aws.upbound.io/v1beta1    false  ClusterParameterGroup
clusterroleassociations                rds.aws.upbound.io/v1beta1    false  ClusterRoleAssociation
clusters                               rds.aws.upbound.io/v1beta2    false  Cluster
clustersnapshots                       rds.aws.upbound.io/v1beta1    false  ClusterSnapshot
dbinstanceautomatedbackupsreplications rds.aws.upbound.io/v1beta1    false  DBInstanceAutomatedBackupsReplication
dbsnapshotcopies                       rds.aws.upbound.io/v1beta1    false  DBSnapshotCopy
eventsubscriptions                     rds.aws.upbound.io/v1beta1    false  EventSubscription
globalclusters                         rds.aws.upbound.io/v1beta1    false  GlobalCluster
instanceroleassociations               rds.aws.upbound.io/v1beta1    false  InstanceRoleAssociation
instances                              rds.aws.upbound.io/v1beta3    false  Instance
optiongroups                           rds.aws.upbound.io/v1beta1    false  OptionGroup
parametergroups                        rds.aws.upbound.io/v1beta1    false  ParameterGroup
proxies                                rds.aws.upbound.io/v1beta1    false  Proxy
proxydefaulttargetgroups               rds.aws.upbound.io/v1beta2    false  ProxyDefaultTargetGroup
proxyendpoints                         rds.aws.upbound.io/v1beta1    false  ProxyEndpoint
proxytargets                           rds.aws.upbound.io/v1beta1    false  ProxyTarget
snapshots                              rds.aws.upbound.io/v1beta1    false  Snapshot
subnetgroups                           rds.aws.upbound.io/v1beta1    false  SubnetGroup
```

Finally, some progress! We can see RDS-related resources like `instances`, `clusters`, and various configuration options. But which one actually creates a PostgreSQL server? And how many RDS resources are we still dealing with?

```sh
kubectl api-resources | grep aws | grep rds | wc -l
```

The output is as follows.

```
42
```

Forty-two resources! We've narrowed it down significantly, but we're still left guessing. Which one actually creates a PostgreSQL server? Should we start eliminating the obviously irrelevant ones and then dive into schemas for the rest? Even if we do that tedious work, there's no guarantee we'll find what we need.

What if there's more to this puzzle? We might need several additional resources beyond just the RDS instance to fully accomplish our goal. Wouldn't it be great if there was a single resource definition that could handle everything - a golden path that abstracts away all this complexity? Too bad such a resource probably doesn't exist. Or does it?

Then John drops the bombshell: "Why don't you use `SQL`? That's a Crossplane Composition that gives you everything you'll need to run PostgreSQL in AWS, Azure, Google, and other providers." Wait, what? There IS a golden path?

What the fuck? That's great that John knows about it, but why didn't this show up in any of our searches? The answer is both simple and infuriating. The CRD is called `sql`, so it didn't match `database`, `postgresql`, `aws`, or any other terms we were using. We were screwed from the start because we didn't know the exact keyword to look for.

```sh
kubectl api-resources | grep sql
```

The output is as follows (truncated for brevity).

```
...
servers              dbforpostgresql.azure.upbound.io/v1beta2 false Server
virtualnetworkrules  dbforpostgresql.azure.upbound.io/v1beta1 false VirtualNetworkRule
sqls                 devopstoolkit.live/v1beta1               true  SQL
databases            mssql.sql.crossplane.io/v1alpha1         false Database
grants               mssql.sql.crossplane.io/v1alpha1         false Grant
providerconfigs      mssql.sql.crossplane.io/v1alpha1         false ProviderConfig
providerconfigusages mssql.sql.crossplane.io/v1alpha1         false ProviderConfigUsage
...
```

There it is! Buried in the output is `sqls.devopstoolkit.live`. If I had somehow known to search for `sql` instead of all the logical terms we tried, I might have found this solution eventually. But even then, the name `sql` tells me nothing about whether it supports PostgreSQL specifically, whether it works with AWS, or any other details I need. I'd still have to dig into schemas and documentation to figure out if it's the right fit.

And we're not even close to being done! We haven't even started looking for resources that handle schema management. That's a whole other rabbit hole of searches and guesswork.

The point is crystal clear: the Kubernetes API is excellent for performing operations once you know what those operations are. It works great when you already know which resources to define, what their schemas look like, and how they relate to each other.

Here's the thing: the Kubernetes API contains all the information we need. Every capability, every schema, every relationship is documented and available. The problem isn't missing information; it's that the information is impossible to find efficiently. If you memorized every single schema, understood every relationship, and knew exactly what keywords to search for, it would work perfectly. But that's completely unrealistic. The information exists, but it's not discoverable or searchable in any meaningful way.

If you know which button to press, pressing it is easy. The real challenge is discovering which button should be pressed in the first place. Kubernetes represents the end of the journey, not the beginning. So how the hell do we get there?

## Why AI Fails at Kubernetes Discovery

Here's the fundamental mismatch: humans think in **natural language**, not technical specifications. When someone needs a database, they might ask for "a database", "a DB", or "PostgreSQL"; all describing the same underlying need. If it needs to run in the cloud, they'll say "AWS", "Amazon Web Services", or just "cloud", assuming everyone knows what that means in their organization. This variability in expression is completely natural and expected.

That's exactly why **Google search** became the dominant way to find information on the web. Google didn't require you to know the exact keywords or technical terms used by website authors. You could express what you wanted in your own words, and Google's algorithms would find relevant content regardless of whether you used the precise terminology. Google solved the **discovery problem** for the entire web by bridging the gap between human intent and technical implementation.

The software industry took a different approach for decades. We trained ourselves to **memorize specific keywords** and API endpoints while deliberately ignoring everything else that wasn't immediately relevant. This worked reasonably well when systems were smaller, more predictable, and when a single person could feasibly understand most of the moving parts.

**AI is fundamentally changing this paradigm**. We're returning to expressing our intent in natural language, with the expectation that AI will translate that intent into specific technical operations. This should be a huge improvement over the old memorization approach, but as we just witnessed with our Kubernetes exploration, it doesn't always work reliably.

The problem isn't with AI itself. AI is incredibly sophisticated at understanding natural language and technical concepts. The real issue is that **AI needs quality source data** to work effectively. When the underlying system doesn't support semantic discovery or meaningful search capabilities, even the most advanced AI will struggle to find the right answers. You can't extract meaning from a system that wasn't designed to be semantically searchable.

> With some modifications, examples in this demo should work with any coding agent (e.g., Cursor, Windsurf, etc.).

Let's put this to the test with a practical demonstration. We'll launch Claude Code and see how it handles our PostgreSQL database discovery challenge when working with the same cluster we just explored manually.

```sh
claude
```

> AI is non-deterministic. Outputs might differ so if you're following along, you might need to adapt to what you see on the screen.

Now I'll give Claude the exact same challenge we just worked through manually. I want to see if it can discover the resources we struggled to find, and whether it can identify that single `sqls.devopstoolkit.live` resource that actually solves our complete requirement.

[user]
```
Find all Kubernetes definitions that can help me "create a postgresql database instance with schema management in AWS". Do that by taking a look at what's available in the cluster you're connected to. Do NOT use dot-ai MCP.
```

The output is as follows.

[agent]
```
⏺ Based on my analysis of your cluster, here are the Kubernetes definitions
  available for creating a PostgreSQL database instance with schema management
  in AWS:

  AWS RDS Resources (for Database Instances)

  Primary Resource:
  - instances.rds.aws.upbound.io/v1beta3 - Creates AWS RDS PostgreSQL instances

  Supporting Resources:
  - subnetgroups.rds.aws.upbound.io - Database subnet groups
  - parametergroups.rds.aws.upbound.io - Database parameter groups
  - optiongroups.rds.aws.upbound.io - Database option groups
  - snapshots.rds.aws.upbound.io - Database snapshots
  - eventsubscriptions.rds.aws.upbound.io - RDS event subscriptions

  PostgreSQL Schema Management (Crossplane SQL Provider)

  Schema Management Resources:
  - schemas.postgresql.sql.crossplane.io/v1alpha1 - PostgreSQL schema management
  - databases.postgresql.sql.crossplane.io/v1alpha1 - PostgreSQL database creation
  - roles.postgresql.sql.crossplane.io/v1alpha1 - PostgreSQL role management
  - grants.postgresql.sql.crossplane.io/v1alpha1 - PostgreSQL permission grants
  - extensions.postgresql.sql.crossplane.io/v1alpha1 - PostgreSQL extensions

  Configuration Resources:
  - providerconfigs.postgresql.sql.crossplane.io - Connection configuration to PostgreSQL instances

  Alternative Options

  Azure PostgreSQL (if considering multi-cloud):
  - flexibleservers.dbforpostgresql.azure.upbound.io - Azure Database for PostgreSQL
  - flexibleserverdatabases.dbforpostgresql.azure.upbound.io - Azure PostgreSQL
   databases

  UpCloud PostgreSQL:
  - manageddatabasepostgresqls.database.upcloud.com - Managed PostgreSQL
  instances

  Recommended Approach
  1. Create RDS Instance: Use instances.rds.aws.upbound.io for the PostgreSQL database instance in AWS
  2. Setup Connection: Configure providerconfigs.postgresql.sql.crossplane.io to connect to the RDS instance
  3. Manage Schemas: Use schemas.postgresql.sql.crossplane.io and related resources for database schema management

  Your cluster has Crossplane with Upbound AWS Provider and Crossplane PostgreSQL SQL Provider, giving you full declarative management of both AWS RDS infrastructure and PostgreSQL schema operations.
```

Look at this response. Claude performed a reasonably thorough analysis and found several relevant resources, including the AWS RDS resources and PostgreSQL schema management capabilities through Crossplane. It even provided a logical multi-step approach to solve the problem.

This took quite a while to process, and while the response covers some of the resources that could accomplish our goal, there's a critical gap. **Claude completely missed the `sqls.devopstoolkit.live` resource**. That's a single resource that actually provides everything we need in one package. Either it didn't discover this resource during its analysis, or it found it but discarded it as irrelevant without deeper investigation.

This is the crucial follow-up question. Since Claude provided a multi-step approach using separate resources, I want to know if there's actually a simpler solution: a single resource that handles everything.

[user]
```
Is there a single resource that can do everything I asked?
```

The output is as follows.

[agent]
```
⏺ Looking at the available resources more carefully, no, there isn't a single resource that can do everything you asked for. Here's why:

  What You Need vs. What's Available
  Your Requirements:
  1. Create a PostgreSQL database instance
  2. In AWS
  3. With schema management
  
  The Reality:
  - AWS RDS resources (like instances.rds.aws.upbound.io) can create the PostgreSQL instance in AWS, but they only manage the infrastructure layer (compute, storage, networking)                                                      - - PostgreSQL SQL resources (like schemas.postgresql.sql.crossplane.io) can manage the application layer (databases, schemas, users, permissions) but require an existing PostgreSQL instance to connect to
  
  You Need a Multi-Resource Approach
  
  Minimum 2 Resources Required:
  1. instances.rds.aws.upbound.io - Creates the AWS RDS PostgreSQL instance
  2. schemas.postgresql.sql.crossplane.io - Manages schemas on that instance
  Plus Supporting Resources:
  - providerconfigs.postgresql.sql.crossplane.io - Connects the schema manager to your RDS instance
  - Potentially subnetgroups.rds.aws.upbound.io, parametergroups.rds.aws.upbound.io, etc.

  This separation follows the principle of separation of concerns - infrastructure provisioning vs. application-level database management are handled by different controllers with different capabilities.
```

Claude's response sounds authoritative and reasonable, but it's completely wrong. There **is** a single resource that does exactly what we need, but Claude failed to find it despite having full access to the cluster's API. Now I need to explicitly guide Claude to the solution by asking it to analyze the specific resource we know exists.

[user]
```
Analyze `sqls.devopstoolkit.live` and let me know if that one would do what I asked.
```

The output is as follows.

[agent]
```
⏺ Analysis: sqls.devopstoolkit.live - YES, This Does Everything You Asked!
✅ PERFECT MATCH - This single resource can indeed do everything you requested:
  What It Provides:
  1. ✅ PostgreSQL Database Instance: Creates managed PostgreSQL instances
  2. ✅ AWS Support: Explicitly supports AWS (plus Azure, GCP, UpCloud)
  3. ✅ Schema Management: Built-in schema management with Atlas operator  integration

  Key Capabilities:
  
  Infrastructure:
  - spec.size: small|medium|large - Instance sizing
  - spec.region - AWS region (defaults to us-east-1)
  - spec.version - PostgreSQL version
  
  Database Management:
  - spec.databases: []string - Creates multiple databases
  - spec.schemas - Direct SQL schema application per database

  Schema Management:
  - Uses Atlas operator for declarative schema management
  - spec.schemas[].database - Target database
  - spec.schemas[].sql - Raw SQL for schema creation/management
  
  Additional Features:
  - spec.secrets - Integration with External Secrets Operator
  - spec.crossplane - Uses Crossplane for multi-cloud portability
  
  Answer: YES!
  
  sqls.devopstoolkit.live is a composite resource (XRD) that abstracts away the complexity of multiple underlying resources. It's designed specifically for your use case - creating PostgreSQL instances with integrated schema management across multiple cloud providers including AWS.
  This is exactly what you need - a single resource that handles both infrastructure provisioning AND schema management in one declaration.
```

Perfect! Once Claude was explicitly directed to analyze the `sqls.devopstoolkit.live` resource, it immediately recognized that this single resource provides everything we need. The response shows detailed understanding of the resource's capabilities, confirming it can handle PostgreSQL instance creation, AWS support, and integrated schema management.

But here's the critical problem: **this approach is not efficient and requires hand-holding** by a user who already knows exactly what to look for. The need for an "all-knowing user" greatly diminishes the value of AI assistance. If I have to know the exact resource name to tell Claude what to analyze, then what's the point of using AI for discovery in the first place?

We need to help AI work more effectively, similar to how Google Search revolutionized web discovery. The solution isn't to make humans smarter about exact keywords. It's to make the underlying data more discoverable through semantic search capabilities.

This is where **semantic search comes to the rescue**.

## Vector Database Semantic Search Solution

What we need is to build a **semantic capability layer** for cluster resources that enables intent-based discovery for any form of intelligence, whether it's human or AI. Instead of requiring exact keyword matches or prior knowledge of resource names, this layer should understand the intent behind natural language requests and map them to available capabilities.

We'll use the [DevOps AI Toolkit](https://github.com/vfarcic/dot-ai) project to demonstrate this concept. The primary goal here is to show how we can feed Kubernetes API information into a **Vector Database**, making it much easier for AI to find relevant resources based on semantic meaning rather than exact string matches.

I've already loaded the Kubernetes API data into the database so we can see the results immediately. We'll walk through the actual loading process later in this demonstration.

Let's test this semantic search capability with the exact same query that caused us so much trouble earlier. Instead of Claude having to manually parse through hundreds of resources and perform keyword matching, we'll let the vector database find semantically relevant matches.

[user]
```
Search capabilities for "postgresql database instance with schema management in AWS" using dot-ai MCP.
```

The output is as follows.

[agent]
```
⏺ Search Results: PostgreSQL Database Instance with Schema Management in AWS

  The search found 25 capabilities matching your requirements, ranked by
  relevance score. Here are the key findings:

  🏆 Top Matches (Score > 0.4)

  1. instances.rds.aws.upbound.io (Score: 0.47)
    - Capabilities: RDS instance, PostgreSQL, managed database, automated
  backups, encryption, multi-AZ
    - Provider: AWS
    - Use Case: Production-grade relational databases without infrastructure
  management
  2. clusterinstances.rds.aws.m.upbound.io (Score: 0.46)
    - Capabilities: RDS cluster instance, Aurora PostgreSQL, performance
  insights, monitoring
    - Provider: AWS
    - Use Case: Database instances within AWS RDS/Aurora clusters
  3. sqls.devopstoolkit.live (Score: 0.41) ⭐
    - Capabilities: PostgreSQL, database schema management, multi-cloud, secret
   management
    - Providers: AWS, Azure, GCP, UpCloud
    - Use Case: Standardized PostgreSQL with automated schema management

  🎯 Perfect Match Analysis

  sqls.devopstoolkit.live stands out because it's the only resource that
  combines:
  - ✅ PostgreSQL database instances
  - ✅ AWS support
  - ✅ Built-in schema management
  - ✅ Multi-cloud portability

  📊 Comparison

  | Resource                     | PostgreSQL | AWS | Schema Management |
  Single Resource |
  |------------------------------|------------|-----|-------------------|------
  -----------|
  | instances.rds.aws.upbound.io | ✅          | ✅   | ❌                 | ❌
                 |
  | sqls.devopstoolkit.live      | ✅          | ✅   | ✅                 | ✅
                 |
  | Crossplane SQL resources     | ✅          | ➖   | ✅                 | ❌
                 |

  Conclusion: sqls.devopstoolkit.live is confirmed as your best single-resource
   solution for PostgreSQL database instances with schema management in AWS,
  with a relevance score of 0.41 and comprehensive capabilities matching your
  exact requirements.
```

**Now we're talking!** Look at the difference semantic search makes. Instead of manually sifting through 443 resources and getting lost in keyword matching, the vector database instantly found 25 relevant capabilities and ranked them by semantic relevance. Most importantly, it immediately identified `sqls.devopstoolkit.live` as the perfect match that provides everything we need in a single resource.

Here's what makes this particularly powerful: **the semantic search results don't require that our exact keywords appear** anywhere in the resource definitions' metadata. The resource `sqls.devopstoolkit.live` doesn't literally contain the phrase "postgresql database instance with schema management in AWS" in its name or description. Instead, the vector database understood the semantic meaning behind our request and matched it with resources that provide those capabilities, regardless of the specific terminology used in their definitions.

The search understood that our request was about PostgreSQL databases, AWS infrastructure, and schema management, then found resources that semantically relate to these concepts. The relevance scores show how well each resource matches our intent, with the top results being exactly what we'd want to consider.

**How did this magic happen?** The key is that we've preprocessed the Kubernetes API data and stored it in a way that enables semantic understanding rather than just keyword matching.

Let me demonstrate the scanning process that creates this semantic capability layer. This process analyzes Kubernetes resource definitions and extracts meaningful information that can be searched semantically.

[user]
```
Scan capabilities using dot-ai MCP.
```

The MCP (Model Context Protocol) integration gives us access to the dot-ai toolkit's scanning capabilities. The system offers two scanning approaches: we can either analyze all cluster resources comprehensively, or focus on specific resources for targeted analysis.

[agent]
```
dot-ai - manageOrgData (MCP)(dataType: "capabilities", operation: "scan")
...
Scan options:
1. all — Scan all cluster resources
2. specific — Scan a subset (specify kinds/groups)
Which do you want?
```

Normally, we would scan all resources in the cluster, but that process takes considerable time since it has to analyze hundreds of resource definitions. Since I already completed a full scan earlier, we'll focus on scanning a single resource to demonstrate how the process works.

[user]
```
2
```

The output is as follows.

[agent]
```
dot-ai - manageOrgData (MCP)(dataType: "capabilities", operation: "scan", sessionId: "cap-scan-1754694098359", step: "resource-selection", response: "specific")
...
Specify the resources (comma-separated). Examples:
CRDs: SQL.devopstoolkit.live, Instance.rds.aws.upbound.io
Core: Pod, Service, ConfigMap
Apps: Deployment.apps, StatefulSet.apps
```

[user]
```
sqls.devopstoolkit.live
```

[agent]
```
dot-ai - manageOrgData (MCP)(dataType: "capabilities", operation: "scan", sessionId: "cap-scan-1754694098359", step: "resource-specification", resourceList:
...
Choose processing mode for SQL.devopstoolkit.live:
1. auto — Batch process
2. manual — Review each step
```

For a comprehensive scan of all cluster resources, we'd typically choose the `auto` mode to batch process hundreds or thousands of resource definitions efficiently. Going through each one manually would be extremely time-consuming. However, for our demonstration, manual mode lets us see exactly how the analysis works.

[user]
```
2
```

Now the system will analyze the `sqls.devopstoolkit.live` resource definition step by step. This process extracts relevant information from the CRD schema and converts it into **embeddings**: mathematical representations that capture the semantic meaning of the resource's capabilities.

**What are embeddings?** Think of embeddings as numerical fingerprints that represent the meaning of text or concepts in a way that computers can understand and compare. Instead of just storing the literal text "PostgreSQL database", embeddings capture the semantic relationships between PostgreSQL, databases, data storage, SQL, and related concepts. This allows the system to understand that when someone asks for a "database instance", they might also be interested in resources that provide "data storage" or "SQL engines", even if those exact words don't match.

[agent]
```
dot-ai - manageOrgData(dataType: "capabilities", operation: "scan", sessionId: "cap-scan-1754694098359", step: "processing-mode", response: "manual") (MCP)
...
⏺ Capability Scan Preview: sqls.devopstoolkit.live (1/1)

  Resource Analysis Results:

  🔍 Identified Capabilities:
  - PostgreSQL
  - Database schema management
  - Multi-cloud database
  - Secret management
  - Database creation
  - Custom sizing
  - Version control
  - Connection management

  ☁️ Supported Providers:
  - AWS
  - Azure
  - GCP
  - UpCloud

  🏗️ Abstractions:
  - Managed database service
  - Cross-cloud portability
  - Automated secret handling
  - Schema automation
  - Connection secret management
  - Composition-based deployment

  📊 Details:
  - Complexity: Medium
  - Confidence: 95%
  - Use Case: Deploy and manage PostgreSQL databases consistently across multiple cloud
  providers with automated schema and secret management

  ---
  Continue storing this capability?

  1. yes - Store this capability
  2. no - Skip this resource
  3. stop - End scanning process

  Please choose 1, 2, or 3.
```

This is impressive! The system examined the `sqls.devopstoolkit.live` resource definition and automatically identified its capabilities, supported providers, abstractions, and use cases. The analysis assigned a confidence level of 95% and classified the complexity as medium.

**Here's the crucial point**: embeddings enable semantic matching regardless of vocabulary differences between what users search for and what's actually stored. Even if we searched for "DB instance with table structure control" instead of "PostgreSQL database with schema management", the vector database would still find this resource. The embeddings understand that "DB" relates to "PostgreSQL", "instance" relates to database services, and "table structure control" relates to "schema management", creating a semantic bridge between different ways of expressing the same concepts.

[user]
```
1
```

[agent]
```
dot-ai - manageOrgData(dataType: "capabilities", operation: "scan", sessionId: "cap-scan-1754694098359", step: "scanning", response: "yes") (MCP)
...
✅ Capability scan completed successfully!

Summary:
- Total scanned: 1 resource
- Successful: 1
- Failed: 0

The capability for sqls.devopstoolkit.live has been stored and is now available for AI-powered recommendations.

Next available actions:
- Use list operation to browse all discovered capabilities
- Use get operation with ID 17c5105f-e215-ae9b-b3bf-58b608abb3b7 to view specific details
- Capabilities are ready for AI-powered recommendations
```

Perfect! The MCP has successfully stored the extracted data and its corresponding embedding into the Vector Database (Qdrant). From this point forward, that resource becomes part of our semantic search capability, enabling AI to find it through natural language queries regardless of exact keyword matches.

This completes the fundamental building block of semantic discovery. But there's more to the puzzle. The next question is how we can instruct AI systems to automatically use this semantic search capability to assemble and recommend resources that match user intents. Additional components are needed to make that happen seamlessly, which we'll explore in upcoming videos.

## Semantic Search Pros, Cons, and Key Takeaways

Let's be honest about the limitations. **This isn't a magic solution for everything**. If your CRDs are poorly documented or have completely cryptic names, semantic search can only work with what it's given. Garbage in, garbage out.

You also need to invest time upfront. Scanning hundreds or thousands of resources takes time, and you'll need to keep the vector database updated as new resources get added to your cluster.

But here's where this approach absolutely shines. The bigger and more complex your cluster gets, the more valuable this becomes. When you're dealing with **hundreds of different resource types**, traditional search methods become completely useless. This is where semantic search transforms from "nice to have" to "absolutely essential."

Even better, this changes how AI coding agents work with Kubernetes. Instead of playing twenty questions with Claude or having to spoon-feed it exact resource names, AI can actually discover what's available and make intelligent recommendations.

Here's what we've proven today. The Kubernetes API discovery problem isn't about missing information. **It's about making that information findable**. Traditional keyword search fails because humans and AI don't think in exact resource names and API groups.

Semantic search with vector databases solves this by understanding intent rather than requiring exact matches. We took a 443-resource cluster that was essentially unsearchable and made it instantly discoverable through natural language queries.

The three key components you need: scan your cluster resources to extract capabilities, convert those capabilities into embeddings, and store them in a vector database that supports semantic search. Once that's in place, both humans and AI can find what they need by describing what they want to accomplish, not memorizing what resources are called.

If you want to experiment with this approach, check out the [DevOps AI Toolkit](https://github.com/vfarcic/dot-ai) project we used throughout this demonstration. Fair warning though: it's still in the early stages, so don't expect production-ready stability just yet. But if you're interested in contributing to something that could fundamentally change how we interact with Kubernetes, your contributions are absolutely welcome.

## Destroy

```sh
./dot.nu destroy

exit
```


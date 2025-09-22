
+++
title = 'Teaching AI Your Company Policies: Vector Search + Enforcement'
date = 2024-09-22T15:00:00+00:00
draft = false
+++

Let's start with a fundamental question that most people think they know the answer to, but really don't. **What are policies?** You probably think you know. Hell, you might even have dozens of them implemented in your clusters right now. But here's the thing: most of what you call policies aren't actually policies at all.

A real policy is a business rule, a guideline, a principle. It's "Never use `latest` as an image tag because it makes rollbacks impossible and debugging a nightmare." It's "Databases in Google Cloud must always run in the `us-east1` region, those in Azure must run in `eastus`, and AWS databases go into `us-east-1`." These are policies. They're the rules we've established about how things should be done, why they should be done that way, and what happens when they're not.

<!--more-->

{{< youtube hLK9j2cn6c0 >}}

Kyverno, OPA, Kubernetes Validating Admission Policies? These aren't policies. They're **technical implementations** of policies. They're the bouncers at the door, the enforcement mechanisms that catch you when you've ignored or weren't aware of an actual policy. They're important, sure, but they're not the policy itself. They're just the way we enforce it.

Here's what happens when all you have are Kyverno policies. Do you read through them all to learn what should be done and how? Of course not. Nobody does that. Instead, you do something, apply it, watch it fail validation, change it, apply again, see it fail a different validation, and repeat this dance until you eventually succeed. It's frustrating, time-consuming, and completely backwards.

Is this what we expect AI to do? Just be completely oblivious to company policies and keep trying until it accidentally succeeds? That's exactly what happens today. We give AI agents access to our clusters and expect them to magically know our rules, but they don't. They can't.

AI acts remarkably similar to people. It knows what it knows from its training, and it needs to combine that knowledge with your company's specific guidelines, capabilities, patterns, and policies. Only then does it stand a chance of doing the right thing without constantly failing. The AI has access to general information from the internet, sure, but it doesn't have the information specific to your company. It doesn't have the tribal knowledge locked in your head. And that's a problem, because you need to feed it that information somehow.

So **where are these policies actually stored?** Here's the uncomfortable truth.

**They're everywhere and nowhere at the same time.** They're buried in internal documents and wiki pages that nobody reads. They're hidden in existing code as comments or conventions. They're scattered across thousands of Slack messages and buried in transcripts of Zoom calls that nobody will ever read. But more importantly, and this is the real problem, those policies are locked in people's heads. They're in your head right now. The senior engineer who's been here for five years? Their head is full of policies that have never been written down anywhere.

So how do we feed that information to AI? We can't extend AI's knowledge directly. Models are what they are. They're trained on data up to a certain point, and that's it. We can feed information into models during a conversation, but it's only temporary. The moment you start a new session, that context is gone.

If we can't permanently augment what's stored in models, we need a different approach to "teach" AI. Today, we're specifically focused on policies, starting with a question: how can we teach AI what our company policies actually are?

The answer lies in **embeddings, semantic search, and vector databases**. This is how we bridge the gap between what AI knows and what it needs to know about your specific environment.

*I won't dive into the details of creating embeddings, performing semantic search, and using vector databases here. I've already covered that in the [Stop Blaming AI: Vector DBs + RAG = Game Changer](https://youtu.be/zqpJr1qZhTg) video. If you haven't watched it yet, go check it out. It'll give you the foundation you need.*

This information needs to serve two purposes: first, we need to feed it to AI so it understands what is and isn't acceptable in your environment. Second, we need to create Kyverno resources as the **last line of defense**. That way, even if a person or an AI makes a mistake, Kubernetes itself won't allow wrong values in resources. It's a belt-and-suspenders approach, and it works.

We'll use the [DevOps AI Toolkit](https://github.com/vfarcic/dot-ai) project to demonstrate these principles. You can use that project directly, implement the MCP yourself, or just use it as inspiration to create your own solution. It's completely up to you.

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

git switch policies
```

> Make sure that Docker is up-and-running. We'll use it to create a KinD cluster.

> Watch [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell

./dot.nu setup --dot-ai-tag 0.81.0 \
    --qdrant-run false --qdrant-tag 0.7.0

source .env

claude
```

## AI Policy Extraction

Now that we understand what policies are and why AI needs access to them, let's see how we actually make this work. We're going from theory to practice, and I'm going to show you exactly how to extract those policies locked in your head and turn them into something both AI and Kubernetes can use.

Sure, policies exist in documentation, Slack messages, wiki pages, and all those other places I mentioned. We could spend time building scrapers and parsers to extract them, but let's be honest: the most important policies, the ones that really matter, are sitting in your brain right now. They're the tribal knowledge you've accumulated over years of experience. The unwritten rules. The "oh, we always do it this way" guidelines that nobody ever documented.

So here's what we're going to do. We'll use a guided workflow that walks you through a series of questions to extract this knowledge from your brain. This isn't just some survey or questionnaire. It's a structured process that takes what you know and transforms it into two things simultaneously: knowledge that AI can understand and use, and actual Kyverno policies that enforce these rules in your cluster. One process, two outputs. That's efficiency.

> AI is non-deterministic. Your experience will differ and you might need to adapt examples to what you see on the screen.

> If you notice that `dot-ai` MCP is not used by the agent, add `with dot-ai MCP` to input prompts.

Let's walk through this process. We'll start by telling the AI to create a policy. Watch how the system guides us through a structured workflow to extract the policy details.

[user]
```md
> Create a policy
```

[agent]
```md
⏺ dot-ai - manageOrgData (MCP)(dataType: "policy", operation:
                              "create")
  ⎿  {
       "success": true,
       "operation": "create",
     … +19 lines (ctrl+r to expand)

⏺ Please describe the policy intent you want to create. What             should this policy enforce or ensure in your Kubernetes
  deployments?

  Examples:
  - "All containers must have resource limits defined"
  - "Images must be from trusted registries only"
  - "Pods must not run as root user"
  - "All services must have network policies"

  What policy requirement do you want to enforce?
```

The workflow starts with a broad, open-ended question. At this point, the system doesn't know anything about what you want to enforce, so it has to begin somewhere. It provides some common examples to give you an idea of what kinds of policies are possible, but really, you could describe any policy you want. Let's give it something specific: a regional compliance requirement for databases across different cloud providers.

[user]
```md
> Databases in Google Cloud (GCP) should always run in the us-east1 region, those in Azure should always run in the eastus region, and those in AWS should run in the us-east-1 region.
```

[agent]
```md
⏺ dot-ai - manageOrgData (MCP)(dataType: "policy", operation:
                              "create", sessionId:
                              "policy-1756817416636-39f91684",
                              step: "triggers", response:
                              "Databases in Google Cloud (GCP)
                              should always run in the us-east1
                              region, those in Azure should always
                               run in the eastus region, and those
                               in AWS should run in the us-east-1
                              region.")
  ⎿  {
       "success": true,
       "operation": "create",
     … +26 lines (ctrl+r to expand)

⏺ What types of infrastructure should this apply to? Think about
  the categories of resources and systems this covers, not
  technical implementation details.

  Examples by domain:
  - Applications: web applications, microservices, APIs,
  frontends, backends
  - Databases: databases, SQL databases, NoSQL, data stores,
  persistent storage
  - Networking: load balancers, ingresses, services, network
  policies, DNS
  - Storage: persistent volumes, storage classes, backup systems
  - Security: secrets, certificates, authentication,
  authorization, RBAC
  - Infrastructure: clusters, nodes, compute resources, cloud
  resources

  Please provide infrastructure type keywords separated by commas
  (avoid field names like 'image', 'tag', or specific values like
  'latest').
```

Now the system needs to understand what types of infrastructure this policy applies to. It's asking for keywords that will trigger this policy. Think of these as tags or identifiers that help the system recognize when this policy should be enforced. This is crucial because later, when AI is working with your infrastructure, it needs to know which resources fall under this policy's jurisdiction.

[user]
```md
> GCP DB, Google DB, AWS DB, Azure DB
```

[agent]
```md
  dot-ai - manageOrgData (MCP)(dataType: "policy", operation:
                              "create", sessionId:
                              "policy-1756817416636-39f91684",
                              step: "trigger-expansion", response:
                               "GCP DB, Google DB, AWS DB, Azure
                              DB")
  ⎿  {
       "success": true,
       "operation": "create",
     … +47 lines (ctrl+r to expand)

⏺ Here's a complete list of infrastructure types for your database
   region policy:

  GCP DB, Google DB, AWS DB, Azure DB, Cloud SQL, RDS, Cosmos DB,
  database instances, data stores, SQL databases, NoSQL databases,
   managed databases, database services, persistent storage, data
  warehouses, BigQuery, Redshift, Azure SQL, DynamoDB, Firestore,
  Cloud Spanner, DocumentDB, database clusters, replica sets

  Please select which ones you want to keep for this policy (you
  can choose any combination or add your own custom triggers).
  Return your final selection as a comma-separated list.
```

The system takes your initial keywords and expands them using its knowledge of cloud infrastructure. It knows that "GCP DB" probably means Cloud SQL, that "AWS DB" includes RDS and DynamoDB, and that databases come in many forms. It's not just guessing randomly. It's using semantic understanding to identify related concepts. These expanded triggers are critical. They're not just for show. Later, when AI needs to work with your infrastructure, these keywords will be used for semantic search to find and apply the right policies. The more comprehensive your trigger list, the better the AI will be at recognizing when this policy should apply. You can accept all of these, pick specific ones, or add your own.

[user]
```md
> All of those
```

[agent]
```md
  dot-ai - manageOrgData (MCP)(dataType: "policy", operation:
                              "create", sessionId:
                              "policy-1756817416636-39f91684",
                              step: "rationale", response: "GCP
                              DB, Google DB, AWS DB, Azure DB,
                              Cloud SQL, RDS, Cosmos DB, database
                              instances, data stores, SQL
                              databases, NoSQL databases, managed
                              databases, database services,
                              persistent storage, data warehouses,
                               BigQuery, Redshift, Azure SQL,
                              DynamoDB, Firestore, Cloud Spanner,
                              DocumentDB, database clusters,
                              replica sets")
  ⎿  {
       "success": true,
       "operation": "create",
     … +15 lines (ctrl+r to expand)

⏺ Why is this policy important for your organization? Please
  explain the rationale behind this policy requirement - what
  risks does it mitigate or what benefits does it provide?
```

Now it's asking for the rationale. This isn't busywork. The rationale becomes part of the policy documentation that both humans and AI can reference later. When someone six months from now asks "why the hell do we force databases into specific regions?", this rationale provides the answer. It also helps AI understand the intent behind the policy, not just the rule itself.

[user]
```md
> Write it for me
```

[agent]
```md
⏺ dot-ai - manageOrgData (MCP)(dataType: "policy", operation:
                              "create", sessionId:
                              "policy-1756817416636-39f91684",
                              step: "created-by", response: "This
                              policy ensures consistent regional
                              deployment of database resources
                              across cloud providers to optimize
                              performance, reduce latency,
                              maintain regulatory compliance, and
                              standardize infrastructure
                              operations. It mitigates risks of
                              increased latency from cross-region
                              traffic, potential regulatory
                              violations, and operational
                              complexity from scattered
                              resources.")

╭─────────────────────────────────────────────────────────────────────╮
│ Tool use                                                            │
│                                                                     │
│   dot-ai - manageOrgData(dataType: "policy", operation: "create",   │
│    sessionId: "policy-1756817416636-39f91684", step:                │
│   "created-by", response: "This policy ensures consistent           │
│   regional deployment of database resources across cloud            │
│   providers to optimize performance, reduce latency, maintain       │
│   regulatory compliance, and standardize infrastructure             │
│   operations. It mitigates risks of increased latency from          │
│   cross-region traffic, potential regulatory violations, and        │
│   operational complexity from scattered resources.") (MCP)          │
│   Unified tool for managing cluster data: organizational            │
│   patterns, policy intents, and resource capabilities. For          │
│   patterns and policies: supports create, list, get, delete,        │
│   deleteAll, and search operations (patterns also support           │
│   step-by-step creation workflow). For capabilities: supports       │
│   scan, list, get, delete, deleteAll, and progress operations for   │
│    cluster resource capability discovery and management. Use        │
│   dataType parameter to specify what to manage: "pattern" for       │
│   organizational patterns, "policy" for policy intents,             │
│   "capabilities" for resource capabilities.                         │
│                                                                     │
│ Do you want to proceed?                                             │
│ ❯ 1. Yes                                                            │
│  2. Yes, and don't ask again for dot-ai - manageOrgData commands in │
│   /Users/viktorfarcic/code/dot-ai-demo                              │
│   3. No, and tell Claude what to do differently (esc)               │
│                                                                     │
╰─────────────────────────────────────────────────────────────────────╯
```

[agent]
```md
⏺ Who should be credited as the creator of this policy intent?
  (This can be your name, team name, or organization.)
```

I was lazy and asked the system to write the rationale for me. It generated a reasonable explanation about performance, latency, compliance, and operational complexity. In a real scenario, you'd probably want to provide your actual business reasons. Maybe you have specific compliance requirements, or maybe you just got a better pricing deal in those regions. Whatever the reason, be specific.

The system then asks who created this policy. This creates an audit trail and accountability.

[user]
```md
> a-team
```

[agent]
```md
  dot-ai - manageOrgData (MCP)(dataType: "policy", operation:
                              "create", sessionId:
                              "policy-1756817416636-39f91684",
                              step: "namespace-scope", response:
                              "a-team")
  ⎿  {
       "success": true,
       "operation": "create",
     … +49 lines (ctrl+r to expand)

⏺ Policy Namespace Scope

  Your policy can be applied cluster-wide or limited to specific
  namespaces.

  Available Namespaces in Your Cluster:

  a-team, atlas-operator, b-team, crossplane-system, default,
  ingress-nginx, kube-node-lease, kube-public, kube-system,
  kyverno, local-path-storage

  Choose the scope for your policy:

  1. Apply to all namespaces (cluster-wide enforcement)
    - Type: all or 1
  2. Apply only to specific namespaces (inclusive list)
    - Type: include: namespace1, namespace2, namespace3
    - Example: include: production, staging
  3. Apply to all namespaces EXCEPT specific ones (exclusion list)
    - Type: exclude: namespace1, namespace2
    - Example: exclude: kube-system, kube-public

  Your choice: [Type your selection]

  ---
  Examples:

  For cluster-wide policy:
  all

  To apply only to production and staging:
  include: production, staging

  To exclude system namespaces:
  exclude: kube-system, kube-public, kube-node-lease

  Note

  System namespaces (kube-system, kube-public, kube-node-lease)
  are often excluded from policies to prevent conflicts with
  Kubernetes core functionality. Consider whether your policy
  should apply to these system namespaces.
```

Here's where it gets interesting. The system discovers what namespaces exist in your cluster and asks where this policy should apply. You can make it cluster-wide, limit it to specific namespaces, or exclude certain namespaces. This granular control is important. Maybe your development namespaces don't need the same rigid regional requirements as production. Notice that we're still just gathering information at this point. The system needs to know the scope, regardless of whether it will eventually generate a Kyverno policy, an OPA policy, or something else.

[user]
```md
> Include a-team and b-team Namespaces
```

[agent]
```md
⏺ dot-ai - manageOrgData (MCP)(dataType: "policy", operation:
                              "create", sessionId:
                              "policy-1756817416636-39f91684",
                              step: "kyverno-generation",
                              response: "include: a-team, b-team")
  ⎿  {
       "success": true,
       "operation": "create",
     … +627 lines (ctrl+r to expand)

⏺ Please review your policy intent:

  Description: Databases in Google Cloud (GCP) should always run
  in the us-east1 region, those in Azure should always run in the
  eastus region, and those in AWS should run in the us-east-1
  region.
  Triggers: GCP DB, Google DB, AWS DB, Azure DB, Cloud SQL, RDS,
  Cosmos DB, database instances, data stores, SQL databases, NoSQL
   databases, managed databases, database services, persistent
  storage, data warehouses, BigQuery, Redshift, Azure SQL,
  DynamoDB, Firestore, Cloud Spanner, DocumentDB, database
  clusters, replica sets
  Rationale: This policy ensures consistent regional deployment of
   database resources across cloud providers to optimize
  performance, reduce latency, maintain regulatory compliance, and
   standardize infrastructure operations. It mitigates risks of
  increased latency from cross-region traffic, potential
  regulatory violations, and operational complexity from scattered
   resources.
  Created By: a-team

  I've also generated a Kyverno ClusterPolicy that enforces this
  requirement:

  Generated Kyverno Policy:
  # MANDATORY SCHEMA-BY-SCHEMA ANALYSIS
  #
  # databaseinstances.sql.gcp.m.upbound.io: HAS
  spec.forProvider.region + GCP provider → MUST generate rule
  # databases.sql.gcp.m.upbound.io: NO region field → Can skip
  # globalclusters.rds.aws.m.upbound.io: HAS
  spec.forProvider.region + AWS provider → MUST generate rule
  # users.sql.gcp.m.upbound.io: NO region field → Can skip
  # clusters.rds.aws.m.upbound.io: HAS spec.forProvider.region +
  AWS provider → MUST generate rule
  # sslcerts.sql.gcp.m.upbound.io: NO region field → Can skip
  # clusterinstances.rds.aws.m.upbound.io: HAS
  spec.forProvider.region + AWS provider → MUST generate rule
  # instances.rds.aws.m.upbound.io: HAS spec.forProvider.region +
  AWS provider → MUST generate rule
  # sqls.devopstoolkit.live: HAS spec.region + multi-cloud → MUST
  generate rule
  # manageddatabaselogicaldatabases.database.upcloud.com: NO
  region field + UpCloud only → Can skip
  # servers.dbforpostgresql.azure.m.upbound.io: HAS
  spec.forProvider.location + Azure provider → MUST generate rule
  # databases.dbforpostgresql.azure.m.upbound.io: NO location
  field → Can skip
  # manageddatabasepostgresqls.database.upcloud.com: HAS
  spec.forProvider.zone + UpCloud only → Can skip
  # optiongroups.rds.aws.m.upbound.io: HAS spec.forProvider.region
   + AWS provider → MUST generate rule
  # parametergroups.rds.aws.m.upbound.io: HAS
  spec.forProvider.region + AWS provider → MUST generate rule
  # flexibleservers.dbforpostgresql.azure.m.upbound.io: HAS
  spec.forProvider.location + Azure provider → MUST generate rule
  # flexibleserverdatabases.dbforpostgresql.azure.m.upbound.io: NO
   location field → Can skip
  # manageddatabasemysqls.database.upcloud.com: HAS
  spec.forProvider.zone + UpCloud only → Can skip
  # manageddatabaseredis.database.upcloud.com: HAS
  spec.forProvider.zone + UpCloud only → Can skip
  # subnetgroups.rds.aws.m.upbound.io: HAS spec.forProvider.region
   + AWS provider → MUST generate rule
  # clusterparametergroups.rds.aws.m.upbound.io: HAS
  spec.forProvider.region + AWS provider → MUST generate rule
  # clusterproviderconfigs.gcp.m.upbound.io: NO region field → Can
   skip
  # dbsnapshotcopies.rds.aws.m.upbound.io: HAS
  spec.forProvider.region + AWS provider → MUST generate rule
  # instancestates.rds.aws.m.upbound.io: HAS
  spec.forProvider.region + AWS provider → MUST generate rule
  # manageddatabaseopensearches.database.upcloud.com: HAS
  spec.forProvider.zone + UpCloud only → Can skip
  # dbinstanceautomatedbackupsreplications.rds.aws.m.upbound.io:
  HAS spec.forProvider.region + AWS provider → MUST generate rule
  # databases.postgresql.sql.crossplane.io: NO region field → Can
  skip
  # databases.mssql.sql.crossplane.io: NO region field → Can skip
  # availabilityzonegroups.ec2.aws.m.upbound.io: HAS
  spec.forProvider.region + AWS provider → Can skip (not database)
  # atlasschemas.db.atlasgo.io: NO region field → Can skip
  # databases.mysql.sql.crossplane.io: NO region field → Can skip
  # providerconfigs.gcp.m.upbound.io: NO region field → Can skip
  # atlasmigrations.db.atlasgo.io: NO region field → Can skip
  # configurations.dbforpostgresql.azure.m.upbound.io: NO location
   field → Can skip
  # providerconfigs.gcp.upbound.io: NO region field → Can skip
  # clustersnapshots.rds.aws.m.upbound.io: HAS
  spec.forProvider.region + AWS provider → MUST generate rule
  # virtualnetworkrules.dbforpostgresql.azure.m.upbound.io: NO
  location field → Can skip
  # manageddatabaseusers.database.upcloud.com: NO region field →
  Can skip
  # instances.ec2.aws.m.upbound.io: HAS spec.forProvider.region +
  AWS provider → Can skip (not database)
  # placementgroups.ec2.aws.m.upbound.io: HAS
  spec.forProvider.region + AWS provider → Can skip (not database)
  #
  flexibleserverconfigurations.dbforpostgresql.azure.m.upbound.io:
   NO location field → Can skip
  # serverkeys.dbforpostgresql.azure.m.upbound.io: NO location
  field → Can skip
  # servergroups.server.upcloud.com: NO region field → Can skip
  # flexibleserveractivedirectoryadministrators.dbforpostgresql.az
  ure.m.upbound.io: NO location field → Can skip
  # ebsencryptionbydefaults.ec2.aws.m.upbound.io: HAS
  spec.forProvider.region + AWS provider → Can skip (not database)
  # storages.storage.upcloud.com: HAS spec.forProvider.zone +
  UpCloud only → Can skip
  # activedirectoryadministrators.dbforpostgresql.azure.m.upbound.
  io: NO location field → Can skip
  # firewallrules.dbforpostgresql.azure.m.upbound.io: NO location
  field → Can skip
  # clusterendpoints.rds.aws.m.upbound.io: HAS
  spec.forProvider.region + AWS provider → MUST generate rule
  # providerconfigusages.gcp.upbound.io: NO region field → Can
  skip
  #
  # RESOURCES REQUIRING VALIDATION RULES: GCP, AWS, Azure database
   resources with region/location fields
  #
  apiVersion: kyverno.io/v1
  kind: ClusterPolicy
  metadata:
    name: require-database-regional-compliance
    labels:
      policy-intent/id: 2e02e3a8-0d74-495a-b319-2eb8bb0defba
    annotations:
      policy-intent/description: "Databases in Google Cloud (GCP)
  should always run in the us-east1 region, those in Azure should
  always run in the eastus region, and those in AWS should run in
  the us-east-1 region."
      policy-intent/rationale: "This policy ensures consistent
  regional deployment of database resources across cloud providers
   to optimize performance, reduce latency, maintain regulatory
  compliance, and standardize infrastructure operations. It
  mitigates risks of increased latency from cross-region traffic,
  potential regulatory violations, and operational complexity from
   scattered resources."
  spec:
    background: false
    validationFailureAction: Enforce
    rules:
    - name: validate-gcp-database-region
      match:
        any:
        - resources:
            kinds:
            - sql.gcp.m.upbound.io/v1beta1/DatabaseInstance
            namespaces:
            - a-team
            - b-team
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
          - expression: >-
              !has(object.spec.forProvider.region) ||
              object.spec.forProvider.region == 'us-east1'
            message: "GCP database instances must be deployed in
  the us-east1 region for compliance"
          - expression: >-
              !has(object.spec.initProvider.region) ||
              object.spec.initProvider.region == 'us-east1'
            message: "GCP database instances must be deployed in
  the us-east1 region for compliance"
    - name: validate-aws-rds-global-cluster-region
      match:
        any:
        - resources:
            kinds:
            - rds.aws.m.upbound.io/v1beta1/GlobalCluster
            namespaces:
            - a-team
            - b-team
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
          - expression: >-
              has(object.spec.forProvider.region) &&
              object.spec.forProvider.region == 'us-east-1'
            message: "AWS RDS Global Clusters must be deployed in
  the us-east-1 region for compliance"
    - name: validate-aws-rds-cluster-region
      match:
        any:
        - resources:
            kinds:
            - rds.aws.m.upbound.io/v1beta1/Cluster
            namespaces:
            - a-team
            - b-team
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
          - expression: >-
              has(object.spec.forProvider.region) &&
              object.spec.forProvider.region == 'us-east-1'
            message: "AWS RDS Clusters must be deployed in the
  us-east-1 region for compliance"
    - name: validate-aws-rds-cluster-instance-region
      match:
        any:
        - resources:
            kinds:
            - rds.aws.m.upbound.io/v1beta1/ClusterInstance
            namespaces:
            - a-team
            - b-team
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
          - expression: >-
              has(object.spec.forProvider.region) &&
              object.spec.forProvider.region == 'us-east-1'
            message: "AWS RDS Cluster Instances must be deployed
  in the us-east-1 region for compliance"
    - name: validate-aws-rds-instance-region
      match:
        any:
        - resources:
            kinds:
            - rds.aws.m.upbound.io/v1beta1/Instance
            namespaces:
            - a-team
            - b-team
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
          - expression: >-
              has(object.spec.forProvider.region) &&
              object.spec.forProvider.region == 'us-east-1'
            message: "AWS RDS Instances must be deployed in the
  us-east-1 region for compliance"
    - name: validate-multicloud-sql-region
      match:
        any:
        - resources:
            kinds:
            - devopstoolkit.live/v1beta1/SQL
            namespaces:
            - a-team
            - b-team
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
          - expression: >-
              !has(object.spec.region) ||
              object.spec.region == 'us-east1' ||
              object.spec.region == 'us-east-1' ||
              object.spec.region == 'eastus'
            message: "Multi-cloud SQL databases must be deployed
  in approved regions (us-east1 for GCP, us-east-1 for AWS, eastus
   for Azure)"
    - name: validate-azure-postgresql-server-region
      match:
        any:
        - resources:
            kinds:
            - dbforpostgresql.azure.m.upbound.io/v1beta1/Server
            namespaces:
            - a-team
            - b-team
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
          - expression: >-
              !has(object.spec.forProvider.location) ||
              object.spec.forProvider.location == 'eastus'
            message: "Azure PostgreSQL Servers must be deployed in
   the eastus region for compliance"
          - expression: >-
              !has(object.spec.initProvider.location) ||
              object.spec.initProvider.location == 'eastus'
            message: "Azure PostgreSQL Servers must be deployed in
   the eastus region for compliance"
    - name: validate-aws-rds-option-group-region
      match:
        any:
        - resources:
            kinds:
            - rds.aws.m.upbound.io/v1beta1/OptionGroup
            namespaces:
            - a-team
            - b-team
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
          - expression: >-
              has(object.spec.forProvider.region) &&
              object.spec.forProvider.region == 'us-east-1'
            message: "AWS RDS Option Groups must be deployed in
  the us-east-1 region for compliance"
    - name: validate-aws-rds-parameter-group-region
      match:
        any:
        - resources:
            kinds:
            - rds.aws.m.upbound.io/v1beta1/ParameterGroup
            namespaces:
            - a-team
            - b-team
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
          - expression: >-
              has(object.spec.forProvider.region) &&
              object.spec.forProvider.region == 'us-east-1'
            message: "AWS RDS Parameter Groups must be deployed in
   the us-east-1 region for compliance"
    - name: validate-azure-postgresql-flexible-server-region
      match:
        any:
        - resources:
            kinds:
            -
  dbforpostgresql.azure.m.upbound.io/v1beta1/FlexibleServer
            namespaces:
            - a-team
            - b-team
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
          - expression: >-
              !has(object.spec.forProvider.location) ||
              object.spec.forProvider.location == 'eastus'
            message: "Azure PostgreSQL Flexible Servers must be
  deployed in the eastus region for compliance"
          - expression: >-
              !has(object.spec.initProvider.location) ||
              object.spec.initProvider.location == 'eastus'
            message: "Azure PostgreSQL Flexible Servers must be
  deployed in the eastus region for compliance"
    - name: validate-aws-rds-subnet-group-region
      match:
        any:
        - resources:
            kinds:
            - rds.aws.m.upbound.io/v1beta1/SubnetGroup
            namespaces:
            - a-team
            - b-team
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
          - expression: >-
              has(object.spec.forProvider.region) &&
              object.spec.forProvider.region == 'us-east-1'
            message: "AWS RDS Subnet Groups must be deployed in
  the us-east-1 region for compliance"
    - name: validate-aws-rds-cluster-parameter-group-region
      match:
        any:
        - resources:
            kinds:
            - rds.aws.m.upbound.io/v1beta1/ClusterParameterGroup
            namespaces:
            - a-team
            - b-team
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
          - expression: >-
              has(object.spec.forProvider.region) &&
              object.spec.forProvider.region == 'us-east-1'
            message: "AWS RDS Cluster Parameter Groups must be
  deployed in the us-east-1 region for compliance"
    - name: validate-aws-rds-db-snapshot-copy-region
      match:
        any:
        - resources:
            kinds:
            - rds.aws.m.upbound.io/v1beta1/DBSnapshotCopy
            namespaces:
            - a-team
            - b-team
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
          - expression: >-
              has(object.spec.forProvider.region) &&
              object.spec.forProvider.region == 'us-east-1'
            message: "AWS RDS DB Snapshot Copies must be deployed
  in the us-east-1 region for compliance"
    - name: validate-aws-rds-instance-state-region
      match:
        any:
        - resources:
            kinds:
            - rds.aws.m.upbound.io/v1beta1/InstanceState
            namespaces:
            - a-team
            - b-team
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
          - expression: >-
              has(object.spec.forProvider.region) &&
              object.spec.forProvider.region == 'us-east-1'
            message: "AWS RDS Instance States must be deployed in
  the us-east-1 region for compliance"
    - name: validate-aws-rds-automated-backups-replication-region
      match:
        any:
        - resources:
            kinds:
            - rds.aws.m.upbound.io/v1beta1/DBInstanceAutomatedBack
  upsReplication
            namespaces:
            - a-team
            - b-team
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
          - expression: >-
              has(object.spec.forProvider.region) &&
              object.spec.forProvider.region == 'us-east-1'
            message: "AWS RDS DB Instance Automated Backups
  Replications must be deployed in the us-east-1 region for
  compliance"
    - name: validate-aws-rds-cluster-snapshot-region
      match:
        any:
        - resources:
            kinds:
            - rds.aws.m.upbound.io/v1beta1/ClusterSnapshot
            namespaces:
            - a-team
            - b-team
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
          - expression: >-
              has(object.spec.forProvider.region) &&
              object.spec.forProvider.region == 'us-east-1'
            message: "AWS RDS Cluster Snapshots must be deployed
  in the us-east-1 region for compliance"
    - name: validate-aws-rds-cluster-endpoint-region
      match:
        any:
        - resources:
            kinds:
            - rds.aws.m.upbound.io/v1beta1/ClusterEndpoint
            namespaces:
            - a-team
            - b-team
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
          - expression: >-
              has(object.spec.forProvider.region) &&
              object.spec.forProvider.region == 'us-east-1'
            message: "AWS RDS Cluster Endpoints must be deployed
  in the us-east-1 region for compliance"

  Choose what to do:

  1. Apply Kyverno policy to cluster - Store policy intent AND
  deploy enforcement to cluster
  2. Store policy intent only - Save for AI guidance without
  cluster enforcement
  3. Cancel - Do nothing

  ⚠️ Warning: Option 1 will deploy active policy enforcement to
  your cluster.

⏺ Update Todos
  ⎿  ☒ Create a new policy using the manageOrgData tool
```

Now comes the magic. The system took all the information we provided and generated two things. First, it created a policy intent, generated embeddings from it, and stored those embeddings in the vector database. This is what enables semantic search later. When AI needs to work with your infrastructure, it can search for relevant policies using natural language, not just exact keyword matches. Second, it generated an actual Kyverno ClusterPolicy with all the technical implementation details.

> If the agent did not show the complete Kyverno policy, you can tell it to `show the entire Kyverno policy`.

Look at how comprehensive this is! The system analyzed the available Custom Resource Definitions in your cluster, figured out which ones are database-related based on our triggers, and generated specific validation rules for each one. It knows that GCP databases use `forProvider.region`, Azure uses `forProvider.location`, and it created CEL expressions to validate them. This isn't some generic template. It's a policy specifically crafted for your cluster's actual resources.

Think about what just happened here. The system generated validation rules for over 20 different resource types. If you had to do this manually, you'd need to identify which CRDs in your cluster are database-related, fetch the schema for each one, figure out which fields contain region or location information, write the appropriate CEL expressions for each, handle both `forProvider` and `initProvider` patterns, and wrap it all in proper Kyverno syntax with namespace scoping. That's **hours of tedious work, condensed into a few seconds.**

The system gives you three options: apply it to the cluster immediately, store just the intent for AI guidance without enforcement, or cancel. This flexibility is important. Maybe you want AI to know about the policy but aren't ready to enforce it yet. Or maybe you want immediate enforcement. Your choice.

*We could instruct the AI to save the Kyverno policy to a file and manage it through GitOps with Argo CD or Flux, but I didn't set those up for this demo. In a real environment, you'd probably want that GitOps workflow.*

Let's go with option 1 and apply it directly to the cluster.

[user]
```md
> 1
```

[agent]
```md
  dot-ai - manageOrgData (MCP)(dataType: "policy", operation:
                              "create", sessionId:
                              "policy-1756817416636-39f91684",
                              step: "complete", response:
                              "apply-to-cluster")
  ⎿  {
       "success": true,
       "operation": "create",
     … +386 lines (ctrl+r to expand)

⏺ Policy Applied to Cluster Successfully!

  Policy ID: 2e02e3a8-0d74-495a-b319-2eb8bb0defba
  Description: Databases in Google Cloud (GCP) should always run
  in the us-east1 region, those in Azure should always run in the
  eastus region, and those in AWS should run in the us-east-1
  region.
  Created By: a-team
  Deployed Policy: policy-2e02e3a8-0d74-495a-b319-2eb8bb0defba

  The policy intent has been stored in the database and the
  Kyverno policy has been applied to your cluster. The policy is
  now actively enforcing database regional compliance for the
  a-team and b-team namespaces.

  Apply output:
  clusterpolicy.kyverno.io/require-database-regional-compliance
  created
```

Perfect! The policy has been applied to the cluster. We now have both the policy intent stored in the vector database for AI semantic search and the actual Kyverno policy enforcing our regional requirements. This dual approach means AI knows about the policy and can consider it when working with infrastructure, while Kubernetes actively enforces it as the last line of defense.

Let's verify our policy was stored correctly by listing all the policies in the system.

[user]
```md
> List policies
```

[agent]
```md
  dot-ai - manageOrgData (MCP)(dataType: "policy", operation:
                              "list")
  ⎿  {
       "success": true,
       "operation": "list",
     … +123 lines (ctrl+r to expand)

⏺ Found 3 policy intents:

  1. Database Regional Compliance (ID:
  2e02e3a8-0d74-495a-b319-2eb8bb0defba)
    - Description: Databases in Google Cloud (GCP) should always
  run in the us-east1 region, those in Azure should always run in
  the eastus region, and those in AWS should run in the us-east-1
  region.
    - Created: 2025-09-02 by a-team
    - Status: Deployed
  (policy-2e02e3a8-0d74-495a-b319-2eb8bb0defba)
  2. No Latest Tag (ID: 7a95808e-1829-4084-b06f-0ef538fd48eb)
    - Description: Applications must NEVER use the latest tag.
    - Created: 2025-08-30 by a-team
    - Status: Deployed
  (policy-7a95808e-1829-4084-b06f-0ef538fd48eb)
  3. SQL Composition Reference (ID:
  ef434183-7872-4d32-bf1f-dc1c3c270f21)
    - Description: sqls.devopstoolkit.live must have
  spec.crossplane.compositionRef.name set. If using AWS, it MUST
  be aws-postgresql, if Google Cloud (GCP) it MUST be
  google-postgresql, and if Azure it MUST be azure-postgresql.
    - Created: 2025-08-30 by a-team
    - Status: Deployed
  (policy-ef434183-7872-4d32-bf1f-dc1c3c270f21)

  All policies are currently deployed and active in your cluster.
```

There it is! Our new database regional compliance policy is listed first, along with two others I created earlier. Each policy has a unique ID, description, creation metadata, and deployment status. This is the information that gets embedded and stored in the vector database for semantic search. When AI needs to work with your infrastructure, it can search through these policies to understand what's allowed and what isn't.

## Policy Enforcement Demo

Now let's see these policies in action. We'll ask the AI to create a database in Azure. Behind the scene, the MCP tool will discover available resources, match them to organizational patterns, and apply our policies. But what we're interested in today is how it uses the policy information we just stored to ensure compliance with our regional requirements.

[user]
```md
> Create a database in Azure
```

[agent]
```md
⏺ I'll help you create a database in Azure using Kubernetes. Let
  me start by creating a todo list to track this task.

  Update Todos
  ⎿  ☐ Get Azure database recommendations from dot-ai
     ☐ Configure database solution based on requirements
     ☐ Generate Kubernetes manifests for Azure database
     ☐ Deploy database to Azure via Kubernetes

⏺ Now let me get recommendations for creating a database in Azure:

  dot-ai - recommend (MCP)(intent: "create database in Azure")
  ⎿  {
       "status": "clarification_available",
       "intent": "create database in Azure",
     … +164 lines (ctrl+r to expand)

⏺ I need more details to provide the best database solution for
  Azure. Here are some key questions to help me recommend the
  right approach:

  Database Type & Technology:
  - What database engine do you need (MySQL, PostgreSQL, SQL
  Server, MongoDB, etc.)?
  - Do you have a specific version requirement?

  Deployment Approach:
  - Do you want to use managed Azure Database services (like Azure
   Database for PostgreSQL) or deploy a self-managed database in
  Kubernetes?
  - Are you planning to deploy this in Azure Kubernetes Service
  (AKS)?

  Requirements:
  - What are your expected performance requirements (concurrent
  connections, transactions per second)?
  - Do you need high availability with automatic failover?
  - Is this for development, staging, or production?

  Security & Compliance:
  - What security requirements do you have (encryption, compliance
   standards)?
  - Do you need integration with Azure Active Directory?

  Could you provide details on these points so I can recommend the
   most suitable database solution for your needs?
```

The AI starts with a guided workflow to understand what we need. Let's keep it simple.

[user]
```md
> Publicly accessible PostgreSQL
```

[agent]
```md
  Update Todos
  ⎿  ☐ Get Azure database recommendations from dot-ai
     ☐ Configure database solution based on requirements
     ☐ Generate Kubernetes manifests for Azure database
     ☐ Deploy database to Azure via Kubernetes

  dot-ai - recommend (MCP)(intent: "create publicly accessible
                          PostgreSQL database in Azure", final:
                          true)
  ⎿ {
      "intent": "create publicly accessible PostgreSQL database in
     Azure",
    … +194 lines (ctrl+r to expand)


  Update Todos
  ⎿  ☒ Get Azure database recommendations from dot-ai
     ☐ Configure database solution based on requirements
     ☐ Generate Kubernetes manifests for Azure database
     ☐ Deploy database to Azure via Kubernetes

⏺ I found 3 solutions for creating a publicly accessible
  PostgreSQL database in Azure. Please choose which one you'd
  prefer:

  Option 1: Multi-cloud PostgreSQL (Score: 98)

  Resources: SQL (devopstoolkit.live/v1beta1)
  - Multi-cloud abstraction with Azure support
  - Built-in secret management and schema handling
  - Simplified configuration
  - Uses organizational pattern for PostgreSQL deployment

  Option 2: Azure PostgreSQL Flexible Server (Score: 95)

  Resources: ResourceGroup, FlexibleServer,
  FlexibleServerFirewallRule, FlexibleServerDatabase
  - Modern Azure PostgreSQL Flexible Server
  - Complete solution with all Azure components
  - Production-ready with enterprise features
  - Uses Azure ResourceGroup organizational pattern

  Option 3: Traditional Azure PostgreSQL Server (Score: 92)

  Resources: ResourceGroup, Server, FirewallRule, Database
  - Classic Azure Database for PostgreSQL
  - Proven stability and reliability
  - Enterprise security and backup features
  - Uses Azure ResourceGroup organizational pattern

  Which solution would you like me to configure and deploy?
```

It presents three options. Let's go with the multi-cloud PostgreSQL option.

[user]
```md
> 1
```

[agent]
```md
  Update Todos
  ⎿  ☒ Get Azure database recommendations from dot-ai
     ☐ Configure database solution based on requirements
     ☐ Generate Kubernetes manifests for Azure database
     ☐ Deploy database to Azure via Kubernetes

  dot-ai - chooseSolution (MCP)(solutionId: "sol_2025-09-02T131741
                               _1e3c6f4d5c94")
  ⎿  {
       "status": "stage_questions",
       "solutionId": "sol_2025-09-02T131741_1e3c6f4d5c94",
     … +90 lines (ctrl+r to expand)

⏺ Great choice! Now I need some basic configuration details for
  your multi-cloud PostgreSQL database:

  Required Configuration:

  1. Database name: What should we name your PostgreSQL database?
  (must be lowercase, alphanumeric with hyphens)
  2. Namespace: Which namespace should it be deployed to?
    - Options: a-team, atlas-operator, b-team, crossplane-system,
  default, ingress-nginx, kube-node-lease, kube-public,
  kube-system, kyverno, local-path-storage
  3. Database size: What size do you need?
    - Options: small, medium, large
  4. Cloud provider composition: Since you want Azure, this should
   be azure-postgresql
  5. Region: Which Azure region?
    - Options: eastus, us-east-1, us-east1 (recommend eastus for
  Azure)

  Please provide these details so I can configure your database
  solution.
```

**This is the moment where policies come alive.**

Now watch this. The system is applying policies it found through semantic search. First, the cloud provider composition: "Since you want Azure, this should be azure-postgresql". That's enforcing one of the policies I created earlier. Second, and more importantly for our demo, look at the region options. The system is specifically recommending `eastus` for Azure.

This isn't random. The MCP tool performed a semantic search, found our database regional compliance policy, understood we're creating an Azure database, and is now guiding us toward the compliant choice. The AI isn't just blindly following rules. It understood the intent of our policy and is proactively helping us stay compliant. This is the difference between AI that knows your policies and AI that doesn't.

[user]
```md
> 1. my-db, 2. a-team; 3. small, 4. azure-postgresql, 5. eastus
```

[agent]
```md
⏺ Perfect! Now let's configure the basic settings for your
  PostgreSQL database:

  Basic Configuration (optional - you can skip this stage):

  1. Databases to create: What databases should be created in your
   PostgreSQL instance?
    - Options: app, users, orders, products, analytics, logs (can
  select multiple)
  2. PostgreSQL version: Which version would you like?
    - Options: 15, 14, 13, 12 (default: 15)
  3. Secret store name: What should we name the secret store for
  database credentials?
    - Default: postgres-secrets
  4. Push credentials to store: Should database credentials be
  automatically pushed to the secret store?
    - Default: true

  You can provide specific answers for these settings or just say
  "skip" to proceed to advanced configuration.
```

I'll configure some basic settings and move through the rest of the workflow quickly.

[user]
```md
> 1. db-01, db-02; 2. 15; 4. false
```

[agent]
```md
  dot-ai - answerQuestion (MCP)(solutionId: "sol_2025-09-02T131741
                               _1e3c6f4d5c94", stage: "basic",
                               answers:
                               {"databases":["db-01","db-02"],"ver
                               sion":"15","push-to-store":false})
  ⎿  {
       "status": "stage_questions",
       "solutionId": "sol_2025-09-02T131741_1e3c6f4d5c94",
     … +78 lines (ctrl+r to expand)

⏺ I notice you only answered 3 of the 4 questions. Would you like
  to specify the secret store name (default is "postgres-secrets")
   or should I proceed with the default? You can:

  - Provide the secret store name
  - Say "default" to use "postgres-secrets"
  - Say "proceed" to continue with current settings

> proceed

  dot-ai - answerQuestion (MCP)(solutionId: "sol_2025-09-02T131741
                               _1e3c6f4d5c94", stage: "basic",
                               answers: {"databases":["db-01","db-
                               02"],"version":"15","push-to-store"
                               :false,"secret-store-name":"postgre
                               s-secrets"})
  ⎿  {
       "status": "stage_questions",
       "solutionId": "sol_2025-09-02T131741_1e3c6f4d5c94",
     … +95 lines (ctrl+r to expand)

⏺ Great! Now let's configure the advanced settings for your
  PostgreSQL database:

  Advanced Configuration (optional - you can skip this stage):

  1. Custom SQL schemas: Do you need to run custom SQL schemas on
  specific databases?
  2. Root password key: What key should be used when pulling the
  root password from secrets? (default: root-password)
  3. Pull to cluster: Should credentials be pulled to a specific
  cluster?
  4. Credential namespace: Which namespace should receive the
  pulled credentials?
    - Options: a-team, atlas-operator, b-team, crossplane-system,
  default, ingress-nginx, etc. (default: default)
  5. Composition update policy: How should composition updates be
  handled?
    - Options: Automatic, Manual (default: Manual)
  6. Network address: Do you need to specify a custom network
  address for UpCloud deployment?
  7. Resource labels: Do you want to add custom labels to the
  database resource? (format: key1=value1,key2=value2)

  You can provide specific answers for any of these settings or
  just say "skip" to proceed to the final open configuration
  stage.
```

The system continues gathering configuration details. I'll ask it to generate a sample schema.

[user]
```md
> 1. Write a simple sample schema for me.
```

[agent]
```md
  dot-ai - answerQuestion (MCP)(solutionId: "sol_2025-09-02T131741
                               _1e3c6f4d5c94", stage: "advanced",
                               answers: {"schemas":"CREATE TABLE
                               users (id SERIAL PRIMARY KEY,
                               username VARCHAR(50) UNIQUE NOT
                               NULL, email VARCHAR(100) UNIQUE NOT
                                NULL, created_at TIMESTAMP DEFAULT
                                CURRENT_TIMESTAMP); CREATE TABLE
                               posts (id SERIAL PRIMARY KEY,
                               user_id INTEGER REFERENCES
                               users(id), title VARCHAR(200) NOT
                               NULL, content TEXT, created_at
                               TIMESTAMP DEFAULT
                               CURRENT_TIMESTAMP);"})
  ⎿  {
       "status": "stage_questions",
       "solutionId": "sol_2025-09-02T131741_1e3c6f4d5c94",
     … +102 lines (ctrl+r to expand)

⏺ Perfect! I've added a simple sample schema that creates:
  - A users table with id, username, email, and created_at columns
  - A posts table with id, user_id (foreign key), title, content,
  and created_at columns

  Would you like to configure any of the remaining advanced
  settings:

  2. Root password key (default: root-password)
  3. Pull to cluster (leave empty for default)
  4. Credential namespace (default: default)
  5. Composition update policy (default: Manual)
  6. Network address (leave empty for automatic)
  7. Resource labels (format: key1=value1,key2=value2)

  Or say "skip" to proceed to the final open configuration stage.
```

Let's skip the remaining advanced settings.

[user]
```md
> skip
```

[agent]
```md
  dot-ai - answerQuestion (MCP)(solutionId: "sol_2025-09-02T131741
                               _1e3c6f4d5c94", stage: "advanced",
                               answers: {})
  ⎿  {
       "status": "stage_questions",
       "solutionId": "sol_2025-09-02T131741_1e3c6f4d5c94",
     … +37 lines (ctrl+r to expand)

⏺ Great! Now for the final open configuration stage:

  Any Additional Requirements?

  Is there anything else about your PostgreSQL database
  requirements or constraints that would help us provide better
  recommendations?

  Examples:
  - Specific performance requirements
  - Backup needs
  - Connection limits
  - Security constraints
  - Integration with existing applications

  You can provide additional details or just say "N/A" if you have
   no additional requirements.
```

No additional requirements needed.

[user]
```md
> N/A
```

[agent]
```md
  dot-ai - answerQuestion (MCP)(solutionId: "sol_2025-09-02T131741
                               _1e3c6f4d5c94", stage: "open",
                               answers: {"open":"N/A"})
  ⎿  {
       "status": "ready_for_manifest_generation",
       "solutionId": "sol_2025-09-02T131741_1e3c6f4d5c94",
     … +47 lines (ctrl+r to expand)

  Update Todos
  ⎿  ☒ Get Azure database recommendations from dot-ai
     ☒ Configure database solution based on requirements
     ☐ Generate Kubernetes manifests for Azure database
     ☐ Deploy database to Azure via Kubernetes

⏺ dot-ai - generateManifests (MCP)(solutionId: "sol_2025-09-02T131
                                  741_1e3c6f4d5c94")
  ⎿  {
       "success": true,
       "status": "manifests_generated",
     … +32 lines (ctrl+r to expand)

⏺ Update Todos
  ⎿  ☒ Get Azure database recommendations from dot-ai
     ☒ Configure database solution based on requirements
     ☒ Generate Kubernetes manifests for Azure database
     ☐ Deploy database to Azure via Kubernetes

⏺ Perfect! I've generated the Kubernetes manifests for your Azure
  PostgreSQL database. The configuration includes:

  - Database name: my-db
  - Namespace: a-team
  - Size: small
  - PostgreSQL version: 15
  - Region: eastus (Azure)
  - Databases: db-01, db-02
  - Sample schema: users and posts tables

  Now I'll deploy it to your Azure Kubernetes cluster:

⏺ dot-ai - deployManifests (MCP)(solutionId: "sol_2025-09-02T13174
                                1_1e3c6f4d5c94")

╭─────────────────────────────────────────────────────────────────────╮
│ Tool use                                                            │
│                                                                     │
│   dot-ai - deployManifests(solutionId:                              │
│   "sol_2025-09-02T131741_1e3c6f4d5c94") (MCP)                       │
│   Deploy Kubernetes manifests from generated solution with          │
│   kubectl apply --wait                                              │
│                                                                     │
│ Do you want to proceed?                                             │
│ ❯ 1. Yes                                                            │
│  2. Yes, and don't ask again for dot-ai - deployManifests commands  │
│  in /Users/viktorfarcic/code/dot-ai-demo                            │
│   3. No, and tell Claude what to do differently (esc)               │
│                                                                     │
╰─────────────────────────────────────────────────────────────────────╯
```

The system generated the manifests and is ready to deploy. Notice that the database configuration includes `region: eastus`, complying with our policy.

*We could skip this last step and instruct client agent to save the manifest into a file so that we manage it through Argo CD or Flux. As I already mentioned, I was too lazy to set up one of those.*

[agent]
```md
⏺ Update Todos
  ⎿  ☒ Get Azure database recommendations from dot-ai
     ☒ Configure database solution based on requirements
     ☒ Generate Kubernetes manifests for Azure database
     ☒ Deploy database to Azure via Kubernetes

⏺ ✅ Successfully deployed your Azure PostgreSQL database!

  Your publicly accessible PostgreSQL database has been created
  with:
  - Name: my-db
  - Namespace: a-team
  - Azure region: eastus
  - PostgreSQL version: 15
  - Databases: db-01, db-02
  - Sample schema with users and posts tables

  The deployment is complete and all components are ready. Your
  database will be provisioned in Azure and accessible publicly
  once the cloud resources finish initializing.
```

The database deployed successfully with the compliant `eastus` region. 

But here's where it gets interesting. Let's test what happens when we try to violate our policy. I'll ask the AI to modify the manifest and change the region to `westus` - which, remember, violates our policy that requires Azure databases to be in `eastus`.

[user]
```md
> Take the manifest we generated earlier with the MCP, change the region to `westus`, and save it to `k8s/db.yaml`.
```

[agent]
```md
⏺ Update Todos
  ⎿  ☐ Read the generated manifest from MCP session
     ☐ Change region from eastus to westus
     ☐ Save modified manifest to k8s/db.yaml

⏺ Update Todos
  ⎿  ☐ Read the generated manifest from MCP session
     ☐ Change region from eastus to westus
     ☐ Save modified manifest to k8s/db.yaml

⏺ Let me get the manifest we generated earlier and modify it:

⏺ Write(k8s/db.yaml)
  ⎿  Wrote 47 lines to k8s/db.yaml
     apiVersion: v1
     kind: ConfigMap
     metadata:
       name: dot-ai-app-my-db-sol-2025-09-02t131741-1e3c6f4d5c94
       namespace: a-team
       labels:
         dot-ai.io/managed: 'true'
         dot-ai.io/app-name: my-db
         dot-ai.io/intent:
     create-publicly-accessible-postgresql-database-in-azure
       annotations:
     … +37 lines (ctrl+r to expand)

⏺ Update Todos
  ⎿  ☒ Read the generated manifest from MCP session
     ☒ Change region from eastus to westus
     ☒ Save modified manifest to k8s/db.yaml

⏺ ✅ Done! I've saved the manifest to k8s/db.yaml with the region
  changed from eastus to westus. The file contains the same
  PostgreSQL configuration as before but now targets the West US
  Azure region.
```

The AI happily changed the region to `westus` and saved the file. It doesn't know this violates our policy because it's just following instructions. 

Time for the moment of truth. Let's see what happens when we try to apply this non-compliant manifest to the cluster.

[user]
```md
> Apply the manifest `k8s/db.yaml` without using the MCP.
```

[agent]
```md
  Bash(kubectl apply -f k8s/db.yaml)
  ⎿  Error: Error from server: error when applying patch:
     {"metadata":{"annotations":{"kubectl.kubernetes.io/last-applie
     d-configuration":"{\"apiVersion\":\"devopstoolkit.live/v1beta1
     \",\"kind\":\"SQL\",\"metadata\":{\"annotations\":{},\"labels\
     ":{\"dot-ai.io/app-name\":\"my-db\",\"dot-ai.io/intent\":\"cre
     ate-publicly-accessible-postgresql-database-in-azure\",\"dot-a
     i.io/managed\":\"true\"},\"name\":\"my-db\",\"namespace\":\"a-
     team\"},\"spec\":{\"crossplane\":{\"compositionRef\":{\"name\"
     :\"azure-postgresql\"}},\"databases\":[\"db-01\",\"db-02\"],\"
     region\":\"westus\",\"schemas\":[{\"database\":\"db-01\",\"sql
     \":\"CREATE TABLE users (id SERIAL PRIMARY KEY, username
     VARCHAR(50) UNIQUE NOT NULL, email VARCHAR(100) UNIQUE NOT
     NULL, created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP); CREATE
     TABLE posts (id SERIAL PRIMARY KEY, user_id INTEGER REFERENCES
      users(id), title VARCHAR(200) NOT NULL, content TEXT,
     created_at TIMESTAMP DEFAULT
     CURRENT_TIMESTAMP);\"}],\"secrets\":{\"pushToStore\":false,\"s
     toreName\":\"postgres-secrets\"},\"size\":\"small\",\"version\
     ":\"15\"}}\n"}},"spec":{"region":"westus"}}
     to:
     Resource: "devopstoolkit.live/v1beta1, Resource=sqls",
     GroupVersionKind: "devopstoolkit.live/v1beta1, Kind=SQL"
     Name: "my-db", Namespace: "a-team"
     for: "k8s/db.yaml": error when patching "k8s/db.yaml":
     admission webhook "validate.kyverno.svc-fail" denied the
     request:

     resource SQL/a-team/my-db was blocked due to the following
     policies

     require-database-regional-compliance:
     … +4 lines (ctrl+r to see all)

⏺ The deployment was blocked by a Kyverno policy that requires
  databases to use specific regions. The policy only allows:
  - us-east1 for GCP
  - us-east-1 for AWS
  - eastus for Azure

  Since you want to use westus, but the policy requires eastus for
   Azure, the manifest was rejected. You'll need to either:

  1. Change the region back to eastus in the manifest
  2. Update the Kyverno policy to allow westus

  Would you like me to change the region back to eastus or help
  you modify the policy?
```

And there it is. **The last line of defense.**

Kyverno blocked the deployment! Even though the AI tried to apply a manifest with `westus`, the cluster rejected it because it violates our policy. The error message is crystal clear: "Multi-cloud SQL databases must be deployed in approved regions (us-east1 for GCP, us-east-1 for AWS, eastus for Azure)".

This is the beauty of the dual approach. The AI was guided toward the right choice through semantic search and policy awareness during the creation process. But when someone (or some AI) tries to bypass that guidance and deploy something non-compliant directly, Kyverno steps in and says "absolutely not".

Without this enforcement, the non-compliant resource would have been created, potentially causing compliance issues, unexpected costs, or performance problems. With it, we have a guarantee that our policies are always respected, no matter how the resources are created.

## Dual Policy Strategy

Let's recap what we've accomplished here. We started with a fundamental problem: policies exist everywhere and nowhere. They're scattered across documentation, Slack messages, wikis, and most importantly, locked in people's heads. AI agents, like humans, stumble through trial and error because they don't know these policies exist. And the technical implementations like Kyverno? They're the bouncers that only tell you "no" after you've already screwed up.

We demonstrated a solution that addresses this problem from two angles. First, we built a guided workflow that extracts policy knowledge from humans through a structured conversation. The system doesn't just collect this information. It generates embeddings and stores them in a vector database, making policies searchable through semantic understanding, not just keyword matching. When AI needs to create infrastructure, it can find relevant policies and understand not just the rules, but the intent behind them.

Second, we automatically generated Kyverno policies from the same information. The system analyzed our cluster's CRDs, identified which resources our policy applies to, and created specific validation rules for over 20 different resource types. Hours of manual work condensed into seconds. This gives us enforcement at the Kubernetes level, the last line of defense when someone or something tries to violate our policies.

The real magic happened when we saw both aspects working together. When we asked AI to create an Azure database, it proactively recommended the compliant region. The policy wasn't just sitting in documentation somewhere. It actively influenced the AI's behavior through semantic search. The system understood we were creating an Azure database and guided us toward `eastus`, exactly as our policy required.

But we also saw what happens when someone tries to bypass that guidance. When we manually changed the region to `westus` and tried to apply it, Kyverno blocked it immediately. This dual approach ensures policies aren't just suggestions or documentation. They're active participants in your infrastructure management, guiding decisions when possible and enforcing rules when necessary.

This isn't about replacing human judgment or creating rigid systems that can't adapt. It's about capturing tribal knowledge before it walks out the door, making it accessible to both humans and AI, and ensuring that our hard-won lessons and business rules are consistently applied. Whether it's a junior engineer, a senior architect, or an AI agent creating resources, they all operate with the same understanding of what's acceptable and why.

The approach we've shown scales beyond just regional compliance for databases. Any policy you can describe can be captured, stored, and enforced this way. Resource limits, naming conventions, security requirements, networking rules, backup policies. If you can explain it, the system can capture it, and both AI and Kubernetes can enforce it.

Ready to implement this yourself? Check out the [DevOps AI Toolkit](https://github.com/vfarcic/dot-ai) project we used in this demo. Try it out, send me your thoughts, open issues if you find bugs or have feature requests. And please, please, please star the repository if you find it useful. Stars help others discover the project and motivate me to keep improving it. This is open source, and your feedback and contributions make it better for everyone.

## Destroy

> Press `ctrl+c` twice to exit Claude Code

```sh
./dot.nu destroy

rm k8s/db.yaml

exit
```


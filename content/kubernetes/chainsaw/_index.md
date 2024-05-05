+++
title = 'Mastering Kubernetes Testing Kyverno Chainsaw!'
date = 2024-04-08T16:00:00+00:00
draft = false
+++

**Testing is important**, no matter what you're working on. If you write Java code, you need to test it. If you're managing infrastructure, you need to test it. If you're working with IoT, you need to test it. There is no excuse not to test while working and before moving it to production. Testing while working allows us to work more efficiently. Testing before moving something to production allows us to have confidence that it will not explode.

Now, today's session will not focus on all the ways we can test something. I will not go into mad rant explaining the importance of **test-driven development**, **test automation**, **CI/CD pipelines**, or anything else related to testing.
<!--more-->

{{< youtube hQJWGzogIiI >}}

Instead, today I want to focus on testing Kubernetes resources.

Most people just write Kubernetes manifests directly as YAML or package them into Helm charts, or Kustomize, or whatever else we might be using. Once it's done, those people would just deploy that something to a Kubernetes cluster. That strategy tends to be successful if you are religious since it relies on prayer to the deity of choice to be successful. "Dear God, bless those resources and let them be run successfully. I have no idea what will happen so I will rely on your infinite wisdom."

**Good engineers test everything, all the time**. They test continuously while developing, and they test through CI/CD pipelines after pushing to Git. The former allows us to write code faster, and the latter gives us needed confidence. Now, when I said that good engineers test everything, I meant **everything**, and that includes Kubernetes resources.

However, we have a problem.

When testing is concerned, the situation with Kubernetes is **pathetic**. We do not have many tools that can help us test Kubernetes resources. We certainly have plethora of tools to test applications running in Kubernetes, but we do not have much to test Kubernetes resources themselves.

There's hope or, to be more precise, there was hope that died. A while ago, we got [KUTTL](https://kuttl.dev).

I already explored KUTTL in [that video](https://youtu.be/ZSTQQNu4laY), so I won't go into details here. Watch it if you haven't already. The link is in the description.

The problem with KUTTL is... No! Wait... It would be easier to explain what is NOT a problem with KUTTL. The main positive thing about KUTTL is that it exists. It's one of the few, if not the only tool of that kind thus making it great simply because there is nothing better.

KUTTL is a project that is not maintained. The last release was published over a year ago. That, by itself, is a huge red flag. On top of that, it is missing too many features that I feel are critical.

* No patching
* No templating
* Insufficient output
* No catch.

I, for example, am in a desperate need of a mechanism to patch or update existing resources, I need some form of templating to avoid repetitions, I need better outputs, and I need some form of a catch mechanism that will allow me to execute additional steps when a test fails so that I get more information about the failure.

That's my list of missing features and I think I found a replacement that meets all of those. That replacement is **[Kyverno Chainsaw](https://kyverno.github.io/chainsaw)**.

Just as my requirements are the result of frustration when working with **KUTTL**. Luckily for me, Kyverno folks faced similar obstacles. They needed a tool to test Kyverno, they used KUTTL, and they quickly hit walls that prevented them to do what they wanted to do.

But... There's a big difference between me and Kyverno folks. While my dissapointed with KUTTL resulted in me crying at night, they did something about it. They tried contributing to KUTTL, but that did not work out for a variety of reasons. Then they decided to create a new tool, a new project, and that is today known as Kyverno Chainsaw.

It's great, so let's see it in action.

## Setup

Make sure that Docker is up and running.

```sh
git clone https://github.com/vfarcic/crossplane-sql

cd crossplane-sql

git pull

git checkout chainsaw
```

> Watch [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell

task cluster-create
```

## Kyverno Chainsaw in Action

Here's what I do when I want to start working on this repo.

```sh
chainsaw test
```

The output is as follows (truncated for brevity).

```
...
=== NAME  chainsaw/azure#01
    | 12:09:52 | azure | @cleanup | DELETE    | DONE  | v1/Namespace @ chainsaw-alive-weevil
=== NAME  chainsaw/azure
    | 12:09:53 | azure | @cleanup | DELETE    | DONE  | v1/Namespace @ chainsaw-touching-treefrog
=== NAME  chainsaw/aws
    | 12:09:56 | aws | @cleanup | DELETE    | DONE  | v1/Namespace @ chainsaw-merry-starfish
--- PASS: chainsaw (0.00s)
    --- PASS: chainsaw/azure#01 (15.37s)
    --- PASS: chainsaw/azure (15.95s)
    --- PASS: chainsaw/aws (18.90s)
PASS
Tests Summary...
- Passed  tests 3
- Failed  tests 0
- Skipped tests 0
Done.
```

That run all the tests related to that project which, by the way, are a few Crossplane Compositions.

That's irrelevant for this story though. I'd run similar set of tests for any type of Kubernetes resources.

Actually, I do not run tests like that but, instead, I run them through Task which executes tests every time I make a change to my Kubernetes manifests or tests themselves. That is also beyond the point. What matters is that I can execute tests whenever I want to validate any of my Kubernetes resources and I can execute something similar from pipeline builds triggered when I push a change to a Git repo.

> If you are not familiar with Task, you should be. You can watch [Say Goodbye to Makefile - Use Taskfile to Manage Tasks in CI/CD Pipelines and Locally](https://youtu.be/Z7EnwBaJzCk) to get familiar with it. It's awesome.

Now, if you are familiar with KUTTL, you might be thinking that Chainsaw is the same, just with a nicer output. If that's what you're thinking, you're wrong. Chainsaw is so much more, and we can see some of that by exploring one of my `Test` definitions.

```sh
cat tests/aws/chainsaw-test.yaml
```

The output is as follows (truncated for brevity).

```yaml
apiVersion: chainsaw.kyverno.io/v1alpha1
kind: Test
metadata:
  name: aws
spec:
  template: true
  bindings:
    - name: hyperscaler
      value: aws
    - name: cluster
      value: eks
  steps:
    - try:
        - apply:
            file: ../common/install.yaml
        ...
```

Just as KUTTL, Chainsaw allows us to run tests based on naming convention where applying resources and testing them is executed based on file name ordering and with asserts always being named `assert`.

However, with the `Test` manifest like this one, we can gain more control over what is executed, when it's executed, what something does, and so on and so forth.

At the very top I'm specifying that Chainsaw should expect some files to serve as `template`. Further on, there are two `bindings`. In this specific case, `hyperscaler` is set to `aws` and `cluster`to `eks`. Templates and bindings allow us to avoid repetition. In my case, there are manifests for Azure, Google Cloud, and, as we can see in this example, for AWS. Since those are often very similar with only changes to a few values, I could define all three once and let Chainsaw replace those bindings with actual values. That's, in a way, similar to what we would normally do with Helm or any other templating engine.

Further on, we have the `try` section that starts with the instruction to apply whatever is defined in `../common/install.yaml`, so let's take a look at that file.

```sh
cat tests/common/install.yaml
```

The output is as follows.

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: (join('-', ['my-db', $hyperscaler, 'password']))
data:
  password: cG9zdGdyZXM=
---
apiVersion: devopstoolkitseries.com/v1alpha1
kind: SQLClaim
metadata:
  name: my-db
spec:
  id: (join('-', ['my-db', $hyperscaler]))
  compositionSelector:
    matchLabels:
      provider: ($hyperscaler)
      db: postgresql
  parameters:
    version: "13.4"
    size: medium
```

This is where I'm using `bindings` we saw in the `Test` file.

Since all three variations are similar, instead of defining three separate files to apply them, I have this one manifest that uses Chainsaw bindings and functions.

For example, the `name` of the secret should be `my-db-aws-password` for AWS, `my-db-azure-password` for Azure, and `my-db-google-password` for GCP. So, instead of hardcoding those values, I'm using the `join` function that combines hard-coded `my-db` and `password` with the `hyperscaler` value defined in `Test` bindings.

We can observe a similar situation with `SQLClaim` `spec.id` field that contains a similar `join` function as the value. Then there is `spec.compositionSelector.matchLabels.provider` that only has the `hyperscaler` value.

As I said, that's similar to what we would expect with something like Helm templating. Now, just to avoid confusion that might lead you to say "but why don't we use Helm templating instead of Chainsaw's syntax"... You'll see later that there is much more to it than simple templating.

Now, to be clear, that `join` function, and many other features of Chainsaw can look scary. The reaction might be "**This is silly**" or "**This is too much to learn**" or "**This does not make sense, I should write tests in a different way**". Those are some of the thoughts I had when I saw Chainsaw for the first time, and we'll comment on how I feel about them now near the end of the video.

Let's go back to the `Test` file and see what else I have there.

```sh
cat tests/aws/chainsaw-test.yaml
```

The output is as follows (truncated for brevity).

```yaml
apiVersion: chainsaw.kyverno.io/v1alpha1
kind: Test
metadata:
  name: aws
spec:
  ...
  steps:
    - try:
        - apply:
            file: ../common/install.yaml
        - assert:
            file: ../common/assert-install.yaml
        - assert:
            file: assert-install.yaml
        ...
```

After `install.yaml` is applied, I want to perform some tests. Some of those are fairly similar no matter whether they are run for AWS resources of something else, while others are very specific to AWS. To solve that, I have two `assert` entries. The common asserts are in the `assert-install.yaml` file, so let's take a look at it.

```sh
cat tests/common/assert-install.yaml
```

The output is as follows.

```yaml
---
apiVersion: devopstoolkitseries.com/v1alpha1
kind: SQLClaim
metadata:
  name: my-db
spec:
  compositionRef:
    name: (join('-', [$hyperscaler, 'postgresql']))
  compositionSelector:
    matchLabels:
      db: postgresql
      provider: ($hyperscaler)
  id: (join('-', ['my-db', $hyperscaler]))
  parameters:
    size: medium
    version: "13.4"
  resourceRef:
    apiVersion: devopstoolkitseries.com/v1alpha1
    kind: SQL
```

This one uses the same `join` function as before but, this time, it is used to generate the asssert manifest. The final output is what is compared against the actual resources in the cluster meaning that it will try to find `SQLClaim` that matches that exact specification.

The second assert manifest is `assert-install.yaml` in the `aws` directory, so let's take a look at that one as well.

```sh
cat tests/aws/assert-install.yaml
```

The output is as follows.

```yaml
---
apiVersion: devopstoolkitseries.com/v1alpha1
kind: SQL
metadata:
  labels:
    crossplane.io/claim-name: my-db
spec:
  claimRef:
    apiVersion: devopstoolkitseries.com/v1alpha1
    kind: SQLClaim
    name: my-db
  compositionRef:
    name: aws-postgresql
  compositionSelector:
    matchLabels:
      db: postgresql
      provider: aws
  compositionUpdatePolicy: Automatic
  id: my-db-aws
  parameters:
    size: medium
    version: "13.4"
  resourceRefs:
  - apiVersion: ec2.aws.upbound.io/v1beta1
    kind: InternetGateway
    name: my-db-aws
  - apiVersion: ec2.aws.upbound.io/v1beta1
    kind: MainRouteTableAssociation
    name: my-db-aws
  - apiVersion: ec2.aws.upbound.io/v1beta1
    kind: RouteTableAssociation
    name: my-db-aws-1a
  - apiVersion: ec2.aws.upbound.io/v1beta1
    kind: RouteTableAssociation
    name: my-db-aws-1b
  - apiVersion: ec2.aws.upbound.io/v1beta1
    kind: RouteTableAssociation
    name: my-db-aws-1c
  - apiVersion: ec2.aws.upbound.io/v1beta1
    kind: RouteTable
    name: my-db-aws
  - apiVersion: ec2.aws.upbound.io/v1beta1
    kind: Route
    name: my-db-aws
  - apiVersion: ec2.aws.upbound.io/v1beta1
    kind: SecurityGroupRule
    name: my-db-aws
  - apiVersion: ec2.aws.upbound.io/v1beta1
    kind: SecurityGroup
    name: my-db-aws
  - apiVersion: ec2.aws.upbound.io/v1beta1
    kind: Subnet
    name: my-db-aws-a
  - apiVersion: ec2.aws.upbound.io/v1beta1
    kind: Subnet
    name: my-db-aws-b
  - apiVersion: ec2.aws.upbound.io/v1beta1
    kind: Subnet
    name: my-db-aws-c
  - apiVersion: ec2.aws.upbound.io/v1beta1
    kind: VPC
    name: my-db-aws
  - apiVersion: kubernetes.crossplane.io/v1alpha1
    kind: ProviderConfig
    name: my-db-aws-sql
  - apiVersion: kubernetes.crossplane.io/v1alpha2
    kind: Object
    name: my-db-aws-secret
  - apiVersion: postgresql.sql.crossplane.io/v1alpha1
    kind: ProviderConfig
    name: my-db-aws
  - apiVersion: rds.aws.upbound.io/v1beta1
    kind: Instance
    name: my-db-aws
  - apiVersion: rds.aws.upbound.io/v1beta1
    kind: SubnetGroup
    name: my-db-aws
---
apiVersion: ec2.aws.upbound.io/v1beta1
kind: InternetGateway
metadata:
  annotations:
    crossplane.io/composition-resource-name: gateway
  labels:
    crossplane.io/claim-name: my-db
  name: my-db-aws
  ownerReferences:
  - apiVersion: devopstoolkitseries.com/v1alpha1
    blockOwnerDeletion: true
    controller: true
    kind: SQL
spec:
  deletionPolicy: Delete
  forProvider:
    region: us-east-1
    tags:
      crossplane-kind: internetgateway.ec2.aws.upbound.io
      crossplane-name: my-db-aws
      crossplane-providerconfig: default
    vpcIdSelector:
      matchControllerRef: true
  managementPolicy: FullControl
  providerConfigRef:
    name: default
...
```

That one is essentially the same as what we would do with KUTTL. There are no templates or functions simply because that assert is not reused. It is very specific to managed resources for AWS. There's nothing fancy there since there is no need for anything fancy.

There's one important note here. Take a look at `spec.resourceRefs`. It is an array of items and the assert validates that exactly the same items specified here are available in the actual resource in the cluster. They even have to be defined in the same order. No more. No less. It needs to be exactly the same.

Now, that's perfectly okay in this specific example but, as you'll see later, that can become a nightmare to assert as we start updating resources. For now, remember that there are currently eighteen entries in `spec.resourceRefs`. That will become important later.

Speaking of updates, let's go back to the `Test` manifest.

```sh
cat tests/aws/chainsaw-test.yaml
```

The output is as follows (truncated for brevity).

```yaml
apiVersion: chainsaw.kyverno.io/v1alpha1
kind: Test
metadata:
  name: aws
spec:
  ...
  steps:
    - try:
        ...
        - patch:
            file: ../common/db.yaml
        - assert:
            file: ../common/assert-db.yaml
        ...
```

After the asserts that validated that the resource initially applied and child resources it created are as they are supposed to be, I wanted to update the `SQLClaim`. With KUTTL, that would mean creating a copy of it, making some modifications, and applying the whole manifest again. That's another example of repetition.

To avoid that, I used Chainsaw's `patch` option, so let's take a look at it.

```sh
cat tests/common/db.yaml
```

The output is as follows.

```yaml
apiVersion: devopstoolkitseries.com/v1alpha1
kind: SQLClaim
metadata:
  name: my-db
spec:
  parameters:
    databases:
      - db-01
      - db-02
      - db-03
```

This is very similar to Kustomize overlay mechanism. Instead of specifying the whole `SQLClaim` again, I'm saying something like "find the `SQLClaim` named `my-db` and update it by adding `spec.parameters.databases` with those three entries. That allows me to avoid repetition, except for the identifiers like `apiVersion`, `kind` and `metadata.name`.

Now, testing that patch could be complicated, but is actually made relatively simple with Chainsaw, so let's take a look at the `assert-db.yaml` file that should validate that change.

```sh
cat tests/common/assert-db.yaml
```

The output is as follows.

```yaml
---
apiVersion: devopstoolkitseries.com/v1alpha1
kind: SQL
metadata:
  labels:
    crossplane.io/claim-name: my-db
spec:
  parameters:
    databases:
      - db-01
      - db-02
      - db-03
  (resourceRefs[?kind == 'Database']):
  - apiVersion: postgresql.sql.crossplane.io/v1alpha1
    name: (join('-', ['my-db', $hyperscaler, 'db-01']))
  - apiVersion: postgresql.sql.crossplane.io/v1alpha1
    name: (join('-', ['my-db', $hyperscaler, 'db-02']))
  - apiVersion: postgresql.sql.crossplane.io/v1alpha1
    name: (join('-', ['my-db', $hyperscaler, 'db-03']))
...
```

The expected outcome of the patch we applied should be the addition of three additional resource spun up by `SQL`. There were eighteen before and now there should be twenty one. Now, at this stage, I'm only interested in those three so having to list all twenty one sounds like a waste of time. That's what I would have to do with KUTTL, but not with Chainsaw.

Instead, I can apply filtering. I can say "Get all `resourceRefs` entries, then filter them so that only those that contain `kind` set to `Database` are left. I don't care about the others. I want only those."

Now, that I filtered entries of the `resourceRefs`, I can assert that they are exactly what I expect them to be. There should be `my-db-aws-db-01`, `my-db-aws-db-02`, and `my-db-aws-db-03`. Since the similar pattern should be asserted for Azure and Google as well, that `name` is assembled using the `join` function and `hyperscaler` variable just as we did before.

There's one more, out of many features, that I'd like to show.

```sh
cat tests/aws/chainsaw-test.yaml
```

The output is as follows (truncated for brevity).

```yaml
apiVersion: chainsaw.kyverno.io/v1alpha1
kind: Test
metadata:
  name: aws
spec:
  ...
  steps:
    - try:
      ...
      catch:
        - get:
            resource: managed
        - describe:
            resource: sqls
            selector: crossplane.io/claim-namespace=$NAMESPACE
```

Just as most other test frameworks, each step has `try`, `catch`, and `finally` blocks. So far, we explored only `try` which I used as a mechanism to `apply` and `patch` resources, and to `assert` them. That's the backbone that everyone will be using. But, in some cases, we might want to leverage `catch` blocks that are executed in case there are failures, and the `finally` block that is executed always at the end. Personally, I did not find `finally` useful in my projects since Chainsaw automatically deletes all the resources created in the `try` block. Nevertheless, if you have some additional cleanup to do, that's when you would do it.

The `catch` block is, at least in my case, very useful. You see, when a test fails, the output Chainsaw provides by default is not always sufficient. I often need more. In this specific case, I need to see all the managed resources running in the cluster as well as the events of the main resource under test.

That's why I added two `catch` statements. The first one is a `get` that returns all `managed` resources and the other one is `describe` that, as the name would suggest, describes the `sqls` resource with the specific label.

Now, to be clear, I could accomplish the same with the `script` that would execute `kubectl get managed` and `kubectl describe` commands. The effect would be the same, and the specifics are probably not important for this story. What matters is that there is a mechanism for me to retrieve additional information in case of the failure of one of the asserts and that I can use that information to deduce what I did wrong.

I only scretched the surface with Kyverno Chainsaw and I'll leave it to you to explore it in more depth, but only after we go through pros and cons.

## Kyverno Chainsaw Pros and Cons

I must admit something. When I first saw Chainsaw, I was not impressed. I thought that it is moving into a wrong direction. I thought that **Kyverno JSON Query Language** might not be the best way to express complex logic and that we would be better off writing those tests in a programming language of choice which, in my case, would be Go.

That might still be the case. There's still something telling me that I might be better of writing tests as "real" code instead of YAML interpolated with Kyverno JSON Query Language. It does not seem right to convert YAML into anything that it is not already; into anything beyond data structures.

Nevertheless, I do think that Chainsaw is, right now, the best tool to test Kubernetes resources. KUTTL is too limiting and results in too much repetition which I was tempted to avoid through some silly workarounds.

Hence, even before we go through Chainsaw pros and cons, I can safely say **use it**. It has its faults, just as anything else does, but it is a huge step forward when compared to what we had in the past.

With that being said, there are a few negative things, so let's start with those first.

**Cons:**
* Docs
* YAML
* Learning curve
* No string interpolation

To begin with, documentation is not very good. I strugled a lot with it. **Docs** might be the main reason why my initial impressions of Chainsaw were negative. Fortunately for me, folks behind the project were very helpful and made me "see the light" eventually. I'm sold now. I am committed to Chainsaw when testing my Kuberentes resources, but that's not thanks to documentation but, rather, very approachable maintainers. Now, to be clear, almost all projects have horrible docs in their early stages so I'm not saying that Chainsaw documentation is worse than others but, rather, that it is just as bad as most early stage projects.

I am not convinced that expressing logic inside **YAML** is a good idea. If we are in need of loops, conditionals, and other constructs, we are, generally speaking, better of with something like Go, or Python, or Java, or CUE, or any other language you might be familiar with. That being said, Kyverno JSON Query Language is very powerful and it works well. It's not something developed specifically for Chainsaw but, rather, something that has been battle tested with Kyverno itself. Hence, I'm split between **this is great** and **this should not be done with YAML**.

Now that I mentioned Kyverno JSON Query Language, there is also a relatively steep **learning curve**, at least when testing is concerned. It might take a while until you get a grip on Kyverno JSON Query Language which is the backbone of Chainsaw. If you are already using Kyverno, you should have no trouble jumping into Chainsaw. But if you are not, it might take a while until you feel comfortable.

Finally, there is **no string interpolation**. Having to use functions like `join` instead of simply interpolating strings and variables makes tests harder to write than they should be and even harder to read by those not versed in it.

There must be other issues but, at least for me, those are the biggest ones and, frankly, I'm not worried about any of those except the doubt whether to use YAML in the first place. I'll explain later why I'm not worried, just after we go through the good things; though the pros.

**Pros:**
* Maintainers
* Templates, bindings, functions
* Output
* Try/catch/finaly
* Patching
* KUTTL compatibility

The first and, potentially most important thing about Chainsaw is the openess, enthysiasm, and proactiveness of the **maintainers**. Now, to be clear, vast majority of the commits are done by two people with occasional commits by a few others so it's not a big project with massive number of maintainers. Never the less, I was impressed to see how proactive they are in reaching to other communities, listening to feedback, adapting the project to the needs of early adopters, and all the other things that projects at such early stage should be doing but are often not. I asked for some features and some of them were already added to the project while others are ongoing. I wanted bindings, I got bindings. I wanted it to be available as a Nix package, and, by the time you watch this, that will probably done. I got burned by not being able to use string interpolations, and I'm sure that will be added soon. That's the reason why I said that I'm not worried about the cons. I'm sure that the issues I discovered will be fixed soon. If you discover something, I'm sure you'll get the same treatment as long as you let the project know what that something is.

Next, **templates, bindings, and functions** are awesome. Chainsaw is piggy backing on Kyverno that already has many of those things nailed down, and that's giving Chainsaw a huge boost. Since the project started by Kyverno, they have the know-how that allows them to move fast on top of what they already know. Those alone enabled me to greatly simplify my tests that were previously written for KUTTL and to do things that I could not do before.

Then there is **output** that is much easier to read and deduce what's wrong.

That output is greatly augmented with additional instructions I can add to the `catch` block so the mere existence of **Try/catch/finaly*** blocks is important, especially since it follows a common practice in testing.

Then there is **patching** which helps greatly with constant updates of the resources for the sake of creating more robust test scenarios.

Finally, if you are already using KUTTL, you can switch to Chainsaw without any modifications. Chainsaw can run **KUTTL** tests as they are. Now, to be clear, you won't get much of the benefits of Chainsaw by only changing the CLI. You will, eventually, have to rewrite some of the tests or start writing new ones in a Chainsaw-friendly way. That's to be expected. But, what matters, at least to KUTTL users, is that you do not need to rewrite all your tests right away. You can just switch to the `chainsaw` CLI right away without any upfront investment and transition manifests later when you see that you'll gain benefits from that effort.

All in all Chainsaw is awesome and I strongly recommend it. It has its quirks but, as far as I know, it is the best tool for testing Kubernetes resources right now.

The community is very welcoming and I suggest you get in touch with them if you have a suggestion for improvement.

## Destroy

```sh
task cluster-destroy

git checkout main

exit
```

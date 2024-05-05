+++
title = 'Is Pkl the Ultimate Data Format? Unveiling the Challenger to YAML, JSON, and CUE'
date = 2024-03-25T16:00:00+00:00
draft = false
+++

**Pkl is CUE killer!**

**Pkl is JSON and YAML killer!**

Those are two sentences I heard lately. The former could be the case, while the latter is just silly. Let me explain.
<!--more-->

{{< youtube Nm1ioWPRRVQ >}}

<img src="/logo/cue.svg" style="width:25%; float:right; padding: 10px">

It all started with a question I got a while ago. **Is Pkl CUE killer?** I love CUE. I use it for all my "complex" manifests, so, naturally, I had to take a look at it. I tried it and I discarded it. It looked silly. However, since it comes from Apple, I gave it another try, and it still looked silly, at least when compared to CUE. Still, I did not give up. I tried it for the third time and something clicked. There's something compelling about it. It started growing on me, so I decided to start using it seriously. I moved a few manifests to [Pkl](https://pkl-lang.org). Now, you might say that a few manifests is not much, but it is, considering that the few manifests I chose to move to `pkl` are around two thousand lines long. It a decently big sample with quite a few repetitions, loops, and conditionals. Hence, I believe I have a good sample for the initial judgement.

Let's start with the use cases.

There are two primary reasons one might adopt [Pkl](https://pkl-lang.org). You might use it as the **configuration format inside your applications**. Instead of reading YAML, JSON, properties, or any other format, our applications might be reading Pkl. I believe that's the primary motivation behind Pkl and, at the same time, something I have zero interest in adopting. YAML works failry well for that. It's easy to read from your code, or from anywhere else. There is no format easier to read than YAML and it is the de-facto standard. I have no intention changing that.

Now, the fact that YAML is easy to read does not mean that it is easy to write. Or, to be more precise, to maintain. Anyone can write YAML, and, I believe that everyone should write YAML but, in some cases, YAML becomes too convoluted when working with big or complex definitions and when we need to update it. That's why many of us are generating rather than writing YAML. That's why we see wide adoption of Helm, Kustomize, Jsonnet, Carvel ytt, and other tools and formats, including my favorite, CUE. We write templates or overlays which produce YAML which, in turn, is consumed by humans or machines.

That's where Pkl comes in. It aims to be a replacement of your favorite tool to **generate static configurations** no matter what the format of that configuration is. It formats into YAML, Json, Jsonnet, PCF, Properties, Plist, TextProto, xml, as well as into its own format. Hence, we can use Pkl to write dynamic templates that can be converted into almost any statis format. That's not much different from CUE. Both are "special" formats that allow us to define data dynamically. One, CUE, is based on Go while the other, Pkl, seem to have taken inspiration from Java and Groovy. At least, that's the impression I got.

Let's take a closer look at it.

## Setup

```sh
git clone https://github.com/vfarcic/crossplane-sql

cd crossplane-sql

git checkout pkl
```

* Download `pkl` by following the instructions at https://pkl-lang.org/main/current/pkl-cli/index.html#download.

## Pkl in Action

Here's one of my Pkl definitions.

```sh
cat pkl/aws.pkl
```

That's a "real world" example of Pkl. That's one of the Pkl definitions I use to generate some of my Crossplane Compositions. We'll take a closer look at it in a moment. For now, let's see what it produces.

We can evaluate that file...

```sh
pkl eval pkl/aws.pkl
```

...and the output is still Pkl except that now templates were converted into the final output.

Now, that won't be useful to majority of us. Unless you plan to rewrite your application to use Pkl format directly, you expect something else. You might, for example, need to format it to YAML. All we have to do to get YAML is to add the `--format` argument.

```sh
pkl eval pkl/aws.pkl --format yaml
```

There we go. I got YAML I needed. Your needs might be different. You might need Json instead, so let's change the value of the `--format` argument, hit the enter key, and...

```sh
pkl eval pkl/aws.pkl --format json
```

...the output is Json.

Pkl also supports Jsonnet, PCF, Properties, Plist, TestProto, and XML.

That's one of the similarities with CUE. Both provide a way to define templates that represent data and data can be output into any data format.

Now, let's get back to the Pkl file we saw earlier and comment on a few of the key features.

```sh
cat pkl/aws.pkl
```

```
...
apiVersion = "apiextensions.crossplane.io/v1"
kind       = "Composition"
...
```

Those few lines look very similar to YAML, except that we're using equal signs instead of colons to separate keys from values.

```
...
metadata {
    name = "aws-postgresql"
    labels {
        provider = "aws"
        db       = "postgresql"
    }
}
spec {
    compositeTypeRef {
        apiVersion = "devopstoolkitseries.com/v1alpha1"
        kind       = "SQL"
    }
    mode = "Pipeline"
    pipeline {
        new {
            functionRef { name = "crossplane-contrib-function-patch-and-transform" }
            ...
```

The next few lines look more like Json, which is normal since Pkl is, I believe, built on top of Java which also likes curly braces (`{` and `}`).

Still, so far there's nothing truly interesting in what I marked so far, so let's go up near the top where I defined a few classes.

```
...
class Subnet {
    hidden zoneVar: String
    hidden cidrBlockVar: String
    name = "subnet-\(zoneVar)"
    base {
        apiVersion = "ec2.aws.upbound.io/v1beta1"
        kind       = "Subnet"
        metadata { labels { zone = "us-east-1\(zoneVar)" } }
        ...
...
```

Now, if you check the documentation, you will not find information what classes (`class`) are or what they're used for. The explanation starts with "Classes are arranged in a single inheritance hierarchy." Great. Now I know how they are arranged without knowing what they are. Well done!

As a matter of fact, you won't find much information about anything in Pkl docs. If there would be a competition for the worst docs ever, Pkl would be a finalist. It would not win the worst documentation ever since Nix would get the gold medal, so Pkl would need to satisfy itself with silver or bronze.

Anyways... From what I gathered, Pkl classes are similar to Java classes. We can use them to create instances of something and that something, in case of Pkl, are data objects.

So, here I have a class called `Subnet` which contains a combination of static data, plus a few variables.

The reason for the existence of that specific `class` is to avoid repetition. I had to define three subnets which are mostly the same except for the zone and CIDR blocks.

So, I defined those two as variables `zoneVar` and `cidrBlockVar`. Both are `String` variables without default values meaning that whomever uses that class needs to specify them. Since, by default, Pkl outputs all data and it treats variables as data, I had to specify that those variablesare `hidden`, not output. Later on, I used those variables to customize the output, like in the case of `base.metadata.labels.zone` key that contains a combination of static text and value of the `zoneVar` variable.

```
                    (Subnet) {
                        zoneVar = "a"
                        cidrBlockVar = "11.0.0.0/24"
                    }
                    (Subnet) {
                        zoneVar = "b"
                        cidrBlockVar = "11.0.1.0/24"
                    }
                    (Subnet) {
                        zoneVar = "c"
                        cidrBlockVar = "11.0.2.0/24"
                    }
...
```

Further down, I'm invoking that class (`Subnet`) three times, each with different values for the zone and the CIDR block. As a result, I'm getting three Subnets without unnecessary duplication.

```
...
spec {
    ...
    pipeline {
        new {
            ...
            input {
                ...
                resources {
                    default {
                        base {
                            apiVersion = "ec2.aws.upbound.io/v1beta1"
                            spec { forProvider { region = "us-east-1" } }
                        }
                        patches { new {
                            type = "PatchSet"
                            patchSetName = "metadata"
                        } }
                    }
                    new {
                        name = "vpc"
                        base {
                            kind = "VPC"
                            spec { forProvider {
                                cidrBlock          = "11.0.0.0/16"
                                enableDnsSupport   = true
                                enableDnsHostnames = true
                            } }
                        }
                    }
                    ...
```

Everything I shown so far are key/value examples and it took me a while to understand how to create lists. As a matter of fact, they are easy once you get over the hurdle of uncomprehensible documentation. If a value is a `new` object, it automatically becomes an item in a list.

Here, for example, I'm saying that the first item in the list of `resources` is whatever is inside the `new` object.

Here's a nice one though. We can define `default` values of objects in a list. In this case, I'm specifying that the default `spec.apiVersion` will be `ec2.aws.upbound.io/v1beta1`, that the `base.spec.forProvider.region` will be, by default `us-east-1`, and so on and so forth. Later on, inside the `new` objects, I can overwrite specific default entries.

There are also templates like the  one I defined in `goTemplate.pkl`.


```sh
cat pkl/goTemplate.pkl
```

The output is as follows.

```
hidden templateVar: String
hidden delimLeft = "{{"
hidden delimRight = "}}"

functionRef { name = "crossplane-contrib-function-go-templating" }
step: String
input {
    apiVersion = "gotemplating.fn.crossplane.io/v1beta1"
    kind       = "GoTemplate"
    source     = "Inline"
    delims {
        left = delimLeft
        right = delimRight
    }
    inline {
      template = templateVar
    }
}
```

That's a template similar to a `class`, yet slightly different, and I dare you to understand the differences from the docs.

Anyways, over here I'm defining some `hidden` variables and some data (`functionRef`, `apiVersion`) just as I did before. The major difference is that it is specified in a separate file which I can reference, as I did in `daprComponents.pkl`.

```sh
cat pkl/daprComponents.pkl
```

The output is as follows.

```
amends "goTemplate.pkl"

step = "dapr-components"
templateVar = """
    {{ if and .observed.composite.resource.spec.parameters.secrets.daprComponents .observed.composite.resource.spec.parameters.secrets.pullToCluster }}
    {{ range .observed.composite.resource.spec.parameters.databases }}
    ---
    apiVersion: kubernetes.crossplane.io/v1alpha2
    kind: Object
    metadata:
      name: {{ $.observed.composite.resource.spec.id }}-dapr-component-{{ . }}
      annotations:
        gotemplating.fn.crossplane.io/composition-resource-name: {{ $.observed.composite.resource.spec.id }}-dapr-component-{{ . }}
    spec:
      providerConfigRef:
        name: {{ $.observed.composite.resource.spec.parameters.secrets.pullToCluster }}
      forProvider:
        manifest:
          apiVersion: dapr.io/v1alpha1
          kind: Component
          metadata:
            name: {{ $.observed.composite.resource.spec.id }}-{{ . }}
            namespace: {{ $.observed.composite.resource.spec.parameters.secrets.pullToClusterNamespace }}
          spec:
            type: state.postgresql
            version: v1
            metadata:
              - name: connectionString
                secretKeyRef:
                  name: {{ $.observed.composite.resource.spec.id }}
                  key: conn-{{ . }}
    {{ end }}
    {{ end }}
    """
```

This is also a template which imports the previous template (`goTemplate.pkl). Now, by default, templates are immutable, unless we choose to `ammend` them, just as I'm doing here.

*You can safely ignore the insanity I have here with the value of `templateVar` set to a GoTemplate inside Pkl. The reasons for that are irrelevant for this story.*

There are many other aspects of Pkl, and here's an important note. Pkl can output results into many different formats, but that does not mean that everything works in all the formats. If, for example, you go through the quickstart, you'll find this example. Try to output it as YAML or JSON, and you'll see that it does not work. The only format Pkl can always output to is Pkl. For everything else, you need to ensure that what you're doing is compatible with YAML, Json, Jsonnet, or whatever the destination format is.

What else...?

It works as a Gradle plugin, and it supports IntelliJ, VS Code, and Neovim. I tend to use VS Code and the plugin is... Pasable. It's not great, but it works, more or less.

There are also language bindings for Java, Kotlin, Swift, and Go which can be useful if you do choose to adopt Pkl as the configuration format instead of a tool that converts templates into something else like YAML or Json. I'm not sure that the former makes sense and, if it doesn't, the bindings are, more or less, useless. Still, you might go all in on Pkl and, if you do, and if you happen to be a Java or Go shop, you'll find those bindings useful.

## Pkl Pros and Cons

Here's the big question. **Should you adopt Pkl** and ditch whatever you're using right now? The answer to that question is... **It depends**. If, for example, you're planning to create YAML that represents Kubernetes resources, you might want to go with something specifically designed to work with Kubernetes. If it's a set of relatively simple manifests that require small amount of changes, what you typically have with your own applications, **Kustomize** is still a better choice. If you are using third-party applications, **Helm** is inevitable. If you are distributing your applications to others, Helm is still inevitable since that's what others are expecting.

Should you then adopt Pkl as the configuration format your applications use instead of Properties files or YAML or whatever you're using today? There's probably no need to switch, especially not this early while Pkl is still in early stages.

So, does that mean that there is **no space for Pkl**?

I think there is a good use case when the end result, YAML, Json, XML, or whatever you're using is sufficiently big with repetitions and frequent updates.

That's the situation I'm in with Crossplane Compositions I shown in this video. There's around two thousand lines of YAML to be generated. It's too much YAML for me to manage directly with too many repetions across Compositions. I have all that complexity in form of Crossplane Compositions precisely so that users of my services, in this case databases, do not need to deal with it. They will just create a few lines of YAML that represent claims and all the required resources will be assembled inside a control plane cluster. Nevertheless, I need to assemble all that YAML somehow, and writing YAML directly is not an option, so I needed something else.

That something else could be Helm charts which I don't like. Helm is unaware that it is working with data. It is a free-text templating engine. On top of that, it's missing too many features I deem important. Hence, Helm is not a good option, at least not for me. Then there are formats like Jsonnet, ytt, and a few others which do not make sense for simple setups, but are great for situations I'm in. However, my choice is none of those. I prefer CUE backed by Timoni.

*If you are not familiar with CUE or Timoni, please watch [Is CUE The Perfect Language For Kubernetes Manifests (Helm Templates Replacement)?](https://youtu.be/m6g0aWggdUQ) and [Is Timoni With CUE a Helm Replacement?](https://youtu.be/bbE1BFCs548).*

So, in my case, if I am to adopt Pkl, it would be as a replacement for CUE which I use to generate YAML. Hence, the initial two questions are good ones.

**Should Pkl replace JSON or YAML?** The answer to that one is a big **No**. If what you have is small enough and not updated frequently, use YAML or Json directly. There's no need to complicate your life with anything else. If your situation is more complicated and you need to generate YAML or Json, than the question is a different one.

**Should Pkl replace Jsonnet or CUE?** or whichever other solution you might be using?

I'm now torn between CUE and Pkl. I feel that CUE is a better choice but with a steeper learning curve. If you master CUE and you learn to deal with its immutability, you should probably stick with it. On the other hand, if you want something easier to learn yet very effective, Pkl is actually a pretty good one. Nevertheless, It'll still take a while until you wrap your head around it. You won't learn it in an hour, so it's somewhere between the simplicity of Kustomize on one side, and CUE or Jsonnet on the other.

Personally, I stopped using anything but Kustomize for simple situations and CUE for more complex scenarios. Now I'm adding Pkl to the mix and I'll do my best to continue using CUE for some of my projects and Pkl for others so that I get a better understanding of the pros and cons of both and, more importantly, figure out which one feels better. After all, if both can accomplish, more or less the same results, it's more about the feeling than anything else. Hence, I'll use both in the upcoming months and try to establish which one of the two I like more.

With all that in mind, here are the pros and cons but, unlike in other videos, they are heavily focused on my use cases.

**Cons**
* Documentation
* Why?
* (Probably) not better than CUE

Starting with negatives or cons, the first one on the list is **documentation**. It's not the worst documentation I saw, that privilege goes to Nix, but it is still pretty bad. Pkl is a language or a DSL making it hard to figure out what is what without a good documentation. If it would be yet another tool, it would be different. It's relatively easy to understand what a CLI does or what to do in a Web UI type of a solution. But this is not it. This is a language and a language needs exceptional documentation and plenty of examples. More over, it's not only about the docs alone. For example, given that Pkl is relatively new, you won't find much help outside the official docs. You cannot rely much on stack overflow and AI tools like Copilot will not help you either. As a matter of fact, I asked GitHub Copilot to explain a Pkl file and it deduced that it is HCL or something similar. All that is to be expected and the phase any new project, especially a language needs to pass through. If Pkl gets wider adoption, it will have more contributions, including the docs, it will have conversations on StackOverflow and Reddit, and AIs will eventually learn what it is. But that's the future. That's not the stage the project is in today.

The second negative point is a single word question: "**Why?**" What is the reason behind the existence of yet another language or a DSL that allows us to transform data into YAML, Json, XML, or any other static format?

My understanding is that Apple uses it internally. It probably works well for them so there is a strong chance that it will work for others. That's great. I will always support companies sharing their internal projects.

There is a page in the docs that compares Pkl with other solutions. It talks about the reasons why not to use a static config format like YAML, Json, and XML, and that makes sense. That's why we have languages like CUE, templates like Helm, overlaying solutions like Kustomize, and other config languages like Jsonnet and HCL. There was no reason to compare itself with static config formats. That's obvious. Then it goes on into a comparison with general purpose languages and that makes sense as well. Most of us do not use Python or Java or JavaScript to create static configs. We have tools for that. We have CUE which is based on Go, Helm which is also based on Go, CDK8S which works with most of the languages, and many others. Hence, I understand why we should not use general purpose languages directly. The dissapointing part is that the docs did not mention any of the libraries and frameworks that are based on general purpose languages. That would be a good comparison. Even if you'd say that CUE and Helm are not good because they use Go syntax which you might not like and CDK8S is too much focused on Kubernetes, there is still Pulumi and many others that can transform your general purpose language of choice into YAML or whichever other format you need.

I suspect that the key is in Java. I suspect that Pkl creators work with Java and wanted something close to Java yet something that is not Java alone.

Finally, the comparison page goes on to compare it with configuration languages like Jsonnet, HCL, and Dhall. There are no good arguments there except embedding into JVM applications, making my suspicion that it's all about Java a valid one.

All in all, I'm not sure that the reasons for creating Pkl are valid beyond the ability to embeed it into JVM applications.

Finally, the last negative point and that it is **not better than CUE**. At best, it is slightly easier to learn than CUE, especially if you are not familiar with Jsonnet and Go since CUE is a combination of those two. Where Pkl shines is for those not familiar with those. It takes time to learn Pkl, but it probably takes even more time to learn CUE so I'll change my statement to "CUE is probably a better choice for complex setups if you are willing to learn it".

Those were negative points that might make you think that I really dislike Pkl, but that's not the case. The more I use it, the more I like it. I am not saying that it will replace my configurations defined as CUE but only that it could. Time will tell.

**Pros**
* (Relatively) easy to learn
* Outputs to any format
* Language bindings

As for the positive points, for the pros...

To begin with, Pkl is **relatively easy to learn** when compared to Jsonnet, CUE, and other languages and formats. It's not easy though. It's just slighly easier, and that's a good thing if the goal is to get wide adoption.

Another great thing, even though it is not unique, is that we can **output Pkl to almost any static configuration format**. If we need YAML, it can be output to YAML. If we need JSON, it can be output to JSON. XML, Properties, PCF... All are supported. Now, to be clear, that's not much of an advantage since CUE and others can also output to almost any format. When the purpose of a language is to generate structured data, it's easy to output that data to any format. Still, Pkl can output to almost anything, and that's a good thing, even though I doubt that you'll need the same data in more than one format.

The last potentially great feature are **language bindings**. If you want to consume Pkl format directly from inside the code of your application, Pkl allows you to do that easily, as long as that language is Java, Kotlin, Swift, or Go. Whether you should do that, whether you should have your app consume Pkl files instead of YAML, Json, or XML is a completely other question. I'd say no, but I might be wrong, so I'm giving language bindings thumbs up just in case.

All in all, Pkl is growing up on me. It started with "**I don't like this**. I don't understand why it exists" and now it's in the "**It might be a good thing**" category. I'll continue using CUE for some of my projects and Pkl for others. I'll let you know which one will prevail.

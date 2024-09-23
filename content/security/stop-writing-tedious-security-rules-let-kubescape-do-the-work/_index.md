
+++
title = 'Stop Writing Tedious Security Rules! Let Kubescape Do the Work'
date = 2024-09-22T16:00:00+00:00
draft = false
+++

Implementation and maintenance of security is **tedious**, especially when runtime portion of it is concerned. For example, if we'd like to be notified when a potential breach is happening, we'll likely use a tool like [Falco](https://www.google.com/search?q=falco+security&sourceid=chrome&ie=UTF-8). It's a great tool. It's potentially one of the best if not the best tool of it's kind. It allows us to define an infinite number of rules that, when one of them is met, will fire notifications. That's the problem though. We have to define all those rules or, at least, accept a significant number of rules that are available out of the box. Essentially, we need to predict everything that should not be allowed to happen or, if we prefer the other way around, everything that is allowed. That is tedious and you are likely going to end up frustrated at best, in an asylum at worst. After all, **who can predict all the bad things that might happen?** and who is fully aware of all high and low level calls that applications are making? I certainly can't.

<!--more-->

{{< youtube xilNX_mh6vE >}}

There's a better way; a better approach. Instead of us trying to figure out what are all the things our applications should and shouldn't be allowed to do, we can have **a process that learns "normal" behavior of an app** and assume that everything outside the ordinary is... bad.

After all, that's what we, humans, do. We define security rules based on what we know about our applications. However, we are unlikely to ever truly know, at least on the system calls level, what an application does. eBPF does not have that problem. We can have an **eBPF process that watches every single activity of an application**, collect all the data, no matter how high or low level that data is, and deduce that's what that application should be doing. From there on, that same process can notify us when something extraordinary happens.

In other words, we can have an eBPF process that learns how an application behaves and notifies us if any out-of-ordinary behavior occurs. That's how we detect malious actors. That's how we find out that someone, or something, breached our defences.

That's where Kubescape or, to be more precise, Kubescape's Runtime Threat Detection comes into play.

TODO: Thumbnail: ZATGiDIDBQk

Kubescape itself is a Kubernetes security platform I already explored in the [How to Secure Kubernetes Clusters with Kubescape and Armo](https://youtu.be/ZATGiDIDBQk). Today I will ignore all the good (and bad) things it does. Instead, I want to focus on a single feature that was released recently.

## Kubescape Runtime Threat Detection

Kubescape Runtime Detection consists of three parts.

The first one is **Anomaly Detection Engine** that detects and alerts on any anomalous behavior. It does that not by following our instructions of what is and what isn't an anomaly by by learning the behavior of applications and reacting when a deviation occurs.

Then there is **Behavioral Analysis Engine** that identifies well known attacks. It, essentially, tells us if someone or something is performing operations that are known to be malicious.

Finally, there is **Malware scanning** that scan for malwares, viruses, and other stuff that might infect your system.

Today, I will not only ignore Kubescape in general but also all those runtime threat detections but one. Today, I want to focus on the **Anomaly Detection Engine** which I will clumsilly describe as a solution similar to Falco but without the parts that can cause nightmares.

Oh... Before we continue, I should say that everything I'll show today is **open source**. There is a commercial version with a few additional features which is, actually, pretty good and probably worth your money. Nevertheless, today is all about open source.

Let's see it in action.

## Setup

```sh
git clone https://github.com/vfarcic/kubescape-node-agent

cd kubescape-node-agent
```

> Make sure that Docker is up-and-running. We'll use it to create a KinD cluster.

> Watch [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell

chmod +x setup.sh

./setup.sh

source .env
```

## Kubescape Anomaly Detection Engine In Action

I already created a Kubernetes cluster and deployed an application. It's a silly demo that, when we send a request to it,...

```sh
curl "http://silly-demo.$INGRESS_HOST.nip.io"
```

The output is as follows.

```
This is a silly demo
```

...does not hide how silly it is.

Nevertheless, that silly demo will be more than enough to demonstrate how anomaly detection works.

Now, let's say that I am a malicious actor that managed to gain access into the cluster and wants to list all the files and directories in the root of the container where the application is running. I would do that by first finding out the name of the Pod I'm interested in,...

```sh
export POD_NAME=$(kubectl --namespace a-team get pods \
    --no-headers --output name | head -1)
```

...and executing `kubectl` `exec` with the `ls /` command.

```sh
kubectl --namespace a-team exec --tty --stdin $POD_NAME -- ls /
```

The output is as follows.

```
bin    etc    lib    mnt    proc   run    srv    tmp    var
dev    home   media  opt    root   sbin   sys    usr
```

Now, that is obviously not an operation that should be allowed, unless the application itself needs to be able to list files and directories. Maybe it does, or maybe it doesn't. As I already mentioned, I honestly do not know what are all the system-level calls the application is doing so, even though, executing `ls` is obviously something that should not be allowed, there are many other calls that should, and even more of others that shouldn't.

We'll fix that by installing Kubescape operator through a Helm chart. But, before we do that, let's take a quick look at the values files I prepared.

```sh
cat kubescape-values.yaml
```

The output is as follows.

```yaml
clusterName: dot
alertCRD:
  installDefault: true
capabilities:
  runtimeDetection: enable
nodeAgent:
  config:
    maxLearningPeriod: "5m"
    stdoutExporter: true
    alertManagerExporterUrls: prometheus-kube-prometheus-alertmanager.monitoring:9093
```

Over there I am enabling `runtimeDetection` since that's the only Kubescape feature we're interested in today.

Further on, I set the `maxLearningPeriod` to `5m`. In a real-world scenario, five minutes would obviously not be enough for Kubescape to learn everything there is to lean about the application. By default, it is set to twenty four hours which should be more than enough for all the behavioral permutations to happen. If your release frequency is once every couple of days, you might want to leave the default value of 24h. On the other hand, all of you advanced folks that are releasing more frequently might want to consider something in between.

In any case, five minutes should be more than enough for a demo. Anything more than that would be a test of my patience.

Finally, I enabled `stdoutExporter` so that the agents output alerts to logs and set `alertManagerExporterUrls` so that those same alerts are also sent to [AlertManager](https://prometheus.io/docs/alerting/alertmanager). Besides those two, we could also use Syslog and HTTP exporters.

Now we can install Kubescape Operator by executing `helm upgrade`.

```sh
helm upgrade --install kubescape kubescape-operator \
    --repo https://kubescape.github.io/helm-charts \
    --namespace kubescape --create-namespace \
    --values kubescape-values.yaml --wait
```

Now that Kubescape is running, it will spend five minutes learning about the behavior of the apps in the cluster. In production there would be nothing for us to do since the application would be receiving "real" traffic but since this is a demo, I'll similate traffic by executing `hey` for `5m`.

```sh
hey -z 5m "http://silly-demo.$INGRESS_HOST.nip.io"
```

*hey* is now "bombing" the application with requests and will continue doing so for a while, so let's fast forward to the end of those five minutes.

The output is as follows (truncated for brevity).

```
...
Status code distribution:
  [200] 125282 responses
```

From now on, Kubescape knows what a "normal" behavior of the application is and will alert us when something abnormal happens. Since we specified that it should output warnings to stdout, we can see what's going on by retrieving `logs` from the `node-agent` and, since we're interested only in one specific application, we'll `grep silly-demo` and, finally we'll make it pretty with `jq`.

```sh
kubectl --namespace kubescape logs \
    --selector app.kubernetes.io/name=node-agent \
    | grep silly-demo | jq .
```

The output is as follows.

```json
{
  "level": "info",
  "ts": "2024-08-05T18:09:08Z",
  "msg": "start monitor on container",
  "container ID": "36b03255dedcf83cb91290565996ed9f0bbfbe485366f891b1b79392c7c645fc",
  "k8s workload": "a-team/silly-demo-6bc44c5d45-s28ln/silly-demo"
}
{
  "level": "info",
  "ts": "2024-08-05T18:14:08Z",
  "msg": "monitoring time ended",
  "container ID": "36b03255dedcf83cb91290565996ed9f0bbfbe485366f891b1b79392c7c645fc",
  "k8s workload": "a-team/silly-demo-6bc44c5d45-s28ln/silly-demo"
}
{
  "level": "info",
  "ts": "2024-08-05T18:09:07Z",
  "msg": "start monitor on container",
  "container ID": "a614c61db9132e85e9710baf49e5fcae60c1e4347e483267ceb58bdfd3b515a5",
  "k8s workload": "a-team/silly-demo-6bc44c5d45-5wsd7/silly-demo"
}
{
  "level": "info",
  "ts": "2024-08-05T18:14:07Z",
  "msg": "monitoring time ended",
  "container ID": "a614c61db9132e85e9710baf49e5fcae60c1e4347e483267ceb58bdfd3b515a5",
  "k8s workload": "a-team/silly-demo-6bc44c5d45-5wsd7/silly-demo"
}
```

So far, nothing "special" happened. The only information we got so far is that it started to `monitor on container` and, five minutes later, it `ended` monitoring. Since, by default, the node agent is running as two replicas, we got those message in stereo.

Other than the information that monitoring started and stopped, the node agent did not detect anything unusual. Let's change that. Let's execute the same `ls` process we run before. That one surrely does not count as expected behavior.

```sh
kubectl --namespace a-team exec --tty --stdin $POD_NAME -- ls /
```

Let's see what we have in the logs now.

```sh
kubectl --namespace kubescape logs \
    --selector app.kubernetes.io/name=node-agent \
    | grep silly-demo | jq .
```

The output is as follows (truncated for brevity).

```json
...
{
  "BaseRuntimeMetadata": {
    "alertName": "Unexpected system call",
    "infectedPID": 4996,
    "fixSuggestions": "If this is a valid behavior, please add the system call \"munmap\" to the whitelist in the application profile for the Pod \"silly-demo-6bc44c5d45-5wsd7\".",
    "md5Hash": "0ba15535232923f2fa90ef5971c3e2fc",
    "sha1Hash": "58d64c911b6f305fddc714d2fd61b69b34a19f13",
    "sha256Hash": "262be835341d87f795283a95fc43eae7bb33bc5c88d52e04e91ce7af3078d0ab",
    "severity": 1,
    "size": "18 MB",
    "timestamp": "2024-08-05T18:20:16.048392104Z"
  },
  "RuleID": "R0003",
  "RuntimeK8sDetails": {
    "clusterName": "dot",
    "containerName": "silly-demo",
    "hostNetwork": false,
    "namespace": "a-team",
    "containerID": "a614c61db9132e85e9710baf49e5fcae60c1e4347e483267ceb58bdfd3b515a5",
    "podName": "silly-demo-6bc44c5d45-5wsd7",
    "podNamespace": "a-team",
    "workloadName": "silly-demo",
    "workloadNamespace": "a-team",
    "workloadKind": "Deployment"
  },
  "RuntimeProcessDetails": {
    "processTree": {
      "pid": 4996,
      "cmdline": "silly-demo",
      "comm": "silly-demo",
      "ppid": 4911,
      "pcomm": "containerd-shim",
      "uid": 0,
      "gid": 0,
      "cwd": "/",
      "path": "/usr/local/bin/silly-demo"
    },
    "uniqueID": 0,
    "containerID": "a614c61db9132e85e9710baf49e5fcae60c1e4347e483267ceb58bdfd3b515a5"
  },
  "event": {
    "runtime": {
      "runtimeName": "containerd",
      "containerId": "a614c61db9132e85e9710baf49e5fcae60c1e4347e483267ceb58bdfd3b515a5"
    },
    "k8s": {
      "node": "ip-192-168-31-181.ec2.internal",
      "namespace": "a-team",
      "podName": "silly-demo-6bc44c5d45-5wsd7",
      "podLabels": {
        "app.kubernetes.io/name": "silly-demo",
        "pod-template-hash": "6bc44c5d45"
      },
      "containerName": "silly-demo"
    },
    "timestamp": 1722882016048392104,
    "type": "normal"
  },
  "level": "error",
  "message": "Unexpected system call: munmap in: silly-demo",
  "msg": "Unexpected system call",
  "time": "2024-08-05T18:20:16Z"
}
...
```

Now, to be clear, exploring logs like those we saw is not ideal. Actually, it so far from ideal that I would say it's almost useless. I would not know what to do with them and I certainly do not want to waste my time tailing logs 24/7 just in case some anomaly is detected. Instead, I want to receive a notification when something's wrong on Slack, email, or something similar. Kubescape, at least the open source version, does not have that capability baked in. We cannot tell it to send a message to Slack or something similar when something's wrong and, even if we could do that, we would be swarmed with too many alerts and quite a few false positives.

What we can do though is instruct it to send alerts to AlertManager. Actually, we already did that when we deployed Kubescape so we can jump straight into it.

```sh
echo "http://alertmanager.$INGRESS_HOST.nip.io"
```

> Open the URL from the output in a browser.

We can see a few groups of alerts but we are interested only in those coming from the `a-team` Namespace since that's where the application is.

> Filter by `namespace="a-team"`

This looks wrong though. It claims that there are `87` alerts while logs showed only a few and even those are too many since there are two replicas of the node agent effectively duplicating the alerts.

![](am.png)

I feel that the node agent keeps sending the same alerts to AlertManager, which is bad, yet not a big problem by itself since we can group alerts. Since we are probably interested in the command (`comm`), we can group alerts by that label.

> Select `Group`, select `Enable custom grouping`, type `comm`, and press the enter key.

![](am-group.png)

That makes much more sense. We can see that there are four distinct alerts. The one that matters is the `ls` command we executed as a way to simulate the "unexpected". The rest are recurring alerts coming from Kubescape, but not directly related to what we're exploring today.

While logs are almost useless, AlertManager is a much better destination, yet its far from perfect. There are quite a few fields that are not sent to AlertManager, and that's to be expected since AlertManager is not supposed to be a dashboard where we observe "stuff" but, rather, a gateway between the source of alerts, in this case Kubescape, and the destination which can be Slack, email, or almost anything else.

Now, I won't go into details how to configure AlertManager to send alerts somewhere. That would be a completely different subject which, if you are interested and express that interest in the comments, I would be more than happy to explore in more detail. Otherwise, I'll assume that you are familiar with AlertManager.

So far, we saw that we do not need to tell Kubescape what is the expected behavior of our apps and that it can notify us when something extraordinary happens.

We also saw that, out of the box, it comes with rules that we might, but also rules that we might NOT need. We can change that as well through the `runtimerulealertbindings` resource that was created when we installed Kubescape. Let's take a look at it.

```sh
kubectl get \
    runtimerulealertbindings.kubescape.io all-rules-all-pods \
    --output yaml
```

The output is as follows (truncated for brevity).

```yaml
apiVersion: kubescape.io/v1
kind: RuntimeRuleAlertBinding
...
spec:
  ...
  rules:
  - ruleName: Unexpected process launched
  - parameters:
      ignoreMounts: true
      ignorePrefixes:
      - /proc
      - /run/secrets/kubernetes.io/serviceaccount
      - /var/run/secrets/kubernetes.io/serviceaccount
      - /tmp
    ruleName: Unexpected file access
  - ruleName: Unexpected system call
  - ruleName: Unexpected capability used
  - ruleName: Unexpected domain request
  - ruleName: Unexpected Service Account Token Access
  - ruleName: Kubernetes Client Executed
  - ruleName: Exec from malicious source
  - ruleName: Kernel Module Load
  - ruleName: Exec Binary Not In Base Image
  - ruleName: Malicious SSH Connection
  - ruleName: Fileless Execution
  - ruleName: XMR Crypto Mining Detection
  - ruleName: Exec from mount
  - ruleName: Crypto Mining Related Port Communication
  - ruleName: Crypto Mining Domain Communication
  - ruleName: Read Environment Variables from procfs
  - ruleName: eBPF Program Load
  - ruleName: Symlink Created Over Sensitive File
  - ruleName: Unexpected Sensitive File Access
  - ruleName: LD_PRELOAD Hook
  - ruleName: Hardlink Created Over Sensitive File
  - ruleName: Exec to pod
  - ruleName: Port forward
```

The key is in the `rules` section. The `Unexpected process launched` is the rule that detected that *ls* command was executed and that it is not considered a "normal" behavior since it was not executed during the learning period.

I won't go through the rest of the rules because they should be self-explanatory and, if that's not the case, you can find a bit more, yet potentially not enough, information in the *Rule bindings* section in the [Rruntime Threat Detection](https://kubescape.io/docs/operator/runtime-threat-detection) page in the documentation.

Future releases should come with additional rules and, more importantly, a mechanism to add custom rules and, hopefully, better documentation.

The key to all this are the `applicationprofile` resources, so let's see what we have in the `a-team` Namespace.

```sh
kubectl --namespace a-team get applicationprofile
```

The output is as follows.

```
NAME                             CREATED AT
replicaset-silly-demo-6bc44c5d45 2024-08-05T18:09:08Z
```

Next, we'll store the name of the profile in the environment variable and...

> Replace `[...]` with the application profile name

```sh
export PROFILE_NAME=[...]
```

...use it to output it.

```sh
kubectl --namespace a-team get applicationprofile $PROFILE_NAME \
    --output yaml
```

The output is as follows (truncated for brevity).

```yaml
apiVersion: spdx.softwarecomposition.kubescape.io/v1beta1
kind: ApplicationProfile
metadata:
  ...
  labels:
    ...
    kubescape.io/workload-kind: Deployment
    kubescape.io/workload-name: silly-demo
    kubescape.io/workload-namespace: a-team
    ...
spec:
  ...
  containers:
  - capabilities: null
    ...
    syscalls:
    - nanosleep
    - sched_yield
    - madvise
    - getsockname
    - write
    - epoll_pwait
    - clone
    - setsockopt
    - sigaltstack
    - rt_sigprocmask
    - futex
    - accept4
    - tgkill
    - rt_sigreturn
    - gettid
    - getpid
    - mmap
    - epoll_ctl
    - read
    - close
...
```

That's the `ApplicationProfile` where the information about the expected behavior of the application was stored after the node agent finished observing it for the given period.

We can see, through the `labels` that it is associated with the `silly-demo` `Deployment` inside the `a-team` Namespace. That's the app. That's missleading though since the profile is generated for the specific ReplicaSet and, the next time we deploy a new release of the application, a new profile will be created after the learning period. That's both good and bad.

Since each new release of an application can change the behavior of the application, it makes perfect sense that each is initiating a new learning process and generating a new profile. That makes perfect sense, except, maybe, if we release very frequently in which case the learning period might be just as long as the release frequency effectively resulting in constant learning without much alerting. Nevertheless, it makes perfect sense to repeat the same process for every release.

The bad news is that the instruction in the logs to whitelist false positives in the profile is kind of silly since a profile is short lived. As a matter of fact, I would not even know where to start if I would want to whitelist anything. I'm not saying that's not possible but, rather, that if it is, it's not documented or explained anywhere (or I missed it).

The good news is that we can ignore the application profile and, instead, silence alerts that are false positives in the AlertManager. It might not be ideal, but it's doable and, judging by the current state of the project, a preferable way to deal with alerts coming from Kubescape.

I feel that we had enough hands-on experience with Kubescape Anomaly Detection Engine to give you an idea how it works, so let's move into my favorite part of every post and talk about the pros and cons and try to deduce whether you should adopt the project.

## Kubescape Anomaly Detection Engine Pros and Cons

Let me start by saying that what we read in this post is the open-source version of the project and that the commercial version solved some if not most of the issues we are about to discuss. I invite you to explore it yourself since, today, the focus is only on open source. All I will say is that the commercial version is awesome and should be considered.

With that out of the way, let's talk about the cons.

**Cons:**

* **Outputs** are not great. Observing alerts in logs is almost impossible, at least directly. If you're shipping logs to a central storage like, for example, Loki, you should be able to make sense of them through filters. If you do not use some centralized logging solution but plan to observe logs through `kubectl logs` you are likely NOT going to be able to make sense of them. Sending alerts to AlertManager is a much better, yet not a great solution either. I'm not sure what the "final" solution should be, yet I am sure that something is missing to make sense of the alerts in an effective manner.
* The idea to edit the **profile** sounds silly. Should we edit profile after each release of an application? That sounds silly. I feel that Kubescape should keep generating application profiles and enable us, users, to create a separate resource with whitelisted processes that is persisted across releases.
* I could argue that Kubescape is better or, at least, less demanding way to detect anomalies in behavior than Falco. However, just as Falco, it is limited to alerts. If someone enters the system and starts doing whatever malicious actors do, we will find out what's happening. We might stop that behavior, but we cannot prevent it. I feel that Kubescape should extend the existing mechanism to not only alert on potentially malicious behavior but, optionally, prevent it in a similar was a KubeArmor prevents it. Since it is based on eBPF, I don't see a technical reason not to do so. Now, I do understand that many will choose not to enable **prevention**, especially early on. But, as time passes and people start trusting the anomaly detection, anomaly prevention will become a desirable progression.
* Finally, there is no obvious or, at least, a documented way, to transfer application **profiles** from one environment to another. Ideally, anomaly detection should work all the time in production, and not only after the learning period which, as per recommendation, is 24h. What might make sense is to run it in a separate environment, let's say staging and, once complete, move it to production together with the new release of the application. Now, I'm fully aware that we could, probably, somehow, hack it, but I'm ignoring that possibility since such an option is not even mentioned in the docs.

**Pros:**

* There are quite a few positive aspect of Kubescape Anomaly Detection Engine but, today, I will focus only on one since that one is the real differentiator. Instead of forcing us to configure expected and unexpected behavior of each app, the engine **learns** what the expected behavior of the app is and considers anything else as unexpected. That might not sound like a big deal, but it is. That solves one of the biggest problems for the tools in that space.

Now, judging by the number of cons and my complains, you might deduce that Kubescape, or, at least, the anomaly detection engine, is not a good choice. That's not how I would describe it. It's a young project so it is to be expected that there are rough edges and that there are still some things for it to discover. I like the direction it is going. I feel that "learning" the behavior of an application is a much better approach than expecting us, humans, to know that and define it in some configuration file. That feature has a lot of potential and I can't wait to see how it will progress.

Try it out and let me know what you think.

## Destroy

```sh
chmod +x destroy.sh

./destroy.sh

exit
```


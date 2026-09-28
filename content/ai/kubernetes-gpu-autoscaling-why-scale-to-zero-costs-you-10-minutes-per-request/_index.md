
+++
title = "Kubernetes GPU Autoscaling: Why Scale to Zero Costs You 10 Minutes Per Request"
date = 2026-09-28T02:54:00+00:00
draft = false
+++

The same question, to the same model, on the same cluster. Four seconds one time, and over ten minutes the next, with nothing broken in between and nobody having touched a line of configuration.

That is the price of turning a GPU off when nobody is using it, and turning it off is the only way to stop paying for it, because the cloud charges you for the machine whether or not anything is running on it. The same ten minutes governs the other direction too. A second replica has to be asked for long before the traffic that needs it arrives, because it will not turn up in time to serve it.

So this is that trade, measured rather than argued. What scale to zero actually saves, what those ten minutes are made of, which parts of them you can attack, and when you have to ask for more.

<!--more-->

{{< youtube wwrPwU-JCMw >}}

## Setup

Everything we'll run is in a single repository. Clone it and move into the directory.

```sh
git clone https://github.com/vfarcic/inference-demo

cd inference-demo
```

> Watch [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell
```

> All outputs in this post come from Google Cloud. AWS is also supported, and only the provider changes.

> Devbox aliases `cat` to `bat`, so file listings are syntax-coloured on screen. Without it you get the same content in plain text.

[user]
```sh
export PROVIDER=google # `aws` is also supported
```


The setup creates a Kubernetes cluster with two node pools. General nodes run the system workloads. The second pool holds the GPUs and is tainted so only inference lands on it, and this time that pool is allowed to shrink all the way to nothing or grow to two nodes. It also installs KEDA, which is the autoscaler this whole video is about, and deploys a model server we can measure. Everything that follows is measured against that server.

```sh
chmod +x dot.nu

./dot.nu setup autoscaling $PROVIDER

source .env
```

## The Cost Of Doing Nothing

We have a language model running on a GPU in Kubernetes. It works. It answers questions. That part is finished, and it is not what this video is about.

This video is about the time when nobody is asking it anything and the meter is still running.

One disclaimer, because it will otherwise nag at anyone doing this seriously. What I'm running here is one small model on a single GPU, which is a video budget rather than a production deployment. Every number in this video gets worse at real scale. The shape of the problem does not.


So, before we change a single thing, let's look at what we are actually paying for. Starting with the machines.

```sh
kubectl get nodes
```

The output is as follows.

```text
NAME                                       STATUS   ROLES    AGE   VERSION
gke-inference-default-pool-6897be68-dn5m   Ready    <none>   14m   v1.35.7-gke.1027000
gke-inference-default-pool-6897be68-x31p   Ready    <none>   20m   v1.35.7-gke.1027000
gke-inference-default-pool-6897be68-xv3v   Ready    <none>   20m   v1.35.7-gke.1027000
gke-inference-gpu-3e020a97-cb9k            Ready    <none>   18m   v1.35.7-gke.1027000
```

Three ordinary machines running the boring parts of the cluster, and one with an NVIDIA L4 bolted to it (`gke-inference-gpu`). That last one is where all the money goes. The other three cost roughly what a cup of coffee costs.

And this is what is running on the expensive one.


```sh
kubectl --namespace inference get pods --selector app=silly-model
```

The output is as follows.

```text
NAME                    READY   STATUS    RESTARTS   AGE
vllm-86d74d98f8-tmbw4   1/1     Running   0          26m
```


The model server has been running on that GPU node for a while, so it is warm. Let's ask it something and time the answer.

```sh
time curl --silent --show-error \
    --header "Content-Type: application/json" \
    --data-binary "@demo/batching-request.json" \
    "http://silly-model.$INGRESS_HOST/v1/chat/completions" \
    | jq --raw-output '.choices[0].message.content'
```

The output is as follows (truncated for brevity).

```text
...

Kubernetes is an open-source platform designed to automate the deployment, scaling, and management of containerized applications.

...

real	0m4.704s
user	0m0.009s
sys	0m0.007s
```

A few seconds, start to finish (`real`). Remember that, because by the end of this video the exact same question will take a very different amount of time.


So what was the GPU doing for the rest of that minute?

```sh
kubectl --namespace inference exec deployment/vllm -- nvidia-smi \
    --query-gpu=memory.used,memory.total,utilization.gpu \
    --format=csv
```

The output is as follows.

```text
memory.used [MiB], memory.total [MiB], utilization.gpu [%]
19974 MiB, 23034 MiB, 0 %
```

Almost all of the card's memory is spoken for (`memory.used`, `memory.total`), and the processor is doing absolutely nothing (`utilization.gpu`). Those two numbers together are the entire problem. The memory is reserved so the model can answer the instant somebody asks, and reserving it costs exactly the same whether anybody asks or not.

I am not going to tell you what my little demo costs, because it is beside the point. This is the cheapest card you would seriously serve from, running the smallest model worth serving. The GPU people actually run large models on costs roughly seventeen times as much per hour, and it almost never arrives on its own. Put eight of them in one machine and you are into tens of thousands a month for the GPUs alone, before anything else on the bill. That is the number that should be bothering you, and every hour of it that goes unused is gone.

Which raises the obvious question. If nobody is using it, why not just turn it off?

## Scaling The Pod To Zero

The obvious move is a HorizontalPodAutoscaler, and as of Kubernetes 1.37 it can finally scale to zero. It still will not help us here. Coming back up needs a signal that survives having no Pods, which rules out CPU and memory, and ours is not a queue sitting there waiting to be read. Ours is an HTTP request arriving for a model that does not exist, and Kubernetes Services do not hold on to those.

So something has to catch that request, keep the caller waiting, and start the model on their behalf. That something is an interceptor, and everything that follows is built around it.


Here is how you describe one.

```sh
cat demo/autoscaling-route.yaml
```

The output is as follows.

```yaml
apiVersion: http.keda.sh/v1beta1
kind: InterceptorRoute
metadata:
  name: vllm
  namespace: inference
spec:
  target:
    service: silly-model
    port: 8000
  rules:
    - hosts:
        - silly-model.34.75.126.70.nip.io
  scalingMetric:
    concurrency:
      targetValue: 16
```

That is an `InterceptorRoute`, and there are three things in it. Which requests belong to this model (`rules`), where to send them once something is actually running (`target`), and how many requests one replica should be expected to juggle at a time (`scalingMetric`, `concurrency`). Remember that last one. It looks like the least interesting field on the screen and it is going to cause us a great deal of trouble later.


That is the routing half. Here is the half that does the scaling.

```sh
cat demo/autoscaling-zero.yaml
```

The output is as follows.

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: vllm
  namespace: inference
spec:
  scaleTargetRef:
    name: vllm
  minReplicaCount: 0
  maxReplicaCount: 1
  pollingInterval: 5
  cooldownPeriod: 120
  triggers:
    - type: external-push
      metadata:
        scalerAddress: keda-add-ons-http-external-scaler.keda:9090
        interceptorRoute: vllm
```

And that is a `ScaledObject`. The route counts requests; this decides what to do about the count. `minReplicaCount: 0` is the whole point, and the count it acts on comes from the interceptor rather than from the Pods, which is what makes zero reachable at all. `cooldownPeriod` is how long the model has to sit idle before KEDA puts it away, and I have set it deliberately low so we are not all staring at a terminal. In production you would want it far higher, for reasons that will be extremely obvious in about ten minutes.


Let's put both of those into the cluster.

```sh
kubectl apply --filename demo/autoscaling-route.yaml

kubectl apply --filename demo/autoscaling-zero.yaml
```

Applying those two does nothing yet, because traffic still goes straight at the model. It has to go to the interceptor instead, otherwise there is nobody home when the model is asleep. That means moving the Ingress, and moving it into a different namespace, because an Ingress can only point at a Service that lives beside it and the interceptor lives in the keda namespace.


Let's take a look at the replacement.

```sh
cat demo/autoscaling-ingress.yaml
```

The output is as follows.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: silly-model
  namespace: keda
spec:
  ingressClassName: traefik
  rules:
    - host: silly-model.34.75.126.70.nip.io
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: keda-add-ons-http-interceptor-proxy
                port:
                  number: 8080
```

Same `host`. Different `namespace`, and a different service `name`. Take the old one out and put this one in.


```sh
kubectl --namespace inference delete ingress silly-model

kubectl apply --filename demo/autoscaling-ingress.yaml
```

That is the whole configuration. Now we do nothing at all, which is the one part of this that requires no effort, and wait for the cooldown to expire.


Let's see what KEDA makes of that.

```sh
kubectl --namespace inference get scaledobject vllm
```

The output is as follows.

```text
NAME   SCALETARGETKIND      SCALETARGETNAME   MIN   MAX   READY   ACTIVE   FALLBACK   PAUSED   TRIGGERS        AUTHENTICATIONS   AGE
vllm   apps/v1.Deployment   vllm              0     1     True    False    False      False    external-push                     21m
```

`ACTIVE` is false. That is KEDA telling us, in its own way, that nobody has asked this model anything for a while and it sees no reason to keep it around.


And the model itself.

```sh
kubectl --namespace inference get pods --selector app=silly-model
```

The output is as follows.

```text
No resources found in inference namespace.
```

And the model server is gone. No Pod, no process, nothing holding that GPU. We asked for a system that costs nothing when nobody is using it, and we appear to have got one in about twenty lines of YAML.

So: solved? Let's check the bill.

## The Bill Did Not Move


There is one thing we have not looked at since we started deleting things, and it is the thing the money is actually attached to.

```sh
kubectl get nodes
```

The output is as follows.

```text
NAME                                       STATUS   ROLES    AGE     VERSION
gke-inference-default-pool-6897be68-dn5m   Ready    <none>   3h41m   v1.35.7-gke.1027000
gke-inference-default-pool-6897be68-x31p   Ready    <none>   3h47m   v1.35.7-gke.1027000
gke-inference-default-pool-6897be68-xv3v   Ready    <none>   3h47m   v1.35.7-gke.1027000
gke-inference-gpu-3e020a97-cb9k            Ready    <none>   3h45m   v1.35.7-gke.1027000
```

The Pod is gone. The GPU node is still there (`gke-inference-gpu`), still running, still charging full price to host absolutely nothing.

Google does not bill you for Pods. It bills you for machines. Deleting the workload changed the cluster, and it did not change the invoice by a single cent.

And nobody is coming to fix that on your behalf. Allocated means billed, used or not, in every cloud there is. Capacity you forgot to hand back is close to the most profitable thing anyone can rent you, because they are not doing anything for it. Your failure to reclaim it is somebody else's margin, and handing it back is entirely your job.


So is that it? Does it just sit there, indefinitely, charging us for a machine with nothing on it? Let's give it a few minutes and look again.

```sh
kubectl get nodes
```

> The Cluster Autoscaler takes its time, and how long it takes varies. If the GPU node is still listed, give it a few minutes and run the command again.

The output is as follows.

```text
NAME                                       STATUS   ROLES    AGE     VERSION
gke-inference-default-pool-6897be68-dn5m   Ready    <none>   3h58m   v1.35.7-gke.1027000
gke-inference-default-pool-6897be68-x31p   Ready    <none>   4h4m    v1.35.7-gke.1027000
gke-inference-default-pool-6897be68-xv3v   Ready    <none>   4h4m    v1.35.7-gke.1027000
```

The GPU node is gone. Three ordinary machines left, no accelerator anywhere in the cluster, and now the cost genuinely is zero. A model nobody was using has stopped costing anything at all.

So something was watching after all. That something is the Cluster Autoscaler, and it is the second of the two here, working in series with the first. Getting rid of the Pod was necessary, because a node with a workload on it is a node nobody can take away, but it was never what stops the meter. The Cluster Autoscaler had to look at the empty machine, decide it was not needed, and hand it back. It takes its time over that, and reasonably so, because deleting a node that turns out to be needed is a far worse mistake than keeping one a few minutes too long.

This is also the one place the two clouds are not interchangeable. On Google the Cluster Autoscaler is part of the control plane and you get it by asking the node pool for it. On EKS it is simply not there. You deploy the Cluster Autoscaler into the cluster yourself, or you run Karpenter instead, and until you do a node group stays at whatever size you set it to no matter how empty it gets.

The two of them run on completely separate clocks, and the gap between them is the part you just paid for. The Pod disappeared, and then we sat there for several minutes at full price for a GPU with nothing on it. And that delay is not yours to set. The Cluster Autoscaler waits a fixed stretch before it will accept that a node is genuinely spare, and both times I ran this it came to about ten minutes. So every time your model goes quiet, you buy something like ten idle GPU-minutes for the privilege of it going away, and no amount of tuning on your side shortens them.

So that is scale to zero, working. The model is gone, the machine is gone, the bill is gone. Solved?

Only if nobody ever asks it anything again.

## The Wall


One request. That is all it takes to find out what those savings actually cost. I'm sending a single question, the same kind we asked at the start when the answer came back in a few seconds, and I am going to leave it running in the background while we watch what happens behind it.

```sh
./dot.nu run openai_load \
    "http://silly-model.$INGRESS_HOST/v1/chat/completions" \
    --model qwen3-8b \
    --cases demo/autoscaling-one-case.json \
    --output tmp/cold-start-download.json &
```


Nothing comes back. The interceptor has caught that request and is holding the connection open, and the person on the other end is looking at a spinner. So while they wait, let's find out what the cluster is doing on their behalf.

```sh
kubectl --namespace inference get pods --selector app=silly-model
```

The output is as follows.

```text
NAME                    READY   STATUS    RESTARTS   AGE
vllm-86d74d98f8-8d8mj   0/1     Pending   0          10s
```

Good news, in a sense. There is a Pod. KEDA noticed the request and asked for the model back. The bad news is `Pending`, which is Kubernetes for "I would very much like to run this and there is nowhere to put it."


Why not? The events attached to that Pod spell it out.

```sh
kubectl --namespace inference describe pod --selector app=silly-model \
    | grep --after-context 6 "^Events:" \
    | cut -c1-140
```

> Without the `cut` each of those messages runs past two hundred characters, most of it a `no new claims to deallocate, preemption: ...` tail that says nothing. Drop the `cut` if you want to read them in full.

The output is as follows.

```text
Events:
  Type     Reason            Age                    From                Message
  ----     ------            ----                   ----                -------
  Warning  FailedScheduling  4m48s                  default-scheduler   0/3 nodes are available: 3 Insufficient nvidia.com/gpu. no new claim
  Normal   TriggeredScaleUp  4m46s                  cluster-autoscaler  Pod triggered scale-up: [{https://www.googleapis.com/compute/v1/proj
  Warning  FailedScheduling  4m11s (x2 over 4m14s)  default-scheduler   0/4 nodes are available: 1 node(s) had untolerated taint(s), 3 Insuf
  Warning  FailedScheduling  3m56s (x2 over 3m59s)  default-scheduler   0/4 nodes are available: 4 Insufficient nvidia.com/gpu. no new claim
```

Four lines, and they are four *different* failures.

It starts with no GPU anywhere in the cluster (`0/3 nodes`), which is exactly what we asked for and is now our problem. That failure is what wakes the node autoscaler (`TriggeredScaleUp`) and gets a machine ordered. Then a fourth node exists, and the Pod still will not go on it, because the node is not finished being born (`untolerated taint`). And then the best one: the node is ready, and it *still* reports no GPU (`4 Insufficient nvidia.com/gpu`). The machine physically has the card in it. Kubernetes cannot see the card yet, because the driver is still installing.


That is three separate kinds of waiting and we have not started the model yet. Eventually it does land, and eventually the request comes back. Here is the whole sequence.

```sh
kubectl --namespace inference get events --sort-by .lastTimestamp \
    --output custom-columns=TIME:.lastTimestamp,REASON:.reason,MSG:.message \
    | tail -13 | cut -c1-125
```

> The namespace holds events from earlier in this session too. The `tail` keeps the ones belonging to this cold start; the `cut` keeps the lines on one line each.

The output is as follows.

```text
2026-09-04T20:14:01Z   SuccessfulCreate             Created pod: vllm-86d74d98f8-8d8mj
2026-09-04T20:14:01Z   ScalingReplicaSet            Scaled up replica set vllm-86d74d98f8 from 0 to 1
2026-09-04T20:14:01Z   FailedScheduling             0/3 nodes are available: 3 Insufficient nvidia.com/gpu. no new claims to 
2026-09-04T20:14:01Z   KEDAScaleTargetActivated     Scaled apps/v1.Deployment inference/vllm from 0 to 1, triggered by 
2026-09-04T20:14:03Z   TriggeredScaleUp             Pod triggered scale-up: [{https://www.googleapis.com/compute/v1/projects/
2026-09-04T20:14:38Z   FailedScheduling             0/4 nodes are available: 1 node(s) had untolerated taint(s), 3 Insufficie
2026-09-04T20:14:53Z   FailedScheduling             0/4 nodes are available: 4 Insufficient nvidia.com/gpu. no new claims to 
2026-09-04T20:15:15Z   Scheduled                    Successfully assigned inference/vllm-86d74d98f8-8d8mj to gke-inference-gp
2026-09-04T20:15:18Z   Pulling                      Pulling image "vllm/vllm-openai:v0.27.1"
2026-09-04T20:19:26Z   Pulled                       Successfully pulled image "vllm/vllm-openai:v0.27.1" in 4m4.956s (4m8.602
2026-09-04T20:19:27Z   Created                      Container created
2026-09-04T20:19:27Z   Started                      Container started
2026-09-04T20:21:40Z   Unhealthy                    Readiness probe failed: Get "http://10.36.2.5:8000/health": dial tcp 10.3
```

Ignore the last line. That is just the readiness probe knocking on a door the engine has not finished building yet. Everything above it is the real story: KEDA reacting in about a second, then a node, then a very large image coming down the wire, then the container finally starting.


And now the number that matters. How long did that person with the spinner actually wait?

```sh
nu -c "open tmp/cold-start-download.json
    | get requests
    | select case_id first_token_ms finish_ms"
```

The output is as follows.

```text
╭───┬────────────┬────────────────┬───────────╮
│ # │  case_id   │ first_token_ms │ finish_ms │
├───┼────────────┼────────────────┼───────────┤
│ 0 │ cold-start │      623251.50 │ 643680.30 │
╰───┴────────────┴────────────────┴───────────╯
```

Read that as minutes rather than milliseconds (`first_token_ms`). Over ten minutes before the first token came out. The identical question, at the start of this video, took a few seconds.

Nothing failed here. Nothing was misconfigured, nothing crashed, no quota was hit. Every component did precisely what it was designed to do, in order, competently. Ten minutes is what it cost us to start a language model that was not already running. That is the wall.

And ten minutes to start something is not, on its own, alarming. If this were a release going out on a Tuesday afternoon, nobody would mention it.

What makes it alarming is where it sits. This is not a deployment. It is a request. Somebody asked a question, and the machinery that answers questions had to be built from nothing before it could hear them, with that person holding the line for every second of it. And when it finally finishes, they still do not have an answer. They have a model that is finally ready to start writing one.

And ours is the friendly version of it. A small model, a single card, a fast cloud network with nothing else competing for it. Everything in that wall that scales with the size of the model only gets longer from here: more weights to fetch, more weights to move onto more cards, more cards that have to be found before any of it can start. Nobody's frontier model comes up quicker than mine did.

It is worth breaking apart, because the four pieces are not equally your problem. KEDA reacting is one second, a rounding error. Getting a machine, booted, with a working GPU driver, is a minute and a quarter, which is quick for what it is. Loading the model is the biggest slice at five minutes, and that is roughly what anybody would expect.

The surprise is the container image, at four minutes. Nine gigabytes of CUDA libraries and Python, dragged across the network before anybody reads a single model weight, and it accounts for well over a third of the wall on its own. Model loading gets optimised constantly. Inference images mostly do not.

So: solved? Obviously not. But now we know where the minutes go, which means we can start attacking them.

## Cutting The Wall Down


So pick a target. Of those four phases the model is the fattest, and the instinct about what to do with it is almost universal. I had it too. The model got downloaded from the internet. It had already been downloaded from the internet an hour earlier. Why are we doing that twice? Give the thing a disk.

```sh
diff demo/autoscaling-vllm.yaml demo/autoscaling-vllm-cached.yaml
```

The output is as follows.

```diff
0a1,12
> apiVersion: v1
> kind: PersistentVolumeClaim
> metadata:
>   name: model-weights
>   namespace: inference
> spec:
>   accessModes:
>     - ReadWriteOnce
>   resources:
>     requests:
>       storage: 30Gi
> ---
9c21
<     weights: download
---
>     weights: cached
24c36
<         weights: download
---
>         weights: cached
30a43,46
>       volumes:
>         - name: weights
>           persistentVolumeClaim:
>             claimName: model-weights
46a63,68
>           env:
>             - name: HF_HOME
>               value: /weights
>           volumeMounts:
>             - name: weights
>               mountPath: /weights
```

A `PersistentVolumeClaim`, one environment variable telling Hugging Face to cache into it (`HF_HOME`), and a label saying `cached` so we can tell the two variants apart. That is the entire change. Twenty lines, no new components, nothing clever. This is the fix everybody reaches for and it is genuinely the right instinct.


Let's put it in.

```sh
kubectl apply --filename demo/autoscaling-vllm-cached.yaml
```


The disk starts out empty, so somebody has to pay for the download exactly once to fill it. Might as well be us. One request does the job.

```sh
./dot.nu run openai_load \
    "http://silly-model.$INGRESS_HOST/v1/chat/completions" \
    --model qwen3-8b \
    --cases demo/autoscaling-one-case.json \
    --output tmp/cache-populate.json
```

The output is as follows (truncated for brevity).

```text
╭───┬────────────┬───────────────┬────────────────┬───────────╮
│ # │  case_id   │ send_after_ms │ first_token_ms │ finish_ms │
├───┼────────────┼───────────────┼────────────────┼───────────┤
│ 0 │ cold-start │             0 │      251818.10 │ 272453.30 │
╰───┴────────────┴───────────────┴────────────────┴───────────╯
...
```

Note that one is quicker (`first_token_ms`), and note why: the node was already running this time and the image was already sitting on it. So what we just measured is the model on its own. Fetching it, putting it on the card, starting the engine. That is the biggest single slice of the wall, and it is the slice the disk is supposed to delete.

Now we let it go quiet, and we wait for the node to be taken away again, so that the comparison is fair. Same starting position, or it proves nothing.


Checking that the GPU node has gone again.

```sh
kubectl get nodes
```

> The Cluster Autoscaler takes its time, and how long it takes varies. If the GPU node is still listed, give it a few minutes and run the command again.

The output is as follows.

```text
NAME                                       STATUS   ROLES    AGE     VERSION
gke-inference-default-pool-6897be68-dn5m   Ready    <none>   5h41m   v1.35.7-gke.1027000
gke-inference-default-pool-6897be68-x31p   Ready    <none>   5h47m   v1.35.7-gke.1027000
gke-inference-default-pool-6897be68-xv3v   Ready    <none>   5h47m   v1.35.7-gke.1027000
```

Gone again, and it took much the same ten minutes it took the first time. That is not the cluster being slow, it is the Cluster Autoscaler's default: a node has to sit unwanted for a fixed stretch before anything will remove it, and that stretch is set by the autoscaler, not by you and not by how much the machine is costing you while it runs down.


Back to where we were before: no replicas, no GPU node, an empty cluster with a full disk sitting in it. Same request as last time.

```sh
./dot.nu run openai_load \
    "http://silly-model.$INGRESS_HOST/v1/chat/completions" \
    --model qwen3-8b \
    --cases demo/autoscaling-one-case.json \
    --output tmp/cold-start-cached.json
```

The output is as follows (truncated for brevity).

```text
╭───┬────────────┬───────────────┬────────────────┬───────────╮
│ # │  case_id   │ send_after_ms │ first_token_ms │ finish_ms │
├───┼────────────┼───────────────┼────────────────┼───────────┤
│ 0 │ cold-start │             0 │      569320.80 │ 589824.50 │
╰───┴────────────┴───────────────┴────────────────┴───────────╯
...
```

And there it is. `first_token_ms` barely moved.

We added a persistent disk, we pre-loaded sixteen gigabytes of model onto it, we removed an entire download from the critical path, and we bought back under a minute of a ten-minute wall. Something like eight percent. If you had shipped that to your team as a cold-start improvement you would have been quietly laughed at.


A result that small deserves suspicion, and the first suspect is the cache itself. Maybe it silently did nothing. So before drawing any conclusion, let's prove it worked.

```sh
kubectl --namespace inference logs deployment/vllm \
    | grep --extended-regexp "Filesystem type|Loading safetensors|init engine" \
    | cut -c1-125
```

The output is as follows (truncated for brevity).

```text
(EngineCore pid=56) INFO 09-04 22:06:08 [weight_utils.py:867] Filesystem type for checkpoints: EXT4. Checkpoint size: 15.26 G
Loading safetensors checkpoint shards:   0% Completed | 0/5 [00:00<?, ?it/s]
...
Loading safetensors checkpoint shards: 100% Completed | 5/5 [01:29<00:00, 17.83s/it]
(EngineCore pid=56) INFO 09-04 22:08:51 [core.py:348] init engine (profile, create kv cache, warmup model) took 73.31 s (comp
```

It worked perfectly. There is no download in that log at all. The engine read the whole model straight off the volume (`Filesystem type for checkpoints: EXT4`), and reading it took most of a minute and a half all by itself (`Loading safetensors checkpoint shards`). Then it spent another minute-odd building the KV cache and compiling CUDA graphs before it would answer anything at all (`init engine`).

Which explains the disappointing result. We did not remove a slow step. We swapped it for another one that takes about as long. A persistent volume is not the fast local storage the word disk makes you picture; it is storage attached over the network, and pulling sixteen gigabytes across it is its own slow errand. The cache was never going to be a shortcut. It was a different route of roughly the same length.

So break open the phase we actually attacked, and it turns out to be three things rather than one. Python and CUDA starting up, a minute and a quarter. Reading fifteen gigabytes off the volume, a minute and a half. Building the KV cache and compiling graphs, another minute and a quarter.

The weights were never the problem. That is the finding, and I did not expect it.

Reading the model is about a third of that, and no storage decision touches the other two thirds. We went after the biggest block without checking what was inside it.

Most of what is left is fixed. You are not going to make Python import faster, you are not going to talk CUDA out of compiling graphs, and the only way to skip getting a machine is to keep one running, which is the thing we are trying to avoid paying for.

That leaves the container image as the biggest slice you could actually attack. We pulled ours from Docker Hub across the public internet, which is both the worst case and the default, so a registry in the same region, a slimmer runtime, or pre-pulling onto the node pool would all buy something back. Not all of it is network, though. Some of those four minutes is unpacking nine gigabytes onto a machine, and that costs the same wherever the bytes came from.

And the image is the one part of this that does *not* grow with your model. Serve something ten times this size and the weights swamp everything else, at which point the biggest slice is also the one you can do least about.

So: solved? No. We chipped a minute off and learned that the wall is mostly load-bearing. If we cannot make it fast, the only thing left is to stop pretending it is fast.

## While You Wait

Everything so far has quietly assumed the caller is willing to hold a connection open for ten minutes. Ours was, because ours was a script with no feelings. A browser is not. A load balancer is not. Your API gateway will give up long before, and so will the person clicking the button.

So if we cannot make the wall short, we can at least stop lying about it. The interceptor is allowed to answer on the model's behalf.


Let's see what that takes.

```sh
diff demo/autoscaling-route.yaml demo/autoscaling-route-placeholder-naive.yaml
```

The output is as follows.

```diff
15a16,23
>   coldStart:
>     placeholder:
>       response:
>         statusCode: 503
>         headers:
>           Retry-After: '60'
>           Content-Type: application/json
>         body: '{"error":{"message":"The model is starting up. Retry in about a minute.","type":"model_cold_start"}}'
```

One addition. A `coldStart` block says that when nothing is ready, do not hold the line, answer straight away instead. We choose the status code, the headers and the `body` the caller gets, so we can hand back something a client actually understands rather than a timeout.


Let's try it.

```sh
kubectl apply --filename demo/autoscaling-route-placeholder-naive.yaml
```


The model is asleep. Let's ask it something.

```sh
curl --silent --show-error --dump-header - \
    --write-out "\nHTTP %{http_code} in %{time_total}s\n" \
    --header "Content-Type: application/json" \
    --data-binary "@demo/batching-request.json" \
    "http://silly-model.$INGRESS_HOST/v1/chat/completions"
```

The output is as follows.

```text
HTTP/1.1 503 Service Unavailable
Content-Length: 100
Content-Type: application/json
Date: Fri, 04 Sep 2026 22:22:04 GMT
Retry-After: 60

{"error":{"message":"The model is starting up. Retry in about a minute.","type":"model_cold_start"}}

HTTP 503 in 0.361242s
```

A fraction of a second instead of ten minutes (`time_total`). A `Retry-After` header the client can actually obey. An error body shaped like the API's own errors, so whatever is calling us can branch on it instead of choking on a dead socket.

The wall has not moved. We have converted one ten-minute silence into a series of polite refusals. But that is a genuine improvement, because a caller can do something with a refusal and can do nothing at all with a spinner. So: solved?


Let's keep asking, and watch the cluster instead of the responses.

```sh
for i in $(seq 1 10); do
    curl --silent --output /dev/null --write-out "%{http_code} " \
        --header "Content-Type: application/json" \
        --data-binary "@demo/batching-request.json" \
        "http://silly-model.$INGRESS_HOST/v1/chat/completions"
    sleep 2
done
```

The output is as follows.

```text
503 503 503 503 503 503 503 503 503 503
```

Ten refusals, all delivered promptly and politely. Now the part that matters, which is what the cluster did about them.


```sh
kubectl --namespace inference get scaledobject vllm

kubectl --namespace inference get pods --selector app=silly-model
```

The output is as follows.

```text
NAME   SCALETARGETKIND      SCALETARGETNAME   MIN   MAX   READY   ACTIVE   FALLBACK   PAUSED   TRIGGERS        AUTHENTICATIONS   AGE
vllm   apps/v1.Deployment   vllm              0     1     True    False    False      False    external-push                     169m
No resources found in inference namespace.
```

Ten requests. Ten refusals. And now look at what did *not* happen. `ACTIVE` is still false. There is still no Pod.

The model is not starting. It is never going to start. Every single caller will be told to come back in sixty seconds, and in sixty seconds they will be told exactly the same thing, forever. Nothing in this cluster is reporting an error. Every component is healthy. The service is simply dead, politely, at scale.

The cause is that field I told you to remember. `concurrency` counts requests that are *in flight*, and being in flight is precisely what a placeholder stops a request from doing. It is answered and gone in a third of a second, so the count never leaves zero, so KEDA is never told that anybody wants this model. The feature we added to make the cold start survivable is the same feature that destroys the evidence that a cold start is needed.


The fix is to count arrivals instead of counting time.

```sh
diff demo/autoscaling-route-placeholder-naive.yaml demo/autoscaling-route-placeholder.yaml
```

The output is as follows.

```diff
14,15c14,17
<     concurrency:
<       targetValue: 16
---
>     requestRate:
>       targetValue: 10
>       window: 1m
>       granularity: 1s
```

`requestRate` counts arrivals over a window no matter how fast each one is answered, so a refused request still counts as somebody asking.


Apply that and the wake-up starts working again.

```sh
kubectl apply --filename demo/autoscaling-route-placeholder.yaml
```


Then we ask again, exactly as before.

```sh
for i in $(seq 1 10); do
    curl --silent --output /dev/null --write-out "%{http_code} " \
        --header "Content-Type: application/json" \
        --data-binary "@demo/batching-request.json" \
        "http://silly-model.$INGRESS_HOST/v1/chat/completions"
    sleep 2
done
```

The output is as follows.

```text
503 503 503 503 503 503 503 503 503 503
```

Ten refusals again. From outside the cluster nothing has changed at all, which is worth sitting with for a second: the broken version and the fixed one are indistinguishable to every caller. The difference is all on the inside.


```sh
kubectl --namespace inference get scaledobject vllm

kubectl --namespace inference get pods --selector app=silly-model
```

The output is as follows.

```text
NAME   SCALETARGETKIND      SCALETARGETNAME   MIN   MAX   READY   ACTIVE   FALLBACK   PAUSED   TRIGGERS        AUTHENTICATIONS   AGE
vllm   apps/v1.Deployment   vllm              0     1     True    True     False      False    external-push                     174m
NAME                    READY   STATUS    RESTARTS   AGE
vllm-7bf646f966-4k275   0/1     Pending   0          67s
```

`ACTIVE` flips to true, a Pod appears, and the wall starts running down in the background while callers get told to retry.

Having just watched one of these fail quietly, the obvious worry is whether this one has a threshold too. Does a trickle of traffic sit below the target forever and never wake anything up? It does not, and the reason is a distinction worth carrying around: waking up and sizing up are two different decisions. The target rate governs how many replicas to run once something is running. Whether to run anything at all is a separate question, and the answer to it is simply whether anybody asked.

One thing about that number, because it matters shortly. It is only deciding whether the model exists at all. It is a perfectly good wake-up signal and a genuinely terrible way to decide how many replicas you need, for reasons we are about to run into.

## The Second Replica

Everything up to here has been about the gap between zero and one. Getting rid of the model when nobody wants it, and getting it back when somebody does. That is one half of autoscaling and it is a perfectly sensible thing to want.

The other half is the direction the word usually brings to mind. Traffic climbs past what a single replica can absorb, and you want a second one. You need both, and most people reach for this one first.

That is a different problem, and it turns out to be the same problem, and it ends worse.

First, some housekeeping, and it is a sharper lesson than it looks. That disk we added is ReadWriteOnce, which means one node at a time. Each of our GPU machines holds exactly one card, and the first replica is already using the card on the node the disk is attached to. So a second replica needs a different node, and on a different node it cannot have the disk.

Which is worse than the cache merely stopping being useful. The cache stops you scaling out at all: the second replica would sit there, never starting, until somebody took the volume away.

Storage that fixes your cold start and storage that survives a second replica are not the same storage, and you have to choose. We are choosing the second one, so we go back to downloading.

Two changes then, and the order matters more than it looks. The ScaledObject goes first, because it is the thing that pins a replica in place. Swap the Deployment while the old object still permits zero and the new Pod gets scaled away in the middle of its own download, which is a genuinely infuriating ten minutes to lose.


Here is what changes.

```sh
diff demo/autoscaling-zero.yaml demo/autoscaling-queue.yaml
```

The output is as follows.

```diff
9,10c9,10
<   minReplicaCount: 0
<   maxReplicaCount: 1
---
>   minReplicaCount: 1
>   maxReplicaCount: 2
14c14
<     - type: external-push
---
>     - type: metrics-api
16,17c16,20
<         scalerAddress: keda-add-ons-http-external-scaler.keda:9090
<         interceptorRoute: vllm
---
>         url: http://silly-model.inference:8000/metrics
>         format: prometheus
>         valueLocation: vllm:num_requests_waiting
>         targetValue: "4"
>         activationTargetValue: "0"
```

`minReplicaCount` goes to one and `maxReplicaCount` to two. Scale to zero is over; from here there is always one replica running and at most one more.

The trigger changes too, and this is where inference stops behaving like a web app. Requests per second is a fine signal when every request costs roughly the same. Here one request might be twenty tokens and the next two thousand, so counting them tells you almost nothing about how hard the GPU is working.

The engine already knows the answer, so we ask it. KEDA scrapes vLLM's own metrics endpoint directly (`url`, `format: prometheus`), and that second field is worth a second look, because it names a text format rather than a dependency. No Prometheus server is involved in this decision. KEDA reads the number straight off the engine, so the thing deciding when to scale is not sitting downstream of a metrics pipeline you have to keep alive as well.


ScaledObject first, then the Deployment, in that order. And the Deployment we are going back to is the original one, with no volume attached, so the model downloads on every start again. The claim itself stays behind in the cluster, unused and still billed, until we tear everything down at the end.

```sh
kubectl apply --filename demo/autoscaling-queue.yaml

kubectl apply --filename demo/autoscaling-vllm.yaml
```


Once it is serving again, here is what KEDA is now watching.

```sh
kubectl --namespace inference exec deployment/vllm -- \
    curl --silent http://localhost:8000/metrics \
    | grep "^vllm:num_requests"
```

The output is as follows.

```text
vllm:num_requests_running{engine="0",model_name="qwen3-8b"} 0.0
vllm:num_requests_waiting{engine="0",model_name="qwen3-8b"} 0.0
vllm:num_requests_waiting_by_reason{engine="0",model_name="qwen3-8b",reason="capacity"} 0.0
vllm:num_requests_waiting_by_reason{engine="0",model_name="qwen3-8b",reason="deferred"} 0.0
```

The one that matters is `vllm:num_requests_waiting`. Those are requests the engine has accepted and has no room to advance. Not requests arriving, not requests in progress. Requests stuck. That is about as close as you get to a machine telling you it is full.

Both are zero right now, because nothing is happening. Time to change that.

The obvious thing is to fire a pile of requests at it and see what happens. I did that first, and it taught me something about my own testing rather than about Kubernetes: the burst was over in under a minute, and a replica takes minutes to exist, so the traffic had gone long before help arrived. What I measured was my own impatience.


So the schedule has to outlast a cold start. Same requests, spread out, arriving steadily for longer than it takes to build a replica.

```sh
./dot.nu generate load_cases
```

The output is as follows.

```text
Wrote demo/autoscaling-sustained-cases.json: 2400 requests at 2/s over 1200s
```

The rate matters more than the total. It has to sit above what one replica can serve, or no queue builds and nothing ever scales. It also has to sit below what two can serve, or the queue never drains and the second replica changes nothing you can see. Between those two numbers is where the hand-over is visible, and neither is as fixed as it sounds. A saturated engine is slower than a merely busy one, because a full batch has its sequences competing for the same cache. Measure that ceiling gently and you will pick a rate the replica quietly absorbs, with nothing ever queuing for KEDA to see.


With that settled, we point it at the model and leave it going.

```sh
./dot.nu run openai_load \
    "http://silly-model.$INGRESS_HOST/v1/chat/completions" \
    --model qwen3-8b \
    --cases demo/autoscaling-sustained-cases.json \
    --output tmp/sustained.json &
```


While that runs, we look in on the replicas.

```sh
kubectl --namespace inference get pods --selector app=silly-model
```

> A single snapshot only catches one moment of this. Run it every so often while the load is going to watch the second replica appear, sit there unready, and eventually start taking a share.

The output is as follows.

```text
NAME                    READY   STATUS    RESTARTS   AGE
vllm-86d74d98f8-f7thp   1/1     Running   0          32m

NAME                    READY   STATUS    RESTARTS   AGE
vllm-86d74d98f8-c7x9r   0/1     Running   0          6m32s
vllm-86d74d98f8-f7thp   1/1     Running   0          39m

NAME                    READY   STATUS    RESTARTS   AGE
vllm-86d74d98f8-c7x9r   1/1     Running   0          13m
vllm-86d74d98f8-f7thp   1/1     Running   0          45m
```

The first snapshot is where we started: one replica, taking all of it. By the second, KEDA has already seen the queue and asked for another, and Kubernetes has already created it. That happened about a minute in.

And then nothing happens for eleven minutes. A node has to be found, nine gigabytes of image pulled onto it, a model loaded, and none of it goes faster because there is a queue waiting on the other side. The third snapshot is the far side of that wait: two replicas, both ready, twelve minutes after the first request arrived.

Every request in that middle stretch was served by a single replica that had already been told it needed help. The help was ordered on time. It just could not get there.


And the requests themselves recorded what all of that felt like from the outside.

```sh
nu -c 'open tmp/sustained.json
    | get requests
    | insert minute {|r| ($r.send_after_ms / 60000) | math floor}
    | group-by minute
    | transpose minute rows
    | each {|g| {minute: $g.minute, sent: ($g.rows | length), ttft_p50_ms: ($g.rows | get ttft_ms | math median | math round)}}'
```

The output is as follows.

```text
╭────┬────────┬──────┬─────────────╮
│  # │ minute │ sent │ ttft_p50_ms │
├────┼────────┼──────┼─────────────┤
│  0 │ 0      │  120 │        2522 │
│  1 │ 1      │  120 │        9049 │
...
│ 10 │ 10     │  120 │       68888 │
│ 11 │ 11     │  120 │       74346 │
│ 12 │ 12     │  120 │       69531 │
│ 13 │ 13     │  120 │       65034 │
│ 14 │ 14     │  120 │       48142 │
...
│ 19 │ 19     │  120 │       19193 │
╰────┴────────┴──────┴─────────────╯
```

Read `ttft_p50_ms` down the column. It starts at two and a half seconds, which is about what one of these answers costs when nothing is queued ahead of it. Then it climbs, every single minute, for eleven minutes, until the median caller is waiting seventy-four seconds to see the first token come back. Thirty times worse than when we started, and every one of those minutes is a minute in which help had already been asked for.

Minute twelve is where it turns. That is the second replica coming ready. From there the line falls: sixty-nine seconds, sixty-five, forty-eight, and it is still falling when the traffic stops. It never does get back to two and a half, because a backlog that took eleven minutes to build does not clear in eight.

And now consider what that means for traffic that is not this patient. We watched a steady stream get help eventually. A spike shorter than the wall gets none at all: the replica is requested, a node is bought, an image is pulled, and by the time any of it is ready the spike is a line on a graph somebody looks at afterwards. You paid for the machine and served nobody with it.

One caveat about how those requests found two replicas, because it will bother anyone who has thought about this properly. The interceptor forwards to the Service, so they were spread by ordinary round-robin. Nothing in this setup knows which replica has a shorter queue or a warmer cache, and you can watch that cost you. While the first replica was still a hundred requests deep, the second one's queue sat at zero: it was taking its half of the new arrivals and nothing else, because round-robin has no mechanism for handing back a backlog. Routing that actually knows is a genuinely interesting problem and it is a different video.

So, the verdict, and it is a narrower one than the word autoscaling makes it sound. Scaling out works. What it cannot do is work *reactively*, and our own configuration is the clearest example of why. We scaled on the queue, which is the honest signal for this replica is full, and full is already too late when the fix takes minutes to arrive.

Notice what full did not mean, though. Nothing failed. Nothing was dropped, nothing timed out, nobody got an error page. The engine batches, so the overflow queued up inside it and got worked through a piece at a time, and every answer arrived late rather than never. That is a far softer landing than most systems give you at capacity, and it is worth knowing before anybody panics.

So this is a choice rather than a catastrophe. You can run hot and accept that some stretch of your traffic is slow while a replica is built, which costs nothing and irritates people. Or you can scale before you are full, on sequences in flight against the engine's limit or on how much of the KV cache has gone, both of which start moving well before anything queues. That gets you a replica that arrives on time, and it charges you every time the signal turns out to be a false alarm.

Which is the real shape of it. Autoscaling an inference workload is not a reaction to traffic. It is a forecast, and the lead time is however long your model takes to come up, so the choice is between paying for wrong guesses and living with slow minutes.

## Destroy

That is everything, so let's not leave a GPU running by accident. This deletes the cluster, the node pools, the disk we created, and the project around them.

```sh
./dot.nu destroy autoscaling $PROVIDER

exit
```

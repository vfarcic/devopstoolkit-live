
+++
title = 'Unlock the Power of GPUs in Kubernetes for AI Workloads'
date = 2023-10-06T16:00:00+00:00
draft = false
+++

Here's a question. Where do we run AI models? Everyone knows the answer to that one. We run them in **servers with GPUs**. GPUs are much more efficient at processing AI models or, to be more precise, at inference.

Here's another question. How do we manage models across those servers? The answer to that question is... Kubernetes.

<!--more-->

{{< youtube zuRKdveFuZ4 >}}

**Kubernetes is the de-facto standard to manage any type of workloads**, be it stateless apps, stateful apps, jobs, AI models, or anything else. We just need to tell Kubernetes how much memory and CPU our workloads need and it will figure out where to put them. However, with AI our primary requirement is not the amount of memory and CPU but, rather, **the amount of GPU** we need, whether GPU is dedicated to a single process exclusively or shared across processes, and a few other things. In other words, the way we manage GPU-based workloads is different from more traditional workloads, yet somehow the same.

Today we'll skip explanation why AI works better with GPUs than CPUs and focus on running AI models and managing GPUs in Kubernetes. We'll explore not only how to run AI models in Kubernetes but also how to do it in a way that **does not result in bankrupcy**.

## Setup

```sh
git clone https://github.com/vfarcic/kubernetes-gpu-demo

cd kubernetes-gpu-demo
```

> Watch https://youtu.be/WiFLtcBvGMU if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell
```

> The setup currently works only with Google Cloud. Some modifications to both the setup and the steps that follow might be required if you prefer using a different provider.

```sh
chmod +x setup.nu

./setup.nu

source .env
```

## Using GPUs for AI in Kubernetes

I already have a Kubernetes cluster running. It's a "normal" cluster that happens to be GKE in Google Cloud. Nevertheless, the logic behind the rest of the post should be the same no matter where your cluster is.

That cluster does not have any nodes with GPUs so there are at least three tasks we need to perform to make it "AI-ready".

To begin with, we need to figure out how to create **nodes or node groups with GPUs**. Now, those nodes would not be of much use by themselves. We would also need to install **device plugins** that will let Pods access specialized hardware features which, in this case, are GPUs. Finally, we need to **instruct Pods to use GPUs**.

Let's start from the begining.

Here are the nodes of my cluster.

```sh
kubectl get nodes
```

The output is as follows (truncated for brevity).

```
NAME                      STATUS  ROLES   AGE   VERSION
gke-dot-default-pool-...  Ready   <none>  112s  v1.29.7-gke.1008000
gke-dot-default-pool-...  Ready   <none>  38s   v1.29.7-gke.1008000
```

As I already mentioned, none of the nodes currently used by that cluster have GPUs. Hence, we should add a node group or a node pool with GPUs to the cluster. But, before we do that, we need to figure out what is available for a given provider. In case of Google Cloud we can list all those currently available.

```sh
gcloud compute accelerator-types list --project $PROJECT_ID
```

The output is as follows (truncated for brevity).

```
NAME                   ZONE        DESCRIPTION
...
nvidia-l4              us-east1-b  NVIDIA L4
nvidia-l4-vws          us-east1-b  NVIDIA L4 Virtual Workstation
nvidia-tesla-a100      us-east1-b  NVIDIA A100 40GB
nvidia-tesla-p100      us-east1-b  NVIDIA Tesla P100
nvidia-tesla-p100-vws  us-east1-b  NVIDIA Tesla P100 Virtual Workstation
...
```

Those are accelerators that we can pick. We also need to select a machine just as we'd normally do except that those with GPUs tend to be labeled differently.

Anyways... We'll `create` a new node pool that will be attached to the `dot` cluster. Since I'm cheap, we'll select the smallest machine type (`a2-highgpu-1g`), only `1` node and, most importantly, choose `nvidia-tesla-a100` as the GPU.

```sh
gcloud container node-pools create dot-gpu \
    --project $PROJECT_ID --cluster dot --zone us-east1-b \
    --machine-type a2-highgpu-1g --num-nodes 1 \
    --no-enable-autoupgrade \
    --accelerator type=nvidia-tesla-a100,count=1,gpu-driver-version=default
```

The output is as follows.

```
...
Creating node pool dot-gpu...done.
Created [https://container.googleapis.com/v1/projects/dot-20240819000228/zones/us-east1-b/clusters/dot/nodePools/dot-gpu].
NAME     MACHINE_TYPE   DISK_SIZE_GB  NODE_VERSION
dot-gpu  a2-highgpu-1g  100           1.29.7-gke.1008000
```

Here are the nodes of the cluster.

```sh
kubectl get nodes
```

The output is as follows.

```
NAME                     STATUS ROLES  AGE   VERSION
gke-dot-default-pool-... Ready  <none> 10m   v1.29.7-gke.1008000
gke-dot-default-pool-... Ready  <none> 8m47s v1.29.7-gke.1008000
gke-dot-dot-gpu-...      Ready  <none> 47s   v1.29.7-gke.1008000
```

That's it. That's all it takes to add nodes with GPUs to the cluster and with that single action we not only added nodes with GPUs (`gke-dot-dot-gpu-*`) to the cluster but we made sure that those nodes have the necessary device plugins installed.

Now, to be clear, not all providers are just as easy as Google Cloud and the instructions for your favorite provide might vary. What matter is that we did two out of three steps to run AI models in the cluster and the rest is going to be the same no matter which provider you prefer using.

We're almost ready to use those new GPU nodes. The only thing missing is to double check `taints` in those nodes.

```sh
kubectl get node \
    --selector cloud.google.com/gke-nodepool=dot-gpu \
    --output jsonpath="{.items[0].spec.taints[0]}" \
    | jq .
```

The output is as follows.

```json
{
  "effect": "NoSchedule",
  "key": "nvidia.com/gpu",
  "value": "present"
}
```

The important thing to note is that those nodes have the **`NoSchedule` taint** effect meaning that no Pods will be scheduled to run in them unless we explicitly specify the matching toleration.

Now we should be able to run our own or third party apps that contain models that require GPUs. We'll opt for the latter since it's easier and, by exploring the outcome, will give us a better understanding what is required.

We'll run Ollama which already has a Helm chart available and all we have to do is specify the type of GPU it should use.

> I will assume that you are familiar with Ollama AI models. If that's not the case and you would like me to explore it in more detail, all you have to do is let me know in the comments of this video.

Here's an example Helm values file we'll use.

```sh
cat ollama-values.yaml
```

The output is as follows.

```yaml
ollama:
  gpu:
    enabled: true
    type: nvidia
    number: 1
  models:
  - llama2
ingress:
  enabled: true
  className: traefik
  hosts:
  - host: ollama.35.229.75.73.nip.io
    paths:
    - path: /
      pathType: Prefix
```

Over there, we're specifying that `gpu` usage should be `enabled`, that we are using `nvidia` for that and that the application should use only `1` GPU. Further on, since Ollama allows us to use quite a few models, we're specifying that today we're interested only in `llama2`. The rest are `ingress` values that are not relevant for today's subject.

Now, to be clear, those values by themselves will not help you understand how to use GPU with your own or with any other third-party models. However, later on, we'll inspect what Ollama deployed and that will enable us to deduce what we should specify if we'd like to use GPU with virtually any application or any model since the logic is always the same.

Let's apply the chart.

```sh
helm upgrade --install ollama ollama \
    --repo https://otwld.github.io/ollama-helm \
    --values ollama-values.yaml \
    --namespace ollama --create-namespace --wait
```

The output is as follows.

```
Release "ollama" does not exist. Installing it now.
NAME: ollama
LAST DEPLOYED: Mon Aug 19 00:23:35 2024
NAMESPACE: ollama
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  http://ollama.35.229.75.73.nip.io/
```

Now comes the important part. Since all the changes we should do to make containers in Pods use GPU are in the manifest of those Pods, we can inspect a Deployment Ollama created. Since that Deployment creates a ReplicaSet which creates the Pods, that should give us a clue as to what we should do to use GPUs.

```sh
kubectl --namespace ollama get deployment ollama \
    --output yaml | yq .
```

The output is as follows (truncated for brevity).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  ...
  name: ollama
  ...
spec:
  ...
  template:
    ...
    spec:
      containers:
        - env:
            ...
            - name: NVIDIA_DRIVER_CAPABILITIES
              value: compute,utility
            - name: NVIDIA_VISIBLE_DEVICES
              value: all
          ...
          resources:
            limits:
              nvidia.com/gpu: "1"
          ...
      tolerations:
        - effect: NoSchedule
          key: nvidia.com/gpu
          operator: Exists
      ...
```

The first thing you'll notice are environment variables `NVIDIA_DRIVER_CAPABILITIES` and `NVIDIA_VISIBLE_DEVICES`. The first one specifies that GPU should accelerate `compute` and `utility`. We could have specified that it should accelerate *video* as well but, since Llama2 model does not support video, there is no need for that one. We could set it to `all` if we're too lazy. That's what is used with *NVIDIA_VISIBLE_DEVICES* which specified which devices will be injected by the NVIDIA Container Runtime.

Those environment variables are not that important. What is important is that `limits` in `resources` is set to use maximum `1` `nvidia.com/gpu`. That should be self-explanatory. I can't affort more than one GPU so that's what I'm using.

Finally, we have `tolerations` which, in general, allow us to tell the scheduler to place Pods only on nodes with matching taints. In this case, we're telling it to place the Pods with the model only on nodes that have the `key` set to `nvidia.com/gpu`. That's the same key that was set to the nodes of the node pool with GPUs we created earlier. As a result, it is clear to Kubernetes that the Pod with Llama2 model should run only on specific nodes, on those with GPUs.

Since those instructions were set to the Deployment template, the end result is that we have a Pod running in the only node with GPU and is using that GPU to accelerate calculations.

Now we can use the local *ollama* client to communicate with the model running in the cluster. To do that, first we'll define the environment variable `OLLAMA_HOST` to point to the Ingress endpoint,...

```sh
export OLLAMA_HOST="http://ollama.$INGRESS_HOST.nip.io"
```

...and list all available models.

```sh
ollama list
```

The output is as follows.

```
NAME          ID           SIZE   MODIFIED
llama2:latest 78e26419b446 3.8 GB 13 minutes ag
```

There is only one model since that's what we specified. The important part is that we proved that the model is running in the cluster and that we can access it.

Let's use it by executing `ollama run llama2` and passing it a question.

```sh
ollama run llama2 "How to run GPU in Kubernetes?"
```

The output is as follows (truncated for brevity).

```
Running a GPU (Graphics Processing Unit) in a Kubernetes cluster can be challenging due to the lack
of native support for GPUs in containers. However, there are several ways to run a GPU in a
Kubernetes cluster:

1. Using NVIDIA's GPU Pod Dispatcher: NVIDIA provides a tool called GPU Pod Dispatcher that allows
you to run GPU-intensive workloads on Kubernetes clusters. The GPU Pod Dispatcher manages the
allocation of GPU resources to containers and schedules them on the appropriate nodes in the cluster.
...
```

The first thing you'll notice is that it was lightning fast. GPU accelerated the calculation and we got an answer almost instantly.

Now, let's say that we would like to run a second model. Normally, that would be something completely different but, for the sake of simplicity, we'll apply the same one again but with a different name.

```sh
helm upgrade --install ollama2 ollama \
    --repo https://otwld.github.io/ollama-helm \
    --values ollama-values.yaml \
    --namespace ollama --create-namespace
```

What do you think happened? Is the second model running?

Let's check it out.

```sh
kubectl --namespace ollama get pods
```

The output is as follows (truncated for brevity).

```
NAME        READY STATUS  RESTARTS AGE
ollama-...  1/1   Running 0        18m
ollama2-... 0/1   Pending 0        2m51s
```

The second model is in the `Pending` state. Kubernetes cannot place it and we can try to figure out why is that so by describing that Pod.

```sh
kubectl --namespace ollama describe pod \
    --selector app.kubernetes.io/instance=ollama2
```

The output is as follows (truncated for brevity).

```
...
Events:
  Type     Reason             Age                From                Message
  ----     ------             ----               ----                -------
  Normal   NotTriggerScaleUp  4m1s               cluster-autoscaler  pod didn't trigger scale-up: 1 Insufficient nvidia.com/gpu
  Warning  FailedScheduling   4m (x2 over 4m1s)  default-scheduler   0/3 nodes are available: 3 Insufficient nvidia.com/gpu. preemption: 0/3 nodes are available: 3 No preemption victims found for incoming pod.
```

We can see that there are no available GPUs. None of the nodes can accomodate that Pod. The explanation for that behavior is simple. We have only a single node with a single GPU. We tried to run two models, each requesting a single GPU. The first one got it, while the second is left wondering why there are no GPUs available for it.

Now, there are two solutions for this problem. The obvious one would be to create another node with a GPU so that it can be assigned to the second model. If our models would be running at full capacity, that could be the solution. However, that's not the case today. The first model is hardly used and it would be a waste of money to get an additional GPU for the second model that also might not be used all the time. A better solution could be to have those two models share the same GPU. After all, GPUs are very expensive and we might not have an infinite number of them.

Fortunately, we can tell Kubernetes nodes to **partition GPUs**. The way how to accomplish that might differ from one provider to another. Today, we'll explore how to do it in Google Cloud and you should be able to adapt it to whichever provider you might be using.

We'll start by destroying the node pool we created,...

```sh
gcloud container node-pools delete dot-gpu --project $PROJECT_ID \
    --cluster dot --zone us-east1-b --quiet
```

...and create a new one, just as we did before but, this time, with an additional `accelerator` instruction to partition GPU into `1g.5gb` sizes.

```sh
gcloud container node-pools create dot-gpu \
    --project $PROJECT_ID --cluster dot --zone us-east1-b \
    --machine-type a2-highgpu-1g --num-nodes 1 \
    --no-enable-autoupgrade \
    --accelerator type=nvidia-tesla-a100,count=1,gpu-driver-version=default,gpu-partition-size=1g.5gb
```

The `gpu-partition-size` of `1g.5gb` refers to a GPU instance with one compute unit or 1/7th of streaming multiprocessors on the GPU, and one memory unit of 5 GB.

Now, to be clear, there's a bit of dark magic involved so you might not be able to set partition sizes without checking the documentation. Don't try to use intuition since it won't get you far. Consult the documentation. Don't try to find logic. Just do what it says.

Anyways... Now that we replaced the old with a new node that partitions GPU usage to up to 7 units, we can take another look at the Pods with Ollama models.

```sh
kubectl --namespace ollama get pods
```

The output is as follows (truncated for brevity).

```
NAME        READY STATUS  RESTARTS AGE
ollama-...  1/1   Running 0        7m50s
ollama2-... 1/1   Running 0        12m
```

We can see that, this time, both Pods are `Running`. Even though each of them requested one GPU, the node itself now allows up to seven Pods to share that same GPU. Hence, both Pods are now running and I won't go bankrupt.

That's it. That's all you should know to run models in containers inside a Kubernetes cluster. Nevertheless, before we part ways, I have a suggestion that might improve your resource utilization even more.

If the models are not used heavily and constantly, we might be able to optimize the setup even more by using Knative. Instead of having long-running Pods, with Knative we can have a system that scales unused Pods to zero or scales them up to whichever number is required to meet the demand. Some might call that serverless computing.

Now, even though Knative is typically used with "normal", often stateless applications, I would argue that it could be a perfect match for running models that require GPUs. Please let me know in the comments if you'd like me to explore such a solution.

Another alternative could be to use KubeVirt to run models inside virtual machines managed by Kubernetes. I can explore that one as well. Just let me know in the comments.

Thank you for watching.
See you in the next one.
Cheers.

## Destroy

```sh
chmod +x destroy.nu

./destroy.nu

exit
```


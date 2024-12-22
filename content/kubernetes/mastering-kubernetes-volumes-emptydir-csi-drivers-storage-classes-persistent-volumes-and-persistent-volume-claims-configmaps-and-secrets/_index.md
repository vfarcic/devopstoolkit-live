+++
title = 'Mastering Kubernetes: Volumes (emptyDir, CSI Drivers, Storage Classes, Persistent Volumes and Persistent Volume Claims, ConfigMaps, and Secrets)'
date = 2024-12-23T14:00:00+00:00
draft = false
+++

Today we'll go through Kubernetes volumes. We'll explore local volumes like emptyDir, CSI Drivers, Storage Classes, Persistent Volumes and Persistent Volume Claims, ConfigMaps, and Secrets. It does not matter what you do in Kubernetes, **volumes are unavoidable**. You will use them, so you need to understand what they're used for, when to use them, and how to use them.

<!--more-->

{{< youtube yYQXKiiJzS8 >}}

## Intro

There's a lot to cover, so let's skip the pep talk and jump right into it.

## Setup

```sh
git clone https://github.com/vfarcic/kubernetes-demo

cd kubernetes-demo

git pull

git checkout volumes
```

> Watch [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell
```

> Watch [The Future of Shells with Nushell! Shell + Data + Programming Language](https://youtu.be/zoX_S6d-XU4) if you are not familiar with Nushell. Alternatively, you can inspect the `setup/kubernetes.nu` script and transform the instructions in it to Bash or ZShell if you prefer not to use that Nushell script.

```sh
chmod +x setup/volume.nu

./setup/volume.nu

source .env
```

## Kubernetes Volumes (emptyDir)

Let's start with the simplest volume usage.

```sh
cat volume/empty-dir.yaml
```

The output is as follows.

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: silly-demo
  labels:
    app.kubernetes.io/name: silly-demo
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: silly-demo
  template:
    metadata:
      labels:
        app.kubernetes.io/name: silly-demo
    spec:
      containers:
        - image: ghcr.io/vfarcic/silly-demo:1.4.235-alpine
          name: silly-demo
          ports:
            - containerPort: 8080    
          readinessProbe:
            httpGet:
              path: /
              port: 8080
          env:
            - name: DB
              value: fs
          volumeMounts:
            - mountPath: /cache
              name: silly-cache
      volumes:
        - name: silly-cache
          emptyDir: {}
```

Pods definition, in this case defined through the `Deployment`, can include a list of `volumes` which are, in some form or another, are directories with a file system that can be local or remote, permanent or ephemeral.

In this specific case, we are specifying that we would like to have a volume type `emptyDir`.

**emptyDir** is an **ephemeral** type of a volume. It exists only as long as the Pod exists. It is created when a Pod is created and it is destroyed when the Pod is no more. In other words, every time we start a Pod, *emptyDir* volume is empty, hence the name.

The important note is that *emptyDir* volumes are removed when Pods are removed, not when containers crash. If a container goes down, a new one is created in the same Pod, so the files we might have in an *emptyDir* are still there.

All that "it exists only as long as a Pod exists" might leave you wondering what the usage of *emptyDir* is.

The obvious one is for local development. If we work with an application that would mount extenal storage as volumes in production, we might want a simpler setup in development given that, more often than not, we do not need persistent storage just to test whether an app works.

Inside the container specification we have `volumeMounts` entry that does the actual mount of the `silly-cache` volume as the `/cache` directory. So, whichever files are in that volume will be accessible inside the */cache* path.

Let's see it in action by applying the manifest.

```sh
kubectl --namespace a-team apply --filename volume/empty-dir.yaml
```

Next, we'll send a *POST* request to the application which, in turn, should store the data to the file system. That way we can observe what's happening with the volume we mounted.

Here it goes.

Sending one `POST` request,...

```sh
curl -XPOST \
    "http://silly-demo.$INGRESS_HOST.nip.io/video?id=1&title=something"
```

...and another for good taste.

```sh
curl -XPOST \
  "http://silly-demo.$INGRESS_HOST.nip.io/video?id=2&title=else"
```

Next, we'll send a request to the app which requires it to fetch data from the volume and send it back to us.

```sh
curl "http://silly-demo.$INGRESS_HOST.nip.io/videos" | jq .
```

The output is as follows.

```json
[
  {
    "id": "1",
    "title": "something"
  },
  {
    "id": "2",
    "title": "else"
  }
]
```

As we can see, the volume seem to be working. The application is writing and reading files stored in it.

If we would like to be 100% sure that's what's happening, we can `exec` into the container associated with the `service` `silly-demo` and list (`ls`) all the files from the `/cache` directory that was mounted as a volume.

```sh
kubectl --namespace a-team exec service/silly-demo \
    --stdin --tty -- ls /cache/
```

The output is as follows.

```
videos.yaml
```

We can see that the app created the `videos.yaml` file.

Now comes an important note related to ephemeral volumes. They exist only as long as the Pod that mounted them exists.

We can demonstrate that by deleting the Pod.

```sh
kubectl --namespace a-team delete pod \
    --selector app.kubernetes.io/name=silly-demo
```

Since the Pod is managed by a ReplicaSet which is managed by a Deployment, the ReplicaSet Controller should have noticed that a Pod is missing and created a new one.

We can confirm that by retrieving all the `pods`.

```sh
kubectl --namespace a-team get pods
```

The output is as follows.

```
NAME           READY STATUS  RESTARTS AGE
silly-demo-... 1/1   Running 0        5s
```

Now comes the critical moment of truth.

If we send the same request to the app as the one we sent earlier,...

```sh
curl "http://silly-demo.$INGRESS_HOST.nip.io/videos"
```

...this time we're getting the output that states that the file `/cache/videos.yaml` does not exist.

Similarly, if we `exec` into the main container of that Pod and list (`ls`) all the files in the `/cache` directory,...

```sh
kubectl --namespace a-team exec service/silly-demo \
    --stdin --tty -- ls /cache/
```

...we can see that there are no files there.

Ephemeral volumes, like *emptyDir* are temporary and should never be used for storing data permanently. More often than not, they should be used for development, for replica-related cache, or for any other usage when there is no need to keep the data after Pods are removed.

If we would like a permanent storage, we need PersistentVolumes but to get to them, first we need to talk about CSI Drivers.

## Kubernetes Container Storage Interface (CSI) Drivers and Storage Classes

Kubernetes itself does not support any particular storage, partly because there are many available options and what we will use depends greatly on where we run our Kubernetes clusters. AWS storage is different from Azure storage which is different from what Google Cloud offers. If we run in our own datacenters, there is almost an infinite number of storage options we could be using. On top of all those, there are also some options that work everywhere and are based on very different principles than "traditional" block storage or Network File System (NFS).

All in all, it would be foolish for Kubernetes to try to bake-in all the avaialble options and almost equally silly to select a few at the expense of all the others. So, Kubernetes decided not to support any storage option but, instead, to define a **standard** that any storage vendor can implement as a way to plug itself into Kubernetes.

That standard is called Container Storage Interface or **CSI**. It acts as a **bridge between Kubernetes and storage systems**. From the Kubernetes perspective, CSI is what allows it to talk to those systems through a unified interface. From the vendor's perspective, it's what allows them to create plugins through which they can expose their storage offerings to Kubernetes users.

Finally, from the end-user perspective, there's not much to think about. The Kubernetes distribution or service we're using most likely has at least one CSI driver installed, or, if none of those matches our needs, we can install additional ones ourselves.

What matters, for you as a Kubernetes user, is that there are storage classes. Those are the types of storage that your cluster can use. One CSI driver results in one or more storage classes.

Let's see which storage classes we have right now.

```sh
kubectl get storageclasses
```

The output is as follows.

```
NAME                  PROVISIONER        RECLAIMPOLICY VOLUMEBINDINGMODE    ALLOW... AGE
azurefile             file.csi.azure.com Delete        Immediate            true     19m
azurefile-csi         file.csi.azure.com Delete        Immediate            true     19m
azurefile-csi-premium file.csi.azure.com Delete        Immediate            true     19m
azurefile-premium     file.csi.azure.com Delete        Immediate            true     19m
default (default)     disk.csi.azure.com Delete        WaitForFirstConsumer true     19m
managed               disk.csi.azure.com Delete        WaitForFirstConsumer true     19m
managed-csi           disk.csi.azure.com Delete        WaitForFirstConsumer true     19m
managed-csi-premium   disk.csi.azure.com Delete        WaitForFirstConsumer true     19m
managed-premium       disk.csi.azure.com Delete        WaitForFirstConsumer true     19m
```

> Outputs in this post are from Azure AKS. Yours might differ depending on the hyperscaler choice you made during the setup.

If we take a look at the `PROVISIONER` column, we can see that all the classes available in my case are coming from the `file.csi.azure.com` and `disk.csi.azure.com` provisioners. Those are the CSI plugins AKS provides out of the box. We'll see them in action soon. For now, what matters, is that we can choose any of those when creating a persistent volume or, to be more precise, when we claim a volume.

We can also observe that the volume binding mode (`VOLUMEBINDINGMODE`) is, in this case, set to `Immediate` for some and `WaitForFirstConsumer` for other classes. We'll see what that means soon.

Let's see one of those Storage Classes in action.

## Kubernetes Persistent Volumes and Persistent Volume Claims

Let's take a look at an example of a Persistent Volume Claim.

```sh
cat volume/persistent-volume-claim.yaml
```

The output is as follows.

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: silly-demo
  labels:
    app.kubernetes.io/name: silly-demo
spec:
  storageClassName: managed
  resources:
    requests:
      storage: 1Gi
  accessModes:
  - ReadWriteOnce
```

That definition claims a persistent volume (`PersistentVolumeClaim`) which, in this scenario, means that it will instruct Kubernetes to request storage using the CSI Driver.

Since, as we saw earlier, there can be multiple Storage Classes, we are requesting a specific one called `managed`. Alternatively, we can remove the `storageClassName` and use whichever is the default Storage Class instead. We'll explore that option later.

Finally, we can set some additional information. In this case, we are requesting `1Gi` of `storage` and we are also specifying that we would like that storage to have both read and write access (`ReadWriteOnce`).

Let's apply that manifest,...

```sh
kubectl --namespace a-team apply \
    --filename volume/persistent-volume-claim.yaml
```

...and take a look at the `persistentvolumeclaims` in the `a-team` Namespace.

```sh
kubectl --namespace a-team get persistentvolumeclaims
```

The output is as follows.

```
NAME       STATUS  VOLUME CAPACITY ACCESS MODES STORAGECLASS VOLUMEATTRIBUTESCLASS AGE
silly-demo Pending                              managed      <unset>               7s
```

The important note is that the status of the claim we just made is `Pending`. The class we just used has the volume binding mode set to *WaitForFirstConsumer*. That means that even though we claimed a storage, we are not consuming it just yet so it is in the pending state. As a result, the claim did not yet create an actual volume. It will do that only when we attach it to a Pod.

We can confirm that's truly the case by listing all `persistentvolumes`.

```sh
kubectl get persistentvolumes
```

The output is as follows.

```
No resources found
```

There is nothing, at least in my case. If you chose a Storage Class with the volume binding mode set to *Immediate*, you should see the volume being created even if there is no Pod consuming it.

Let's change that. Let's update our Deployment to use the claim we just applied instead of the *emptyDir* volume we are using right now.

Here's the updated manifest.

```sh
cat volume/persistent-volume.yaml
```

The output is as follows (truncated for brevity).

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: silly-demo
  ...
spec:
  ...
  template:
    ...
    spec:
      containers:
        - image: ghcr.io/vfarcic/silly-demo:1.4.235-alpine
          name: silly-demo
          ...
          volumeMounts:
            - mountPath: /cache
              name: silly-cache
      volumes:
        - name: silly-cache
          persistentVolumeClaim:
            claimName: silly-demo
```

The only change is in the `volumes` section. Instead of simply specifying *emptyDir*, this time we are instructing it to use a `persistentVolumeClaim` instead. More specifically, we are instructing the Deployment to attach a volume with the `claimName` `silly-demo`. That's the claim we created earlier.

Everything else is exactly the same as before. We did not even have to change the `volumeMounts` section.

Let's apply that Deployment,...

```sh
kubectl --namespace a-team apply \
    --filename volume/persistent-volume.yaml
```

...and take another look at the `persistentvolumes`.

```sh
kubectl get persistentvolumes
```

The output is as follows.

```
NAME    CAPACITY ACCESS... RECLAIM... STATUS CLAIM      STORAGECLASS VOLUME... REASON AGE
pvc-... 1Gi      RWO       Delete     Bound  silly-demo managed      <unset>          10s
```

We can see that, a volume requested through the claim was now created. The Storage Class was waiting for the consumer and now that there is a Pod that claims it, there is no need to wait any more.

Let's see whether, this time, the storage is indeed persistent and not ephemeral like with `emptyDir`. We'll confirm that through the same commands as those we executed earlier.

We'll send a `POST` request to the application which should store the data to the volume.

```sh
curl -XPOST \
    "http://silly-demo.$INGRESS_HOST.nip.io/video?id=1&title=something"
```

We'll send a second `POST` request for good taste.

```sh
curl -XPOST \
    "http://silly-demo.$INGRESS_HOST.nip.io/video?id=2&title=else"
```

Finally, we'll send a *GET* request to the application which should read and return the data back to us.

```sh
curl "http://silly-demo.$INGRESS_HOST.nip.io/videos" | jq .
```

The output is as follows.

```json
[
  {
    "id": "1",
    "title": "something"
  },
  {
    "id": "2",
    "title": "else"
  }
]
```

Just as before, we'll also `exec` into the container and list (`ls`) all the files in the `/cache` directory.

```sh
kubectl --namespace a-team exec service/silly-demo \
    --stdin --tty -- ls /cache/
```

The output is as follows.

```
lost+found videos.yaml
```

We can see that there is a file `videos.yaml`.

Everything we did and observed so far is the same as when we worked with ephemeral volumes. What comes next should be different.

We'll delete the Pod,...

```sh
kubectl --namespace a-team delete pod \
    --selector app.kubernetes.io/name=silly-demo
```

...and the ReplicaSet should create a new one.

When we did that with ephemeral volumes, everything we stored in them was lost. This time, however, we are using a Persistent Volume and, as a result, the new Pod that was just created should have attached the same storage as the one that was used by the old Pod. All the data should be there as if nothing happened.

Let's confirm that by going into the container in the new Pod (`exec`) and listing all the files in the `/cache` directory.

```sh
kubectl --namespace a-team exec service/silly-demo \
    --stdin --tty -- ls /cache/
```

The output is as follows.

```
lost+found videos.yaml
```

We can see that the file `videos.yaml` is still there and you should trust me when I say that the content of that file is the same. Data was persisted on an external storage and the new Pod automatically attached to it. The best news is that we did not have to do anything "special" for all that to happen. It was easy, but we can make it even easier.

But, before we do jump into the easy mode, let's first delete the Persistent Volume,...

```sh
kubectl --namespace a-team delete \
    --filename volume/persistent-volume.yaml
```

...and the Claim,...

```sh
kubectl --namespace a-team delete \
    --filename volume/persistent-volume-claim.yaml
```

...and take another look at the `storageclasses`.

```sh
kubectl get storageclasses
```

The output is as follows.

```
NAME                  PROVISIONER        RECLAIMPOLICY VOLUMEBINDINGMODE    ALLOW... AGE
azurefile             file.csi.azure.com Delete        Immediate            true     39m
azurefile-csi         file.csi.azure.com Delete        Immediate            true     39m
azurefile-csi-premium file.csi.azure.com Delete        Immediate            true     39m
azurefile-premium     file.csi.azure.com Delete        Immediate            true     39m
default (default)     disk.csi.azure.com Delete        WaitForFirstConsumer true     39m
managed               disk.csi.azure.com Delete        WaitForFirstConsumer true     39m
managed-csi           disk.csi.azure.com Delete        WaitForFirstConsumer true     39m
managed-csi-premium   disk.csi.azure.com Delete        WaitForFirstConsumer true     39m
managed-premium       disk.csi.azure.com Delete        WaitForFirstConsumer true     39m
```

You'll notice that one of the classes has `(default)` next to the name. Actually, in your case that might not the true if you chose to use AWS.

If you do see *(default)*, it means that, as you can guess, one of the classes is used by default meaning that we do not have to specify the *storageClassName* in our claims.

Here's an example.

```sh
cat volume/persistent-volume-claim-default.yaml
```

The output is as follows (truncated for brevity).

```yaml
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: silly-demo
  labels:
    app.kubernetes.io/name: silly-demo
spec:
  resources:
    requests:
      storage: 1Gi
  accessModes:
    - ReadWriteOnce
...
```

Everything is exactly the same as before, except that the `PersistentVolumeClaim` does not have the *storageClassName* set. That's very handy since it means that users do not need to think which storag class to use, unless they have "special" requirements.

> Unfortunately, none of the Storage Classes in AWS are set to be default. I don't understand why AWS choose not to set a default Storage Class, but it is what it is. You will have to modify one of them using `kubectl edit` or skip the rest of the commands in this section.

Let's apply the Claim and the Deployment,...

```sh
kubectl --namespace a-team apply \
    --filename volume/persistent-volume-claim-default.yaml
```

...and retrieve `persistentvolumeclaims` and `persistentvolumes`.

```sh
kubectl --namespace a-team \
    get persistentvolumeclaims,persistentvolumes
```

The output is as follows.

```
NAME                             STATUS VOLUME          CAPACITY ACCESS MODES STORAGECLASS VOLUMEATTRIBUTESCLASS AGE
persistentvolumeclaim/silly-demo Bound  pvc-a717bafe... 1Gi      RWO          default      <unset>               8s

NAME                             CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM             STORAGECLASS VOLUMEATTRIBUTESCLASS REASON AGE
persistentvolume/pvc-a717bafe... 1Gi      RWO          Delete         Bound  a-team/silly-demo default      <unset>                      3s
```

We can see that the end result is, more or less, the same as if we specified the desired Storage Class, except that now we are using whichever is the default one.

Before we move to the next section, let me stress that volumes work differently depending on the type of the root resource. If we are using Kubernetes **Deployments**, all the **Pods are sharing the same volume**. On the other hand, if we choose to use **Stateful Sets**, each **Pod gets a separate volume**. I won't go into more details since I already explained that in [Mastering Kubernetes: Workloads APIs (Deployment, StatefulSet, ReplicaSet, Pod, etc.)](https://youtu.be/U6weXlzQxoY).

With that out of the way, let's move on to Config Maps.

## Kubernetes Config Maps

**Config Maps** are volumes as well. They are persistent, but, unlike other persistent volumes, they are **read-only**. Applications in containers where ConfigMaps are attached can read but cannot write data. That's not the only difference though. ConfigMaps are structured data that can be transformed into environment variables and a few other formats. As such, ConfigMaps are perfect for cases when we want to inject some additional configuration.

Here's an example.

```sh
cat volume/config-map.yaml
```

The output is as follows (truncated for brevity).

```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: silly-demo
  labels:
    app.kubernetes.io/name: silly-demo
data:
  videos.yaml: |
    - id: "1"
      title: something
    - id: "2"
      title: else
    - id: "3"
      title: something new
  message: Hello, DevOps Toolkit!
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: silly-demo
  ...
spec:
  ...
  template:
    ...
    spec:
      containers:
        - image: ghcr.io/vfarcic/silly-demo:1.4.235-alpine
          ...
          envFrom:
            - configMapRef:
                name: silly-demo
          env:
            - name: MESSAGE
              valueFrom:
                configMapKeyRef:
                  name: silly-demo
                  key: message
          volumeMounts:
            - name: cache
              mountPath: /cache
      volumes:
        - name: cache
          configMap:
            name: silly-demo
```

Over there, we are defining a `ConfigMap` `silly-demo` with two entries in `data`. Each of those entries much contain a key that is used to identify it. The first one is `videos.yaml` with a text value that happens to be YAML. The second entry is identified with the key `message` and also contains some text. As a matter of fact, each value in a ConfigMap is always text or string.

Further on, inside the `Deployment`, we are defining a volume (`volumes`) just as we did before except that, this time, the volume is a `configMap`. Similarly, just as before, we can mount it as well (`volumeMounts`). The major difference is that we can not only mount it as any other volume, but we can also extract data from a ConfigMap.

In this case, we are specifying that we would like to convert all ConfigMap values info environment variables (`envFrom`).

We can also transform a key from a ConfigMap into an environment variable with a different name. That's what we are doing in the `env` section where we are specifying that we would like to have an environment variable `MESSAGE` with the value coming from the ConfigMap `silly-demo` and the key `message`.

As a result of that setup, we will get files from the keys in the ConfigMap stored as files, all ConfigMap keys converted into environment variables, and the value of the *message* key converted into the environment variable *MESSAGE*.

Let's see whether that's what will really happen by applying those manifests,...

```sh
kubectl --namespace a-team apply \
    --filename volume/config-map.yaml
```

...and retrieving `configmaps`.

```sh
kubectl --namespace a-team get configmaps
```

The output is as follows.

```
NAME             DATA AGE
kube-root-ca.crt 1    37m
silly-demo       2    8s
```

We can see that the ConfiMap was indeed created. That was to be expected.

Let's see which environment variables are available inside the container by executing `env` inside it.

```sh
kubectl --namespace a-team exec service/silly-demo \
    --stdin --tty -- env
```

The output is as follows (truncated for brevity).

```
...
message=Hello, DevOps Toolkit!
videos.yaml=- id: "1"
  title: something
- id: "2"
  title: else
- id: "3"
  title: something new

MESSAGE=Hello, DevOps Toolkit!
...
```

We can see that environment variables `message` and `videos.yaml` are there. Those were created because we specified with *envFrom* that we would like to have all entries in the ConfigMap converted into environment variables.

There is also `MESSAGE` (in capital letters) variable because we explicitly stated that we would like it to map to the *message* key in the ConfigMap.

That's not all though. We should also have some files in the `/cache` directory. Let's check it out by listing all the files in that folder inside the container.

```sh
kubectl --namespace a-team exec service/silly-demo \
    --stdin --tty -- ls /cache/
```

The output is as follows.

```
message videos.yaml
```

We can see that both `message` and `videos.yaml` files are there. That happened because we also chose to mount the ConfigMap just as we would mount any other type of volume except that, this time, it converted each key in the map into a separate file.

To be sure that's what really happened, let's output the content of the `videos.yaml` file.

```sh
kubectl --namespace a-team exec service/silly-demo \
    --stdin --tty -- cat /cache/videos.yaml \
    | yq .
```

The output is as follows.

```yaml
- id: "1"
  title: something
- id: "2"
  title: else
- id: "3"
  title: something new
```

We can see that the content of that file is exactly the same as the value of that key in the map.

There is one more crucial type of volumes we should explore. But, before we do that, we'll first remove the ConfigMap we just created.

```sh
kubectl --namespace a-team delete \
    --filename volume/config-map.yaml
```

## Kubernetes Secrets

Kubernetes Secrets are similar to ConfigMaps. Both can be mounted as volumes or transformed into environment variables. There are two major differences though.

First of all, **Secrets** are **encrypted at rest**. All Kubernetes resources are stored in etcd in plain-text, except Secrets. Secrets are encrypted at rest meaning that they are stored encrypted in etcd making them a better choice for confidential information. To be more precise, we can choose to have secrets encrypted, which is the case in most managed Kubernetes offerings, but, if you're running Kubernetes yourself, that needs to be specifically enabled.

The second difference is that values in manifests are **base64 encoded**. Here's an example.

```sh
cat volume/secret.yaml
```

The output is as follows (truncated for brevity).

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: silly-demo
  labels:
    app.kubernetes.io/name: silly-demo
data:
  videos.yaml: LSBpZDogIjEiCiAgdGl0bGU6IHNvbWV0aGluZwotIGlkOiAiMiIKICB0aXRsZTogZWxzZQotIGlkOiAiMyIKICB0aXRsZTogc29tZXRoaW5nIG5ldwo=
  message: SGVsbG8sIERldk9wcyBUb29sa2l0IQo=
  silly: ZGVtbwo=
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: silly-demo
  ...
spec:
  ...
  template:
    ...
    spec:
      containers:
        - image: ghcr.io/vfarcic/silly-demo:1.4.235-alpine
          ...
          volumeMounts:
            - name: cache
              mountPath: /cache
      volumes:
        - name: cache
          secret:
            secretName: silly-demo
```

The definition of the `Secret` is almost identical to the ConfigMap we explored earlier. The only tangible difference is that the values in `data` keys (`videos.yaml`, `message`, `silly`) are base64 encoded. Bear in mind that something being encoded is not the same as being encrypted. Encoded data is easier to manage, and to encrypt, but not any safer by itself. We'll see that in a moment.

Inside the `Deployment` we can use Secrets in the same way as ConfigMaps. In this particular case, we are specifying the `silly-demo` `secret` volume and mounting it (`volumeMounts`) just as any other volume.

Now, let's get back to encoding...

If you wondered how we got the *message* value to be *SGVsbG8sIERldk9wcyBUb29sa2l0IQo=*, the answer is simple. That is base 64 encoded version of the text *Hello, DevOps Toolkit!*. Here's the proof.

We'll `echo` `'Hello, DevOps Toolkit!'` and pipe the output to `base64` command available in any Shell.

```sh
echo 'Hello, DevOps Toolkit!' | base64
```

The output is as follows.

```
SGVsbG8sIERldk9wcyBUb29sa2l0IQo=
```

We can see that the output is exactly the same as the value we specified in the Secret manifest.

Let's see it in action by applying the new manifest,...

```sh
kubectl --namespace a-team apply --filename volume/secret.yaml
```

...and retrieving the Secrets.

```sh
kubectl --namespace a-team get secrets
```

The output is as follows.

```
NAME       TYPE   DATA AGE
silly-demo Opaque 3    5s
```

The secret is there and the Deployment that uses it should be running as well. As a result, if we `exec` into the container it created and list all the files in the `/cache` directory,...

```sh
kubectl --namespace a-team exec service/silly-demo \
    --stdin --tty -- ls /cache/
```

The output is as follows.

```
message silly videos.yaml
```

...we can see that a file was created for each key in the Secret.

We can confirm that further by outputting one of those files.

```sh
kubectl --namespace a-team exec service/silly-demo \
    --stdin --tty -- cat /cache/silly
```

The output is as follows.

```
demo
```

All in all, if we ignore that the values in Secret manifests are base64 encoded and that they are encrypted at rest, Secrets are defined and used in the same way as ConfigMaps.

There are many other types and variations of volumes but those we explored today should be more than enough to start navigating through the maze.


## Destroy

```sh
chmod +x destroy/volume.nu

./destroy/volume.nu

exit
```


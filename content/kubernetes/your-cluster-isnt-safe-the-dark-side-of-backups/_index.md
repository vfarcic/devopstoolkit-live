
+++
title = 'Your Cluster Is Not Safe: The Dark Side of Backups'
date = 2024-12-01T16:00:00+00:00
draft = false
+++

We all want to feel safe. That's why we create backups. We want to know that our systems will survive no mather what happens.

The feeling of safety is very very important, and I need to appologise in advance for what I'm about to say.

You are **NOT safe**. If the disaster happens, you might not be able to survive it.

Here's why.

**Backups alone are not enough!**; especially if its only one type of a backup.

When the day of reckoning comes, **you will NOT be saved!**

Let me prove that to you.

<!--more-->

{{< youtube lSRdVzXqFXE >}}

## Setup

```sh
rm -rf velero-demo
```

> Watch the [GitHub CLI (gh) - How to manage repositories more efficiently](https://youtu.be/BII6ZY2Rnlc) video if you are not familiar with GitHub CLI.

```sh
gh repo fork vfarcic/velero-demo --clone --remote

cd velero-demo

gh repo set-default
```

> Watch [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell
```

> Please watch [The Future of Shells with Nushell! Shell + Data + Programming Language](https://youtu.be/zoX_S6d-XU4) if you are not familiar with Nushell. Alternatively, you can inspect the `setup.nu` script and transform the instructions in it to Bash or ZShell if you prefer not to use that Nushell script.

```sh
chmod +x setup-cnpg-crossplane.nu

./setup-cnpg-crossplane.nu

source .env
```

## Before The Disaster

I already have an application running in my cluster.

```sh
kubectl --namespace a-team get all,ingresses
```

The output is as follows.

```
NAME                                                  READY   STATUS    RESTARTS   AGE
pod/silly-demo-1                                      1/1     Running   0          4m24s
pod/silly-demo-595c89b567-gj4fj                       1/1     Running   0          5m19s
pod/silly-demo-videos-atlas-dev-db-678f49ffb9-r658p   1/1     Running   0          3m29s

NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/silly-demo      ClusterIP   10.100.4.103     <none>        8080/TCP   5m20s
service/silly-demo-r    ClusterIP   10.100.49.95     <none>        5432/TCP   5m19s
service/silly-demo-ro   ClusterIP   10.100.141.126   <none>        5432/TCP   5m19s
service/silly-demo-rw   ClusterIP   10.100.142.203   <none>        5432/TCP   5m19s

NAME                                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/silly-demo                       1/1     1            1           5m19s
deployment.apps/silly-demo-videos-atlas-dev-db   1/1     1            1           5m19s

NAME                                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/silly-demo-595c89b567                       1         1         1       5m19s
replicaset.apps/silly-demo-videos-atlas-dev-db-678f49ffb9   1         1         1       5m19s

NAME                                   CLASS     HOSTS                             ADDRESS                                                                   PORTS   AGE
ingress.networking.k8s.io/silly-demo   traefik   silly-demo.52.86.219.243.nip.io   a00117ef59e2f48aa8dacfd39899e1fd-1803699707.us-east-1.elb.amazonaws.com   80      5m20s
```

It's a stateless app that uses PostgreSQL to store data. There is a `deployment` of the app (`silly-demo`), and a Deployment of Atlas (`silly-demo-videos-atlas-dev-db`) that manages PostgreSQL schemas.

There is a `persistentvolume`...

```sh
kubectl --namespace a-team get persistentvolumes
```

The output is as follows.

```
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
persistentvolume/pvc-ec0cc2c3-d41a-4fa8-87e5-48b2f3ffbf07   1Gi        RWO            Delete           Bound    a-team/silly-demo-1   gp2            <unset>                          5m52s
```

...that contains data from the database.

There are also custom resources `clusters`, and `atlasschemas`.

```sh
kubectl --namespace a-team get clusters,atlasschemas
```

The output is as follows.

```
NAME                                    AGE   INSTANCES   READY   STATUS                     PRIMARY
cluster.postgresql.cnpg.io/silly-demo   5m87s                     Cluster in healthy state   silly-demo-1

NAME                                          READY   REASON
atlasschema.db.atlasgo.io/silly-demo-videos   False   VerifyingFirstRun
```

Those are not special either. `Cluster` resources come from CNPG that is designed to run PostgreSQL in a cloud native way. `AtlasSchema` is a custom resource from the Atlas Operator that, as we mentioned earlier, is in charge of managing database schemas.

> If you are not familiar with CNPG and the Atlas Operator, you might want to check out [Should We Run Databases In Kubernetes? CloudNativePG (CNPG) PostgreSQL](https://youtu.be/Ny9RxM6H6Hg) and [Kubernetes? Database Schema? Schema Management with Atlas Operator](https://youtu.be/1iZoEFzlvhM).

There is nothing truly special with those resources. They are still a very very simple example. It's so simple that I won't even dive into the details.

So far, I don't see a problem. Velero can handle those without any issues. "**Right?**" Well... "**Wrong!**", but we'll get to that.

Now, to make it just marginally more complex situation, I have a Crossplane claim running in my cluster. Here it goes...

```sh
kubectl --namespace infra get sqlclaims
```

The output is as follows.

```
NAME    SYNCED   READY   CONNECTION-SECRET   AGE
my-db   True     False                       3m47s
```

That claim represents a database server, and consists of a bunch of other resources.

```sh
kubectl get managed
```

The output is as follows.

```
NAME                             SYNCED   READY   EXTERNAL-NAME                       AGE
route.ec2.aws.upbound.io/my-db   True     True    r-rtb-00161d9b28292789d1080289494   4m3s

NAME                                       SYNCED   READY   EXTERNAL-NAME           AGE
internetgateway.ec2.aws.upbound.io/my-db   True     True    igw-06f07944db89c5883   4m7s

NAME                                                 SYNCED   READY   EXTERNAL-NAME                AGE
mainroutetableassociation.ec2.aws.upbound.io/my-db   True     True    rtbassoc-0ed65b23c7197c10c   4m7s

NAME                                                SYNCED   READY   EXTERNAL-NAME                AGE
routetableassociation.ec2.aws.upbound.io/my-db-1a   True     True    rtbassoc-055b508b098ecdd4c   4m8s
routetableassociation.ec2.aws.upbound.io/my-db-1b   True     True    rtbassoc-07422d44457f01f73   4m8s
routetableassociation.ec2.aws.upbound.io/my-db-1c   True     True    rtbassoc-0d90c51759baa34fb   4m8s

NAME                                  SYNCED   READY   EXTERNAL-NAME           AGE
routetable.ec2.aws.upbound.io/my-db   True     True    rtb-00161d9b28292789d   4m8s

NAME                                         SYNCED   READY   EXTERNAL-NAME       AGE
securitygrouprule.ec2.aws.upbound.io/my-db   True     True    sgrule-1364994703   4m9s

NAME                                     SYNCED   READY   EXTERNAL-NAME          AGE
securitygroup.ec2.aws.upbound.io/my-db   True     True    sg-0839bca02e3629a18   4m9s

NAME                                SYNCED   READY   EXTERNAL-NAME              AGE
subnet.ec2.aws.upbound.io/my-db-a   True     True    subnet-00a7554e7a87104da   4m10s
subnet.ec2.aws.upbound.io/my-db-b   True     True    subnet-063055c491be14340   4m10s
subnet.ec2.aws.upbound.io/my-db-c   True     True    subnet-0c8f8916b9b1fb4ca   4m10s

NAME                           SYNCED   READY   EXTERNAL-NAME           AGE
vpc.ec2.aws.upbound.io/my-db   True     True    vpc-09f9e3df7626f19e0   4m14s

NAME                                           KIND     PROVIDERCONFIG   SYNCED   READY   AGE
object.kubernetes.crossplane.io/my-db-secret   Secret   my-db-sql        False            4m14s

NAME                                                READY   SYNCED   AGE
database.postgresql.sql.crossplane.io/my-db-db-01           False    4m15s
database.postgresql.sql.crossplane.io/my-db-db-02           False    4m15s

NAME                                SYNCED   READY   EXTERNAL-NAME   AGE
instance.rds.aws.upbound.io/my-db   False    False                   4m16s

NAME                                   SYNCED   READY   EXTERNAL-NAME   AGE
subnetgroup.rds.aws.upbound.io/my-db   False    True    my-db           4m18s
```

> That output is from the managed resources from the AWS provider. Depending on your setup choice, the output in your case might differ.

Still, there is nothing alarming. There is still no reason to think that we are not safe with cluster backups created with, let's say, Velero. "Right?" Well... "Wrong!", but we'll get to that.

## Disaster Recovery with Kuberentes Backups (Velero)

We are very fortunate that Joe is working with us. He has a special skill. He is clairvoyant. He can see the future and he is always right. He's the one who predicted the fall of block chain, and now he's predisting that our cluster is about to get busted. That's why Joe is on our payrol. He can tell us when a cluster is going to go down, most of the time because he's messing it up. Let's make a backup while we still can.

```sh
velero backup create pre-disaster
```

> I won't go into details how Velero works. I already explained it in [Master Kubernetes Backups with Velero: Step-by-Step Guide](https://youtu.be/OzoC-wGfBnw). That was the one with reinbows and unicorns, while this one is mostly doom and gloom.

Now, let's say that a new cluster was auto-magically created and that we would like to restore the backup in hopes that everything will continue working on that new cluster as if nothing happened to the old one. If we accomplish that, we can avoid firing Joe for messing it up again.

Let's do it. Let's `restore` the `pre-disaster` backup or, to be more precise, whatever we have in that backup associated with the `a-team` Namespace.

```sh
velero --kubeconfig kubeconfig-dot2.yaml restore create \
    --from-backup pre-disaster --include-namespaces a-team
```

Everything should be working? Right?

If we take a look at all the resources in that Namespace,...

```sh
kubectl --kubeconfig kubeconfig-dot2.yaml --namespace a-team \
    get all
```

The output is as follows.

```
NAME                                                  READY   STATUS    RESTARTS   AGE
pod/silly-demo-595c89b567-gj4fj                       1/1     Running   0          24s
pod/silly-demo-videos-atlas-dev-db-678f49ffb9-r658p   1/1     Running   0          24s

NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/silly-demo      ClusterIP   10.100.234.248   <none>        8080/TCP   25s
service/silly-demo-r    ClusterIP   10.100.8.142     <none>        5432/TCP   25s
service/silly-demo-ro   ClusterIP   10.100.134.203   <none>        5432/TCP   25s
service/silly-demo-rw   ClusterIP   10.100.234.202   <none>        5432/TCP   25s

NAME                                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/silly-demo                       1/1     1            1           25s
deployment.apps/silly-demo-videos-atlas-dev-db   1/1     1            1           25s

NAME                                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/silly-demo-595c89b567                       1         1         1       25s
replicaset.apps/silly-demo-videos-atlas-dev-db-678f49ffb9   1         1         1       25s
```

Everything seem to be just peachy.

Did it restore persistent volumes with data as well?

```sh
kubectl --kubeconfig kubeconfig-dot2.yaml --namespace a-team \
    get persistentvolumes
```

The output is as follows (truncated for brevity).

```
NAME       CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM               STORAGECLASS VOLUMEATTRIBUTESCLASS REASON AGE
pvc-ec0... 1Gi      RWO          Delete         Bound  a-team/silly-demo-1 gp2          <unset>                      65s
```

It indeed did. Well done Velero. You're amazing.

How about custom resources like CNPG clusters and Atlas schemas?

```sh
kubectl --kubeconfig kubeconfig-dot2.yaml --namespace a-team \
    get clusters,atlasschemas
```

The output is as follows.

```
NAME                                  AGE INSTANCES READY STATUS                                    PRIMARY
cluster.postgresql.cnpg.io/silly-demo 87s                 Unable to create required cluster objects 

NAME                                          READY   REASON
atlasschema.db.atlasgo.io/silly-demo-videos   False   VerifyingFirstRun
```

F**k! It's not working.

Let's see what's wrong.

```sh
kubectl --kubeconfig kubeconfig-dot2.yaml --namespace a-team \
    describe cluster silly-demo
```

The output is as follows (truncated for brevity).

```
...
Status:
  Conditions:
    Last Transition Time:  2024-11-08T23:22:55Z
    Message:               Cluster Is Not Ready
    Reason:                ClusterIsNotReady
    Status:                False
    Type:                  Ready
  Image:                   ghcr.io/cloudnative-pg/postgresql:17.0
  Latest Generated Node:   1
  Phase:                   Unable to create required cluster objects
  Phase Reason:            refusing to reconcile service: silly-demo-r, not owned by the cluster
  Target Primary:          silly-demo-1
Events:                    <none>
```

The problem is in Kubernetes itself. We often have resources owned by some other resources. A Deployment creates a ReplicaSet which creates Pods. As a result, a Pod is owned by a ReplicaSet which is owned by a Deployment. Now, in some cases that is and in others that isn't a problem when backups are concerned. In this case, we do have a problem.

When we restored the backup, among other resources we restored `silly-demo-r` Service as well as the CNPG *Cluster* resource. As a result, that *Cluster* resource did not create that service but saw that it already exists but is not owned by it. The end result is that it is failing.

One solution could be to apply filters to the backup restore operation. We could have, for example, said that we do NOT want to restore Services, but that would not work either because there are other Services in that Namespace that are not created by other resources and, hence, not owned by them. We could have had even more elaborated filters that would exclude Services with specific labels, but, in those cases, we would start facing very complex setups with potentially infinite number of filters.

On top of all that, the resources we restored with Velero are likely not up-to-date. They are not resources in the state they were before the crash but resources in the state at the time we created the last backup.

Even if we did manage to solve the filtering conundrum and we would start making backups much more frequently, we are still faced with a potential issue with data.

Do we trust Velero alone to backup database data? I think we shouldn't. Databases have elaborated tools to backup data that are much safer than more generic backups like Velero. If we rely on Velero for data, there is a chance that backups of database data will be corrupted or incomplete. On top of that, data restoration needs to be coordinated with initialization of database servers.

Now, most of those problems can be solved with Velero, but are likely already solved in database operators. CNPG spec, for example, already has entries that allow us to specify data required to backup and restore its data safely and without the need for any "special" operations.

The problem, however, is that we did not specify that we want CNPG to create and restore backups, so we cannot leverage it in the current situation since the cluster where PostgreSQL was running is now dead or, to be more precise, we are pretending it's dead.

All in all, Velero poses problems when restoring Kubernetes resources as well as when working with database data. If that would be the only tool we have at our disposal, we might try to solve those, but that's not the case. We have alternatives and, for those two problems, they are GitOps and database-specific backups. We won't be able to demonstrate the latter since we did not create CNPG backup while we still could, but we certainly can switch to GitOps. But, before we do that, let me give you an advanced warning. GitOps might not solve our issues either.

Let's start over and delete the Namespace with the resources we restored.

```sh
kubectl --kubeconfig kubeconfig-dot2.yaml delete namespace a-team
```

## Disaster Recovery with GitOps (Argo CD)

Assuming that databases are creating their own data backups which are restored automatically whenever we spin them up, we are left with the issue of backups for Kubernetes resources not being up-to-date and causing potential problems like those with ownership we saw earlier.

Let's see whether we can solve that one with GitOps.

Here's an Argo CD Application definition.

```sh
cat apps/silly-demo.yaml
```

The output is as follows.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: silly-demo
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/vfarcic/velero-demo
    targetRevision: HEAD
    path: app/overlays/full
  destination:
    server: https://kubernetes.default.svc
    namespace: a-team
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
      allowEmpty: true
```

Typically, we would be using App of Apps model where a single Argo CD Application would synchronize everything in a cluster. Today, however, I want to keep it simple so we have an `Application` that will synchronize everything in the `app/overlays/full` directory of the `velero-demo` repo.

Instead of restoring a backup, we can just apply that resource and let Argo CD synchronize whatever is in that repo into the new cluster.

So, let's apply it, and...

```sh
kubectl --kubeconfig kubeconfig-dot2.yaml apply \
    --filename apps/silly-demo.yaml
```

...wait for a while for Argo CD to do the work.

A while later, we can retrieve `all` Kubernetes core resources, and persistent volumes, and CNPG clusters, and Atlas schemas.

```sh
kubectl --kubeconfig kubeconfig-dot2.yaml --namespace a-team \
    get all,ingresses,persistentvolumes,clusters,atlasschemas
```

The output is as follows (truncated for brevity).

```
NAME                                   READY STATUS  RESTARTS AGE
pod/silly-demo-1                       1/1   Running 0        2m18s
pod/silly-demo-595c89b567-bxlv5        1/1   Running 0        2m49s
pod/silly-demo-videos-atlas-dev-db-... 1/1   Running 0        117s

NAME                  TYPE      CLUSTER-IP     EXTERNAL-IP PORT(S)  AGE
service/silly-demo    ClusterIP 10.100.71.12   <none>      8080/TCP 2m50s
service/silly-demo-r  ClusterIP 10.100.248.163 <none>      5432/TCP 2m50s
service/silly-demo-ro ClusterIP 10.100.62.94   <none>      5432/TCP 2m50s
service/silly-demo-rw ClusterIP 10.100.90.155  <none>      5432/TCP 2m50s

NAME                                           READY UP-TO-DATE AVAILABLE AGE
deployment.apps/silly-demo                     1/1   1          1         2m50s
deployment.apps/silly-demo-videos-atlas-dev-db 1/1   1          1         2m50s

NAME                                               DESIRED CURRENT READY AGE
replicaset.apps/silly-demo-595c89b567              1       1       1     2m50s
replicaset.apps/silly-demo-videos-atlas-dev-db-... 1       1       1     2m50s

NAME                                 CLASS   HOSTS                           ADDRESS    PORTS AGE
ingress.networking.k8s.io/silly-demo traefik silly-demo.52.86.219.243.nip.io aa81601... 80    2m51s

NAME                        CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM               STORAGECLASS VOLUMEATTRIBUTESCLASS REASON AGE
persistentvolume/pvc-11f... 1Gi      RWO          Delete         Bound  a-team/silly-demo-1 gp2          <unset>                      2m48s

NAME                                  AGE   INSTANCES READY STATUS                   PRIMARY
cluster.postgresql.cnpg.io/silly-demo 2m51s 1         1     Cluster in healthy state silly-demo-1

NAME                                        READY REASON
atlasschema.db.atlasgo.io/silly-demo-videos True  Applied
```

This time, everything seem to be working correctly, with the assumption that we did add backups to CNPG using its own backup and restore machanism.

However, "everything is working" is only an illusion. Everything is working in such a simple setup when we're looking only at a single application. If we instructed Argo CD to synchronize everything that should be running in that cluster at once, it would almost certainly fail. Argo CD assumes that we do things in certain order and that order is specified through Sync Waves. Without them, it would try to synchronize something that depends on something else, fail to do so, and stop the process resulting in only a fraction of the resources that should be running in that cluster.

> I won't go deeper into the issues with Argo CD synchronization since I already did that in [Argo CD Synchronization is BROKEN! It Should Switch to Eventual Consistency!](https://youtu.be/t1Fdse-F9Jw). Watch it if you haven't already.

Still, Argo CD synchronization issues can be solved with a lot of work, yet that is what we're trying to avoid in the first place. So, Velero can restore all resources if we spend the time defining all the filters, and Argo CD can do the same if we define an infinite number of sync waves. Both are likely not going to work on the new cluster in the first attempt but, if we practice a lot and we are persistent enough, we might get there with both.

Argo CD, however, has an advantage over backups. It's source of the desired state is most likely up-to-date. If we are practicing GitOps, we always commit changes to the desired state to Git. So, Git always has the latest desired state. Backups, on the other hand, makes periodic snapshots of the actual state. As a result, GitOps synchronization into a new cluster is more likely to result in that cluster being as close as it can be to the old one that was destroyed (assuming that data is backup up by whichever database we're using).

However, even if we ignore the issue with Argo CD sync waves and we do restore all the resources from Git, it will still not work.

Let me demonstrate that through Crossplane.

## Disaster Recovery of Mutated Resources

I already have a claim in my cluster which expanded into a bunch of managed resources. To be more precise, I have a Crossplane claim in the old cluster that we are pretending to be destroyed. Still, let's stop pretending for a second that it is not running and take a look at it.

```sh
crossplane beta trace sqlclaim my-db --namespace infra
```

The output is as follows (truncated for brevity).

```
NAME                                  SYNCED READY STATUS
SQLClaim/my-db (infra)                True   True  Available
└─ SQL/my-db-sprhz                    True   True Available
   ├─ InternetGateway/my-db           True   True  Available
   ├─ MainRouteTableAssociation/my-db True   True  Available
   ├─ RouteTableAssociation/my-db-1a  True   True  Available
   ├─ RouteTableAssociation/my-db-1b  True   True  Available
   ├─ RouteTableAssociation/my-db-1c  True   True  Available
   ├─ RouteTable/my-db                True   True  Available
   ├─ Route/my-db                     True   True  Available
   ├─ SecurityGroupRule/my-db         True   True  Available
   ├─ SecurityGroup/my-db             True   True  Available
   ├─ Subnet/my-db-a                  True   True  Available
   ├─ Subnet/my-db-b                  True   True  Available
   ├─ Subnet/my-db-c                  True   True  Available
   ├─ VPC/my-db                       True   True  Available
   ├─ ProviderConfig/my-db-sql        -      -     
   ├─ ProviderConfig/my-db-sql        -      -     
   ├─ Object/my-db-secret             False  -     Available
   ├─ Database/my-db-db-01            False  -     Available
   ├─ Database/my-db-db-02            False  -     Available
   ├─ ProviderConfig/my-db            -      -     
   ├─ SubnetGroup/my-db               False  True  Available
   └─ Instance/my-db                  False  True  Available
```

Everything seem to be working well... because it is working. If that cluster was destroyed, we could have restored those resources using Argo CD. Right? Wrong!

One thing we might easily overlook are mutations. Almost all Kubernetes resources mutate over time. Their specs are modified by the scheduler, their statuses are updated, and events are generated. Some of those mutations are not important and the resources will behave correctly even if we apply those same resources into a new cluster without those specific mutations. That's what we do with Argo CD, and many people do not understand that the desired state stored in Git is not the same as the actual state in a cluster.

Still, we can ignore many of those mutations, but not all.

Here's an example.

```sh
kubectl get managed --output yaml
```

The output is as follows.

```yaml
...
- apiVersion: ec2.aws.upbound.io/v1beta1
  kind: SecurityGroupRule
  metadata:
    annotations:
      crossplane.io/composition-resource-name: securityGroupRule
      crossplane.io/external-create-pending: "2024-11-20T12:47:45Z"
      crossplane.io/external-create-succeeded: "2024-11-20T12:47:45Z"
      crossplane.io/external-name: sgrule-1730703853
...
```

> Before we continue, let me stress that the example comes from Crossplane, but we can observe issues that originate from mutations in many others.

Let's, for example, take a look at the `external-name` annotation. That annotation was not defined as the desired state. We won't find it in Git. It was added by the controller. While mutations often do not matter, this one does, in some cases.

That specific resource manages a VPC in AWS. Since AWS autogenerates IDs of some (but not all) its resources, Crossplane mutated that resource by, among other things, injecting that label. Through it, Crossplane is able to establish the relation between that resource and the Route in AWS and do whatever needs to be done.

Here's the question.

What would happen if we instruct Argo CD to synchronize the claim from Git into the new cluster?

Since that annotation is added at runtime and is part of the desired state in Git, Argo CD would create that resource without it. Crossplane, on the other hand, would not be able to find that Route in AWS and create a new one. That's probably not what we wanted. The intention should be for Crossplane in the new cluster to continue managing that specific Route.

So, in that specific scenario, Argo CD is not an option, at least not right away. So, we are forced to go back to Velero. Since some of the resources are cluster scoped, while others are in a specific Namespace, we'll restore two parts of the backup.

We'll restore all the resources with the label `crossplane.io/claim-name=my-db`,...

```sh
velero --kubeconfig kubeconfig-dot2.yaml restore create \
    --from-backup pre-disaster \
    --selector "crossplane.io/claim-name=my-db"
```

...and all those inside the Namespace `infra`.

```sh
velero --kubeconfig kubeconfig-dot2.yaml restore create \
    --from-backup pre-disaster \
    --include-namespaces infra
```

Let's double check that worked by retrieving all Crossplane managed resources.

```sh
kubectl --kubeconfig kubeconfig-dot2.yaml get managed
```

The output is as follows.

```
NAME                           SYNCED READY EXTERNAL-NAME                   AGE
route.ec2.aws.upbound.io/my-db True   True  rtb-00161d9b28292789d_0.0.0.0/0 37s

NAME                                     SYNCED READY EXTERNAL-NAME         AGE
internetgateway.ec2.aws.upbound.io/my-db True   True  igw-06f07944db89c5883 41s

NAME                                               SYNCED READY EXTERNAL-NAME              AGE
mainroutetableassociation.ec2.aws.upbound.io/my-db True   True  rtbassoc-0ed65b23c7197c10c 41s

NAME                                              SYNCED READY EXTERNAL-NAME              AGE
routetableassociation.ec2.aws.upbound.io/my-db-1a True   True  rtbassoc-055b508b098ecdd4c 43s
routetableassociation.ec2.aws.upbound.io/my-db-1b True   True  rtbassoc-07422d44457f01f73 43s
routetableassociation.ec2.aws.upbound.io/my-db-1c True   True  rtbassoc-0d90c51759baa34fb 43s

NAME                                SYNCED READY EXTERNAL-NAME         AGE
routetable.ec2.aws.upbound.io/my-db True   True  rtb-00161d9b28292789d 43s

NAME                                       SYNCED READY EXTERNAL-NAME     AGE
securitygrouprule.ec2.aws.upbound.io/my-db True   True  sgrule-1364994703 43s

NAME                                   SYNCED READY EXTERNAL-NAME        AGE
securitygroup.ec2.aws.upbound.io/my-db True   True  sg-0839bca02e3629a18 43s

NAME                              SYNCED READY EXTERNAL-NAME            AGE
subnet.ec2.aws.upbound.io/my-db-a True   True  subnet-00a7554e7a87104da 44s
subnet.ec2.aws.upbound.io/my-db-b True   True  subnet-063055c491be14340 44s
subnet.ec2.aws.upbound.io/my-db-c True   True  subnet-0c8f8916b9b1fb4ca 44s

NAME                         SYNCED READY EXTERNAL-NAME         AGE
vpc.ec2.aws.upbound.io/my-db True   True  vpc-09f9e3df7626f19e0 48s

NAME                                         KIND   PROVIDERCONFIG SYNCED READY AGE
object.kubernetes.crossplane.io/my-db-secret Secret my-db-sql      True         48s

NAME                                              READY SYNCED AGE
database.postgresql.sql.crossplane.io/my-db-db-01       True   50s
database.postgresql.sql.crossplane.io/my-db-db-02       True   49s

NAME                              SYNCED READY EXTERNAL-NAME AGE
instance.rds.aws.upbound.io/my-db True   True                50s

NAME                                 SYNCED READY EXTERNAL-NAME AGE
subnetgroup.rds.aws.upbound.io/my-db True   True  my-db         52s
```

It's working and the fact that all the resources became `READY` right away signals that Crossplane correctly identified the resources that should be managed through the *external-name* annotation. Otherwise, some of them would not be `READY` since it takes a while to create some of those resources in AWS.

Similarly, we can confirm that the claim itself, the resource that manages all those managed resources, was created as well.

```sh
kubectl --kubeconfig kubeconfig-dot2.yaml --namespace infra \
    get sqlclaims
```

The output is as follows.

```
NAME    SYNCED   READY   CONNECTION-SECRET   AGE
my-db   True     True                        3m20s
```

It's there as well. It's status is set to `READY` only after all those managed resources are ready so that's yet another confirmation that it worked as expected.

Now we should be able to point Argo CD to the Git repo where the desired state of that claim is stored. Argo CD would not do anything right away since the actual state is the same as the desired state. However, the future updates to the desired state will be synced into the cluster.

So, what did we learn today?

## What Did We Learn?

The main outcome of today's exercisees is depression knowing that, if a disaster occurs, there is no single tool, or maybe even a combination of tools that will enable us to recover easily.

There is no simple and fast disaster recovery, especially if we wait for the disaster to happen to validate whether our processes work.

If we want to take disaster recovery seriously, we have to practice it all the time and the best way to do that is to **enforce destruction of clusters**. My recommendation is to **never upgrade clusters** but to always create new ones when we want to jump to the next Kubernetes release. If we do that, we will force ourselves to practice disaster recorvery couple of times a year, given that's the cadence of Kubernetes releases. It will be painful at first, but, over time, we might manage to strike the right balance between Kubernetes backups, database-specific backups, GitOps, and whatever else we might be using.

The problem is that it will never end. Even if we do manage to get to the point that we can reliably restore a cluster with everything in it, the situation will change. New applications are added, existing resources are evolving, and so on and so forth. Disaster recovery is a practice that needs to be exercised often if we want to be sure it is actually working.

As for tools... I recommend taking database backups using database-specific processes to create and restore them. Use Velero for resources that contain mutations that are critical yet missing from the desired state. Use Argo CD for all the other resources and make sure that you set up Argo CD or Flux in a way that resources are synced no matter the current state of the cluster.

Disaster recovery is **painful** and it never gets easy. The best we can do it make sure it is reliable.

## Destroy

```sh
chmod +x destroy.nu

./destroy.nu

exit
```


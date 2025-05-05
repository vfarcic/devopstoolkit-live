
+++
title = 'AWS Controllers for Kubernetes (ACK): Is It Worth It?'
date = 2025-05-04T16:00:00+00:00
draft = false
+++

It's not a secret that I believe in Kubernetes and that I think that it is much more than a "**thingy where we run containers**". It is an extensible API with controllers that can manage any type of resources. It is a **control plane** for everything.

That's why, a while ago, I was very excited when AWS Controllers for Kubernetes (ACK) project was born. It allows us to extend Kubernetes with Custom Resource Definitions and controllers that allow us to manage AWS resources through Kubernetes.

<!--more-->

{{< youtube Ht1ob_57f08 >}}

Now, to be honest, I never liked the idea of having tools provided by cloud or hyperscaler vendors to manage their resources. I never liked AWS CloudFormation or Azure Bicep for a very simple reason. Those are limited to the hyperscaler that released them, and we are almost never doing everything in a single hyperscaler or Cloud provider. There are always other services we might be using so I never liked the idea that I would manage some resources with CloudFormation, others with Bicep, something else for Grafana and Datadog, what-so-not for GitHub, Helm for Kuberentes, and so on and so forth. That's why **Terraform was great** back in the day **before we got Kubernetes**. It was a vendor-neutral way to manage any type of resources anywhere.

That changed with Kubernetes. Now it does not matter that we might be using it to manage apps with Knative, clusters with ACK, databases with CNPG, and so on and so forth. Those are all CRDs and controllers that follow the same guidelines. They all feel and behave in a similar way because they all standardized on Kubernetes API and they all work seamesly with the rest of the ecosystem.

All in all, having Kubernetes controllers provided by different vendors independently of each other is not only great but very welcome. That's why ACK, when it arrived a few years ago, was exciting.

The bad news is that, back then, it was too green. It had very **low coverage**, too many **expected features were missing**, and it **did not work well**. So, I put it on hold, until today. Most of controllers are now GA. Many of them are, at least according to AWS, stable and production ready. So, I decided to get back to it and check it out. I wanted to see whether it is everything I hoped it would be. More importantly, I wanted to see whether I can switch to it and replace the controllers I'm currently using.

So, today we'll explore ACK and try to figure out whether you should ditch whichever tool and format you're currently using to manage resources in AWS. After all, if it works well, and if AWS stands behind it, why wouldn't you use ACK as a building block of your control plane? With drift-detection and reconciliation, the ability to wrap it into your own controllers, integration with Argo CD, Flux, Kyverno and other projects in the Kubernetes ecosystem, ACK is a very strong candidate to be just what we need to build parts of an internal developer platform related to AWS. It's not all we need, but it could be a very important piece of the puzzle.

It's awesome, at least on paper, and I am very excited. Let's get going.

## Setup

```sh
git clone https://github.com/vfarcic/ack-demo

cd ack-demo
```

> Make sure that Docker is up-and-running. We'll use it directly as well as to create a KinD cluster.

> Watch [Nix for Everyone: Unleash Devbox for Simplified Development](https://youtu.be/WiFLtcBvGMU) if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell
```

> Watch [The Future of Shells with Nushell! Shell + Data + Programming Language](https://youtu.be/zoX_S6d-XU4) if you are not familiar with Nushell. Alternatively, you can inspect the `dot.nu` script and transform the instructions in it to Bash or ZShell if you prefer not to use that Nushell script.

```sh
chmod +x dot.nu

./dot.nu setup

source .env
```

## AWS ACK CRDs and CRs

If we would like to manage AWS resources through Kubernetes, ACK seems to be the way to go. After all, AWS would not create it and maintain it if they don't believe in it. As a matter of fact, having projects like ACK done by AWS itself is a validation that Kubernetes itself expanded way beyond "a thingy that can run containers." Kubernetes is a control plane we can use to manage any type of resources no matter where they are, and ACK could be a very important part of that story.

So, we can head out to the [Services](https://aws-controllers-k8s.github.io/community/docs/community/services/) section of the documentation and see all groups of services we can manage. There is, for example, `EC2` and `RDS` which we'll use today. Both are `RELEASED` and both are generally available (`GENERAL AVAILABILITY`).

I will not bore you with the installation of ACK controllers. All we have to do is install a Helm chart and then go through pain of setting up IRSA. If we survive IRSA nightmare, we get CRDs that represent specific groups of services in AWS. So, to streamline the process and avoid boring you to death with the installation and setup, I already did all that and now my EKS cluster has the ACK controllers for EC2 and RDS. We can observe what we got by retrieving all `crds` and filtering the output with `k8s.aws` which is the API they all use.

```sh
kubectl get crds | grep "k8s.aws"
```

The output is as follows.

```
adoptedresources.services.k8s.aws                     2025-04-06T16:43:29Z
capacityreservations.ec2.services.k8s.aws             2025-04-06T16:43:26Z
cninodes.vpcresources.k8s.aws                         2025-04-06T16:34:43Z
dbclusterparametergroups.rds.services.k8s.aws         2025-04-06T16:43:56Z
dbclusters.rds.services.k8s.aws                       2025-04-06T16:43:56Z
dbclustersnapshots.rds.services.k8s.aws               2025-04-06T16:43:56Z
dbinstances.rds.services.k8s.aws                      2025-04-06T16:43:57Z
dbparametergroups.rds.services.k8s.aws                2025-04-06T16:43:57Z
dbproxies.rds.services.k8s.aws                        2025-04-06T16:43:57Z
dbsnapshots.rds.services.k8s.aws                      2025-04-06T16:43:57Z
dbsubnetgroups.rds.services.k8s.aws                   2025-04-06T16:43:57Z
dhcpoptions.ec2.services.k8s.aws                      2025-04-06T16:43:26Z
elasticipaddresses.ec2.services.k8s.aws               2025-04-06T16:43:27Z
fieldexports.services.k8s.aws                         2025-04-06T16:43:29Z
flowlogs.ec2.services.k8s.aws                         2025-04-06T16:43:27Z
globalclusters.rds.services.k8s.aws                   2025-04-06T16:43:57Z
instances.ec2.services.k8s.aws                        2025-04-06T16:43:27Z
internetgateways.ec2.services.k8s.aws                 2025-04-06T16:43:27Z
natgateways.ec2.services.k8s.aws                      2025-04-06T16:43:27Z
networkacls.ec2.services.k8s.aws                      2025-04-06T16:43:27Z
policyendpoints.networking.k8s.aws                    2025-04-06T16:34:43Z
routetables.ec2.services.k8s.aws                      2025-04-06T16:43:27Z
securitygrouppolicies.vpcresources.k8s.aws            2025-04-06T16:34:43Z
securitygroups.ec2.services.k8s.aws                   2025-04-06T16:43:28Z
subnets.ec2.services.k8s.aws                          2025-04-06T16:43:28Z
transitgateways.ec2.services.k8s.aws                  2025-04-06T16:43:28Z
vpcendpoints.ec2.services.k8s.aws                     2025-04-06T16:43:28Z
vpcendpointserviceconfigurations.ec2.services.k8s.aws 2025-04-06T16:43:28Z
vpcpeeringconnections.ec2.services.k8s.aws            2025-04-06T16:43:28Z
vpcs.ec2.services.k8s.aws                             2025-04-06T16:43:28Z
```

There can be two reactions to that list of EC2 and RDS CRDs.

One could be "Wow! Now I can manage EC2 `instances`, `vpcs`, RDS `dbinstances`, and other resources in AWS." That's the positive reaction based on prospects of using Kubernetes as a control plane to manage you "stuff" no matter where that stuff is. That's the reaction of optimism of a person who uses only the most common AWS services.

Another reaction could be "What the f\*\*k! Is that it? Where are the rest of AWS services?"

There are around 250 API endpoints for AWS EC2 and RDS services while ACK gives us around 30. That means that the current coverage is slightly above 10%. That does not look good, especially considering that ACK has been released more than three years ago and, for example, EC2 controller is currently version 1.4, meaning that it reached the stability usually associated with projects that pass from version 0.something to version 1. ACK is not a kid anymore, yet it provides abysmally low coverage of AWS services.

Nevertheless, that is not necessarily a bad thing. Chances are that the CRDs ACK offers do cover what you need. If that's the case, my rant about the coverage was silly. It's all good. Who cares if what is missing is not what you need.

So, I'll make a pause here... Think about something positive like rainbows and unicorns... Happy thoughts... Now I'm back.

Let's say that we would like to run an RDS PostgreSQL database server. Since this is AWS, nothing is straightforward and we need to create quite a few resources to get the database server itself.

Here's an example of all the resources we might need.

```sh
cat rds.yaml
```

The output is as follows (truncated for brevity).

```yaml
---
apiVersion: ec2.services.k8s.aws/v1alpha1
kind: VPC
...
apiVersion: ec2.services.k8s.aws/v1alpha1
kind: InternetGateway
...
apiVersion: ec2.services.k8s.aws/v1alpha1
kind: RouteTable
...
apiVersion: ec2.services.k8s.aws/v1alpha1
kind: SecurityGroup
...
apiVersion: ec2.services.k8s.aws/v1alpha1
kind: Subnet
metadata:
  name: my-db-a
  ...
apiVersion: ec2.services.k8s.aws/v1alpha1
kind: Subnet
metadata:
  name: my-db-b
  ...
apiVersion: ec2.services.k8s.aws/v1alpha1
kind: Subnet
metadata:
  name: my-db-c
  ...
apiVersion: ec2.services.k8s.aws/v1alpha1
kind: Subnet
metadata:
  name: my-db-x
  ...
apiVersion: rds.services.k8s.aws/v1alpha1
kind: DBSubnetGroup
...
apiVersion: rds.services.k8s.aws/v1alpha1
kind: DBInstance
metadata:
  name: my-db
  annotations:
    services.k8s.aws/region: us-east-1
spec:
  dbSubnetGroupRef:
    from:
      name: my-db
  vpcSecurityGroupRefs:
    - from:
        name: my-db
  masterUsername: masteruser
  engine: postgres
  publiclyAccessible: true
  allocatedStorage: 200
  masterUserPassword:
    key: password
    name: my-db
  storageEncrypted: true
  dbInstanceIdentifier: my-db
  dbInstanceClass: db.m5.large
  engineVersion: "16.3"
---
apiVersion: services.k8s.aws/v1alpha1
kind: FieldExport
metadata:
  name: my-db-username
...
apiVersion: services.k8s.aws/v1alpha1
kind: FieldExport
metadata:
  name: my-db-endpoint
...
apiVersion: services.k8s.aws/v1alpha1
kind: FieldExport
metadata:
  name: my-db-port
...
```

We can see that all the resources we might need to run RDS in AWS are there. We have `VPC`, an `InternetGateway`, a `RouteTable`, a `SecurityGroup`, few `Subnet`s, a `DBSubnetGroup`, and, finally, a `DBInstance`.

If we focus on the `DBInstance`, which repesents the actual RDS database server, we can see that all the fields we might normally have to specify are there. There are `dbSubnetGroupRef` and `vpcSecurityGroupRefs` which reference other resources in that manifest. There are also typical fields like the `engine`, whether it should be `publiclyAccessible`, `allocatedStorage`, and so on and so forth. As a matter of fact, if you ignore that we are looking at YAML, you might mistake that definition with Terraform, Pulumi, Ansible, CloudFormation, or any other tool you might be currently using to manage resources. That's normal since all those tools, including ACK, are communicating with AWS API and, as such, their schemas are, more or less, matching the schema of that same API.

This, however, is different from those other tools because it is Kubernetes-native meaning that we get all the good things we get from Kubernetes, be it drift-detection, reconciliation, or anything else. Similarly, we can combine ACK with any tool we might be using in Kubernetes meaning that we can manage it with Argo CD or Flux, package it all up into a Helm chart, wrap it up into kro or Crossplane Compositions, apply policies with Kubernetes Validating Admission Policy or Kyverno, and so on and so forth.

You must know why Kubernetes is awesome and how powerful its ecosystem is so I won't waste my time justifying why using Kubernetes CRDs and controllers is a good thing.

Finally, there are a few `FieldExport` resources which, unlike others, might not be as intuitive and easy to figure out. I'll get to those later.

Since I did not set up Argo CD or Flux, we'll apply those resources directly with `kubectl`.

```sh
kubectl --namespace a-team apply --filename rds.yaml
```

So far, so good (more or less). Let's see whether we can observe what's going on.

## AWS ACK Observability

Here's a problem though. I would really like to be able to retrieve all the resources managed by ACK or all the resources somehow related to AWS. I would really like to know what is running in that Namespace, at least when ACK is concerned. That, however, is not possible, or, at least, I do not know how to do it. There is no *ack* or *aws* or any other similar alias. So, to retrieve all those, we need to execute something like `kubectl --namespace a-team get` and start writing each of them separately. To make things even more complicated, there could be multiple resources with the same *kind* so we might have to use full APIs.

```sh
kubectl --namespace a-team get \
    vpcs.ec2.services.k8s.aws,internetgateways.ec2.services.k8s.aws,routetables.ec2.services.k8s.aws,securitygroups.ec2.services.k8s.aws,subnets.ec2.services.k8s.aws,dbsubnetgroups.rds.services.k8s.aws,dbinstances.rds.services.k8s.aws
```

The output is as follows.

```
NAME                             ID                      STATE
vpc.ec2.services.k8s.aws/my-db   vpc-07cdab5bba559b994   available

NAME                                         ID
internetgateway.ec2.services.k8s.aws/my-db   igw-0cf1e6f9740700bfd

NAME                                    ID
routetable.ec2.services.k8s.aws/my-db   rtb-0ee52adc68ea0baac

NAME                                       ID
securitygroup.ec2.services.k8s.aws/my-db   sg-07fc8328613f6cfaa

NAME                                  ID                         STATE
subnet.ec2.services.k8s.aws/my-db-a   subnet-0168f5b5e1ace9a73   available
subnet.ec2.services.k8s.aws/my-db-b   subnet-0340ef39c2332c2c6   available
subnet.ec2.services.k8s.aws/my-db-c   subnet-0b36d95f548ba2627   available
subnet.ec2.services.k8s.aws/my-db-x                              

NAME                                       AGE
dbsubnetgroup.rds.services.k8s.aws/my-db   114s

NAME                                    STATUS
dbinstance.rds.services.k8s.aws/my-db   creating
```

That was painful and I honestly do not understand why AWS did not add aliases like `aws` to all ACK controllers and something like `ec2` to all the resources of the EC2 controllers.

Nevertheless, we got the list of all of those, and are now faced with yet another dissapointment.

I would expect each of those resources to print some useful information besides the `ID` and the `STATUS`. It would be nice to see the region where that `vpc` is running or the address through which we could access the `dbinstance` or any other useful information. None of it is there. It seems that ACK controllers are all generic, and that's dissapointing. Hopefully, we'll see the info might might need if we describe any of those resources. We'll check that later.

Now, not only that print columns are generic, but even the few fields that always appear are somehow random. Some resources have the `STATE` while others have the `STATUS` which, probably, serve the same purpose. Maybe the team working on the EC2 controller never coordinated with the team working on the RDS controller?

To make things even worse, the `STATE` is not really working either. There is no value for the `my-db-x` `subnet`. I recon that I made some mistake, and that's fine. We all mess it up from time to time. Still, I would expect that mistake to be somehow reflected in the `STATE` column. Heck, for all I know, it's working perfectly just as the `securitygroup` might work working without any status or state.

Let's see whether we can figure out what's wrong, if anything, with that subnet by describing the resource.

```sh
kubectl --namespace a-team \
    describe subnet.ec2.services.k8s.aws my-db-x
```

The output is as follows.

```
Name:         my-db-x
Namespace:    a-team
Labels:       <none>
Annotations:  services.k8s.aws/region: us-east-1
API Version:  ec2.services.k8s.aws/v1alpha1
Kind:         Subnet
Metadata:
  Creation Timestamp:  2025-04-06T23:29:17Z
  Finalizers:
    finalizers.ec2.services.k8s.aws/Subnet
  Generation:        1
  Resource Version:  19111
  UID:               5b51c287-ee3c-4b8b-889a-b370e042a011
Spec:
  Availability Zone:  us-east-1x
  Cidr Block:         11.0.3.0/24
  Route Table Refs:
    From:
      Name:  my-db
  Tags:
    Key:    name
    Value:  my-db
    Key:    zone
    Value:  us-east1x
  Vpc Ref:
    From:
      Name:  my-db
Status:
  Ack Resource Metadata:
    Owner Account ID:  036548781187
    Region:            us-east-1
  Conditions:
    Last Transition Time:  2025-04-06T23:29:22Z
    Status:                True
    Type:                  ACK.ReferencesResolved
    Message:               api error InvalidParameterValue: Value (us-east-1x) for parameter availabilityZone is invalid. Subnets can currently only be created in the following availability zones: us-east-1a, us-east-1b, us-east-1c, us-east-1d, us-east-1e, us-east-1f.
    Status:                True
    Type:                  ACK.Terminal
    Last Transition Time:  2025-04-06T23:29:22Z
    Message:               Resource not synced
    Reason:                resource is in terminal condition
    Status:                False
    Type:                  ACK.ResourceSynced
Events:                    <none>
```

This is a good news bad news situation. Since I am still in the good mood and full of happy thoughts, I'll start with the good news.

We can see what's wrong. The `availabilityZone is invalid` and I should have used one of the `zones` that are listed there. Now I know what I did wrong. Great!

As for the bad news... ACK does not adhere to any sort of a standard. There is no *Ready* condition typically present in most Kubernetes resources, be it those baked into Kubernetes or those coming from custom controllers. Now, you might say that "ACK is *special* and should not follow Kubernetes standards." That would be a silly thing to say, but I'll roll with it. The conditions between different ACK resources are not standardized either. They are all over the place and whatever we learn by observing the conditions in that subnet will be different from conditions in other resources.

That's not all bad news though. There's more.

There are no `Events` so we don't know what the controller did. How is that possible? How could someone design a controller and forget to record events? Or, if that was a conscious decision, how did anyone come to the conclusion that events are useless? I hope it's the former so I'll have to check who the contributors are in hope that they are my age or older. Being over fifty gives us the advantage of being able to use "I forgot, my memory is not as good as it was before" as an excuse that gets us out of quite a few "situations".

Here's another example of a mess related to statuses, conditions, and events.

```sh
kubectl --namespace a-team \
    describe dbinstance.rds.services.k8s.aws my-db
```

The output is as follows (truncated for brevity).

```
...
Status:
  Ack Resource Metadata:
    Arn:                   arn:aws:rds:us-east-1:036548781187:db:my-db
    Owner Account ID:      036548781187
    Region:                us-east-1
  Activity Stream Status:  stopped
  Certificate Details:
    C A Identifier:  rds-ca-rsa2048-g1
    Valid Till:      2026-04-06T23:31:20Z
  Conditions:
    Last Transition Time:     2025-04-06T23:32:23Z
    Status:                   True
    Type:                     ACK.ReferencesResolved
    Last Transition Time:     2025-04-06T23:32:23Z
    Status:                   False
    Type:                     ACK.ResourceSynced
    Last Transition Time:     2025-04-06T23:32:23Z
    Message:                  Late initialization successful
    Reason:                   Late initialization successful
    Status:                   True
    Type:                     ACK.LateInitialized
  Customer Owned IP Enabled:  false
  Db Instance Port:           0
  Db Instance Status:         configuring-enhanced-monitoring
  Db Parameter Groups:
    Db Parameter Group Name:  default.postgres16
    Parameter Apply Status:   in-sync
  Db Subnet Group:
    Db Subnet Group Description:  I am too lazy to write descriptions
    Db Subnet Group Name:         my-db
    Subnet Group Status:          Complete
    Subnets:
      Subnet Availability Zone:
        Name:             us-east-1c
      Subnet Identifier:  subnet-0b36d95f548ba2627
      Subnet Outpost:
      Subnet Status:  Active
      Subnet Availability Zone:
        Name:             us-east-1b
      Subnet Identifier:  subnet-0340ef39c2332c2c6
      Subnet Outpost:
      Subnet Status:  Active
      Subnet Availability Zone:
        Name:             us-east-1a
      Subnet Identifier:  subnet-0168f5b5e1ace9a73
      Subnet Outpost:
      Subnet Status:  Active
    Vpc ID:           vpc-07cdab5bba559b994
  Dbi Resource ID:    db-2NOHJMPDDGYBPY6MHJZAOQOJQI
  Endpoint:
    Address:                            my-db.cn31sgaonaq0.us-east-1.rds.amazonaws.com
    Hosted Zone ID:                     Z2R2ITUGPM61AM
    Port:                               5432
  Iam Database Authentication Enabled:  false
  Instance Create Time:                 2025-04-06T23:32:02Z
  Option Group Memberships:
    Option Group Name:  default:postgres-16
    Status:             in-sync
  Pending Modified Values:
  Vpc Security Groups:
    Status:                 active
    Vpc Security Group ID:  sg-07fc8328613f6cfaa
Events:                     <none>
```

Besides the fact that there are no events, I dare you to find the status of that *DBInstance*. Is it working or not? If it is, what is happening to it?

Go ahead. Find it.

It is in the `Db Instance Status` field. That's the one that tells us what's going on. The information is there. We just need to try not to give up looking for it.

Let's wait until the *DBInstance* is fully available before we take a look at a few other features of ACK.

```sh
kubectl --namespace a-team \
    get dbinstances.rds.services.k8s.aws --watch
```

The output is as follows (truncated for brevity).

```
NAME    STATUS
...
my-db   configuring-enhanced-monitoring
my-db   backing-up
...
my-db   available
```

> Wait until the `STATUS` is `available` and the press `ctrl+c` to stop watching.

There we go. It is now available and we can turn our attention to the features we expect all Kubernetes resources to have.

## ACK Drift-Detection and Reconciliation

One of the features we expect from any Kubernetes resource is drift-detection and reconciliation.

I expect every single controller in Kubernetes to detect drifts between the desired and the actual state and to do reconciliation if a drift is detected. Let's see whether that is the case with ACK.

I am going to simulate a "disaster" scenario by going to AWS console and delete the RDS instance. The `status` clearly says that it is `Deleting` it.

![](console.png)

Now, the logical outcome should be that the controller detected the change of the actual state and, to begin with, changed the status in the Custom Resource as well.

Let's double check that.

```sh
kubectl --namespace a-team \
    get dbinstances.rds.services.k8s.aws --watch
```

The output is as follows.

```
NAME    STATUS
my-db   available
```

Okay. I was probably too hasty. Drift-detection loop might be set to one minute or something similar.

Let's wait for a while longer...

The output is as follows.

```
NAME    STATUS
my-db   available
```

ACK still claims that the instance is `available` even though we saw that it is not. That's bad, really bad. Imagine that you did not saw me deleting that instance from the console and that you are monitoring the state of the system through those Custom Resources directly, or through Grafana, or anything else. You would, rightly, conclude that everything works as expected, until you start receiving phone calls from people complaining that everything went to hell.

> Press `ctrl+c` to stop watching.

One possible explanation is that `get` command does not give us the information we might need, so let's describe that resource. That should surely give us a clue that something is wrong. Right?

```sh
kubectl --namespace a-team \
    describe dbinstance.rds.services.k8s.aws my-db
```

The output is as follows (truncated for brevity).

```
Status:
...
  Conditions:
    Last Transition Time:     2025-04-06T23:37:27Z
    Status:                   True
    Type:                     ACK.ReferencesResolved
    Last Transition Time:     2025-04-06T23:37:27Z
    Message:                  Late initialization successful
    Reason:                   Late initialization successful
    Status:                   True
    Type:                     ACK.LateInitialized
    Last Transition Time:     2025-04-06T23:37:27Z
    Message:                  Resource synced successfully
    Reason:                   
    Status:                   True
    Type:                     ACK.ResourceSynced
  Customer Owned IP Enabled:  false
  Db Instance Port:           0
  Db Instance Status:         available
...
Events:                     <none>
```

Oh my!

According to `Status` `Conditions`, everything is probably okay. We already saw that those are mostly useless, so I'll ignore those.

We can also see that the `Db Instance Status` is `available` so it must be working as expected.

`Events` are silent, so it is almost certainly working as expected and the only conclusion I can make is that I did not deleted it from the Console.

There are only two explanations. Either I was halicinating, or the controller does not provide drift-detection.

Actually, there is a third explanation which, in my opinion, is worse than the other two.

"By default, all ACK controllers attempt to detect drift once every 10 hours."

That was the direct quote from the documentation. Honestly, I would prefer halucinations. If only those three options are given to me, halicinations sound like the preferable one, with "someone forgot to include drift-detection in the controllers" being the second. It makes much more sense to forget to do something than to make a conscious decision to make default drift-detection set to ten hours. Imagine the situation in which your system detects that the database is not working ten hours after it actually became unavailable. How silly would that be?

I expect every project to have sensible defaults, and I don't see why ACK should be an exception. I understand that drift-detection loop of only a couple of seconds could put too much stress on the AWS API. It would be okay if it's once a minute, or once every five minutes, or maybe even once every ten minutes. Drift-detection set to every ten hours does not make any sense as the default value.

It could be worse. It could have been hard-coded to ten hours. Furtunately, it's not, and we can change the `defaultResyncPeriod`. Let's set it to `60` seconds, which might be the other extreme which I might not recommend in production, but should be okay to check whether it even works.

```sh
helm upgrade --install --create-namespace ack-rds-controller \
    oci://public.ecr.aws/aws-controllers-k8s/rds-chart \
    --version=1.4.14 \
    --namespace ack-system \
    --set aws.region=us-east-1 \
    --set reconcile.defaultResyncPeriod=60
```

Let's see whether the change in the re-sync period will result in the change of the status.

```sh
kubectl --namespace a-team get dbinstances --watch
```

The output is as follows.

```
NAME    STATUS
my-db   available
my-db   available
my-db   available
```

Even after waiting for over a minute, the status is still `available`.

Let's wait a bit more...

```
NAME    STATUS
my-db   available
my-db   available
my-db   available
my-db   available
my-db   available
my-db   available
my-db   available
my-db   available
my-db   available
my-db   available
my-db   available
my-db   available
my-db   creating
my-db   creating
...
```

After a while, the status changed to `creating`, and that might only increase the confusion.

Here's what's happening.

ACK does not really track statuses. It had no idea that the instance status changed to *deleting*. It was oblivious what's going on. Since it takes quite some time until AWS actually deletes the instance, it was claiming that it was available all that time. But, once the instance was fully removed, the controller detected that the DB does not exist any more and started creating a new one. That's when we saw the status change to `creating`.

Now, since this is public, I need to choose my words carefully. I cannot say that the experience is s\*\*t, nor that it s\*\*ks, nor use any of the words I would use among friends. Hence, I will say that the experience is "dissapointing".

Still, we got the database we were trying to create and drift-detection and reconciliation is working. Even though the experience is dissapointing, it is still better than if we tried to do the same without the controllers. Kubernetes is the way to go and having controllers like those from ACK is a good thing.

> Press `ctrl+c` to stop watching.

We're not yet done though. We still need to figure out how to connect our applications to that database and to do that we need to fetch credentials.

## ACK Secrets Management

Very often, we need some type of credentials to access AWS services. Databases are a good example. We often create them so that applications can store and access data and, to do that, apps need credentials.

Typically, I would expect to see something like *secret.name* and *secret.namespace* fields in the spec of specific resources and, if those are set, I would expect it to create that secret with whatever is needed to connect to the service. That's not much to ask and a pattern used by quite a few controllers.

When it comes to credentials, we have yet another good news, bad news, type of the situation.

The bad news is that ACK does not have a built-in mechanism to store credentials related to resources we are managing. There is no option to specify in, let's say, *DBInstance* the name and the Namespace where it should store IP, port, username, and password inside a Secret that could, later on, be mounted to whichever application needs it.

The good news is that there is a mechanism to do that. It requires more work than I would expect, but it is there.

Here's an example.

```sh
cat rds.yaml
```

The output is as follows (truncated for brevity).

```yaml
...
apiVersion: services.k8s.aws/v1alpha1
kind: FieldExport
metadata:
  name: my-db-username
spec:
  to:
    name: my-db-password
    kind: secret
    key: username
  from:
    path: .spec.masterUsername
    resource:
      group: rds.services.k8s.aws
      kind: dbinstance
      name: my-db
---
apiVersion: services.k8s.aws/v1alpha1
kind: FieldExport
metadata:
  name: my-db-endpoint
spec:
  to:
    name: my-db-password
    kind: secret
    key: endpoint
  from:
    path: .status.endpoint.address
    resource:
      group: rds.services.k8s.aws
      kind: dbinstance
      name: my-db
---
apiVersion: services.k8s.aws/v1alpha1
kind: FieldExport
metadata:
  name: my-db-port
spec:
  to:
    name: my-db-password
    kind: secret
    key: port
  from:
    path: .status.endpoint.port
    resource:
      group: rds.services.k8s.aws
      kind: dbinstance
      name: my-db
```

We're using `FieldExport` resource that is installed with all ACK controllers so fetch `masterUsername` from `dbinstance` and put it into the `username` field of the `my-db-password` `secret`. Then we are doing a similar process for the `endpoint` and `port`. We're not retrieving the password since RDS requires us to have it in a secret which happens to be the same one we're extending and I created it during the setup.

In that example, all the data is coming from the same resource, but, if needed, we could have assembled data from multiple sources as well. *FieldExport* is flexible. It allows us to generate Secrets, ConfigMaps, and probably other resources using data from other sources. It's simple and good at the same time.

Still, it would be much easier to simply say: "You know what data you're generating. Don't make me spell it out for you. Just put it into that secret."

What matters is that we are able to create Secrets with the information needed to connect to resources in AWS, so it is a story with happy ending.

We can confirm that it's all working by retrieving the `secret` `my-db` from the cluster.

```sh
kubectl --namespace a-team get secret my-db-password --output yaml
```

The output is as follows.

```yaml
apiVersion: v1
data:
  endpoint: bXktZGIuY24zMXNnYW9uYXEwLnVzLWVhc3QtMS5yZHMuYW1hem9uYXdzLmNvbQ==
  password: T1QrOXZQcDhMdXhoeFVQWVpLSk1kUG1YM04xTzBTd3YzWG5ZVjI0UFZzcz0=
  port: NTQzMg==
  username: bWFzdGVydXNlcg==
kind: Secret
...
```

We can see that the `endpoint`, `port`, and `username` were added to the Secret so it can be mounted to whichever app should connect to the database.

All that's left is to see what to do with all those resources.

Let's `delete` what we applied so far before we start over.

```sh
kubectl --namespace a-team delete --filename rds.yaml
```

## Wrap-Up ACK Resources

More often than not, we do not keep a bunch of Kubernetes resources as endless lines of YAML in a file. Instead, we tend to wrap them into something more meaningful. For some, that something would be [Helm](https://helm.sh/) that would have all those resources as templates and expose parts that matter as values. We won't do that today, for two reasons. First, if I show you an example with Helm, my face would turn red and I could not resist explaining, yet again, why Helm is a bad option for anything but third-party operators. The second reason is that ACK is all about using Custom Resource Definitions and controllers to manage resources, so it's only natural that we create our own CRDs if we would like to move into higher-level abstractions.

However, since we are focused on Kubernetes Custom Resource Definitions and controllers, it might make more sense to create our own higher-level CRDs. Given that it's almost just as easy to create CRDs as Helm values and controllers as Helm templates, I don't see a reason not to do it.

We can create our own controllers and, often, CRDs with tools like [KubeVela](https://kubevela.io/), [kro](https://kro.run/), [Crossplane](https://www.crossplane.io/), and quite a few others. I will not discuss here which one you should use, but jump straight into showing you how that might look like with Crossplane since that's the one I tend to use.

```sh
cat dot-sql.yaml
```

The output is as follows.

```yaml
---
apiVersion: devopstoolkit.live/v1beta1
kind: SQL
metadata:
  name: my-db
spec:
  version: "16.3"
  size: medium
  region: us-east-1
  crossplane:
    compositionSelector:
      matchLabels:
        provider: aws-ack
        db: postgresql
```

That's a simple user-friendly interface that exposes "stuff" that matters and hides lower-level complexity most people do not care about. It should be self-explanatory what each field represents, so let's just apply it.

```sh
kubectl --namespace a-team apply --filename dot-sql.yaml
```

Do you remember how we could not retrieve all the ACK resources we created earlier without specifying each and every single one of them? Well... Now we can since the Crossplane Composite Resource we just applied acts as a parent that composes children resources. As a result, we can use something like the `kubectl tree` plugin to retrieve tree-like structure of all children resources of the `sqls` resource `my-db`.

```sh
kubectl tree --namespace a-team sqls my-db
```

```
NAMESPACE  NAME                       READY REASON   AGE
a-team     SQL/my-db                  False Creating 75s
a-team     ├─DBInstance/my-db         -              75s
a-team     ├─DBSubnetGroup/my-db      -              75s
a-team     ├─InternetGateway/my-db    -              75s
a-team     ├─RouteTable/my-db         -              75s
a-team     ├─Secret/d7abb0e9-0f1f-... -              75s
a-team     ├─SecurityGroup/my-db      -              75s
a-team     ├─Subnet/my-db-a           -              75s
a-team     ├─Subnet/my-db-b           -              75s
a-team     ├─Subnet/my-db-c           -              75s
a-team     └─VPC/my-db                -              75s
```

There we go. We got the same ACK resources as before and those, in turn, are now managing actual AWS resource.

The only problem we're having now is that the Composite Resource `SQL` has no idea whether the resources it composed are `READY` or not since ACK controllers do not provide "standard" Kubernetes conditions. None of them has the `READY` status, so the Composite Resource is getting confused. I could have fixed that by telling Crossplane what to look for, but I was too lazy to do that. You are witnessing procrastination in action.

Since Crossplane is not the subject of this video, I'll skip the explanation how I constructed that Composite Resource Definition.

If you'd like to take a look at the source code execute the command that follows to open the repository.

```sh
gh repo view vfarcic/crossplane-sql --branch v2 --web
```

Once inside, open `package/compositions.yaml`. You'll see Composite Resource Definitions for quite a few PostgreSQL implementations. If you're interested in ACK part of it, search for `ack`.

## AWS ACK Pros and Cons

Typically, I would finish a video with pros and cons. That's not what I'll do today since there is only one positive thing I can say about ACK.

It provides Kubernetes Custom Resource Definitions and controllers that allow us to manage AWS resources. That's great since Kubernetes is the best place to manage resources no matter whether they are running in the same cluster, or elsewhere. It is a control plane so we need something like ACK if some of those resources we're managing are in AWS. Moreover, I cannot imagine a better company to create and maintain Kubernetes AWS controllers than AWS itself. They own their services so they should own Kubernetes controllers that manage those services.

On the negative side, I can only say that almost everything is wrong. ACK has very small AWS coverage, default reconciliation loop of ten hours is silly, there are no aliases like *aws* or *ack*, there is no easy built-in mechanism to generate secrets, conditions are all over the place, events are non existent, and so on and so forth. I'm not listing cons mainly because there are so many that it's easier to say that **AWS does not take ACK seriously**, and that's a shame.

That does not mean that we should not manage AWS resources with Kubernetes controllers. We should since there are much better alternatives to ACK, so don't give up on using Kubernetes as a control plane due to bad experience with ACK. Even if that would be the only option, Kubernetes as a control plane wins over any other alternative and ACK is better than nothing.

All in all, I'm **grateful that ACK exists**, but **even more grateful that it is not the only option** so I don't have to use it.

I suspect that I'm on some kind of AWS black list because of some statements I made in other videos. After this one, my suspicion will turn into near certainty, so here's the message to AWS that I hope will help me avoid being in their list of undesirables.

Here's my video version of an open letter.

*Dear AWS,*

*There are many things you do great. There are many services and projects that lead the way and are shining light in this industry. You're great on so many levels. Nevertheless, ACK is garbage.*

*What pains me is that I know that you can do it well and I can only conclude that ACK is not your focus. Please change that. Please make ACK great. I'd love to have AWS controllers that are actually good and maintained by you. Until that happens, there is only one place suitable for garbage.*

## Destroy

```sh
kubectl --namespace a-team delete --filename dot-sql.yaml

kubectl --namespace a-team get \
    vpcs.ec2.services.k8s.aws,internetgateways.ec2.services.k8s.aws,routetables.ec2.services.k8s.aws,securitygroups.ec2.services.k8s.aws,subnets.ec2.services.k8s.aws,dbsubnetgroups.rds.services.k8s.aws,dbinstances.rds.services.k8s.aws
```

> Wait until all resources are removed.


```sh
./dot.nu destroy

exit
```


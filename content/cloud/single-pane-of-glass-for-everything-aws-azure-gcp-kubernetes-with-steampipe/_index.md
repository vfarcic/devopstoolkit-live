
+++
title = 'Single Pane of Glass for Everything (AWS, Azure, GCP, Kubernetes, ...) with Steampipe'
date = 2024-06-17T16:00:00+00:00
draft = false
+++

Right now, I am connected to a PostgreSQL database and you are thinking that is boring. Who wants to see me querying a database?

Well... Wait until you see what I can query from such a database.

<!--more-->

{{< youtube AhdBJQVx0bQ >}}

I can, for example, ask it to retrieve data by selecting `cluster`, `namespace`, and `name` columns from all Kubernetes clusters (`kubernetes_all`), and that the data should represent all Pods (`kubernetes_pod`) in those clusters.

```sql
select _ctx -> 'connection_name' as cluster, namespace, name from kubernetes_all.kubernetes_pod;
```

The output is as follows (truncated for brevity).

```

+---------------------+-------------+------------------------------------+
| cluster             | namespace   | name                               |
+---------------------+-------------+------------------------------------+
| "kubernetes_aks_03" | kube-system | coredns-autoscaler-c6649b67c-mgrmc |
| "kubernetes_aks_03" | kube-system | cloud-node-manager-4wm6x           |
| "kubernetes_aks_03" | kube-system | metrics-server-5cd44496f4-mfz6v    |
| "kubernetes_aks_03" | a-team      | silly-demo-7796698d8d-28fz5        |
...
```

Okay. That might still be boring, since you might be able to accomplish a similar result by creating a simple script that executes kubectl get in a loop. Or, it might be boring because if you already saw the [Single Pane of Glass for Kubernetes Clusters with Clusterpedia](https://youtu.be/Ca1qxZoxBkg) video which does something similar.

Here comes the punch.

I can also query resources in my Azure account, like "give me a list of all (`*`) Kubernetes clusters (`azure_kubernetes_cluster`)."

```sql
select * from azure_kubernetes_cluster
```

```
+--------+-------------------------------------------+---->
| name   | id                                        | typ>
+--------+-------------------------------------------+---->
| dot-03 | /subscriptions/.../managedClusters/dot-03 | Mic>
| dot-02 | /subscriptions/.../managedClusters/dot-02 | Mic>
| dot-01 | /subscriptions/.../managedClusters/dot-01 | Mic>
+--------+-------------------------------------------+---->
(END)
```

Now that is already more interesting. I can query Kubernetes resources and I can query those in Azure, and I can do that using the same interface and the same syntax. Yet, that might not be enough to convince you so let me tell you that I can also query AWS, and Google Cloud, and Open AI, and GitHub, and Terraform, and Slack, and almost any other source.

It's like having a database that contains information about everything, everywhere, all at once.

Isn't that awesome?

Isn't it great to have the ability to store information about everything we do, wherever we do it, in a database, and query that data in the same way we would query any other data?

That's what Steampipe enables us to do. It allows us to pull data from almost any source into a PostgreSQL database and query it. On top of that, there is almost no learning curve. If you have at least a basic knowledge of SQL, you can use Steampipe. If you don't, then you have a serious problem that cannot be addressed by this video.

If you're still here, it means that basic SQL does not scare you, hence I can give you TL;DR; right away.

Steampipe has its quirks and rough edges. Yet, if you manage resources in multiple locations, as almost everyone does, it is amazing. It might easily become one of your best friends, which might, or might not, turn up to be an asshole later on.

So, what is Steampipe?

A short description is that it is a tool that **queries APIs**, **stores the data** behind those APIs into a database, and allows us to **query that data**.

Those APIs can be almost anything. It supports over one hundred and forty of them through plugins so I can safely say that if you work with something that is not supported, you are working with antiquities and should consider replacing it, unless you consider yourself a collector of long-time gone systems.

The data is stored in a PostgreSQL or a SQLite database meaning that we can query that data just as we would query any other data. We can join data from different sources, we can filter them, or we can crunch it in any other way.

That's enough of a pep talk.

Let's dive into it. Let's see Steampipe in action.

## Setup

```sh
git clone https://github.com/vfarcic/steampipe-demo

cd steampipe-demo
```

> Watch https://youtu.be/WiFLtcBvGMU if you are not familiar with Devbox. Alternatively, you can skip Devbox and install all the tools listed in `devbox.json` yourself.

```sh
devbox shell

chmod +x setup.sh

./setup.sh

source .env
```

## Steampipe in Action

I already setup a few Kubernetes clusters in Azure and each of those clusters has some "random" resources running in it. Think of it as a mini simulation of what I might have in production.

Besides that, I did not do anything else except install Steampipe CLI. There was no special setup done in advance, at least not when Steampipe is concerned.

To begin with, we might want to take a look at the currently available plugins.

> Open https://hub.steampipe.io in a browser

We can see that **AWS**, **Google Cloud**, **Azure**, **OpenAI**, **GitHub**, **Terraform**, **Slack**, and **Kubernetes**. That's only a fraction of **143 plugins**.

The only thing we need to do is install one of those plugins, so let's start with `kubernetes`.

```sh
steampipe plugin install kubernetes
```

Installation of that plugin created a `kubecrnetes.spc` config file, so let's take a look at what we got.

```sh
cat ~/.steampipe/config/kubernetes.spc
```

The output is as follows (truncated for brevity).

```spc
connection "kubernetes" {
  plugin = "kubernetes"
  ...
  custom_resource_tables = ["*"]
}
```

If we ignore the comments in that file, it all boils down to two lines. It is the configuration for the `kubernetes` plugin that instructs Steampipe to pull not only Kubernetes resources that are baked into the cluster but also all (`["*"]`) custom resources (`custom_resource_tables`).

Since we did not specify otherwise, whichever Kubernetes cluster our kube config is pointing to will be synchronized into a PostgreSQL database, eventually.

However, that's not what we might want. What would be the point of having the ability to query data based on resourecs in a single cluster? There would be no need for Steampipe for something like that. We could do that with kubectl.

So, let's change the config to use three clusters.

```sh
echo '
connection "kubernetes_aks_01" {
  plugin         = "kubernetes"
  config_path    = "/Users/viktorfarcic/code/steampipe-demo/kubeconfig-01.yaml"
  config_context = "dot-01"
  custom_resource_tables = ["*"]
}
connection "kubernetes_aks_02" {
  plugin         = "kubernetes"
  config_path    = "/Users/viktorfarcic/code/steampipe-demo/kubeconfig-02.yaml"
  config_context = "dot-02"
  custom_resource_tables = ["*"]
}
connection "kubernetes_aks_03" {
  plugin         = "kubernetes"
  config_path    = "/Users/viktorfarcic/code/steampipe-demo/kubeconfig-03.yaml"
  config_context = "dot-03"
  custom_resource_tables = ["*"]
}
connection "kubernetes_all" {
  plugin      = "kubernetes"
  type        = "aggregator"
  connections = ["kubernetes_*"]
}
' | tee ~/.steampipe/config/kubernetes.spc
```

That new configuration specifies Kube config path (`config_path`), the context (`config_context`), and, just as in the default config, all custom resources (`custom_resource_tables`), for each of the clusters. As an added bonus, the last entry creates `kubernetes_all` which, as the name suggests, will aggregate data from all three clusters. We are accomplishing that by saying that entry is of the `aggregator` `type` and that it references all the `connections` with names starting with `kubernetes_` which, in this example, are literally all connections.

Let me give you a big of a warning though. We can specify any number of custome resources in the `custom_resource_tables` array but, if they differ, Steampipe will get confused. You see, even though Steampipe is awesome, it's not very bright. It gets easily confused. Hence, keep all entries the same. If you try to specify something like "sync those resources from that cluster but some other resources from some other cluster", Steampipe will go bazooka. It'll get so confused that it will start producing unexpected and, sometimes, missleading results.

Anyway... Now that we configured Steampipe to work with three Kubernetes clusters, we can use the CLI to `query` it.

```sh
steampipe query
```

From here on, we can, for example, ask it to `inspect` itself.

```sh
.inspect
```

The output is as follows (truncated for brevity).

```
+-------------------+---------------------------------------------------+-------+
| connection        | plugin                                            | state |
+-------------------+---------------------------------------------------+-------+
| kubernetes_aks_01 | hub.steampipe.io/plugins/turbot/kubernetes@latest | ready |
| kubernetes_aks_02 | hub.steampipe.io/plugins/turbot/kubernetes@latest | ready |
| kubernetes_aks_03 | hub.steampipe.io/plugins/turbot/kubernetes@latest | ready |
| kubernetes_all    | hub.steampipe.io/plugins/turbot/kubernetes@latest | ready |
+-------------------+---------------------------------------------------+-------+
...
```

We can see that it has established connections with the three clusters as well as the aggregate connection to all the clusters.

If we'd like to see which tables it created for, let's say, the aks_01 cluster, we can inspect that as well.

```sh
.inspect kubernetes_aks_01
```

The output is as follows (truncated for brevity).

```
+------------------------------------+--------------------------------------------------+
| table                              | description                                      |
+------------------------------------+--------------------------------------------------+
...
| kubernetes_defaultvpcdhcpoptions   | DefaultVPCDHCPOptions is the Schema for...       |
| kubernetes_deployment              | Kubernetes Deployment enables declarative...     |
| kubernetes_deploymentruntimeconfig | A DeploymentRuntimeConfig is used to configure...|
| kubernetes_ebsdefaultkmskey        | EBSDefaultKMSKey is the Schema for the...        |
...
+------------------------------------+--------------------------------------------------+
```

We can see that tables representing resources and custom resources are there, all prefixed with `kubernetes` to distinguish them from tables coming from other sources. For example, if we are interested in Deployments, the data is stored in the `kubernetes_deployment` table.

Similarly, we can output all the tables, no matter from which source they were created.

```sh
.tables
```

The output is as follows (truncated for brevity).

```
...
| kubernetes_vpngatewayroutepropagation | VPNGatewayRoutePropagation is...   |
+---------------------------------------+------------------------------------+
 ==> kubernetes_aks_03
+--------------+-------------------------------------------------------------+
| table        | description                                                 |
+--------------+-------------------------------------------------------------+
| helm_chart   | Lists the configuration settings from the configured charts |
| helm_release | List all of the releases of chart in a Kubernetes cluster   |
...
+--------------+-------------------------------------------------------------+
 ==> kubernetes_all
+--------------+-------------------------------------------------------------+
| table        | description                                                 |
+--------------+-------------------------------------------------------------+
| helm_chart   | Lists the configuration settings from the configured charts |
| helm_release | List all of the releases of chart in a Kubernetes cluster   |
...
+--------------+-------------------------------------------------------------+
...
```

Since, for now, the only source are Kubernetes clusters, we can see tables coming from all clusters (`kubernetes_all`), from AKS 03 (`kubernetes_aks_03`), and so on and so forth.

Once we discover what is available, we can start querying the database and, for example, retrieve all deployments.

```sql
select * from kubernetes_deployment
```

The output is as follows (truncated for brevity).

```
+------------+-------------------+--------------+----------+---------------------------->
| name       | namespace         | uid          | replicas | selector_query             >
+------------+-------------------+--------------+----------+---------------------------->
...
| silly-demo | a-team            | 6b5b6747-... | 2        | app.kubernetes.io/name=sill>
...
| crossplane | crossplane-system | e22389b1-... | 1        | app=crossplane,release=cros>
...
+------------+-------------------+--------------+----------+---------------------------->
(END)
```

We can see that `silly-demo` from the `a-team` Namespace is there, that `crossplane` from `crossplane-system` is there as well, and so on and so forth. Since, more often than not, we cannot fit all the columns and rows on the screen, we can scroll left, right, up, and down and observe whichever data we're interested in.

> Scroll left, right, up, and down.

Once we're done, we just need to type `q` to exit the view.

> Press `q` to exit the table

There are many other quality of life features like, for example, auto-complete.

We can, for example, just start typing something like `select` all (`*`) columns `from` `kuber` and select what we're looking for by pressing tab to choose the next match, or shift tab to select the previous, or we can continue typing until the number of entries is limited to close to what we need, select what we're looking for, and press the enter key.

> Press `tab` to select the next item in the list and `shift+tab` for the previous. Select `kubernetes_pod` and press the `enter` key.

```sql
select * from kubernetes_all.kubernetes_pod;
```

The output is as follows (truncated for brevity).

```
+-------------------------------------------+-------------------+--------------+---------+>
| name                                      | namespace         | uid          | selector|>
+-------------------------------------------+-------------------+--------------+---------+>
...
| crossplane-contrib-provider-sql-...       | crossplane-system | aaab0120-... | <null>  |>
| upbound-provider-azure-dbforpostgresql... | crossplane-system | e926f630-... | <null>  |>
| crossplane-contrib-function-patch-and-... | crossplane-system | a4e19584-... | <null>  |>
| coredns-767bfbd4fb-96gbk                  | kube-system       | 54dcfad0-... | <null>  |>
...
```

Now that you saw me retrieving all the columns, I should say that you should probably never do that. If you ever worked with SQL databases, you know that * can cause significant performance issues. To be clear, that's not necessarily the issue of Steampipe but, rather, the issue with PostgreSQL or any other similar database. The more data we have, the more dangerous is to retrieve all the data from a table using *.

> Press `q` to exit the table

Instead, we should find out which fields we're interested in and specify them explicitly.

We can do that by taking a look at a table in the `.inspect` command.

```sh
.inspect kubernetes_aks_01.kubernetes_pod
```

The output is as follows (truncated for brevity).

```
+---------------------------------+---------+---------------------------------------------+
| column                          | type    | description                                 |
+---------------------------------+---------+---------------------------------------------+
| _ctx                            | jsonb   | Steampipe context in JSON form.             |
| active_deadline_seconds         | text    | Optional duration in seconds the pod may... |
| affinity                        | jsonb   | If specified, the pod's scheduling...       |
| annotations                     | jsonb   | Annotations is an unstructured key value... |
| automount_service_account_token | boolean | AutomountServiceAccountToken indicates...   |
| conditions                      | jsonb   | Current service state of pod.               |
| container_statuses              | jsonb   | The list has one entry per container in...  |
| containers                      | jsonb   | List of containers belonging to the pod.    |
...
```

There we go. That's the list off all the fields that represent Kubernetes Pods, and we can easily observe two potential issues there.

One potential issue is that values of fields deeper than two levels are stored as Jsonb. For example, the spec.containers field in a Pod is stored as `containers` field of type `jsonb`. Whatever is inside containers, and there's a lot, is all stored as a big blob. In some cases, that's only an annoyance while in others it prevents us to do what needs to be done.

Let me explain though a few examples.

For example, if we would like to output the connection name which represents a cluster, we will quickly find out that there is no such field. Instead, it is stored in the _ctx field together with some other information. So, we'd need to `select` the `_ctx` together with whichever other fields we need.

```sql
select _ctx, namespace, name from kubernetes_all.kubernetes_pod;
```

The output is as follows (truncated for brevity).

```
+------------------------------------------------------------------------------+-------------+>
| _ctx                                                                         | namespace   |>
+------------------------------------------------------------------------------+-------------+>
| {"connection_name":"kubernetes_aks_03","steampipe":{"sdk_version":"5.10.0"}} | kube-system |>
| {"connection_name":"kubernetes_aks_03","steampipe":{"sdk_version":"5.10.0"}} | kube-system |>
| {"connection_name":"kubernetes_aks_03","steampipe":{"sdk_version":"5.10.0"}} | kube-system |>
| {"connection_name":"kubernetes_aks_03","steampipe":{"sdk_version":"5.10.0"}} | a-team      |>
...
```

As we can see, the name of the cluster is stored as Jsonb field `connection_name` inside `_ctx`.

> Press `q` to exit the table

Fortunately, we can select specific fields from Jsonb and output their values as columns using ->. So, we can say something like `select` `_ctx` column, extract `connection_name` and output it as a column `cluster`.

```sql
select _ctx -> 'connection_name' as cluster, namespace, name from kubernetes_all.kubernetes_pod;
```

The output is as follows.

```
+---------------------+-------------+------------------------------------+
| cluster             | namespace   | name                               |
+---------------------+-------------+------------------------------------+
| "kubernetes_aks_03" | kube-system | coredns-autoscaler-c6649b67c-mgrmc |
| "kubernetes_aks_03" | kube-system | cloud-node-manager-4wm6x           |
| "kubernetes_aks_03" | kube-system | metrics-server-5cd44496f4-mfz6v    |
| "kubernetes_aks_03" | a-team      | silly-demo-7796698d8d-28fz5        |
...
```

That output makes more sense. The additional instructions we had to perform represent annoyance with values stored as Jsonb. We can extract data we need, but the syntax becomes more complicated. It's not the end of the world. It's just a tiny annoyance, for now.

> Press `q` to exit the table

Let's turn to less depressing parts of Steampipe like, for example, the fact that we can use `where` statements to filter the results.

We can, for example, specify the `cluster`, `name`, `ready_replicas`, and `replicas` fields from `kubernetes_deployment` table and add the `where` statement that will limit the output to data from Pods residing in the `namespace` `a-team` and the `name` set to `silly-demo`.

```sql
select _ctx -> 'connection_name' as cluster, name, ready_replicas, replicas from kubernetes_all.kubernetes_deployment where namespace = 'a-team' and name = 'silly-demo';
```

The output is as follows.

```
+---------------------+------------+----------------+----------+
| cluster             | name       | ready_replicas | replicas |
+---------------------+------------+----------------+----------+
| "kubernetes_aks_01" | silly-demo | 2              | 2        |
| "kubernetes_aks_03" | silly-demo | 2              | 2        |
| "kubernetes_aks_02" | silly-demo | 2              | 2        |
+---------------------+------------+----------------+----------+
```

That's awesome and, potentially, easier than if we tried to accomplish the same with kubectl looping against all the clusters and using --namespace field and label selectors.

We can even do fuzzy search with the like statement that will return all Deployments with the `name` that contains the word `something`.

```sql
select _ctx -> 'connection_name' as cluster, name, ready_replicas, replicas from kubernetes_all.kubernetes_deployment where namespace = 'a-team' and name like '%something%';
```

The output is as follows.

```
+---------------------+----------------+----------------+----------+
| cluster             | name           | ready_replicas | replicas |
+---------------------+----------------+----------------+----------+
| "kubernetes_aks_02" | something-else | 2              | 2        |
+---------------------+----------------+----------------+----------+
```

That output contains almost the same data we would get with kubectl get deployments but with data from multiple clusters.

I can safely say that any type of queries beyond the simplest ones become much easier to do with SQL than kubectl combined with a lot of shell scripting like loops, greps, regexes, and so on and so forth.

We can also combine queries with a union. For example, let's say that we are interested only in deployments from clusters 01 and 02. We could get those by selecting deployments from one cluster and combining them using `union` with the data from the other cluster.

```sql
select _ctx -> 'connection_name' as cluster, name, ready_replicas, replicas from kubernetes_aks_01.kubernetes_deployment where namespace = 'a-team' union all select _ctx -> 'connection_name' as cluster, name, ready_replicas, replicas from kubernetes_aks_02.kubernetes_deployment where namespace = 'a-team'
```

The output is as follows.

```
+---------------------+----------------+----------------+----------+
| cluster             | name           | ready_replicas | replicas |
+---------------------+----------------+----------------+----------+
| "kubernetes_aks_01" | silly-demo     | 2              | 2        |
| "kubernetes_aks_02" | something-else | 2              | 2        |
| "kubernetes_aks_02" | how-about-this | 0              | 2        |
| "kubernetes_aks_02" | silly-demo     | 2              | 2        |
+---------------------+----------------+----------------+----------+
```

We can also order results by using the `order by` SQL instruction.

```sql
select _ctx -> 'connection_name' as cluster, name, namespace, ready_replicas, replicas from kubernetes_all.kubernetes_deployment order by _ctx, namespace
```

The output is as follows (truncated for brevity).

```
+---------------------+----------------------+-------------------+----------------+----------+
| cluster             | name                 | namespace         | ready_replicas | replicas |
+---------------------+----------------------+-------------------+----------------+----------+
| "kubernetes_aks_01" | silly-demo           | a-team            | 2              | 2        |
...
| "kubernetes_aks_02" | how-about-this       | a-team            | 0              | 2        |
| "kubernetes_aks_02" | silly-demo           | a-team            | 2              | 2        |
| "kubernetes_aks_02" | something-else       | a-team            | 2              | 2        |
| "kubernetes_aks_02" | upbound-provider-... | crossplane-system | 1              | 1        |
...
+---------------------+----------------------+-------------------+----------------+----------+
```

There we go. The results are now ordered by the context which contains the `cluster` and by the `namespace`.

As you saw from the configuration, we are not limited only to resources baked into Kubernetes but can also query custom resources. For example, we can select data from `kubernetes_sql` table that contains SQL resources based on custom resource definition.

```sql
select _ctx -> 'connection_name' as cluster, name, creation_timestamp from kubernetes_all.kubernetes_sqlclaim;
```

The output is as follows.

```
+---------------------+----------+---------------------------+
| cluster             | name     | creation_timestamp        |
+---------------------+----------+---------------------------+
| "kubernetes_aks_02" | my-db-02 | 2024-05-12T21:42:24+02:00 |
| "kubernetes_aks_01" | my-db-01 | 2024-05-12T21:41:55+02:00 |
+---------------------+----------+---------------------------+
```

Let's get back to the depressing part of Steampipe, mostly caused by the usage of Jsonb data types.

We can extract data from Jsonb in `select` statements so we can say that we want `size` from the `parameters` field as well as `version` from the same field. We already saw that in action. What we did not try is to use data extracted from Jsonb in `from` or `where` statements. In this example, I would like to join data from tables `kubernetes_sqlclaim` and `kubernetes_sql`. To do that, we need to specify which fields contain matching data which, in this case would mean that `sqlclaim.name` contains the data that match `sql.labels` field `crossplane.io/claim-name`.

You can probably guess what will happen, but let's see it just in case you guessed wrong.

```sh
select sqlclaim.name, sqlclaim.namespace, sqlclaim.parameters -> 'size' as size, sqlclaim.parameters -> 'version' as version, sql.name as sql_name, sql.labels -> 'crossplane.io/claim-name' as claim_name from kubernetes_all.kubernetes_sqlclaim as sqlclaim join kubernetes_all.kubernetes_sql as sql on sqlclaim.name = sql.labels -> 'crossplane.io/claim-name';
```

The output is as follows.

```
Error: operator does not exist: text = jsonb (SQLSTATE 42883)
```

Nope. We cannot combine text and jsonb, at least not in from and where statements. Unfortunately, parent/child relationships in Kubernetes are always based on labels thus making it close to impossible to get such data from Steampipe.

Similarly, we could not, for example, retrieve all resources that failed to start, since conditions are also Jsonb field, not to mention events that are not even stored in the database.

We could, probably, perform some "dark magic" rituals to make it work, but I don't think its worth it. Using Jsonb to store complex data instead of extracting them to separate columns might have seemed like a good idea that simplifies the work on Steampipe but, ultimately, it is a huge mistake.

Another mistake is performance.

I'll demonstrate that by starting over, so let me `.exit` Steampipe,...

```sh
.exit
```

...and execute an ad-hoc query.

```sh
steampipe query \
    "select * from kubernetes_all.kubernetes_server;"
```

The output is as follows (truncated for brevity).

```
...
| my-db-01 | 1386bd62-f93a-4bda-9a8b-96aee64ec3b2 | Server | dbforpostgresql.azure.upbound.io/v1beta1 |           | 2024-05-12T21:41:56+02:00 | {"crossplane.io/claim-name":"my-db-01","crossplane.io/claim-namespace":"a-team","crossplane.io/composite":"my-db-01-k5cbq"} | dot-01       | deployed    | {"crossplane.io/composition-resource-name":"server","crossplane.io/external-create-failed":"2024-05-12T20:12:02Z","crossplane.io/external-create-pending":"2024-05-12T20:12:02Z","crossplane.io/external-create-succeeded":"2024-05-12T19:42:06Z","crossplane.io/external-name":"my-db-01"} | {"name":"default"}  | <null>                        | {"name":"my-db-01","namespace":"a-team"} | Delete          | {"administratorLogin":"postgres","administratorLoginPasswordSecretRef":{"key":"password","name":"my-db-01-password","namespace":"a-team"},"autoGrowEnabled":true,"location":"eastus","publicNetworkAccessEnabled":true,"resourceGroupName":"my-db-01","resourceGroupNameRef":{"name":"my-db-01"},"resourceGroupNameSelector":{"matchControllerRef":true},"skuName":"B_Gen5_1","sslEnforcementEnabled":false,"sslMinimalTlsVersionEnforced":"TLSEnforcementDisabled","storageMb":5120,"version":"11"}                                                | {}            | ["*"]               | <null>              | {}                                                                                                                                                                              ...
```

You'll notice that it took a couple of seconds to retrieve the information. Steampipe does not proactivelly pull information. Instead, it does the pulling on-demand. Now, you might say "that's okay since it took only a couple of seconds" and you would be right, when small setups like the one in this demo are used. On bigger systems, it can easily take minutes, or even more to load fresh data or see cached data that is out of date. Clusterpedia solves that problem by proactively storing all events into the database. That is a separate process unrelated with querying data. The price to pay for that is that Clusterpedia needs agents running, while Steampipe doesn't. Hence, you need to figure out whether you prefer up-to-date lightning fast retrieval of data through Clusterpedia or agentless model of Steampipe that is slower and can return outdated information.

Also, please take a look at that output from ad-hoc queries. It's useless and I honestly don't know what to do with it.

We can output it to other formats like, for example, Json.

```sh
steampipe query \
    "select * from kubernetes_all.kubernetes_server;" \
    --output json | jq .
```

The output is as follows (truncated for brevity).

```json
[
  {
    ...
    "for_provider": {
      "administratorLogin": "postgres",
      "administratorLoginPasswordSecretRef": {
        "key": "password",
        "name": "my-db-01-password",
        "namespace": "a-team"
      },
      "autoGrowEnabled": true,
      "location": "eastus",
      "publicNetworkAccessEnabled": true,
      "resourceGroupName": "my-db-01",
      "resourceGroupNameRef": {
        "name": "my-db-01"
      },
      "resourceGroupNameSelector": {
        "matchControllerRef": true
      },
      "skuName": "B_Gen5_1",
      "sslEnforcementEnabled": false,
      "sslMinimalTlsVersionEnforced": "TLSEnforcementDisabled",
      "storageMb": 5120,
      "version": "11"
    },
    ...
```

That's not very useful either. However, I can blame myself for that one. I should have limited the number of selected fields. Also, you will likely do the queries from the Steampipe Shell and use those ad-hoc queries in automation like CI pipelines (assuming that you need Steampipe there).

Here comes the best part that distinquishes Steampipe from, let's say, Clusterpedia. We can query data coming from almost any source, so let's add, for example, Azure to the mix.

```sh
steampipe plugin install azure
```

Now we can go back to the Steampipe Shell...

```sh
steampipe query
```

...and inspect `azure` resources.

```sh
.inspect azure
```

The output is as follows (truncated for brevity).

```
+-------------------------------+-------------------------------+
| table                         | description                   |
+-------------------------------+-------------------------------+
...
| azure_tenant                  | Azure Tenant                  |
| azure_virtual_network         | Azure Virtual Network         |
| azure_virtual_network_gateway | Azure Virtual Network Gateway |
+-------------------------------+-------------------------------+
```

From here on, the process is the same. The scope increased so now we can, for example, ask it to list all Azure Resource Groups (`azure_resource_group`).

```sql
select * from azure_resource_group
```

The output is as follows.

```
+-------------------------------------+-----------------------------------------+----------->
| name                                | id                                      | provisioni>
+-------------------------------------+-----------------------------------------+----------->
| dot-20240512212559                  | .../dot-20240512212559                  | Succeeded >
| my-db-02                            | .../my-db-02                            | Succeeded >
| MC_dot-20240512212559_dot-03_eastus | .../MC_dot-20240512212559_dot-03_eastus | Succeeded >
| vscs-rg-13cbf76                     | .../vscs-rg-13cbf76                     | Succeeded >
| DefaultResourceGroup-EUS            | .../DefaultResourceGroup-EUS            | Succeeded >
| MC_dot-20240512212559_dot-01_eastus | .../MC_dot-20240512212559_dot-01_eastus | Succeeded >
| dotsweden                           | .../dotsweden                           | Succeeded >
| my-db-01                            | .../my-db-01                            | Succeeded >
| MC_dot-20240512212559_dot-02_eastus | .../MC_dot-20240512212559_dot-02_eastus | Succeeded >
| NetworkWatcherRG                    | .../NetworkWatcherRG                    | Succeeded >
+-------------------------------------+-----------------------------------------+----------->
(END)
```

 Press `q` to exit the table

That's it. That's Steampipe in a nutshel.

Let's talk about pros and cons.

## Steampipe Pros and Cons

<img src="/logo/clusterpedia.png" style="width:25%; float:right; padding: 10px">

Steampipe reminds me of Clusterpedia. The major difference is that Steampipe is generic while Clusterpedia is focused on data coming exclusively from Kubernetes clusters. Both approaches have their pros and cons.

Both are using PostgreSQL to store and query data. The major difference is how we query that data. When using Clusterpedia, we might not even be aware that there is a database behind it since it exposes everything through a Kubernetes-compatible API. Hence, we can use `kubectl` to query data as if all the clusters are a single cluster with resources having additional labels added to it. Clusterpedia can do that since it has a limited scope. It works only with Kubernetes resources.

![](/logo/aws.png?height=5vw&classes=inline)
![](/logo/azure.png?height=3vw&classes=inline)
![](/logo/google-cloud.png?height=3vw&classes=inline)
![](/logo/kubernetes.png?height=5vw&classes=inline)
![](/logo/slack.png?height=5vw&classes=inline)
![](/logo/openai.jpeg?height=5vw&classes=inline)

Steampipe is a completely different beast. It's scope is, essentially, everything. AWS, Azure, Google Cloud, Kubernetes, Slack, OpenAI, or almost anything else one can imagine can be a source of data. The price for such a wide scope is that we cannot work with data from any of those sources "natively". We cannot query AWS resources using `aws` CLI, nor Kubernetes resources using `kubectl`. It's all SQL.

Now, that is not a bad thing. SQL is simple yet much more effective way to query data than querying APIs directly or through CLIs. As a matter of fact, I still don't understand why we still keep etcd in Kubernetes. It should be replaced by PostgreSQL or any other SQL database.

Still, no matter how great SQL is, Steampipe has a few issues, so let's start with those.

**Cons**
* No history
* No default views
* Performance
* Jsonb
* Import rules

To begin with, Steampipe does not keep **history** of resources. We cannot go back in time to look at changes, explore issues, or whatever we might want to do with historical data. I feel that is lost opportunity since PostgreSQL is certainly capable of having a large number of records.

Next, it is irritating that there are **no default views**. If we execute `kubectl get` or `aws list` or `gcloud list` commands we always get a selected number of columns in the output. With Steampipe, if we execute `select` without specifying the columns, we get all. That's a pity since, more often than not, I tend to list resources based on certain criteria and then dig deeper. The Kubernetes plugin could have gotten the list of columns that should be displayed by default from CRDs and could have created some views or used some other mechanism to create defaults.

Now, to be clear, it's not a big deal to specify all the columns we need, but it would be nice to have default views.

Next, querying data sources can be slow since Steampipe needs to contact each API on demand something and wait for the response. It does use caching to somehow mitigate that but the cache is often obsolete. It would be better, but more obtrusive, to have some kind agents that feed the database whenever destination resources change thus keeping it always up-to-date and lightning fast to query. Anyway... **Performance** can be an issue.

Now comes the part that annoys me the most. It uses **Jsonb** fields to store data as blobs. That creates complications when specifying fields we want and, at the same time, it makes it very difficult, if not impossible, to use data from blobs in `from` and `where` statements. Hence, we cannot run queries that would, for example, retrive all unhealty Pods, at least not without performing witcraft rituals.

Filtering data that should be imported, like custom resources in Kubernetes, cannot differ from one cluster to another. Actually, we can specify resources that differ from one cluster to another, but, in those cases, Steampipe goes crazy. So, even though in theory **import rules** are flexible, in practice they are not.

Not everything is bad though. As a matter of fact, Steampipe is a great tool and many of those downsides come from the fact that it's scope is unlimited. It can work with anything, and the price to pay for that are quirks like those we just discussed.

**Pros**
* Any data source
* SQL
* Auto-complete

That leads me to be biggest advantage of Steampipe. It works with **almost everything**. We can instruct it to fetch data from almost any cloud service known to men. That's huge. That's amazing and probably outweights the downsides often caused by such a wide scope.

The next big one is **SQL**. Everyone knows SQL so there is almost nothing to learn. It's powerful, yet simple. I cannot imagine any better option for such a scope.

Given that we might deal with many tables and columns, the addition of the **auto-complete** feature is like a cherry on top of a cake. It simplifies greatly the process of writing SQL queries.

Here it goes. Here's the final verdict.

If you need a single pane of glass only for Kubernetes clusters, Clusterpedia is a better choice. However, if we need to extend the scope to other data sources, Steampipe is the right choice.

Personally, I believe that we should use Kubernetes to manage resources no matter where they are so, in my case, all the data is available in Kubernetes clusters. That's why I embraced Crossplane. Kubernetes should be the control plane for everything. With that in mind, for me, Clusterpedia is a better choice. However, I am fully aware that not everyone shares the opinion that Kubernetes should be the control plane that manages everything. For those, Steampipe is the right choice.

## Destroy

```sh
.exit

chmod +x destroy.sh

./destroy.sh

exit
```


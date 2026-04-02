# Adding mroe nodes to the cluster

### Introduction

- Sharding enables us to scale an index' data volume
  - But eventually, we'll need to add additional nodes
  - Also, replication requires at elast two nodes
- In this lecture, we'll add two more nodes to our cluster
- This lecture is optional, but why not stick around anyway?
- This approach cannot be used with cloud deployments.

Let us check the cluster's health

- Run

```
GET /_cluster/health
```

- You'll seee

```json
{
  "cluster_name": "elasticsearch",
  "status": "yellow",
  "timed_out": false,
  "number_of_nodes": 1,
  "number_of_data_nodes": 1,
  "active_primary_shards": 40,
  "active_shards": 40,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 1,
  "unassigned_primary_shards": 0,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 97.5609756097561
}
```

This is because we have one index with an unassigned replica shard. We can view this via:

- Run

```
GET
```

- You'll see:

```json
[
  {
    "index": "pages",
    "shard": "0",
    "prirep": "p",
    "state": "STARTED",
    "docs": "0",
    "store": "249b",
    "dataset": "249b",
    "ip": "127.0.0.1",
    "node": "Nvs-MacBook-Pro.local"
  },
  {
    "index": "pages",
    "shard": "0",
    "prirep": "r",
    "state": "UNASSIGNED",
    "docs": null,
    "store": null,
    "dataset": null,
    "ip": null,
    "node": null
  }
]
```

### Add new nodes

Extract the `elasticsearch` archive from earlier to two new nodes/folders:

```shell
mkdir second-node && tar -xzvf elasticsearch-9.3.2-darwin-aarch64.tar.gz -C second-node --strip-components=1
mkdir third-node && tar -xzvf elasticsearch-9.3.2-darwin-aarch64.tar.gz -C third-node --strip-components=1
```

> Note:  
> Do not copy over an existing directory, as it contains the configs for that node.

- Open the following file `second-node/config/elasticsearch.yml`:

```yaml
# ...
node.name: second-node
# ...
```

- Open the following file `third-node/config/elasticsearch.yml`:

```yaml
# ...
node.name: third-node
# ...
```

### Let's start the new nodes

We need enrollment tokens to configure our new nodes.

- Ensure your root node is running,the run

```shell
elasticsearch-9.3.2/bin/elasticsearch-create-enrollment-token --scope node
```

- From your `second-node`

```shell
second-node/bin/elasticsearch --enrollment-token <the-token>
```

Let's check the cluster's health:

- Run

```
GET /_cluster/health

```

- You'll see that we are now green:

```json
{
  "cluster_name": "elasticsearch",
  "status": "green",
  "timed_out": false,
  "number_of_nodes": 2,
  "number_of_data_nodes": 2,
  "active_primary_shards": 40,
  "active_shards": 80,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 0,
  "unassigned_primary_shards": 0,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 100
}
```

Let's check the distribution of shards:

- Run:

```
GET /_cat/shards?v&format=json
```

- You'll see:

```json
[
  {
    "index": "pages",
    "shard": "0",
    "prirep": "r",
    "state": "STARTED",
    "docs": "0",
    "store": "249b",
    "dataset": "249b",
    "ip": "127.0.0.1",
    "node": "second-node"
  },
  {
    "index": "pages",
    "shard": "0",
    "prirep": "p",
    "state": "STARTED",
    "docs": "0",
    "store": "249b",
    "dataset": "249b",
    "ip": "127.0.0.1",
    "node": "Nvs-MacBook-Pro.local"
  },
  {
    "index": ".kibana_security_session_1",
    "shard": "0",
    "prirep": "r",
    "state": "STARTED",
    "docs": "3",
    "store": "20.2kb",
    "dataset": "20.2kb",
    "ip": "127.0.0.1",
    "node": "second-node"
  },
  {
    "index": ".kibana_security_session_1",
    "shard": "0",
    "prirep": "p",
    "state": "STARTED",
    "docs": "3",
    "store": "20.2kb",
    "dataset": "20.2kb",
    "ip": "127.0.0.1",
    "node": "Nvs-MacBook-Pro.local"
  }
]
```

- Our replica shard is now `STARTED`, we are in business! The kibana shards are have also scaled up to utilize the new node.

### Let's add a third node

#### Caution

- After adding a third node, at least two nodes are required to run the cluster
  - If two nodes are lost, the cluster cannot elect a master node
- If you want to run just a single node moving forward, **don't add a third node**
  - Alternatively, you can start fresh with a single-node cluster afterwards.

#### Let's do this!

We need enrollment tokens to configure our new nodes.

- Ensure your root node is running,the run

```shell
elasticsearch-9.3.2/bin/elasticsearch-create-enrollment-token --scope node
```

- From your `third-node`

```shell
third-node/bin/elasticsearch --enrollment-token <the-token>
```

Let's check the cluster's health:

- Run

```
GET /_cat/shards?v
```

- You'll see:

| Index                               | Node                  |
| ----------------------------------- | --------------------- |
| pages                               | Nvs-MacBook-Pro.local |
| pages                               | second-node           |
| .kibana_security_solution_9.3.2_001 | second-node           |
| .kibana_security_solution_9.3.2_001 | third-node            |
| .kibana_9.3.2_001                   | Nvs-MacBook-Pro.local |
| .kibana_9.3.2_001                   | second-node           |
| .kibana_search_solution_9.3.2_001   | second-node           |
| .kibana_search_solution_9.3.2_001   | third-node            |
| .kibana_security_session_1          | Nvs-MacBook-Pro.local |
| .kibana_security_session_1          | third-node            |

- ES distributes the shards across all three nodes, even though realistically two nodes could get the job done. This is to ensure even higher availability, and we don't want the third node to be idle.

#### Let's kill one of the nodes.

- If we do a CTRL+C on one of the nodes, it is shut down gracefully.
- We want to forcefully shutdown the node, so let's kill the terminal window

If we look at the terminal of our master node:

```
[994][WARN ][o.e.c.NodeConnectionsService] [Nvs-MacBook-Pro.local] failed to connect to {third-node}{6zVLQboIRa6fYZhgPic7nQ}{EimbU9C8R2iXrMoOG_t1wg}{third-node}{127.0.0.1}{127.0.0.1:9302}{cdfhilmrstw}{9.3.2}{8000099-9060000}{ml.allocated_processors=11, ml.allocated_processors_double=11.0, ml.max_
jvm_size=2147483648, ml.config_version=12.0.0, transform.config_version=10.0.0, xpack.installed=true, ml.machine_memory=19327352832} (tried [1] times)org.elasticsearch.transport.ConnectTransportException: [third-node][127.0.0.1:9302] connect_exception
        at org.elasticsearch.server@9.3.2/org.elasticsearch.transport.TcpTransport$ChannelsConnectedListener.onFailure(TcpTransport.java:1168)
        at org.elasticsearch.server@9.3.2/org.elasticsearch.action.support.SubscribableListener$FailureResult.complete(SubscribableListener.java:435)
        at org.elasticsearch.server@9.3.2/org.elasticsearch.action.support.SubscribableListener.tryComplete(SubscribableListener.java:347)
        at org.elasticsearch.server@9.3.2/org.elasticsearch.action.support.SubscribableListener.setResult(SubscribableListener.java:372)
        at org.elasticsearch.server@9.3.2/org.elasticsearch.action.support.SubscribableListener.onFailure(SubscribableListener.java:284)

See logs for more details.

[076][INFO ][o.e.c.s.ClusterApplierService] [Nvs-MacBook-Pro.local] removed {{third-node}{6zVLQboIRa6fYZhgPic7nQ}{EimbU9C8R2iXrMoOG_t1wg}{third-node}{127.0.0.1}{127.0.0.1:9302}{cdfhilmrstw}{9.3.2}{8000099-9060000}}, term: 6, version: 384, reason: Publication{term=6, version=384}
[103][INFO ][o.e.c.r.DelayedAllocationService] [Nvs-MacBook-Pro.local] scheduling reroute for delayed shards in [59.1s] (26 delayed shards)
[109][INFO ][o.e.c.c.NodeLeftExecutor ] [Nvs-MacBook-Pro.local] node-left: [{third-node}{6zVLQboIRa6fYZhgPic7nQ}{EimbU9C8R2iXrMoOG_t1wg}{third-node}{127.0.0.1}{127.0.0.1:9302}{cdfhilmrstw}{9.3.2}{8000099-9060000}] with reason [disconnected]
[423][WARN ][o.e.c.r.a.AllocationService] [Nvs-MacBook-Pro.local] [.kibana_task_manager_9.3.2_001][0] marking unavailable shards as stale: [slWICba-RKa-bKN5vwAPQw]
[147][WARN ][o.e.c.r.a.AllocationService] [Nvs-MacBook-Pro.local] [.kibana_usage_counters_9.3.2_001][0] marking unavailable shards as stale: [iTHs5O57SWC8XzLmMPVMLA]
```

You can take note of:

```
scheduling reroute for delayed shards
```

When you

- Run

```
GET /_cat/shards?v
```

- You'll see that a number of shards are `UNASSIGNED`, which will be resolved internally by ES as they are assigned to a new node in around a minute, depending on the size of the shards.
- This is by design, as we don't want to move entire shards, which could be large, because of a temporary network failure.
- https://www.elastic.co/docs/deploy-manage/distributed-architecture/shard-allocation-relocation-recovery/delaying-allocation-when-node-leaves

After a while, we'll see we're green:

```
[2026-04-02T22:33:04,754][INFO ][o.e.c.r.a.AllocationService] [Nvs-MacBook-Pro.local] current.health="GREEN" message="Cluster health status changed from [YELLOW] to [GREEN] (reason: [shards started [[.ds-.logs-elasticsearch.deprecation-default-2026.04.02-000001][0]]])." previous.health="YELLOW" reason="shards started [[.ds-.logs-elasticsearch.deprecation-default-2026.04.02-000001][0]]"
```

And this will be reflected in our shards query

- Run

```
GET /_cat/shards?v
```

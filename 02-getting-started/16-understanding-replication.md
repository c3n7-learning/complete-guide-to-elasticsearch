# Understanding replication

## Introduction

- What happens if a node's hard drive fails? The data is lost.
- Hardware can fail at any time, so we need to handle that somehow.
- Elasticsearch supports replication for fault tolerance.
- Replication is supported natively and enabled by default.
- With many databases, setting up replication can be a pain
- Replication is _extremely_ easy with Elastisearch.

### How does replication work?

Introduction

- Replication is configured at the index level
- Replication works by creating copies of shards, referred to as _replica shards_.
- A shard that has been replicated, is called a _primary shard_.
- A primary shard and its replica shards are referred to as a _replication group_.
- Replica shards are a complete copy of a shard.
- A replica shard can serve search requests, exactly like its primary shard.
- The number of replicas can be configured at index creation

```
                           Example Index
    ╔═════════════════════════════════════════════════════════════════════╗
    ║                                                                     ║
    ║  ┌─────────────────────────────┐    ┌─────────────────────────────┐ ║
    ║  │                             │    │                             │ ║
    ║  │ ┌─────────┐ ┌─────────────┐ │    │ ┌─────────┐ ┌─────────────┐ │ ║
    ║  │ │ Primary │ │ Replica A1  │ │    │ │ Primary │ │ Replica B1  │ │ ║
    ║  │ │ Shard A │ │             │ │    │ │ Shard B │ │             │ │ ║
    ║  │ └─────────┘ └─────────────┘ │    │ └─────────┘ └─────────────┘ │ ║
    ║  │             ┌─────────────┐ │    │             ┌─────────────┐ │ ║
    ║  │             │ Replica A2  │ │    │             │ Replica B2  │ │ ║
    ║  │             │             │ │    │             │             │ │ ║
    ║  │             └─────────────┘ │    │             └─────────────┘ │ ║
    ║  │                             │    │                             │ ║
    ║  │   Replication group A       │    │   Replication group B       │ ║
    ║  └─────────────────────────────┘    └─────────────────────────────┘ ║
    ║                                                                     ║
    ╚═════════════════════════════════════════════════════════════════════╝
```

How does replication help us if a node fails and its disk is corrupted?

- Replica shards are never stored on the same node as the primary shard.

```
    Node A                           Node B
  ╔═══════════╗                   ╔═══════════╗
  ║           ║                   ║           ║
  ║ ┌───────┐ ║                   ║ ┌───────┐ ║
  ║ │Shard A│ ║                   ║ │Shard B│ ║
  ║ └───────┘ ║                   ║ └───────┘ ║
  ║           ║                   ║           ║
  ║ ┌───────┐ ║                   ║ ┌───────┐ ║
  ║ │Rep B1 │ ║                   ║ │Rep A1 │ ║
  ║ └───────┘ ║                   ║ └───────┘ ║
  ║           ║                   ║           ║
  ║ ┌───────┐ ║                   ║ ┌───────┐ ║
  ║ │Rep B2 │ ║                   ║ │Rep A2 │ ║
  ║ └───────┘ ║                   ║ └───────┘ ║
  ║           ║                   ║           ║
  ╚═══════════╝                   ╚═══════════╝
```

So:

- Replication only makes sense if one has multiple nodes
- If we have only one node, ES won't create replica, even if we configure an index to have multiple replicas.

### Choosing the number of replica shards

Depends:

- How many replica shards are ideal, depends on the use case
- E.g., is the data stored elsewhere, such as in an RDBMS?
- Is it OK for data to be unavailable while you restore it?
- For mission critical systems, downtime is not acceptable
- Replicate shards _once_ if data loss is not a disaster
- For critical systems, data should be replicated at least twice.

### Snapshots

- ES supports taking snapshots as backups
- Snapshots can be used to restore to a given point in time
- Snapshots can be taken at the index level, or for the entire cluster
- Use snapshots for backups, and replication for high availability (and performance)

### Increasing query throughput with replication

Replication has more benefits:

- It increases the throughput of our queries, as these replicas can also be queries and help relieve the load from our primary shard.

So:

- Replica shards of a replication group can serve different search requests simultaneously
  - This increases the number of requests that can be handled at the same time
- ES intelligently routes requests to the _best_ shard (more on that later)
- CPU parallelization improves performance if multiple replica shards are stored on the same node.

### A small demo

Let's create a new index:

- Run

```
PUT /pages
```

- A new index `pages` is created, with 1 shard and 1 replica shard.

```json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "pages"
}
```

Let's check the health of the cluster:

- Run:

```
GET /_cluster/health
```

- You'll see that the cluster's health is `yellow`, why?

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

Let's see how our indices look like

- Run:

```shell
GET /_cat/indices?format=json
```

- You'll see:

```json
[
  {
    "health": "yellow",
    "status": "open",
    "index": "pages",
    "uuid": "x7OV8r1RQ0CFvJ5cJjkphw",
    "pri": "1",
    "rep": "1",
    "docs.count": "0",
    "docs.deleted": "0",
    "store.size": "227b",
    "pri.store.size": "227b",
    "dataset.size": "227b"
  }
]
```

- We can see that the new index is `yellow`, because the index has a replica shard, but have only one node!
- So it is not provisioned, and is pending to be allocated, because ES has no node to allocate it to
- The index is usable (even though it is `yellow`), but we risk losning our data.

To see the number of shards:

- Run:

```shell
GET /_cat/shards?v&format=json
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

- We have two shards, the primary shard (`"prirep": "p"`), and the replica shard (`"prirep": "r"`).
- Our replica shard is `UNASSIGNED` because there's no node to send the data to.

For comparison, let's check kibana's configs:

- Run

```
GET /_cat/indices?v&expand_wildcards=all&format=json
```

- You'll see

```json
[
  {
    "health": "green",
    "status": "open",
    "index": ".kibana_security_session_1",
    "uuid": "eRZLRD5QSGGuf6JOS5xSfw",
    "pri": "1",
    "rep": "0",
    "docs.count": "1",
    "docs.deleted": "0",
    "store.size": "6.9kb",
    "pri.store.size": "6.9kb",
    "dataset.size": "6.9kb"
  }
]
```

- Most, if not all, of kibana's indices have 1 shard, the primary shard. This makes sense as the indices are quite small
- Doesn't this put as at the risk of centralization and data loss in case a node fails? Yes and no
- Kibana's indices have a config called `auto_expand_replicas`, which is set to `0-1` for the Kibana indices. So, once we add a new node, new replicas are added dynamically.
- If we have one node, we'll have 0 replica shards. If we have more than one node, we'll have 1 replica shard.

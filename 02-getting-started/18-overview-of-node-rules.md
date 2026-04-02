# Overview of node roles

## Here are the available node roles

### Master-eligible

Configuration - `node-master: true|false` :

- The node may be elected as the cluster's master node
- A master ndoe is responsible for creating and deleting indices, among others.
- A node with this role will not automatically become the master node
  - (unless there are no other master-eligible nodes)
- May be used for having dedicated master node
  - Useful for large clusters
  - If the master nodes is used for search, then we could be in trouble, as the master node might not have the resources to do it's internal work.

### Data

Configuration - `node.data: true|false`

- Enables a node to store data
- Storing data includes performing queries related to the data, such as search queries
- For relatively small clusters, this role is almost always enabled
- Useful for having dedicated master nodes
- Used as part of configuring a dedicatd master node (where you disable this role for the master-eligible nodes)

### Ingest

Configuration - `node.ingest: true|false`

- Enables a node to run ingest pipelines
- Ingest pipelines are a series of steps (processors) that are performed when indexing documents
  - Processors may manipulate documents, e.g. resolving an IP to lat/long
- Think of this as: A simplified version of Logstash, directly within Elasticsearch
  - If you are doing more complex operations, it would be wiser to use Logstash
  - Ingest pipelines can do complex operations, but still, use logstash for the complex stuff
  - For smaller operations, Ingest pipelines can help
  - Internally it's used by things like `filebeat`
  - e.g. when parsing access logs for Nginx, you could use ingest pipelines, but maybe if you have high traffic, you can explore dedicated ingest node.
- This role is mainly useful to have dedicated ingest nodes

## Machine Learning

Configuration:

- `node.ml: true|false`
- `xpack.ml.enabled: true|false`

Notes:

- `node.ml` identifies a node as a machine learning node.
  - This leats the node run ML jobs.
- `xpack.ml.enabled` enables/disables the machine learning API for the node.
- Useufl for running ML jobs that don't affect other tasks.

## Coordination

Configuration:

- `node.master: false`
- `node.data: false`
- `node.ingest: false`
- `node.ml: false`
- `xpack.ml.enabled: false`

Notes:

- Refers to the distribution of queries and the aggregation of results.
- Such a node does not search data on its own, but it delegates that to data nodes.
- Useful for coordination-only nodes (for large clusters)
  - As it can essentially be used as a load-balancer
- Configured by disabling other roles
  - There's no specific role for this, so the aforementioned is the only way.

### Voting-only

Configuration - `node.voting_only: true|false`

- Rarely used, and you almost certainly won't use it either
- A node with this role, wil participate in the voting for a new master node
- The node cannot be elected as the master node itself though
- Only used for large clusters

Let's look at our cluster

- Run:

```
GET /_cat/nodes?v&format=json
```

- You'll see:

```json
[
  {
    "ip": "127.0.0.1",
    "heap.percent": "35",
    "ram.percent": "100",
    "cpu": "34",
    "load_1m": "4.89",
    "load_5m": null,
    "load_15m": null,
    "node.role": "cdfhilmrstw",
    "master": "-",
    "name": "second-node"
  },
  {
    "ip": "127.0.0.1",
    "heap.percent": "30",
    "ram.percent": "100",
    "cpu": "34",
    "load_1m": "4.89",
    "load_5m": null,
    "load_15m": null,
    "node.role": "cdfhilmrstw",
    "master": "-",
    "name": "third-node"
  },
  {
    "ip": "127.0.0.1",
    "heap.percent": "28",
    "ram.percent": "99",
    "cpu": "34",
    "load_1m": "4.89",
    "load_5m": null,
    "load_15m": null,
    "node.role": "cdfhilmrstw",
    "master": "*",
    "name": "Nvs-MacBook-Pro.local"
  }
]
```

- `dim` means `Data`, `Ingest` and `Master`
- These are the default roles, unless configured otherwise
- Our first node is marked as master, only because we didn't have any other nodes at that time, so it had to bear that role.

## When to change node roles

- It depends
- Is almost always useful for large clusters
- Typically done when optimizing the cluster to scale the number of requests
- You will often times change other things _first_
  - e.g. the number of nodes, shards, replica shards, etc
- Better understand what hardware resourcs are used for
- Only change roles if you know what you are doing.

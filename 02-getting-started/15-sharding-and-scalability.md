# Sharding and Scalability

Introduction

- If we want to store over 1TB of data on a node with 500GB, it's not possible
- However, if we have two nodes with 500GB each, we can store the entire dataset of 1TB

How does this work? Sharding.

- Sharding is a way to divide indices into smaller pieces
- Each piece is referred to as a shard
- Sharding is done at the index level
- The main purpose is to horizontally scale the data volume

If we have an index with a size of 600GB, we could shard it as follows:

```
┌─────────────────┐                 ┌─────────────────┐
│     Node A      │                 │     Node B      │
│ (capacity:500GB)│                 │ (capacity:500GB)│
│                 │                 │                 │
│  ┌───────────┐  │                 │  ┌───────────┐  │
│  │  Shard A  │  │                 │  │  Shard B  │  │
│  └───────────┘  │                 │  └───────────┘  │
└─────────────────┘                 └─────────────────┘

                    ┌─────────────────┐
                    │      Index      │
                    │    (600 GB)     │
                    └─────────────────┘
```

### Let's dive a bit deeper...

- A shard is an independent index... kind of
- Each shard is an Apache Lucene index
- An elasticsearch index consists of one or more Lucene indices
- A shard has no predefined size; it grows as documents are added to it
- A shard may store up to two billion documents

### The purpose of sharding

- Mainly to be able to store more documents, billions
- To easier fit large indices onto nodes
- Improved performance
  - Parallelization of queries increases the throughput of an index

Remember this command? Run it in kibana

```shell
GET /_cat/nodes?v
```

You'll see

```json
[
  {
    "health": "green",
    "status": "open",
    "index": ".kibana-siem-rule-migrations-integrations",
    "uuid": "Dld2xAYxSvysbshfhf0niw",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".apm-custom-link",
    "uuid": "Wo7Q2mBYQpSRHVCna8TrHw",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  }
]
```

Right now think of `pri` as the number of shards we have

### Configuring the number of shards

- An index contains a single shard by default
- Indices in Elasticsearch < 7.0.0 were created with five shards
  - This often led to _over sharding_
- If you need to increase the number of shards, you can use the Split API
- If you need to reduce the number of shards, use the Shrink API

### How many shards are optimal?

- There is no formula that will yield a number for us
- There are many factors involved, so it _depends_
- Factors include the number of nodes and their capacity, the # of indices and their size, the # of queries, etc.

Some good convention you could with:

- Anticipate millions of documents? Consider adding a couple of shards
- Need to store some thousand documents? Stick to the default settings

### Summary

- Sharding splits indices into smaller pieces
- Sharding increases the number of documents an index can store
- Sharding makes it easier to fit large indices onto nodes
- Sharding may improve query throughput
- An index defaults to having one shard
- Add a couple of shards for large indices; otherwise use default settings

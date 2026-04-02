# Understanding the basic architecture

Introduction

- Elastic search works with nodes
- You can have more than one node running on a machine
- Each node stores a subset of your data, allowing you to store lots of data
- Ideally though, you want each node working in a segragated unit e.g. virtual machine or host.

Clusters

- A selection of nodes can be grouped into a cluster.
- Cross cluster-communication is possible, but not the norm
- You can have one cluster for search, and another for APM

How do we create a cluster

- When we started the initial node, it checked if there was a cluster it was instructed to join, or if none is found, it creates its own cluster
- An elastic search node will always have to be part of a cluster

```
╔═══════════════════════════════════════════════════╗
║                    CLUSTER                        ║
╠═══════════════════════════════════════════════════╣
║                                                   ║
║    ┌─────────────┐              ┌─────────────┐   ║
║    │    NODE A   │              │    NODE B   │   ║
║    │   (Orange)  │◄────────────►│   (Green)   │   ║
║    │             │              │             │   ║
║    └─────────────┘              └─────────────┘   ║
║           │                            │          ║
║           │          ┌─────────────┐   │          ║
║           │          │    NODE C   │   │          ║
║           └─────────►│    (Red)    │◄──┘          ║
║                      │             │              ║
║                      └─────────────┘              ║
║                                                   ║
╚═══════════════════════════════════════════════════╝
```

### Indexing documents

When you index a document, for example the following JSON

```json
{
  "name": "Bo Andersen",
  "country": "Denmark"
}
```

It is stored as:

```json
{
  "_index": "people",
  "_type": "_doc",
  "_id": "123",
  "_version": 1,
  "_seq_no": 0,
  "_primary_term": 1,
  "_source": {
    "name": "Bo Andersen",
    "country": "Denmark"
  }
}
```

### How are documents stored?

- Withing indices
- An index is a collection of documents which have similar charecteristics which are similar.
- An index can have as many documents as you want, without limits.
- Search queries are run against indices
- YOu can have a "products" index, or a "restaurants" index.

### Summary

- _Nodes_ store the data that we add to Elasticsearch
- A _cluster_ is a collection of nodes
- Data is stored as _documents_, which are JSON objects
- Documents are grouped together with _indices_

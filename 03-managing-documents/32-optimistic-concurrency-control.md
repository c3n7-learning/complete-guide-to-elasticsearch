# 32. Optimistic Concurrency Control

## Introduction

- Prevent overwriting documents inadvertently due to concurrent operations
- There are many scenarios in which this can happen, where the ordering the write operations can mess things up
  - E.g. handling concurrent visitors for a web application

Supports two users are checking out with the values {"in_stock": 6} at the same time

- Assume we have 11 items in stock
- The first thread deducts 6 from stock, and sets the value to 5
- The second thread also thinks it has 11, and sets the value to 5 as well
- There are no errors, and the system thinks all is well
- But now we are selling an item that should out of stock!

How do we solve for this?

- Previously, update operations included the `_version` returned by document retrieval.
- The update operation would then fail .g. if both send `_version: 1`, and we find that ES has `_version: 2`
- This would fail though, but was solved by primary and sequence numbers.

How do we use `primary` and `sequence` numbers?

- Do a pass which looks like
  - We retrieve the product and take note of the `_primary_term` and `_seq_no` params
  - We then do a checkout and POST while including the `if_primary_term` and `if_seq_no` params
- If the version changed, an error is thrown. We retry the entire journey again.

```
┌─────────────────────┐                              ┌───────────────┐
│ {                   │  <---- Retrieve product ---- │               │
│   "primary_term": 1,│                              │ Elasticsearch │
│   "_seq_no": 71     │  -----> Update product ----> │               │
│ }                   │                              └───────────────┘
└─────────────────────┘

POST /products/_update/100?if_primary_term=1&if_seq_no=71
```

## Demo

Retrieve the current document:

- Run:

```
GET /products/_doc/101
```

- Output:

```
{
  "_index": "products",
  "_id": "101",
  "_version": 2,
  "_seq_no": 11,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "name": "Blender",
    "price": 399,
    "in_stock": 6
  }
}
```

Now update the document:

- Run:

```
POST /products/_update/101?if_primary_term=1&if_seq_no=11
{
    "doc": {
        "in_stock": 123
    }
}
```

- Output:

```json
{
  "_index": "products",
  "_id": "101",
  "_version": 3,
  "result": "updated",
  "_shards": {
    "total": 3,
    "successful": 3,
    "failed": 0
  },
  "_seq_no": 15,
  "_primary_term": 1
}
```

- Try to rerun the POST again. You'll see the output:

```json
{
  "error": {
    "root_cause": [
      {
        "type": "version_conflict_engine_exception",
        "reason": "[101]: version conflict, required seqNo [11], primary term [1]. current document has seqNo [15] and primary term [1]",
        "index_uuid": "cQSijgusQuKDWEECZY6eaQ",
        "shard": "0",
        "index": "products"
      }
    ],
    "type": "version_conflict_engine_exception",
    "reason": "[101]: version conflict, required seqNo [11], primary term [1]. current document has seqNo [15] and primary term [1]",
    "index_uuid": "cQSijgusQuKDWEECZY6eaQ",
    "shard": "0",
    "index": "products"
  },
  "status": 409
}
```

## How do we handle failures?

Handle the situation at the application level:

- Retrieve the document again
- use the `_primary_term` and `_seq_no` for a new update request
- Remember to perform any calculations that use field values again

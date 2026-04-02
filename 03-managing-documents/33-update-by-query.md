# Update by Query

## Introduction

- We already covered how to update _one_ document at a time
- Let's now update multiple documents within a single query
  - Similary to an UPDATE query in a RDBMS
- The query uses three concepts that we have jsut covered
  - Primary terms
  - Sequence numbers
  - Optimistic concurrency control

## Demo:

Suppose someone has created multiple products, and all these products have been deducted by 1

- Let's first create the product we deleted earlier:

```
PUT /products/_doc/100
{
    "name": "Toaster",
    "price": 49,
    "in_stock": 4
}
```

Now let's do an `_update_by_query` update, we'll use a `script` to specify the operation`, and the `query` to specify the _selector_:

- Run:

```
POST /products/_update_by_query
{
    "script": {
      "source": "ctx._source.in_stock--"
    },
    "query": {
        "match_all": {}
    }
}
```

- Output:

```json
{
  "took": 349,
  "timed_out": false,
  "total": 3,
  "updated": 3,
  "deleted": 0,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1,
  "throttled_until_millis": 0,
  "failures": []
}
```

### Behind the scenes

When we run an `_update_by_query`:

1. The coordinating node takes a snapshot of the system
2. The coordinating node dispatches search queries to all the relevant shards
3. The coordinating node dispatches bulk updates to documents that matched the search queries

The operations are run sequentially

- This is so that if something fails, the query is retried up to 10 times, and this can configured.
- If the affected query is not successful, the whole query is marked as failed, and _aborted_, but not _rolled back_.
- The query is not run within a transaction as we'd expect in a traditional RDBMS. That is by design

```
                           ┌─────────┐
                           │  Cloud  │
                           │ Storage │
                           └─────────┘
                                ^
                                │ Take snapshot
                                │
                               (✓)

POST /products/_update_by_query ────────> ┌──────────────┐
                                          │ Coordinating │ ───> (✓) ───> ┌─────────────────┐
                                          │     Node     │               │ Replication     │
                                          └──────────────┘               │ group A         │
                                               │                         └─────────────────┘
                                               │
                                               ├────> (✗) ───> ┌─────────────────┐
                                               │               │ Replication     │
                                               │               │ group B         │
                                               │               └─────────────────┘
                                               │
                                               └────> (...) ───> ┌─────────────────┐
                                                                 │ Replication     │
                                                                 │ group C         │
                                                                 └─────────────────┘
```

- In the above diagram, documents in RG-C will remain in the old state, even though RG-A was okay.

### How the snapshot is used

- Prevents overwriting changes made after the snapshot was taken
  - The query may take a while to finish if updating many documents
- Each document's primary term and sequence number is used.
  - A document is only updated if the values match the ones from the snapshot
  - As you know, this is called optimistic concurrency control
- Number of version conflicts is return within the `version_conflicts` key.

If you don't want the query to abort in case of `version_conflicts`, you can update your query as follows:

```
POST /products/_update_by_query
{
    "conflicts": "proceed",
    "script": {
      "source": "ctx._source.in_stock--"
    },
    "query": {
        "match_all": {}
    }
}
```

## Verify the update query worked

Let's use a search query. We will cover search queries later:

```
POST /products/_update_by_query
{
    "conflicts": "proceed",
    "script": {
      "source": "ctx._source.in_stock--"
    },
    "query": {
        "match_all": {}
    }
}
```

- Output:

```json
{
  "took": 62,
  "timed_out": false,
  "_shards": {
    "total": 2,
    "successful": 2,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "products",
        "_id": "101",
        "_score": 1,
        "_source": {
          "price": 399,
          "name": "Blender",
          "in_stock": 122
        }
      },
      {
        "_index": "products",
        "_id": "100",
        "_score": 1,
        "_source": {
          "price": 49,
          "name": "Toaster",
          "in_stock": 3
        }
      },
      {
        "_index": "products",
        "_id": "nrr9T50Bv160fQzNvU9C",
        "_score": 1,
        "_source": {
          "price": 64,
          "name": "Coffeee Maker",
          "in_stock": 9
        }
      }
    ]
  }
}
```

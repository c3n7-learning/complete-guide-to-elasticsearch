# Batch Processing

## Introduction to the Bulk API

- You learned how to index, update, and delete documents
- Let's see how we can perform these actions on _many_ documents with a _single_ query
  - That's done with Bulk API
- The Buk API excpts data formated using the NDJSON specification

```
action_and_metadata\n
optional_source\n
action_and_metadata\n
optional_source\n
```

## Demo

- The index action essentially do an _update or create_ operation.
- The create action will fail if the document already exists

To do a bulk action:

- Run:

```
POST /_bulk
{ "index": { "_index": "products", "_id": 200 } }
{ "name": "Espresso Machine", "price": 199, "in_stock": 5 }
{ "create": { "_index": "products", "_id": 201 } }
{ "name": "Milk Frother", "price": 149, "in_stock": 14 }
```

- Output:

```
{
  "errors": false,
  "took": 0,
  "items": [
    {
      "index": {
        "_index": "products",
        "_id": "200",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 3,
          "successful": 3,
          "failed": 0
        },
        "_seq_no": 3,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "create": {
        "_index": "products",
        "_id": "201",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 3,
          "successful": 3,
          "failed": 0
        },
        "_seq_no": 4,
        "_primary_term": 1,
        "status": 201
      }
    }
  ]
}
```

Let's see if we managed to create the documents:

- Run:

```
GET /products/_search
{
    "query": {
        "match_all": {}
    }
}
```

- Output:

```json
{
  "took": 9,
  "timed_out": false,
  "_shards": {
    "total": 2,
    "successful": 2,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "products",
        "_id": "200",
        "_score": 1,
        "_source": {
          "name": "Espresso Machine",
          "price": 199,
          "in_stock": 5
        }
      },
      {
        "_index": "products",
        "_id": "201",
        "_score": 1,
        "_source": {
          "name": "Milk Frother",
          "price": 149,
          "in_stock": 14
        }
      }
    ]
  }
}
```

## Demo Update

Write the query

- You could run the following query, but we could make it better:

```
POST /_bulk
{ "update": { "_index": "products", "_id": 201 } }
{ "doc": { "price": 129 } }
{ "delete": { "_index": "products", "_id": 200} }
```

We could make it better if we're running all operations in one index

- Run:

```
POST /products/_bulk
{ "update": { "_id": 201 } }
{ "doc": { "price": 129 } }
{ "delete": { "_id": 200} }
```

- Run:

```
GET /products/_search
{
    "query": {
        "match_all": {}
    }
}
```

- Output:

```json
{
  "took": 6,
  "timed_out": false,
  "_shards": {
    "total": 2,
    "successful": 2,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "products",
        "_id": "201",
        "_score": 1,
        "_source": {
          "name": "Milk Frother",
          "price": 129,
          "in_stock": 14
        }
      }
    ]
  }
}
```

## Things to be aware of

1 of 3

- The HTTP `Content-Type` header should be set as follows
  - `Content-Type: application/x-ndjson`
  - `application/json` is accepted, but that's not the correct way
- The console tool (kibana) handles this for us
  - The ES SDKs also handle this for us
  - Using HTTP clients, we need to handle this ourselves
- You'll see how to do this in the next lecture

2 of 3

- Each line **must** end with a newline character (`\n` or `\r\n`)
- This **includes** the last line
  - In a text editor, this means that the last line should be an empty new line.
- Automatically handled with the Console tool
- Typically a script will generate the bulk file, in which case you need to handle this
- Don't literally type out `\n` or `\r\n` in a text editor

3 of 3

- A failed actions will **not** affect other actions
  - Neither will the bulk request as a whole be aborted
- The Bulk API returns detailed information about each aciton
  - Inspect the `items` key to see if a given action succeeded.
    - The order is the same as the actions within the request
  - The `errors` key conveniently tells us if any errors occured.

## When to use the Bulk API

- When you need to perform lots of write operations at the same time
  - E.g when importing data or modifying lots of data
- The Bulk API is more efficient than sending individual write requests.
  - A lot of network round trips are avoided

## Two more things...

- Routing is used to resolve a document's shard
  - Can be customized if necessary
- The Bulk API supports optimistic concurency control
  - Include `if_primary_term` and `if_seq_no` parameters within the action metadata

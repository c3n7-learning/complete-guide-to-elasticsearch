# Querying by Parent ID

- The query is called `parent_id`
  - `parent_id.type`: type of document we want returned. We add the child relation here
  - `parent_id.id`: id of the parent

Example #1

- Run:

```http
GET /department/_search
{
  "query": {
    "parent_id": {
      "type": "employee",
      "id": 1
    }
  }
}
```

Example #2:

- Run:

```
GET /department/_search
{
  "query": {
    "parent_id": {
      "type": "employee",
      "id": 2
    }
  }
}
```

- In the results, you'll see a new field called `join_field`:
  - the `_routing` field is also added; it is the ID of the parent document
  - id uses the id of the parent document to figure out which shard the document is on

```json
{
  "took": 48,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.9444616,
    "hits": [
      {
        "_index": "department",
        "_id": "4",
        "_score": 0.9444616,
        "_routing": "2",
        "_source": {
          "name": "John Doe",
          "age": 44,
          "gender": "M",
          "join_field": {
            "name": "employee",
            "parent": 2
          }
        }
      },
      {
        "_index": "department",
        "_id": "7",
        "_score": 0.9444616,
        "_routing": "2",
        "_source": {
          "name": "Jane Park",
          "age": 23,
          "gender": "F",
          "join_field": {
            "name": "employee",
            "parent": 2
          }
        }
      }
    ]
  }
}
```

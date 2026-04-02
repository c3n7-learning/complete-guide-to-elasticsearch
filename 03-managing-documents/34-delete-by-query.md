# Delete by Query

## Introduction

- We already covered how to delete one document at a time
- Let's now delete multiple documents withing a single query
- The query is very similar to the Update by Query API

To delete by query, the below will delete all documents:

- Run:

```json
POST /products/_delete_by_query
{
    "query": {
        "match_all": {}
    }
}
```

Then attempt to retreive products:

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
  "took": 13,
  "timed_out": false,
  "_shards": {
    "total": 2,
    "successful": 2,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 0,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  }
}
```

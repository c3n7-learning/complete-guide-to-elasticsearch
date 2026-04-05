# Querying Child Documents by Parent

Reference:

- https://www.elastic.co/docs/reference/query-languages/query-dsl/query-dsl-has-parent-query#_sorting_2

Introduction

- Returns child documents where the parent documents match some criteria
- This can be accomplished by `has_parent`
  - `has_parent.parent_type` - the name of the parent relationship
  - `has_parent.query` - the query to filter the parent document

Run:

```http
GET /department/_search
{
  "query": {
    "has_parent": {
      "parent_type": "department",
      "query": {
        "term": {
          "name.keyword":  "Development"
        }
      }
    }
  }
}
```

Output:

```json
{
  "took": 20,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 4,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "department",
        "_id": "3",
        "_score": 1,
        "_routing": "1",
        "_source": {
          "name": "Bo Andersen",
          "age": 28,
          "gender": "M",
          "join_field": {
            "name": "employee",
            "parent": 1
          }
        }
      }
    ]
  }
}
```

## Relevance Scoring

- The query ignores the relevance score from the matching parent documents
- The default behaviour is to add a score of 1
- We can change this by setting `score: true`

Run:

```http
GET /department/_search
{
  "query": {
    "has_parent": {
      "score": true,
      "parent_type": "department",
      "query": {
        "term": {
          "name.keyword": "Development"
        }
      }
    }
  }
}
```

Output:

```json
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 4,
      "relation": "eq"
    },
    "max_score": 1.7917595,
    "hits": [
      {
        "_index": "department",
        "_id": "3",
        "_score": 1.7917595,
        "_routing": "1",
        "_source": {
          "name": "Bo Andersen",
          "age": 28,
          "gender": "M",
          "join_field": {
            "name": "employee",
            "parent": 1
          }
        }
      }
    ]
  }
}
```

- Why is this relevant? Well, there's no guarantee that a child one have one parent
- If we used `match` queries, we could have more than one parent document
  - The default behaviour would just give us any document that matches
- By setting `score: true`, we implicitly get the best matching parent document which scored the highest

> Note:  
> Because it performs a join, the `has_parent` query is slow compared to other queries. Its performance degrades as the number of matching parent documents increases. Each `has_parent` query in a search can increase query time significantly.

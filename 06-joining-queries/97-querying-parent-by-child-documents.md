# Querying parent by child documents

- [Custom Sorting](https://www.elastic.co/docs/reference/query-languages/query-dsl/query-dsl-has-child-query#_sorting)

## Finding parents with child documents matching a `bool` query

- The query is called `has_child`
  - `has_child.type`: the child relation
  - `has_child.query`: the query to filter the child documents with

Run:

```
GET /department/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      "query": {
        "bool": {
          "must": [
            {
              "range": {
                "age": {
                  "gte": 50
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "gender.keyword": "M"
              }
            }
          ]
        }
      }
    }
  }
}
```

Output:

```
{
  "took": 6,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
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
        "_index": "department",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "Development",
          "join_field": "department"
        }
      }
    ]
  }
}
```

## Taking relevance scores into account with `score_mode`

- `score_mode`: (Optional, string) Indicates how scores for matching child documents affect the root parent document's `relevance score`

| Score mode     | Explanation                                                                    |
| -------------- | ------------------------------------------------------------------------------ |
| min            | The lowest score of matching child documents is mapped into the parent.        |
| max            | The highest score of matching child documents is mapped into the parent.       |
| sum            | The matching children's scores are summed up and mapped into the parent.       |
| avg            | The average score based on matching child documents is mapped into the parent. |
| none (default) | Scores of child documents are ignored. Default value.                          |

Run:

```http
GET /department/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      "score_mode": "sum",
      "query": {
        "bool": {
          "must": [
            {
              "range": {
                "age": {
                  "gte": 50
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "gender.keyword": "M"
              }
            }
          ]
        }
      }
    }
  }
}
```

Output:

```json
{
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.4418328,
    "hits": [
      {
        "_index": "department",
        "_id": "1",
        "_score": 1.4418328,
        "_source": {
          "name": "Development",
          "join_field": "department"
        }
      }
    ]
  }
}
```

- This is useful when we don't just want documents that match the query, but documents which match the query better

## Specifying the minimum and maximum number of children

- We use `min_children` and `max_children` for that
- We no longer get results because we specified "min_children" to 2
  - It filters out documents with child documents less than 2 in number

Run:

```
GET /department/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      "score_mode": "sum",
      "min_children": 2,
      "max_children": 5,
      "query": {
        "bool": {
          "must": [
            {
              "range": {
                "age": {
                  "gte": 50
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "gender.keyword": "M"
              }
            }
          ]
        }
      }
    }
  }
}
```

Output:

```json
{
  "took": 8,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
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

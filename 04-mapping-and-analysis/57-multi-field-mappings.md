# Multi Field Mappings

A field may be mapped in multiple ways

- A text field may be mapped as a keyword field at the same time

Let's create an index that we'll trash later:

- Every field may store 0 or more values, that's why we haven't explicitly specified `ingredients` as an array (as ES doesn't have an array type per se)

```
PUT /multi_field_test
{
    "mappings": {
        "properties": {
            "description": {
                "type": "text"
            },
            "ingredients": {
                "type": "text"
            }
        }
    }
}
```

- Let's assume we also want an index page that lists all ingredients, and how many recipes use them, on top of regular search
  - Aggregation can not work on text fields, it can work on dates/numbers, but not text
  - We'll need to use the `keyword` datatype

```
PUT /multi_field_test
{
    "mappings": {
        "properties": {
            "description": {
                "type": "text"
            },
            "ingredients": {
                "type": "text",
                "fields": {
                    "keyword": {
                        "type": "keyword"
                    }
                }
            }
        }
    }
}
```

- You might be confused that the field-mapping and the data type are both `keyword`, but it's a convention. You could name your mapping anything.

- Output:

```json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "multi_field_test"
}
```

Let's now index a document:

- Run:

```
POST /multi_field_test/_doc
{
  "description": "To make this spaghetti carbonara, you first need to...",
  "ingredients": ["Spaghetti", "Bacon", "Eggs"]
}
```

Description

| TERM      | DOCUMENT #1 |
| --------- | ----------- |
| to        | X           |
| make      | X           |
| this      | X           |
| spaghetti | X           |
| carbonara | X           |

ingredients

| TERM      | DOCUMENT #1 |
| --------- | ----------- |
| spaghetti | X           |
| bacon     | X           |
| eggs      | X           |

`ingredients.keyword`

| TERM      | DOCUMENT #1 |
| --------- | ----------- |
| Spaghetti | X           |
| Bacon     | X           |
| Eggs      | X           |

## Let's query the data

Demo #1:

- Run

```
GET /multi_field_test/_search
{
    "query": {
        "match_all": {}
    }
}
```

- Output:

```json
{
  "took": 5,
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
        "_index": "multi_field_test",
        "_id": "n7rIVJ0Bv160fQzNa0_R",
        "_score": 1,
        "_source": {
          "description": "To make this spaghetti carbonara, you first need to...",
          "ingredients": ["Spaghetti", "Bacon", "Eggs"]
        }
      }
    ]
  }
}
```

Demo #2:

- Full text search
- Run

```
GET /multi_field_test/_search
{
    "query": {
        "match": {
          "ingredients": "Spaghetti"
        }
    }
}
```

- Output:

```json
{
  "took": 4,
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
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "multi_field_test",
        "_id": "n7rIVJ0Bv160fQzNa0_R",
        "_score": 0.2876821,
        "_source": {
          "description": "To make this spaghetti carbonara, you first need to...",
          "ingredients": ["Spaghetti", "Bacon", "Eggs"]
        }
      }
    ]
  }
}
```

Demo #3:

- Exact match
- Run:

```
GET /multi_field_test/_search
{
    "query": {
        "term": {
          "ingredients.keyword": "Spaghetti"
        }
    }
}
```

- Output:

```json
{
  "took": 4,
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
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "multi_field_test",
        "_id": "n7rIVJ0Bv160fQzNa0_R",
        "_score": 0.2876821,
        "_source": {
          "description": "To make this spaghetti carbonara, you first need to...",
          "ingredients": ["Spaghetti", "Bacon", "Eggs"]
        }
      }
    ]
  }
}
```

## Clean up

```
DELETE /multi_field_test
```

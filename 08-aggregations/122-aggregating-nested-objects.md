# Aggregating nested objects

Assume we want to see the ingredient with the least needed amount:

Run:

```http
GET /recipes/_search
{
    "size": 0,
    "aggs": {
      "min_ingredient_amount": {
        "min": {
          "field": "ingredients.amount"
        }
      }
    }
}
```

Output:

```json
{
  "took": 24,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 21,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "min_ingredient_amount": {
      "value": null
    }
  }
}
```

- The aggregation produces null, which is wrong.

To aggregate nested documents, we use the `nested` aggregation

- Run:

```http
GET /recipes/_search
{
  "size": 0,
  "aggs": {
    "ingredients": {
      "nested": {
        "path": "ingredients"
      }
    }
  }
}
```

- Output:

```json
{
  "took": 11,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 21,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "ingredients": {
      "doc_count": 226
    }
  }
}
```

- Now we see some values, yay!

We can nest other aggregations that we've previously used

- Run:

````
```http
GET /recipes/_search
{
  "size": 0,
  "aggs": {
    "ingredients": {
      "nested": {
        "path": "ingredients"
      },
      "aggs": {
        "max_ingredient_amount": {
          "max": {
            "field": "ingredients.amount"
          }
        }
      }
    }
  }
}
````

- Output:

```json
{
  "took": 12,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 21,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "ingredients": {
      "doc_count": 226,
      "max_ingredient_amount": {
        "value": 950
      }
    }
  }
}
```

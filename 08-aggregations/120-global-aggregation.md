# `global` aggregation

- Even if we have included a query that filters out what the aggregation would see, `global` allows us to act as if the global query isn't there, and break out of the context
- The `global` parameter can only be placed in the top level of the aggregation context.

## Break out of the aggregation context

```http
GET /orders/_search
{
  "size": 0,
  "query": {
    "range": {
      "total_amount": {
        "gte": 100
      }
    }
  },
  "aggs": {
    "all_orders": {
      "global": { },
      "aggs": {
        "stats_amount": {
          "stats": {
            "field": "total_amount"
          }
        }
      }
    }
  }
}
```

Output:

```
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 489,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "all_orders": {
      "doc_count": 1000,
      "stats_amount": {
        "count": 1000,
        "min": 10.27,
        "max": 281.77,
        "avg": 109.20961,
        "sum": 109209.61
      }
    }
  }
}
```

- You can see min values of `10.27`, and we have 1000 stats amount total, instead of the `hits.total.value=489`

## Adding aggregation without global context:

```
GET /orders/_search
{
  "size": 0,
  "query": {
    "range": {
      "total_amount": {
        "gte": 100
      }
    }
  },
  "aggs": {
    "all_orders": {
      "global": { },
      "aggs": {
        "stats_amount": {
          "stats": {
            "field": "total_amount"
          }
        }
      }
    },
    "stats_expensive": {
      "stats": {
        "field": "total_amount"
      }
    }
  }
}
```

Output:

```json
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 489,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "all_orders": {
      "doc_count": 1000,
      "stats_amount": {
        "count": 1000,
        "min": 10.27,
        "max": 281.77,
        "avg": 109.20961,
        "sum": 109209.61
      }
    },
    "stats_expensive": {
      "count": 489,
      "min": 100.05,
      "max": 281.77,
      "avg": 157.32703476482618,
      "sum": 76932.92
    }
  }
}
```

- You can see that `hits.total.value = aggregations.stats_expensive.count = 489`

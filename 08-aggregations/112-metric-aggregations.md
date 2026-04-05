# Metric aggregations

They are classified into two (some heavy weight jargon 😂):

1. Single-Value Numeric Metric Aggregations
   - Simply output a single value, e.g. a sum of numbers
2. Multi-Value Numeric Metric Aggregations
   - Yield several values

## Calcualting statistics with `sum`, `avg`, `min`, and `max` aggregations

- Let's aggregate all documents. If we wanted, we could add a query filter
- Setting the `size` to zero because we don't need the source documents, only the aggregations
- We could use `aggregations`, but `aggs` is what we'll use

```http
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "total_sales": {
      "sum": {
        "field": "total_amount"
      }
    },
    "avg_sale": {
      "avg": {
        "field": "total_amount"
      }
    },
    "min_sale": {
      "avg": {
        "field": "total_amount"
      }
    },
    "max_sale": {
      "avg": {
        "field": "total_amount"
      }
    }
  }
}
```

## Retrieving the number of distinct values

> Definitions (google):  
> Cardinality measures the size of a set (number of elements) or the uniqueness of data values in a database column

- We'll use `cardinality` for this:
- It produces approximate numbers. Getting exact numbers would take up too much resources
  - ES does some pretty complex and near-to-accurate approximations as possible, but this is something to keep in mind
  - If we need to improve the accuracy of the approximation, we can tune that

```http
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "total_salesmen": {
      "cardinality": {
        "field": "salesman.id"
      }
    }
  }
}
```

## Retrieving the number of values

```http
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "values_count": {
      "value_count": {
        "field": "total_amount"
      }
    }
  }
}
```

## Using `stats` aggregation for common statistics

- `min`, `max`, `avg`, `sum`, `value_count` all in one:

Run:

```http
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "amount_stats": {
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
      "value": 1000,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "amount_stats": {
      "count": 1000,
      "min": 10.27,
      "max": 281.77,
      "avg": 109.20961,
      "sum": 109209.61
    }
  }
}
```

# Nested Aggregations

Buckets allow us to do cool stuff

- Each bucket can have sub-aggregations
- We can even nest bucket aggregations within bucket aggregations
- Metric aggregations contain simple single values and cannot contain nested aggregations

## Retreiving statistics for each status

- Aggregations are run based on the context they are in
- If it is at the root, it does aggregates at the root, if it is nested within a bucket, the context is constrained to what the bucket sees

```http
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status"
      },
      "aggs": {
        "status_stats": {
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
      "value": 1000,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "status_terms": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "processed",
          "doc_count": 209,
          "status_stats": {
            "count": 209,
            "min": 10.27,
            "max": 281.77,
            "avg": 109.30703349282295,
            "sum": 22845.17
          }
        },
        { "...": "..." }
      ]
    }
  }
}
```

## Narrowing down the aggregation context

Run:

```GET
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
    "status_terms": {
      "terms": {
        "field": "status"
      },
      "aggs": {
        "status_stats": {
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

```json
{
  "took": 19,
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
    "status_terms": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "pending",
          "doc_count": 110,
          "status_stats": {
            "count": 110,
            "min": 100.07,
            "max": 260.03,
            "avg": 159.29090909090908,
            "sum": 17522
          }
        }
      ]
    }
  }
}
```

- You can now see the total number of hits has reduced, and the status_stats have been affected as well

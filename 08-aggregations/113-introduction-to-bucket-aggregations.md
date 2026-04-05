# Introduction to bucket aggregations

- Instead of calculating metrics for fields
- Bucket Aggregations create sets/buckets of documents
- Instead of calculating values, Bucket aggregations create buckets/sets of documents
- Remember that counts are approximates. We'll dive deeper into this later.

## Creating a bucket for each `status` value

Run:

```
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status"
      }
    }
  }
}
```

Output:

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
          "doc_count": 209
        },
        {
          "key": "completed",
          "doc_count": 204
        },
        {
          "key": "pending",
          "doc_count": 199
        },
        {
          "key": "cancelled",
          "doc_count": 196
        },
        {
          "key": "confirmed",
          "doc_count": 192
        }
      ]
    }
  }
}
```

- `sum_other_doc_count`: ES returns the top unique terms. If you have many different terms, some of them will not appear in the results. This value is thus the sum of the terms that were not part of this response.

You can test this with:

```http
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "total_amount"
      }
    }
  }
}
```

## Including `20` terms instead of the default `10`

```http
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "total_amount",
        "size": 20
      }
    }
  }
}
```

## Aggregating documents with missing field (or `NULL`)

What of documents with missing values for the `status` field?

- We can add a default value for such with `missing`
- We won't see anything though as all our documents have a status

```
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status",
        "missing": "N/A"
      }
    }
  }
}
```

## Changing the minimum document count for a bucket to be created

- the default value for `min_doc_count` is 1.
- Now we should see the `N/A` bucket

```
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status",
        "missing": "N/A",
        "min_doc_count": 0
      }
    }
  }
}
```

## Ordering the buckets

```
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status",
        "missing": "N/A",
        "min_doc_count": 0,
        "order": {
          "_key": "asc"
        }
      }
    }
  }
}
```

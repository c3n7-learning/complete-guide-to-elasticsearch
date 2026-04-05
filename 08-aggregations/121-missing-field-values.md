# Missing field values

## Adding test documents

```
PUT /orders/_doc/1001
{
  "total_amount": 100
}

PUT /orders/_doc/1002
{
  "total_amount": 200,
  "status": null
}
```

## Aggregating documents with missing field value

Run:

```
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "orders_without_status": {
      "missing": {
        "field": "status"
      }
    }
  }
}
```

Output:

```
{
  "took": 21,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1002,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "orders_without_status": {
      "doc_count": 2
    }
  }
}
```

## Combining `missing` aggregation with other aggregations

```
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "orders_without_status": {
      "missing": {
        "field": "status"
      },
      "aggs": {
        "missing_sum": {
          "sum": {
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
      "value": 1002,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "orders_without_status": {
      "doc_count": 2,
      "missing_sum": {
        "value": 300
      }
    }
  }
}
```

## Deleting test documents

```
DELETE /orders/_doc/1001

DELETE /orders/_doc/1002
```

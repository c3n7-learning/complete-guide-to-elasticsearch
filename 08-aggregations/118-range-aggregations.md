# Range Aggregations

There are two aggregations

- `range`
- and `date_range`

## `range` aggregation

- Perhaps we want to know which orders are between 0-50, 50-100, and 100-infinity

```http
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "amount_distribution": {
      "range": {
        "field": "total_amount",
        "ranges": [
          {
            "to": 50
          },
          {
            "from": 50,
            "to": 100
          },
          {
            "from": 100
          }
        ]
      }
    }
  }
}
```

## `date_range`aggregation

Run:

```http
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "purchased_ranges": {
      "date_range": {
        "field": "purchased_at",
        "ranges": [
          {
            "from": "2016-01-01",
            "to": "2016-01-01||+6M"
          },
          {
            "from": "2016-01-01||+6M",
            "to": "2016-01-01||+1y"
          }
        ]
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
      "value": 1000,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "purchased_ranges": {
      "buckets": [
        {
          "key": "2016-01-01T00:00:00.000Z-2016-07-01T00:00:00.000Z",
          "from": 1451606400000,
          "from_as_string": "2016-01-01T00:00:00.000Z",
          "to": 1467331200000,
          "to_as_string": "2016-07-01T00:00:00.000Z",
          "doc_count": 481
        },
        {
          "key": "2016-07-01T00:00:00.000Z-2017-01-01T00:00:00.000Z",
          "from": 1467331200000,
          "from_as_string": "2016-07-01T00:00:00.000Z",
          "to": 1483228800000,
          "to_as_string": "2017-01-01T00:00:00.000Z",
          "doc_count": 519
        }
      ]
    }
  }
}
```

## Specifying the date format

```http
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "purchased_ranges": {
      "date_range": {
        "field": "purchased_at",
        "format": "yyyy-MM-dd",
        "ranges": [
          {
            "from": "2016-01-01",
            "to": "2016-01-01||+6M"
          },
          {
            "from": "2016-01-01||+6M",
            "to": "2016-01-01||+1y"
          }
        ]
      }
    }
  }
}
```

- Output:

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
      "value": 1000,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "purchased_ranges": {
      "buckets": [
        {
          "key": "2016-01-01-2016-07-01",
          "from": 1451606400000,
          "from_as_string": "2016-01-01",
          "to": 1467331200000,
          "to_as_string": "2016-07-01",
          "doc_count": 481
        },
        {
          "key": "2016-07-01-2017-01-01",
          "from": 1467331200000,
          "from_as_string": "2016-07-01",
          "to": 1483228800000,
          "to_as_string": "2017-01-01",
          "doc_count": 519
        }
      ]
    }
  }
}
```

- Instructor recommends to only do this if we need it, and do the formatting in our application if we can

## Enabling keys for the buckets

```
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "purchased_ranges": {
      "date_range": {
        "field": "purchased_at",
        "format": "yyyy-MM-dd",
        "keyed": true,
        "ranges": [
          {
            "from": "2016-01-01",
            "to": "2016-01-01||+6M"
          },
          {
            "from": "2016-01-01||+6M",
            "to": "2016-01-01||+1y"
          }
        ]
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
      "value": 1000,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "purchased_ranges": {
      "buckets": {
        "2016-01-01-2016-07-01": {
          "from": 1451606400000,
          "from_as_string": "2016-01-01",
          "to": 1467331200000,
          "to_as_string": "2016-07-01",
          "doc_count": 481
        },
        "2016-07-01-2017-01-01": {
          "from": 1467331200000,
          "from_as_string": "2016-07-01",
          "to": 1483228800000,
          "to_as_string": "2017-01-01",
          "doc_count": 519
        }
      }
    }
  }
}
```

## Defining the bucket keys

```http
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "purchased_ranges": {
      "date_range": {
        "field": "purchased_at",
        "format": "yyyy-MM-dd",
        "keyed": true,
        "ranges": [
          {
            "from": "2016-01-01",
            "to": "2016-01-01||+6M",
            "key": "first_half"
          },
          {
            "from": "2016-01-01||+6M",
            "to": "2016-01-01||+1y",
            "key": "second_half"
          }
        ]
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
    "purchased_ranges": {
      "buckets": {
        "first_half": {
          "from": 1451606400000,
          "from_as_string": "2016-01-01",
          "to": 1467331200000,
          "to_as_string": "2016-07-01",
          "doc_count": 481
        },
        "second_half": {
          "from": 1467331200000,
          "from_as_string": "2016-07-01",
          "to": 1483228800000,
          "to_as_string": "2017-01-01",
          "doc_count": 519
        }
      }
    }
  }
}
```

## Adding a sub-aggregation

```
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "purchased_ranges": {
      "date_range": {
        "field": "purchased_at",
        "format": "yyyy-MM-dd",
        "keyed": true,
        "ranges": [
          {
            "from": "2016-01-01",
            "to": "2016-01-01||+6M",
            "key": "first_half"
          },
          {
            "from": "2016-01-01||+6M",
            "to": "2016-01-01||+1y",
            "key": "second_half"
          }
        ]
      },
      "aggs": {
        "bucket_stats": {
          "stats": {
            "field": "total_amount"
          }
        }
      }
    }
  }
}
```

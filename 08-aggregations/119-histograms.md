# Histrograms

- We've worked with ranges, but we had to explicitly specify the values ahead of time
- What if we wanted to define intervals of 25, that would require us to know the min/max amounts. Even then, we would also potentially miss some values in-between
- Documents are rounded down to the nearest minimum value
  - Assume we have a document with value 60
  - We have two buckets, with 50 and 75 respectively
  - The document will be assigned to bucket #50

## Distribution of `total_amount` with interval of `25`

```http
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "amount_distribution": {
      "histogram": {
        "field": "total_amount",
        "interval": 25
      }
    }
  }
}
```

Output:

```json
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
      "value": 1000,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "amount_distribution": {
      "buckets": [
        {
          "key": 0,
          "doc_count": 42
        },
        {
          "key": 25,
          "doc_count": 122
        },
        { "...": "..." }
      ]
    }
  }
}
```

- By default, buckets are created for each interval between the min and max values, even if the document count in them might be zero

## Requiring minimum 1 document per bucket

```
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "amount_distribution": {
      "histogram": {
        "field": "total_amount",
        "interval": 25,
        "min_doc_count": 1
      }
    }
  }
}
```

## Specifying fixed bucket boundaries

- This filters out documents with a total amount of `< 100`

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
    "amount_distribution": {
      "histogram": {
        "field": "total_amount",
        "interval": 25,
        "min_doc_count": 1
      }
    }
  }
}
```

To show buckets below 100 even if they don't contain documents?

- To force buckets to be present within boundaries:
- We can use the `extended_bounds` param for this
- Take note of `min_doc_count`, to include intervals with documents gte 0

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
    "amount_distribution": {
      "histogram": {
        "field": "total_amount",
        "interval": 25,
        "min_doc_count": 0,
        "extended_bounds": {
          "min": 0,
          "max": 500
        }
      }
    }
  }
}
```

## Aggregating by month with the `date_histogram` aggregation

```
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "orders_over_time": {
      "date_histogram": {
        "field": "purchased_at",
        "calendar_interval": "month"
      }
    }
  }
}
```

[More formats are accepted such as](https://www.elastic.co/docs/reference/aggregations/search-aggregations-bucket-datehistogram-aggregation#calendar_intervals):

- minute
- hour
- day
- week
- month
- quarter
- year

## Offsets

- Both histogram and date_histogram aggregations support an offset.
  - for histogram, a numeric value is used
  - for date_histogram, a date/date-math value is used
- This can help with pagination

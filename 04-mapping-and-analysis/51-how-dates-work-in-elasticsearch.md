# How dates work in Elasticsearch

## Introduction

- Specified in one of three ways:
  - Specially formatted strings
  - Milliseconds since the epoch (long)
  - Seconds since the epoch (integer)
- Epoch refers to the 1st of January, 1970
- Custom formats are supported

## Default behaviour of `date` fields

- Three supported formats:
  - A date _without_ time
  - A date _with_ time
  - Milliseconds since the epoch(long)
- UTC timezone assumed of none is specified
- Dates must be formatted according to the [ISO8601 specification](https://en.wikipedia.org/wiki/ISO_8601)

## How `date` fields are stored

- Stored internally as milliseconds since the epoch (long)
- Any valid value that you supply at index time is converted to a long value internally
- If a date contains a TZ, it will converted to the UTC timezone
- The same conversion will happen for search queries as well

Example #1:

```
PUT /reviews/_doc/2
{
  "rating": 4.5,
  "content": "Not bad. Not bad at all!",
  "product_id": 123,
  "created_at": "2015-03-27",
  "author": {
    "first_name": "Average",
    "last_name": "Joe",
    "email": "avgjoe@example.com"
  }
}
```

Example #2:

- `Z` means we are in UTC `T` timezone (ISO 8601)

```
PUT /reviews/_doc/3
{
  "rating": 3.5,
  "content": "Could be better",
  "product_id": 123,
  "created_at": "2015-04-15T13:07:41Z",
  "author": {
    "first_name": "Spencer",
    "last_name": "Pearson",
    "email": "spearson@example.com"
  }
}
```

Example #3:

- In UTC + 1

```
PUT /reviews/_doc/4
{
  "rating": 5.0,
  "content": "Incredible!",
  "product_id": 123,
  "created_at": "2015-01-28T09:21:51+01:00",
  "author": {
    "first_name": "Adam",
    "last_name": "Jones",
    "email": "adam.jones@example.com"
  }
}
```

Example #4:

- Using Seconds from Epoch
- Don't use Unix timestamp, unless you multiply it by 1000

```
# 2015-07-04T12:01:24Z
PUT /reviews/_doc/5
{
  "rating": 4.5,
  "content": "Very useful",
  "product_id": 123,
  "created_at": 1436011284000,
  "author": {
    "first_name": "Taylor",
    "last_name": "West",
    "email": "twest@example.com"
  }
}
```

Let's query our data:

- Run:

```
GET /reviews/_search
{
    "query": {
        "match_all": {}
    }
}
```

- Output:

```json
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
      "value": 5,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "reviews",
        "_id": "1",
        "_score": 1,
        "_source": {
          "rating": 5,
          "content": "Outstanding course! Bo really taught me a lot about Elasticsearch!",
          "product_id": 123,
          "author": {
            "first_name": "John",
            "last_name": "Doe",
            "email": "johndoe123@example.com"
          }
        }
      },
      {
        "_index": "reviews",
        "_id": "2",
        "_score": 1,
        "_source": {
          "rating": 4.5,
          "content": "Not bad. Not bad at all!",
          "product_id": 123,
          "created_at": "2015-03-27",
          "author": {
            "first_name": "Average",
            "last_name": "Joe",
            "email": "avgjoe@example.com"
          }
        }
      },
      {
        "_index": "reviews",
        "_id": "3",
        "_score": 1,
        "_source": {
          "rating": 3.5,
          "content": "Could be better",
          "product_id": 123,
          "created_at": "2015-04-15T13:07:41Z",
          "author": {
            "first_name": "Spencer",
            "last_name": "Pearson",
            "email": "spearson@example.com"
          }
        }
      },
      {
        "_index": "reviews",
        "_id": "4",
        "_score": 1,
        "_source": {
          "rating": 5,
          "content": "Incredible!",
          "product_id": 123,
          "created_at": "2015-01-28T09:21:51+01:00",
          "author": {
            "first_name": "Adam",
            "last_name": "Jones",
            "email": "adam.jones@example.com"
          }
        }
      },
      {
        "_index": "reviews",
        "_id": "5",
        "_score": 1,
        "_source": {
          "rating": 4.5,
          "content": "Very useful",
          "product_id": 123,
          "created_at": 1436011284000,
          "author": {
            "first_name": "Taylor",
            "last_name": "West",
            "email": "twest@example.com"
          }
        }
      }
    ]
  }
}
```

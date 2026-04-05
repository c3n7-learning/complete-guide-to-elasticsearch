# Sorting results

## Sorting by ascending order (implicitly)

Run:

```
GET /recipes/_search
{
  "_source": false,
  "query": {
    "match_all": {  }
  },
  "sort": [
    "preparation_time_minutes"
  ]
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
      "value": 21,
      "relation": "eq"
    },
    "max_score": null,
    "hits": [
      {
        "_index": "recipes",
        "_id": "5",
        "_score": null,
        "sort": [
          8
        ]
      },
      {
        "_index": "recipes",
        "_id": "6",
        "_score": null,
        "sort": [
          10
        ]
      }
    ]
  }
}
```

- Now we ahve a new key `sort`. It contains an array of the values that the query was sorted by

## Sorting by descending order

Run:

```
GET /recipes/_search
{
  "_source": false,
  "query": {
    "match_all": {  }
  },
  "sort": [
    { "created": "desc" }
  ]
}
```

- Output:

```json
{
  "took": 10,
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
    "hits": [
      {
        "_index": "recipes",
        "_id": "12",
        "_score": null,
        "sort": [1495286127000]
      },
      {
        "_index": "recipes",
        "_id": "10",
        "_score": null,
        "sort": [1493306092000]
      }
    ]
  }
}
```

- Dates have the number of milliseconds since 1970

## Sorting by multiple fields

```
GET /recipes/_search
{
  "_source": ["preparation_time_minutes", "created"],
  "query": {
    "match_all": {  }
  },
  "sort": [
    { "preparation_time_minutes": "asc" },
    { "created": "desc" }
  ]
}
```

# The match query

- The most widely used full text query
- Matches documents that contain one or more of the specified terms
- The search term is analyzed and the result is looked up in the field's inverted index

Run:

```
GET /products/_search
{
    "query": {
        "match": {
          "name": "pasta"
        }
    }
}
```

Output:

```json
{
  "took": 74,
  "timed_out": false,
  "_shards": {
    "total": 2,
    "successful": 2,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 12,
      "relation": "eq"
    },
    "max_score": 5.286037,
    "hits": []
  }
}
```

Run:

```
GET /products/_search
{
    "query": {
        "match": {
          "name": "PASTA"
        }
    }
}
```

Output:

```json
{
  "took": 74,
  "timed_out": false,
  "_shards": {
    "total": 2,
    "successful": 2,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 12,
      "relation": "eq"
    },
    "max_score": 5.286037,
    "hits": []
  }
}
```

## Searching for multiple words

For The query:

```
GET /products/_search
{
    "query": {
        "match": {
          "name": "PASTA CHICKEN"
        }
    }
}
```

- The analyzer converts `"PASTA CHICKEN"` to `["pasta", "chicken"]`

- This is then compared against the inverted index.
  - Either of the term is matched.

| TERM        | DOCUMENT #1 |
| ----------- | ----------- |
| "spaghetti" | X           |
| "pasta"     | X           |
| "beef"      | X           |
| "chicken"   | X           |
| "spinach"   | X           |

### `operator` parameter

This matches `"pasta"` or `"chicken"`

```
GET /products/_search
{
    "query": {
        "match": {
          "name": "PASTA CHICKEN"
        }
    }
}
```

This matches `"pasta"` AND `"chicken"`

```
GET /products/_search
{
  "query": {
    "match": {
      "name": {
        "query": "PASTA CHICKEN",
        "operator": "AND"
      }
    }
  }
}
```

Output:

```json
{
  "took": 111,
  "timed_out": false,
  "_shards": {
    "total": 2,
    "successful": 2,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 7.5215597,
    "hits": [
      {
        "_index": "products",
        "_id": "67",
        "_score": 7.5215597,
        "_source": {
          "name": "Pasta - Bauletti Chicken White",
          "price": 135,
          "in_stock": 30,
          "sold": 357,
          "tags": ["Meat", "Pasta"],
          "description": "Donec odio justo, sollicitudin ut, suscipit a, feugiat et, eros.",
          "is_active": true,
          "created": "2015/03/23"
        }
      }
    ]
  }
}
```

- You can use this for searching for numbers, dates, and boolean values.
- The query is pretty forgiving

## Lecture Summary

- The `match` query is a fundamental query in ES
- Used for most full text searches
- Powerful & flexible when using advanced parameters
- Supports most data types (e.g. dates and numbers)
  - **Recommendation**: Use term level queries if you know the input value
- If the analyzer outputs multiple terms, at least one must match by default
  - This can be changed by setting the `operator` parameter to `"and"`

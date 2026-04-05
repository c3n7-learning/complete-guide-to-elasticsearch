# Searching for terms

- We'll search for tags with the value `vegetable`
- We are searching for a single value instead of a string, because in ES, every value can accept 0 or more values, effectively making every value potentially an array

Run:

```
GET /products/_search
{
    "query": {
        "term": {
            "tags.keyword": "vegetable"
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
    "total": 2,
    "successful": 2,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 0,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  }
}
```

Remember, keyword searches are case sensitive!

- Run:

```
GET /products/_search
{
  "query": {
    "term": {
      "tags.keyword": "Vegetable"
    }
  }
}
```

- Output:

```
{
  "took": 8,
  "timed_out": false,
  "_shards": {
    "total": 2,
    "successful": 2,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 69,
      "relation": "eq"
    },
    "max_score": 2.4221473,
    "hits": [
      {
        "_index": "products",
        "_id": "13",
        "_score": 2.4221473,
        "_source": {
          "name": "Radish",
          "price": 91,
          "in_stock": 13,
          "sold": 88,
          "tags": [
            "Vegetable"
          ],
          "description": "Suspendisse ornare consequat lectus.",
          "is_active": true,
          "created": "2014/09/25"
        }
      }
    ]
  }
}
```

## Searching for booleans

Run:

```
GET /products/_search
{
  "query": {
    "term": {
      "is_active": true
    }
  }
}
```

## Searching for numbers

Run:

```
GET /products/_search
{
  "query": {
    "term": {
      "in_stock": 1
    }
  }
}
```

## Searching Dates

Run:

```
GET /products/_search
{
  "query": {
    "term": {
      "created": "2007/10/14"
    }
  }
}
```

## Searching timestamps

Run:

```
GET /products/_search
{
  "query": {
    "term": {
      "created": "2007/10/14 12:34:56"
    }
  }
}
```

## Shorthand vs Explicit Syntax

So far we've been using the shorthand syntax:

```
GET /products/_search
{
  "query": {
    "term": {
      "tags.keyword": "Vegetable"
    }
  }
}
```

There is a more explicit syntax:

```
GET /products/_search
{
  "query": {
    "term": {
      "tags.keyword": {
        "value": "Vegetable"
      }
    }
  }
}
```

It comes in handy if we want to add parameters.

- For example:

```
GET /products/_search
{
  "query": {
    "term": {
      "tags.keyword": {
        "value": "vegetable",
        "case_insensitive": true
      }
    }
  }
}
```

## Searching for multiple terms

The pseudocode would look like:

```
tags.keyword CONTAINS "Soup" AND/OR "Meat"
```

Run:

```
GET /products/_search
{
  "query": {
    "terms": {
      "tags.keyword": ["Soup", "Meat"]
    }
  }
}
```

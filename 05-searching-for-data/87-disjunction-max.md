# Disjunction Max (`dis_max`)

- A document matches a `dis_max` query if atleast one of the specified queries match:
- The best matching score is used to calculate the relevance score for the document

For example:

- SQL: `SELECT * FROM products WHERE name LIKE '%vegetable%' OR tags LIKE '%vegetable%'`

```http
GET /products
{
  "query": {
    "dis_max": {
      "queries": {
        { "match": { "name": "vegetable" } },
        { "match": { "tags": "vegetable" } },
      }
    }
  }
}
```

- Assuming this result:

```
{
  "name": "Vegetable Broth", (2.7086513)
  "tags": ["Vegetable"] (5.6236019)
}
```

- The score for the document is `5.62`

## Specifying a tie breaker

You can specify a `tie_breaker`

- This is a way to reward documents matching all classes in the query
- It accepts values between 0 and 1

```
GET /products/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "name": "vegetable" } },
        { "match": { "tags": "vegetable" } }
      ],
      "tie_breaker": 0.3
    }
  }
}
```

- The score will be

```
5.6236019 + (2.7086513 * 0.3) = 6.43619729
```

## How `multi_match` works

Consider this query:

```
GET /products/_search
{
  "query": {
    "multi_match": {
      "query": "vegetable",
      "fields": ["name", "tags"]
    }
  }
}
```

We said it would be translated into:

```
GET /products/_search
{
  "query": {
    "match": {
      "name": "vegetable"
    }
  }
}

GET /products/_search
{
  "query": {
    "match": {
      "description": "vegetable"
    }
  }
}
```

Well, ES needs to combine these queries into one compound query to be able to run them:

- `dis_max` comes in handy for that

```
GET /products/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "name": "vegetable" } },
        { "match": { "tags": "vegetable" } }
      ]
    }
  }
}
```

If we specify a tie breaker:

```
GET /products/_search
{
  "query": {
    "multi_match": {
      "query": "vegetable",
      "fields": ["name", "tags"],
      "tie_breaker": 0.3
    }
  }
}

          |
          |
GET /products/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "name": "vegetable" } },
        { "match": { "tags": "vegetable" } }
      ],
      "tie_breaker": 0.3
    }
  }
}
```

## Lecture Summary

- The `dis_max` query is a compound query
  - A document matches if atleast one leaf query matches
- The best matching query clause's relevance score is used for a document's `_score`
- `tie_breaker` can be used to "reward" documents that match multiple queries
- `multi_match` queries are often translated into `dis_max` queries internally

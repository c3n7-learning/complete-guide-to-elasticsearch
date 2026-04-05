# Introduction to relevance sorting

## For term-level queries

We've explored term-level queries

For example

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

Does the document contain the `"Vegetable"` tag?

- ✅ `_score = 1.0`
- ❌ The document doesn't match

## For full-text queries though:

```
GET /products/_search
{
  "query": {
    "match": {
      "name": "pasta chicken"
    }
  }
}
```

ES first retrieves Matching documents

```json
[
  {
    "name": "Pasta Penne",
    "_score": 3.7691
  },
  {
    "name": "Chicken Breast",
    "_score": 2.53271
  },
  {
    "name": "Spinach Pasta",
    "_score": 2.19671
  },
  {
    "name": "Pasta with Chicken",
    "_score": 7.26801
  }
]
```

ES then returns thens sorts the query results by `_score`:

```json
[
  {
    "name": "Pasta with Chicken",
    "_score": 7.26801
  },
  {
    "name": "Pasta Penne",
    "_score": 3.7691
  },
  {
    "name": "Chicken Breast",
    "_score": 2.53271
  },
  {
    "name": "Spinach Pasta",
    "_score": 2.19671
  }
]
```

## Lecture Summary

- Query results are sorted descendingly by the `_score` metadata field
  - A floating point number of how well a document matches a queyr
- Documents matching term level queries are generally scored 1.0
  - Either a document matches, or it doesn't (simply filtered out)
- Full text queries are not for exact matching
  - How well a document matches is now a factor
  - The most relevant resutls are placed highest (e.g. like on Google)

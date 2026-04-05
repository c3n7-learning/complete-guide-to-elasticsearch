# Boosting Query

- The `bool` query enables us to increase relevance scores with `should`
- What if we want to _decrease_ relevance scores for some documents?
  - This can be done with the `boosting` query

Consider the following query

- `size` increases the number of documents per page

```http
GET /products/_search
{
  "size": 20,
  "query": {
    "match": {
      "name": "juice"
    }
  }
}
```

Introducing the boosting query:

- It's basic structure looks like
- The `negative_boost` is multiplied on documents which match the negative score

```http
GET /products/_search
{
  "size": 20,
  "query": {
    "boosting": {
      "positive": {

      },
      "negative": {

      },
      "negative_boost": 0.5
    }
  }
}
```

## Matching juice products, but deprioritize apple juice

- Assume we don't love apple juice

```http
GET /products/_search
{
  "size": 20,
  "query": {
    "boosting": {
      "positive": {
        "match": {
          "name": "juice"
        }
      },
      "negative": {
        "match": {
          "name": "apple"
        }
      },
      "negative_boost": 0.5
    }
  }
}
```

- ES first matches the documents matching the positive clause. Documents in the negative clause then have their relevance scores reduced.

How is `negative_boost` calculated

- Documents which only match the positive clause, have their relevance scores unmodified
- Documents which also match

## Without filtering (depriorityze everything apples)

What if i don't like apples in general? I want to see everything except apple juice!

- The positive query **must** match something.
- We can use a `match_all()` in that

```
GET /products/_search
{
  "size": 20,
  "query": {
    "boosting": {
      "positive": {
        "match_all": { }
      },
      "negative": {
        "match": {
          "name": "apple"
        }
      },
      "negative_boost": 0.5
    }
  }
}
```

## More examples

### "I Like pasta"

Boost the relevance scores for pasta products

```http
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match_all": {}
        }
      ],
      "should": [
        {
          "term": {
            "tags.keyword": "Pasta"
          }
        }
      ]
    }
  }
}
```

### "I don't like meat"

Reduce the relevance scores for meat products

```
GET /products/_search
{
  "size": 20,
  "query": {
    "boosting": {
      "positive": {
        "match_all": {}
      },
      "negative": {
        "term": {
          "tags.keyword": "Meat"
        }
      },
      "negative_boost": 0.5
    }
  }
}
```

### "Pasta products, preferably without meat"

Reduce the relevance scores for meat products

```
GET /products/_search
{
  "size": 20,
  "query": {
    "boosting": {
      "positive": {
        "term": {
          "tags.keyword": "Pasta"
        }
      },
      "negative": {
        "term": {
          "tags.keyword": "Meat"
        }
      },
      "negative_boost": 0.5
    }
  }
}
```

### "I like pasta, but not meat"

- Give me all products, preferably with the Pasta tag, preferably without the Meat tag

```
GET /products/_search
{
  "query": {
    "boosting": {
      "positive": {
        "bool": {
          "must": [
            { "match_all": {} }
          ],
          "should": [
            {
              "term": {
                "tags.keyword": "Pasta"
              }
            }
          ]
        }
      },
      "negative": {
        "term": {
          "tags.keyword": "Meat"
        }
      },
      "negative_boost": 0.5
    }
  }
}
```

## Lecture Summary

- The `bool` query can increase relevance scores with `should`
- The `boosting` query can `decrease` relevance scores with `negative`
  - Documents must match the `positive` query clause
  - Documents that match the `negative` query clause have their relevance scores decreased
  - Use `match_all` query for `positive` if you don't want to filter documents
  - Can be used with any query (including compound queries, such as `bool`)

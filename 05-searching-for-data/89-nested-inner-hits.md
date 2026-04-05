# Nested Inner Hits

## Introduction

- With the `nested` query, matches are "root documents"
  - e.g. recipes when searching for ingredients
- Sometimes we might want to know what nested object matched insted of just something
- Nested inner hits tell us which nested object(s) matched the query
  - e.g. which ingredient(s) matched in a recipe

Use the param `inner_hits` parameter for this:

```
GET /recipes/_search
{
  "query": {
    "nested": {
      "path": "ingredients",
      "inner_hits": { },
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "ingredients.name": "parmesan"
              }
            },
            {
              "range": {
                "ingredients.amount": {
                  "gte": 100
                }
              }
            }
          ]
        }
      }
    }
  }
}
```

- Now we see this:

```
{
  "_index": "recipes",
  "_id": "21",
  "_score": 4.4102354,
  "_source": {},
  "inner_hits": {
    "ingredients": {
      "hits": {
        "total": {
          "value": 1,
          "relation": "eq"
        },
        "max_score": 4.4102354,
        "hits": [
          {
            "_index": "recipes",
            "_id": "21",
            "_nested": {
              "field": "ingredients",
              "offset": 1
            },
            "_score": 4.4102354,
            "_source": {
              "name": "Grated Parmesan cheese",
              "amount": 175,
              "unit": "grams"
            }
          }
        ]
      }
    }
  }
}
```

Structure of inner hits

- The `"inner_hits.ingredients"` path by default has the name of the nested path specified in the query
- The `ingredients.hits.*._nested` contains information about the nested object that was matched
  - The `offest` key specifies as the nested object's offset key in the documents `_source.ingredients` object. Think of this as an array index.
  - _array_ indices are counted from 0

Structure using dot notation

```
hits.hits[0]._source: { ... }
hits.hits[0].inner_hits.ingredients.hits.hits[0]._index: "recipes"
hits.hits[0].inner_hits.ingredients.hits.hits[0]._type: "_doc"
hits.hits[0].inner_hits.ingredients.hits.hits[0]._id: "21"
hits.hits[0].inner_hits.ingredients.hits.hits[0]._score: 4.4102354
hits.hits[0].inner_hits.ingredients.hits.hits[0]._nested.field: "ingredients"
hits.hits[0].inner_hits.ingredients.hits.hits[0]._nested.offset: 1
hits.hits[0].inner_hits.ingredients.hits.hits[0]._source.amount: 175
hits.hits[0].inner_hits.ingredients.hits.hits[0]._source.unit: "grams"
hits.hits[0].inner_hits.ingredients.hits.hits[0]._source.name: "Grated Parmesan cheese"
```

## Parameters

- `name` - change the key that appears directly within the inner_hits object. It defaults to the path specified in the nested query
- `size` - how many inner hits we want to be returned for each matching document. Defaults to 3, sorted by relevance scores

```
GET /recipes/_search
{
  "query": {
    "nested": {
      "path": "ingredients",
      "inner_hits": {
        "name": "my_hits",
        "size": 10
      },
      "query": {
        "bool": {
          "must": [
            { "match": { "ingredients.name": "parmesan" } },
            { "range": { "ingredients.amount": { "gte": 100 } } }
          ]
        }
      }
    }
  }
}
```

Outputs:

```
{
  "hits": {
    ...
    "hits": [
      {
        ...
        "inner_hits": {
          "my_hits": {
            "hits": {
              ...
            }
          }
        }
      }
    ]
  }
}
```

## Lecture Summary

- Nested inner hits tell us which nested object(s) matched a query
  - e.g. which ingredient(s) matched in a recipe
- Without inner hits, we only know that _something_ matched
- Simply add the `inner_hits` parameter to the `nested` query
  - Supply the `{}` as the value for the default behavior
  - Information about the matched nested object(s) is added to search results
  - Use the `offset` key to find each object's position within `_source`
  - Customize results with the `name` and `size` parameters

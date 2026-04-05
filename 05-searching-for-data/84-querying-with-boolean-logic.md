# Querying with Boolean Logic

- https://www.elastic.co/docs/reference/query-languages/query-dsl/query-dsl-bool-query

## `must`

Query clauses added within the `must` occurrence type are required to match

- SQL: `SELECT * FROM products WHERE tags IN ("Alcohol")`

```http
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "tags.keyword": "Alcohol"
          }
        }
      ]
    }
  }
}
```

## `must_not`

Query classes added within the `must_not` occurence type are required to _not_ match

- SQL: `SELECT \* FROM products WHERE tags IN ("Alcohol") AND tags NOT IN ("Wine")

```http
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "tags.keyword": "Alcohol"
          }
        }
      ],
      "must_not": [
        {
          "term": {
            "tags.keyword": "Wine"
          }
        }
      ]
    }
  }
}
```

## `should`

Matching query classes within the `should` occurence type boost a matching document's relevance score:

- They are not required to match, if they do, they'll boost the relevance score.
- The query below will favour results with beer, then the ones that don't will follow.

```http
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "tags.keyword": "Alcohol"
          }
        }
      ],
      "must_not": [
        {
          "term": {
            "tags.keyword": "Wine"
          }
        }
      ],
      "should": [
        {
          "term": {
            "tags.keyword": "Beer"
          }
        }
      ]
    }
  }
}
```

An example with a few more `should` query clauses:

- Documents with 'Beer' in the name and description will be given a boosted relevance score

```http
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "tags.keyword": "Alcohol"
          }
        }
      ],
      "must_not": [
        {
          "term": {
            "tags.keyword": "Wine"
          }
        }
      ],
      "should": [
        {
          "term": {
            "tags.keyword": "Beer"
          }
        },
        {
          "match": {
            "name": "beer"
          }
        },
        {
          "match": {
            "description": "Beer"
          }
        }
      ]
    }
  }
}
```

We could visualize the scoring to be something similar to what's shown below:

| Document    | Name (1)                 | Tags (2)                        | Score 1   | Score 2  | Score 3   | Score 4 | Total Score |
| ----------- | ------------------------ | ------------------------------- | --------- | -------- | --------- | ------- | ----------- |
| Document #1 | Beer - Corona            | ["Alcohol", "Beverage", "Beer"] | 1.9903715 | 5.450562 | 5.9541097 | N/A     | 13.395043   |
| Document #2 | Heineken                 | ["Alcohol", "Beverage", "Beer"] | 2.067343  | 4.760324 | N/A       | N/A     | 6.827667    |
| Document #3 | Schnapps - Peach Walkers | ["Alcohol", "Beverage"]         | 2.067343  | N/A      | N/A       | N/A     | 2.067343    |

### Important things about `should`

Since only `should` query clauses are specified, at lest one of them must match:

- If a `bool` query only contains should clauses, **at least one must match**
- Useful if you just want `something` to match and reward matching documents
  - If nothing were required to match, we would get irrelevant results
- If a query clause exists for `must`, `must_not` or `filter`, no `should` clause is required to match
  - Any `should` clauses are only used to boost relevance scores

```http
GET /products/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "tags.keyword": "Beer"
          }
        },
        {
          "match": {
            "name": "beer"
          }
        }
      ]
    }
  }
}
```

Since a `must` query claue is specified, all of the `should` query clauses are optional. They are therefore only used to boost the relevance scores of matching documents:

```http
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "tags.keyword": "Alcohol"
          }
        }
      ],
      "should": [
        {
          "term": {
            "tags.keyword": "Beer"
          }
        },
        {
          "match": {
            "name": "beer"
          }
        }
      ]
    }
  }
}
```

### `minimum_should_match`

This shown above behaviour can be configured with the `minimum_should_match` parameter as follows:

- The parameter supports more complex behaviour, e.g. percentages

```http
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "tags.keyword": "Alcohol"
          }
        }
      ],
      "should": [
        {
          "term": {
            "tags.keyword": "Beer"
          }
        },
        {
          "match": {
            "name": "beer"
          }
        }
      ],
      "minimum_should_match": 1
    }
  }
}
```

## `filter`

- Query clauses defined within the `filter` occurence type must match.
- This is similar to the `must` occurence type.
- The difference is that `filter` query clauses do not affect relevance scores and may be cached.
  - This improves the performance of the query
  - Query results can be cached and reused
- This makes sense for queries in which relevance scores don't matter (e.g. term queries)

```http
GET /products/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "tags.keyword": "Alcohol"
          }
        }
      ]
    }
  }
}
```

### Example #1

SQL: `SELECT * FROM products WHERE (tags IN ("Beer") OR name LIKE '%Beer%') AND in_stock <= 100`

#### Variation #1:

- We put the stock level filter because it shouldn't have an effect on relevance scoring
- We put the 'should' query in a nested `bool`, so as to make them match at least 1, and have them mandatory (i.e. not optional) (i.e. at least one must match)

```http
GET /products/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "range": {
            "in_stock": {
              "lte": 100
            }
          }
        }
      ],
      "must": [
        {
          "bool": {
            "should": [
              {
                "term": {
                  "tags.keyword": "Beer"
                }
              },
              {
                "term": {
                  "name": "Beer"
                }
              }
            ]
          }
        }
      ]
    }
  }
}
```

#### Variation #2:

- We could rely on the `minimum_should_match` parameter, which would make at least one of the `should` clauses required to match.

```http
GET /products/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "range": {
            "in_stock": {
              "lte": 100
            }
          }
        }
      ],
      "should": [
        {
          "term": {
            "tags.keyword": "Beer"
          }
        },
        {
          "match": {
            "name": "Beer"
          }
        }
      ],
      "minimum_should_match": 1
    }
  }
}
```

## Example #2:

SQL: `SELECT * FROM products WHERE tags IN ("Beer") AND (name LIKE '%Beer%' OR description LIKE '%Beer%') AND in_stock <= 100`

#### Variation #1:

```http
GET /products/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "range": {
            "in_stock": {
              "lte": 100
            }
          }
        },
        {
          "term": {
            "tags.keyword": "Beer"
          }
        }
      ],
      "should": [
        {
          "match": {
            "name": "Beer"
          }
        },
        {
          "match": {
            "description": "Beer"
          }
        }
      ],
      "minimum_should_match": 1
    }
  }
}
```

#### Variation #2:

- The should clause above could be replaced with a multi-match query
- This query differs slightly though in terms of relevance scoring

```http
GET /products/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "range": {
            "in_stock": {
              "lte": 100
            }
          }
        },
        {
          "term": {
            "tags.keyword": "Beer"
          }
        }
      ],
      "must": [
        {
          "multi_match": {
            "query": "Beer",
            "fields": [
              "name",
              "description"
            ]
          }
        }
      ]
    }
  }
}
```

## Occurence Types

- `must`: Query clauses are required tomatch and will contribute to relevance scores
- `filter`: Query classes are required to match, but will not contribute to relevance scores. Query classes may therefore be cached for improved performance
- `must_not`: Query classes must **not** match and do not affect relevance scoring. Query classes may therefore be cached for improved performance
- `should`: Query classes **should** match. Relevance scores of matching documents are boosted for each matching query clause. Behaviour can be adjusted with `minimum_should_match`

Here's a summary

| Occurrence type | Required to match? | Affects relevance scores? | Can be cached? |
| --------------- | ------------------ | ------------------------- | -------------- |
| must            | Yes                | Yes                       | No             |
| filter          | Yes                | No                        | Yes            |
| must_not        | No                 | No                        | Yes            |
| should          | Conditional        | Yes                       | No             |

## The `match` query, revisited

- `match` queries are actually translated to `bool` queries under the hood

### Single term

This query:

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

Is translated to:

- Why a term-level query for this though! It's because this bool query output is based on the analysis process.

```
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "name": "pasta"
          }
        }
      ]
    }
  }
}
```

- The inverted index has terms that have been analyzed. So now we are comparing values that are in the same format

| Term      | Document #1 |
| --------- | ----------- |
| "chicken" | X           |
| "pasta"   | X           |
| "with"    | X           |

### Multiple terms (or operator)

This query:

```http
GET /products/_search
{
  "query": {
    "match": {
      "name": "PASTA CHICKEN"
    }
  }
}
```

Is translated to:

- Since match queries don't require both terms to be present in the document, the `should` occurrence type makes sense.

```http
GET /products/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "name": "pasta"
          }
        },
        {
          "term": {
            "name": "chicken"
          }
        }
      ]
    }
  }
}
```

### Multiple terms (and operator)

This query:

```http
GET /products/_search
{
  "query": {
    "match": {
      "name": {
        "query": "PASTA CHICKEN",
        "operator": "and"
      }
    }
  }
}
```

Is translated to:

- Since now we require both terms to be present in the document, the `must` occurrence type makes sense.

```http
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "name": "pasta"
          }
        },
        {
          "term": {
            "name": "chicken"
          }
        }
      ]
    }
  }
}
```

> Note:  
> This is an oversimplification. It does more advanced stuff. This is just a simple look into how things work under the hood.

# Fuzzy Match Query (handling typos)

Default (no typo)

- Run:

```
GET /products/_search
{
  "query": {
    "match": {
      "name": {
        "query": "lobster"
      }
    }
  }
}
```

- Output:

```json
{
  "hits": {
    "total": {
      "value": 5,
      "relation": "eq"
    },
    "max_score": 5.677772,
    "hits": []
  }
}
```

With typos:

- Run:

```
GET /products/_search
{
  "query": {
    "match": {
      "name": {
        "query": "l0bster"
      }
    }
  }
}
```

- Output:

```json
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
      "value": 0,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  }
}
```

## Searching with `fuzziness` se to `auto`

With typo:

- Run:

```
GET /products/_search
{
  "query": {
    "match": {
      "name": {
        "query": "l0bster",
        "fuzziness": "auto"
      }
    }
  }
}
```

- Output:

```json
{
  "hits": {
    "total": {
      "value": 5,
      "relation": "eq"
    },
    "max_score": 4.866661,
    "hits": []
  }
}
```

Without typo:

- Run:

```
GET /products/_search
{
  "query": {
    "match": {
      "name": {
        "query": "lobster",
        "fuzziness": "auto"
      }
    }
  }
}
```

- Output:
  - We see an additional document with the text `Oyster`. Why? (edit distance of 2)

```json
{
  "hits": {
    "total": {
      "value": 7,
      "relation": "eq"
    },
    "max_score": 5.677772,
    "hits": []
  }
}
```

How does it work?

- Fuzziness is implemented by calculating a so-called Levenshtein Distance
  - Think of it as an `edit` distance
- Assume the text `lobster` and `l0bster`
  - The edit distance is the minimum number of single character edits to to change one word to another
  - This could be addition, deletion, substitution
  - For this case, the value is 1
  - With `Levenshtein` distance, we are not doing our math same as `slop` distance

Automatic Fuziness

- the maximum edit distance is 2
- research has shown that 80% of human errors can be corrected with an edit distance of 1
- so, edit distance of 2 is overkill and should cover a lot
- higher fuzziness values
  - reduce performance
  - make us have unexpected results, results would be all over the place
  - that's why leaving `auto` as the algorithm is probably a good idea

| TERM LENGTH | MAXIMUM EDIT DISTANCE |
| ----------- | --------------------- |
| 1-2         | 0                     |
| 3-5         | 1                     |
| >5          | 2                     |

## Fuzziness is per term (and specifying an integer)

- Fuziness `edit` distance of 1 is applied to each term
- So, for the value, each term is afforded 1 edit distance, giving us a total of 2 edit distance

```
GET /products/_search
{
  "query": {
    "match": {
      "name": {
        "query": "l0bster love",
        "operator": "and"
        "fuzziness": 1
      }
    }
  }
}
```

Output:

```json
{
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 9.843444,
    "hits": [
      {
        "_index": "products",
        "_id": "19",
        "_score": 9.843444,
        "_ignored": ["description.keyword"],
        "_source": {
          "name": "Lobster - Live"
        }
      }
    ]
  }
}
```

## Switching letters around with transpositions

- A guy named Frederick Damerau expanded the Levenshtein algorithm to support transpositions
- Transpositions count as a single edit

```
AB -> BA
```

Run:

```
GET /products/_search
{
  "query": {
    "match": {
      "name": {
        "query": "lvie",
        "fuzziness": 1
      }
    }
  }
}
```

Output:

```json
{
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 4.976783,
    "hits": [
      {
        "_index": "products",
        "_id": "19",
        "_score": 4.976783,
        "_ignored": ["description.keyword"],
        "_source": {
          "name": "Lobster - Live"
        }
      }
    ]
  }
}
```

## Disabling transpositions

- Without transpositions, two edits would be needed

```
GET /products/_search
{
  "query": {
    "match": {
      "name": {
        "query": "lvie",
        "fuzziness": 1,
        "fuzzy_transpositions": false
      }
    }
  }
}
```

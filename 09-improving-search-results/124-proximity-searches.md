# Proximity searches

- When searching for phrases, each word in the phrase has to match
- That's very strict! We might want to relax that constraint a bit
- We can allow a number of terms to be in between the terms we expect
- Instead of the sequence being in the order of `n+1`, we can allow it to be `n+2` or `n+5`

## Adding test documents

```
PUT /proximity/_doc/1
{
  "title": "Spicy Sauce"
}

PUT /proximity/_doc/2
{
  "title": "Spicy Tomato Sauce"
}

PUT /proximity/_doc/3
{
  "title": "Spicy Tomato and Garlic Sauce"
}

PUT /proximity/_doc/4
{
  "title": "Tomato Sauce (spicy)"
}

PUT /proximity/_doc/5
{
  "title": "Spicy and very delicious Tomato Sauce"
}
```

## Default behaviour

Run:

```
GET /proximity/_search
{
  "query": {
    "match_phrase": {
      "title": "spicy sauce"
    }
  }
}
```

Output:

```
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0.21585016,
    "hits": [
      {
        "_index": "proximity",
        "_id": "1",
        "_score": 0.21585016,
        "_source": {
          "title": "Spicy Sauce"
        }
      }
    ]
  }
}
```

## Adding the `slop` parameter to a `match_phrase` query

- Instead of n+1, we can work with N+(1+slop) as the max
- We are doing a search within a proximity of 1

Run:

```
GET /proximity/_search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "spicy sauce",
        "slop": 1
      }
    }
  }
}
```

Output:

```json
{
  "took": 18,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.21585016,
    "hits": [
      {
        "_index": "proximity",
        "_id": "1",
        "_score": 0.21585016,
        "_source": {
          "title": "Spicy Sauce"
        }
      },
      {
        "_index": "proximity",
        "_id": "2",
        "_score": 0.12672737,
        "_source": {
          "title": "Spicy Tomato Sauce"
        }
      }
    ]
  }
}
```

With slop value of 2:

- Instead of n+1, we can work with `N+(1+slop) = N+1+2 = N+3` as the max:
- With `slop`, we allow the `edit` distance to not exceed the slop distance. So the sequence doesn't have to match, as long as the `edit` distance is not exceeded.

```
GET /proximity/_search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "spicy sauce",
        "slop": 2
      }
    }
  }
}
```

Output:

```json
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": 0.21585016,
    "hits": [
      {
        "_index": "proximity",
        "_id": "1",
        "_score": 0.21585016,
        "_source": {
          "title": "Spicy Sauce"
        }
      },
      {
        "_index": "proximity",
        "_id": "2",
        "_score": 0.12672737,
        "_source": {
          "title": "Spicy Tomato Sauce"
        }
      },
      {
        "_index": "proximity",
        "_id": "4",
        "_score": 0.094962806,
        "_source": {
          "title": "Tomato Sauce (spicy)"
        }
      }
    ]
  }
}
```

- `slop` is the maximum `edit` distance for all of the terms
- The more the terms, the higher your slop should be
- It's really a case-specific param that you can tweak to match your needs
- A higher `edit` distance is incurred when moving terms around
  - An edit distance of 2 is needed, for example, for this example below

|          | Position 1 | Position 2   | Position 3 |
| -------- | ---------- | ------------ | ---------- |
| Document | tomato     | sauce        | spicy      |
| Query    | spicy      | sauce        |            |
| Slop 1   | ———►       | spicy\|sauce |            |
| Slop 2   |            | sauce        | ———► spicy |

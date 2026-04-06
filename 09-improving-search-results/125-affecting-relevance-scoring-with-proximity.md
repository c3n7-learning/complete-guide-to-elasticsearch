# Affecting Relevance Scoring with Proximity

Consider these results we got before

```json
[
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
```

- Results which match the sequence better get a higher score
- The closer the proximity, the higher the relevance score
  - The lower the edit-distance, the higher the relevance score
- This is an over-simplification though

## A simple `match` query within a `bool` query

Run:

```
GET /proximity/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": {
              "query": "spicy sauce"
            }
          }
        }
      ]
    }
  }
}
```

- Remember that the default for `bool` for `match` is `or`. So both terms are not required for a document to match. The more the terms, the higher the score.
- But we give up on `edit` distance math, and the sequence could be off

## Boosting relevance based on proximity

- If the `match_phrase` is matched, the relevance score for the document is boosted. You can watch the `_score` for various documents before and after

```
GET /proximity/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": {
              "query": "spicy sauce"
            }
          }
        }
      ],
      "should": [
        {
          "match_phrase": {
            "title": {
              "query": "spicy sauce"
            }
          }
        }
      ]
    }
  }
}
```

## Adding the `slop` parameter

- Let's allow a maximum edit distance of `5`

```
GET /proximity/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": {
              "query": "spicy sauce"
            }
          }
        }
      ],
      "should": [
        {
          "match_phrase": {
            "title": {
              "query": "spicy sauce",
              "slop": 5
            }
          }
        }
      ]
    }
  }
}
```

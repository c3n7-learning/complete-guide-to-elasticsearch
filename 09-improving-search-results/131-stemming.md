# Stemming

## Creating a test index

```
PUT /stemming_test
{
  "settings": {
    "analysis": {
      "filter": {
        "synonym_test": {
          "type": "synonym",
          "synonyms": [
            "firm => company",
            "love, enjoy"
          ]
        },
        "stemmer_test" : {
          "type" : "stemmer",
          "name" : "english"
        }
      },
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "synonym_test",
            "stemmer_test"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "description": {
        "type": "text",
        "analyzer": "my_analyzer"
      }
    }
  }
}
```

## Adding a test document

```
PUT /stemming_test/_doc/1
{
  "description": "I love working for my firm!"
}
```

## Matching the document with the base word (`work`)

```
GET /stemming_test/_search
{
  "query": {
    "match": {
      "description": "enjoy work"
    }
  }
}
```

Output:

```json
{
  "took": 34,
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
    "max_score": 0.7176595,
    "hits": [
      {
        "_index": "stemming_test",
        "_id": "1",
        "_score": 0.7176595,
        "_source": {
          "description": "I love working for my firm!"
        }
      }
    ]
  }
}
```

## The query is stemmed, so the document still matches

```
GET /stemming_test/_search
{
  "query": {
    "match": {
      "description": "love working"
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
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0.7176595,
    "hits": [
      {
        "_index": "stemming_test",
        "_id": "1",
        "_score": 0.7176595,
        "_source": {
          "description": "I love working for my firm!"
        }
      }
    ]
  }
}
```

Visualization:

```
┌─────────────┐   "working"   ┌─────────────┐     "work"      ┌─────────────┐
│             │ ─────────────>│             │ ───────────────>│             │
│ "working"   │               │  ANALYSIS   │                 │   "work"    │
│             │               │ (stemming)  │                 │     ✓       │
└─────────────┘               └─────────────┘                 └─────────────┘

MATCH QUERY                        ANALYSIS                    INVERTED INDEX
```

## Synonyms and stemmed words are still highlighted

```
GET /stemming_test/_search
{
  "query": {
    "match": {
      "description": "enjoy work"
    }
  },
  "highlight": {
    "fields": {
      "description": {}
    }
  }
}
```

Output:

```json
{
  "took": 57,
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
    "max_score": 0.7176595,
    "hits": [
      {
        "_index": "stemming_test",
        "_id": "1",
        "_score": 0.7176595,
        "_source": {
          "description": "I love working for my firm!"
        },
        "highlight": {
          "description": ["I <em>love</em> <em>working</em> for my firm!"]
        }
      }
    ]
  }
}
```

- Even though stemming removes 'ing' from 'working', the full term is still highlighted

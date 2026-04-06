# Adding Synonyms

## Creating an index with custom analyzer

```
PUT /synonyms
{
  "settings": {
    "analysis": {
      "filter": {
        "synonym_test": {
          "type": "synonym",
          "synonyms": [
            "awful => terrible",
            "awesome => great, super",
            "elasticsearch, logstash, kibana => elk",
            "weird, strange"
          ]
        }
      },
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "synonym_test"
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

- `awful => terrible` - the term "awful" should be replaced with the term "terrible"
- `awesome => great, super` - replace the term "awesome" with multiple terms, i.e. "great", "super"
- `elasticsearch, logtstash, kibana => elk` - replace multiple terms (elasticsearch, logtstash, kibana) with one term (elk)
- `weird, strange` - both of the terms will be placed at the same position (mutual association)

Good to note:

- The order of filters is important
- Note that we specified the `lowercase` filter to come before the `synonym_test` filter.
- That is why we define our synonyms in lowercase as well, because we want all the terms to be converted to lowercase first, then the synonyms to be appliede
- We could have it as `lowercase -> synonym_test ->lowercase` if we wanted, but that doesn't make much sense does it.
- Just remember, ordering is important!
  - e.g., we could probably want to do synonyms before stemming

#### `awful => terrible`

```
┌─────────────┐    "awful"     ┌─────────────┐    "terrible"   ┌─────────────┐
│             │ ──────────────>│             │ ───────────────>│             │
│  "awful"    │                │  ANALYSIS   │                 │ "terrible"  │
│             │                │             │                 │     ✓       │
└─────────────┘                └─────────────┘                 └─────────────┘

MATCH QUERY                       ANALYSIS                    INVERTED INDEX
```

> Note  
> ⚠️ Remember, term level queries are not analyzed, so don't try to combine term level queries with synonyms
> In theory you can, but it wouldn't match so well, because the term "awful" has not been stored in our inverted index

```
┌─────────────┐    "awful"       ┌─────────────┐
│             │ ────────────────>│             │
│  "awful"    │                  │ "terrible"  │
│             │                  │     ✗       │
└─────────────┘                  └─────────────┘

TERM QUERY                       INVERTED INDEX
```

#### `awesome => great, super`

"You are my friend, awesome"

- The replacement words are both stored in the inverted index. Each position may contain one or more words

| Position | Term           |
| -------- | -------------- |
| 1        | you            |
| 2        | my             |
| 3        | friend         |
| 4        | are            |
| 5        | great \| super |

## Testing the analyzer (with synonyms)

Example #1:

- Run:

```
POST /synonyms/_analyze
{
  "analyzer": "my_analyzer",
  "text": "awesome"
}
```

- Output:

```json
{
  "tokens": [
    {
      "token": "great",
      "start_offset": 0,
      "end_offset": 7,
      "type": "SYNONYM",
      "position": 0
    },
    {
      "token": "super",
      "start_offset": 0,
      "end_offset": 7,
      "type": "SYNONYM",
      "position": 0
    }
  ]
}
```

Example #2:

- Run:

```
POST /synonyms/_analyze
{
  "analyzer": "my_analyzer",
  "text": "Elasticsearch"
}
```

- Output:

```json
{
  "tokens": [
    {
      "token": "elk",
      "start_offset": 0,
      "end_offset": 13,
      "type": "SYNONYM",
      "position": 0
    }
  ]
}
```

Example #3:

- Run:

```
POST /synonyms/_analyze
{
  "analyzer": "my_analyzer",
  "text": "weird"
}
```

- Output:

```json
{
  "tokens": [
    {
      "token": "weird",
      "start_offset": 0,
      "end_offset": 5,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "strange",
      "start_offset": 0,
      "end_offset": 5,
      "type": "SYNONYM",
      "position": 0
    }
  ]
}
```

Example #4:

- Run:

```
POST /synonyms/_analyze
{
  "analyzer": "my_analyzer",
  "text": "Elasticsearch is awesome, but can also seem weird sometimes."
}
```

- Output:

```json
{
  "tokens": [
    {
      "token": "elk",
      "start_offset": 0,
      "end_offset": 13,
      "type": "SYNONYM",
      "position": 0
    },
    {
      "token": "is",
      "start_offset": 14,
      "end_offset": 16,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "great",
      "start_offset": 17,
      "end_offset": 24,
      "type": "SYNONYM",
      "position": 2
    },
    {
      "token": "super",
      "start_offset": 17,
      "end_offset": 24,
      "type": "SYNONYM",
      "position": 2
    },
    {
      "token": "but",
      "start_offset": 26,
      "end_offset": 29,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "can",
      "start_offset": 30,
      "end_offset": 33,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "also",
      "start_offset": 34,
      "end_offset": 38,
      "type": "<ALPHANUM>",
      "position": 5
    },
    {
      "token": "seem",
      "start_offset": 39,
      "end_offset": 43,
      "type": "<ALPHANUM>",
      "position": 6
    },
    {
      "token": "weird",
      "start_offset": 44,
      "end_offset": 49,
      "type": "<ALPHANUM>",
      "position": 7
    },
    {
      "token": "strange",
      "start_offset": 44,
      "end_offset": 49,
      "type": "SYNONYM",
      "position": 7
    },
    {
      "token": "sometimes",
      "start_offset": 50,
      "end_offset": 59,
      "type": "<ALPHANUM>",
      "position": 8
    }
  ]
}
```

## Adding a text document

```
POST /synonyms/_doc
{
  "description": "Elasticsearch is awesome, but can also seem weird sometimes."
}
```

## Searching the index for synonyms

Example #1:

- Run:

```
GET /synonyms/_search
{
  "query": {
    "match": {
      "description": "great"
    }
  }
}
```

- Output:

```json
{
  "took": 10,
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
    "max_score": 0.3107994,
    "hits": [
      {
        "_index": "synonyms",
        "_id": "-zG-YZ0BspJIKqSE6iPM",
        "_score": 0.3107994,
        "_source": {
          "description": "Elasticsearch is awesome, but can also seem weird sometimes"
        }
      }
    ]
  }
}
```

Example #2:

- Run:

```
GET /synonyms/_search
{
  "query": {
    "match": {
      "description": "awesome"
    }
  }
}
```

- Output:

```json
{
  "took": 29,
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
    "max_score": 0.4168806,
    "hits": [
      {
        "_index": "synonyms",
        "_id": "-zG-YZ0BspJIKqSE6iPM",
        "_score": 0.4168806,
        "_source": {
          "description": "Elasticsearch is awesome, but can also seem weird sometimes"
        }
      }
    ]
  }
}
```

```
┌─────────────┐   "awesome"   ┌─────────────┐  "great","super"┌─────────────┐
│             │ ─────────────>│             │ ───────────────>│             │
│ "awesome"   │               │  ANALYSIS   │                 │  "great"    │
│             │               │             │                 │  "super" ✓  │
└─────────────┘               └─────────────┘                 └─────────────┘

MATCH QUERY                        ANALYSIS                    INVERTED INDEX
```

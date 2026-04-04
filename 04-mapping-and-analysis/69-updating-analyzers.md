# Updating Analyzers

## Add `description` mapping using `my_custom_analyzer`

```
PUT /analyzer_test/_mapping
{
  "properties": {
    "description": {
      "type": "text",
      "analyzer": "my_custom_analyzer"
    }
  }
}
```

## Let's index a document to test this out:

```
POST /analyzer_test/_doc
{
    "description": "Is that Peter's cute-looking dog?"
}
```

## Search query using `keyword` analyzer

`my_custom_analyzer` will remove stop words such as "that", so effectively we'll be searching for `[]`, i.e. nothing.

```
GET /analyzer_test/_search
{
  "query": {
    "match": {
      "description": {
        "query": "that"
      }
    }
  }
}
        |
        |
my_custom_analyzer
        |
       [ ]
```

By specifying the `analyzer` while searching, we tell ES to use that instead of the configured one, in this case, `my_custom_analyzer`:

```
GET /analyzer_test/_search
{
  "query": {
    "match": {
      "description": {
        "query": "that",
        "analyzer": "keyword"
      }
    }
  }
}
        |
        |
     keyword
        |
    ["that"]
```

- However, the document was indexed with the custom analyzer, so stop words were removed. Meaning our search will still yield nothing.
- Why do we still go ahead with this? Because using the `keyword` analyzer helps us find out if there is a document with the word of our choice, in this case: `that`

| TERM    | DOCUMENT #1 |
| ------- | ----------- |
| peter's | X           |
| cute    | X           |
| looking | X           |
| dog     | X           |

Let's try that out:

- Run

```
GET /analyzer_test/_search
{
  "query": {
    "match": {
      "description": {
        "query": "that",
        "analyzer": "keyword"
      }
    }
  }
}
```

- Output:

```json
{
  "took": 6,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
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

- No reponse, as expected. the word `that` was removed by our analyzer.

## Close `analyzer_test` index

```
POST /analyzer_test/_close
```

## Update `my_custom_analyzer` (remove `stop` token filter)

```
PUT /analyzer_test/_settings
{
  "analysis": {
    "analyzer": {
      "my_custom_analyzer": {
        "type": "custom",
        "tokenizer": "standard",
        "char_filter": ["html_strip"],
        "filter": [
          "lowercase",
          "asciifolding"
        ]
      }
    }
  }
}
```

## Open `analyzer_test` index

```
POST /analyzer_test/_open
```

## Retrieve index settings

```
GET /analyzer_test/_settings
```

We can see that the `stop` token filter has been removed

```
"my_custom_analyzer": {
        "filter": [
        "lowercase",
        "asciifolding"
        ],
        "char_filter": [
        "html_strip"
        ],
        "type": "custom",
        "tokenizer": "standard"
    }
```

## Reindex documents

This new update will only apply on new documents going forward. To update the historical documents that were indexed using the old version of the analyzer:

- You could create a new index and copy over documents to it.
- We'll use the `update_by_query` api. This will apply to all documents as we haven't specified a match_by query

```
POST /analyzer_test/_update_by_query?conflicts=proceed
```

## Run search query again

Run:

```
GET /analyzer_test/_search
{
    "query": {
        "match": {
          "description": {
            "query": "that",
            "analyzer": "keyword"
          }
        }
    }
}
```

Output:

```json
{
  "took": 221,
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
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "analyzer_test",
        "_id": "pbp7WZ0Bv160fQzNH0_F",
        "_score": 0.2876821,
        "_source": {
          "description": "Is that Peter's cute-looking dog?"
        }
      }
    ]
  }
}
```

## Lecture Summary

- Analyzers can be updated
- Pay attention to existing documents
  - They were analyzed using the old version of the analyzer
  - Reindex those documents to avoid headaches
- Try to get analyzers right before indexing documents
  - Not always possible though, in which case you now know what to do

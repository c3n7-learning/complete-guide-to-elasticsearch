# Adding Analyzers to existing indices

We already have this analyzer from the previous section:

```
PUT /analyzer_test
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "char_filter": [
            "html_strip"
          ],
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "stop",
            "asciifolding"
          ]
        }
      }
    }
  }
}
```

Let's try to add a new analyer called `my_second_analyzer`:

- Run:

```
PUT /analyzer_test/_settings
{
  "analysis": {
    "analyzer": {
      "my_second_analyzer": {
        "type": "custom",
        "char_filter": [
          "html_strip"
        ],
        "tokenizer": "standard",
        "filter": [
          "lowercase",
          "stop",
          "asciifolding"
        ]
      }
    }
  }
}
```

- Output:

```json
{
  "error": {
    "root_cause": [
      {
        "type": "illegal_argument_exception",
        "reason": "Can't update non dynamic settings [[index.analysis.analyzer.my_second_analyzer.tokenizer, index.analysis.analyzer.my_second_analyzer.type, index.analysis.analyzer.my_second_analyzer.char_filter, index.analysis.analyzer.my_second_analyzer.filter]] for open indices [[analyzer_test/V4kQ2YlYQxWgzMMNn3zP7Q]] unless the `reopen` query parameter is set to true. Alternatively, close the indices, apply the settings changes, and reopen the indices"
      }
    ],
    "type": "illegal_argument_exception",
    "reason": "Can't update non dynamic settings [[index.analysis.analyzer.my_second_analyzer.tokenizer, index.analysis.analyzer.my_second_analyzer.type, index.analysis.analyzer.my_second_analyzer.char_filter, index.analysis.analyzer.my_second_analyzer.filter]] for open indices [[analyzer_test/V4kQ2YlYQxWgzMMNn3zP7Q]] unless the `reopen` query parameter is set to true. Alternatively, close the indices, apply the settings changes, and reopen the indices"
  },
  "status": 400
}
```

- We get the error `Can't update non dynamic settings .. .for open indices`

## Open & closed indices

- An open index is available for indexing and search results
- A closed idnex will refuse requests
  - Read and write requries are blocked
  - Necessary for performing some operations.

## Dynamic and Static settings

- Dynamic settings can be changed without closing the index first
  - Requires no downtime
- Static settings require the index to be closed first
  - The index will be briefly unavailable
- Analysis settings are static settings

## Let's close our index

Run:

- We can have or not have the preceding `/` when using kibana

```
POST analyzer_test/_close
```

Output:

```json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "indices": {
    "analyzer_test": {
      "closed": true
    }
  }
}
```

Let's run the index settings update:

- Run:

```
PUT /analyzer_test/_settings
{
  "analysis": {
    "analyzer": {
      "my_second_analyzer": {
        "type": "custom",
        "char_filter": [
          "html_strip"
        ],
        "tokenizer": "standard",
        "filter": [
          "lowercase",
          "stop",
          "asciifolding"
        ]
      }
    }
  }
}
```

- Output

```json
{
  "acknowledged": true
}
```

Let's now reopen our index:

```
POST /analyzer_test/_open
```

Now let's view our settings:

- Run:

```
GET /analyzer_test/_settings
```

- Output:

```json
{
  "analyzer_test": {
    "settings": {
      "index": {
        "routing": {
          "allocation": {
            "include": {
              "_tier_preference": "data_content"
            }
          }
        },
        "verified_before_close": "true",
        "number_of_shards": "1",
        "provided_name": "analyzer_test",
        "creation_date": "1775319422641",
        "analysis": {
          "analyzer": {
            "my_second_analyzer": {
              "filter": ["lowercase", "stop", "asciifolding"],
              "char_filter": ["html_strip"],
              "type": "custom",
              "tokenizer": "standard"
            },
            "my_custom_analyzer": {
              "filter": ["lowercase", "stop", "asciifolding"],
              "char_filter": ["html_strip"],
              "type": "custom",
              "tokenizer": "standard"
            }
          }
        },
        "number_of_replicas": "1",
        "uuid": "V4kQ2YlYQxWgzMMNn3zP7Q",
        "version": {
          "created": "9060000"
        }
      }
    }
  }
}
```

- We can see our two analyzers are now available
- Closing/reopening our index is required for analysis settings

## Opening & closing indices

- Fairly quick, but might not be an option for production clusters
  - e.g. mission critical systems where downtime is unacceptable
- Alternatively, reindex documents into a new index
  - Create a new index with the updated settings
  - Use an index alias for the transition

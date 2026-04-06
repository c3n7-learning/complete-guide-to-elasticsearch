# Adding Synonyms from File

- The path should be an absolute path, or relative to the config directory

## Adding index with custom analyzer

Navigate to the config directory

```shell
$ cd bin/elasticsearch-9.3.2/config
$ mkdir analysis
$ cd analysis
$ touch synonyms.txt
```

This should be the content of `synonyms.txt`:

```
# this is a comment

awful => terrible
awesome => great, super
elasticsearch, logstash, kibana => elk
weird, strange
good, nice, great
```

Delete the existing index

```http
DELETE /synonyms
```

Restart the node before doing the imports

- Run:

```http
PUT /synonyms
{
  "settings": {
    "analysis": {
      "filter": {
        "synonym_test": {
          "type": "synonym",
          "synonyms_path": "analysis/synonyms.txt"
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

## Testing the analyzer

```
POST /synonyms/_analyze
{
  "analyzer": "my_analyzer",
  "text": "Elasticsearch"
}
```

## Notes

1. If you have multiple nodes, copy `analysis.txt` there too
2. After the index is created, if new entries are added to the synonyms file
   - That index is only available after restarting a node
   - Existing documents won't be updated
   - Only new documents will be updated going forward
   - The search queries will be analyzed with the new synonyms as well, so you might end up not getting results from the old index documents

Solution?

- Update By Query API (`POST /synonyms/_update_by_query`)

# Highlighting matches in fields

References:

- https://www.elastic.co/docs/reference/elasticsearch/rest-apis/highlighting
- https://www.elastic.co/docs/reference/elasticsearch/rest-apis/highlighting-settings

## Adding a test document

```
PUT /highlighting/_doc/1
{
  "description": "Let me tell you a story about Elasticsearch. It's a full-text search engine that is built on Apache Lucene. It's really easy to use, but also packs lots of advanced features that you can use to tweak its searching capabilities. Lots of well-known and established companies use Elasticsearch, and so should you!"
}
```

## Highlighting matches within the `description` field

```
GET /highlighting/_search
{
  "_source": false,
  "query": {
    "match": { "description": "Elasticsearch story" }
  },
  "highlight": {
    "fields": {
      "description" : {}
    }
  }
}
```

Output:

```json
{
  "took": 72,
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
    "max_score": 0.68324494,
    "hits": [
      {
        "_index": "highlighting",
        "_id": "1",
        "_score": 0.68324494,
        "_ignored": ["description.keyword"],
        "highlight": {
          "description": [
            "Let me tell you a <em>story</em> about <em>Elasticsearch</em>.",
            "Lots of well-known and established companies use <em>Elasticsearch</em>, and so should you!"
          ]
        }
      }
    ]
  }
}
```

- Description is a list because matches could be far-between each other.
- ES returns fragments because the field could be very long.
- Consider a document with 500 words, wouldn't make much sense would it.
- The fragments give the user a bit of context.

In addition

- ES highlights the original term in case we have synonyms in the query / document
  - e.g, if the synonym for `weird` is `strange`, and we such `weird`, even though internally the query is replaced with `strange`, `weird` will be highlighted.
- Same is the case with stemming and so forth.

## Specifying a custom tag

```
GET /highlighting/_search
{
  "_source": false,
  "query": {
    "match": { "description": "Elasticsearch story" }
  },
  "highlight": {
    "pre_tags": [ "<strong>" ],
    "post_tags": [ "</strong>" ],
    "fields": {
      "description" : {}
    }
  }
}
```

Output:

```json
{
  "took": 7,
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
    "max_score": 0.68324494,
    "hits": [
      {
        "_index": "highlighting",
        "_id": "1",
        "_score": 0.68324494,
        "_ignored": ["description.keyword"],
        "highlight": {
          "description": [
            "Let me tell you a <strong>story</strong> about <strong>Elasticsearch</strong>.",
            "Lots of well-known and established companies use <strong>Elasticsearch</strong>, and so should you!"
          ]
        }
      }
    ]
  }
}
```

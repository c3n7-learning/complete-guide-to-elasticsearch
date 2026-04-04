# Analyzers and search queries

- Assue we pass the text below through a custom analyzer that also does stemming
  - "I loved drinking bottles of wine on last year's vacation."
- The custom analyzer will give us
  - `["i", "love", "drink", "bottl", "of", "wine", "on", "last", "year", "vacat"]`

What will happen when we do this search?

```
GET /stemming_test/_search
{
  "query": {
    "match": {
      "description": "drinking"
    }
  }
}
```

Against this index?

```json
{
  "properties": {
    "description": {
      "type": "text",
      "analyzer": "stemming_analyzer"
    }
  }
}
```

- The custom analyzer is passed through the same analyzer, so in the background, the word `drink` will be searched

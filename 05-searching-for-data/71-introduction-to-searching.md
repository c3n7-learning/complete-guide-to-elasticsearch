# Introduction to Searching

## URI Searches

These use query params

- This use Apache Lucene syntax
- Is simple, but does not give us access to all that ES provides
- We won't use this

```
GET /products/_search?q=name:sauvignon AND tags:wine
```

## Query DSL

We'll prefer this:

- It's verbose but gives us access to all the search features
- The request body is a json object

```
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "sauvignon"
          }
        },
        {
          "match": {
            "tags": "wine"
          }
        }
      ]
    }
  }
}
```

## Our first search query

- We specify the type of our query under the `query` object
- We'll go with `match_all` for now

Run:

```
GET /products/_search
{
    "query": {
        "match_all": {  }
    }
}
```

Output:

```
{
  "took": 245,
  "timed_out": false,
  "_shards": {
    "total": 2,
    "successful": 2,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1000,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "products",
        "_id": "1",
        "_score": 1,
        "_ignored": [
          "description.keyword"
        ],
        "_source": {
          "name": "Wine - Maipo Valle Cabernet",
          "price": 152,
          "in_stock": 38,
          "sold": 47,
          "tags": [
            "Beverage",
            "Alcohol",
            "Wine"
          ],
          "description": "Aliquam augue quam, sollicitudin vitae, consectetuer eget, rutrum at, lorem. Integer tincidunt ante vel ipsum. Praesent blandit lacinia erat. Vestibulum sed magna at nunc commodo placerat. Praesent blandit. Nam nulla. Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede. Morbi porttitor lorem id ligula.",
          "is_active": true,
          "created": "2004/05/13"
        }
      },
      {
        "..."
      }
    ]
  }
}
```

- `took` is how much ES took to process the query since it receives the query
- `timed_out` is whether or not the request timed out
- `_shards.total` specifies how many shards were queries. This includes shards that weren't allocated
- `_shards.successful` - number of shards that executed our request successfully
- `_shards.skipped`- number of shards that skipped our requests. Possible if it determines if it determines it doesn't have documents that fulfil our request
- `shards.failed` - number of shards that executed our request unsuccessfully

Under the `hits` object:

- Contains the documents that match the query and some metadata
- `total.value` - number of documents that match the query
- `total.relation` - a value of `eq` if the value key contains an accurate number. Which is almost always the case. It could also contain `gte` in some cases
- `max_score` - The highest score of the matched document. It is `1.0` because we used `match_all`, and it simply returns all documents without computing fancy relevance
- `hits` - Our documents are here

`hits.hits.*`:

- `_index` - name of the index where the document is stored. When searching for data, we can search multiple indices at once, so this helps us know
- `_id` - id of the document
- `_score` - relevance score
- `_ignore` - keys that were ignored when the document was indexed. We set the `description.keyword` key to ignore descriptions with a length greater than 256
- `_source` - the actual document. The source object we discussed earlier.

## Lecture Summary

- There are two ways of searching:
  - Request URI
    - Search queries are added to the URL
    - Uses Apache Lucene's query syntax
    - Only supports relatively simple queries
  - Query DSL
    - Search queries are defined as JSOn within the request body
    - More verbose, but supports all features
    - We will be using it throughout this course

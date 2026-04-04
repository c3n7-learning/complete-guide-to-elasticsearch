# Defining Field Aliases

## Introduction to field aliases

- Field names can be changed when reindexing documents
  - Probably not worth it for lots of documents
- An alternative is to use _field aliases_
  - Does not require documents to be reindexed
  - Let's add one pointing from _comment_ to _content_
  - Aliases can be used within queries
  - Aliases are defined with a field mapping

To create an alias

- Run:

```
PUT /reviews/_mapping
{
    "properties": {
        "comment": {
            "type": "alias",
            "path": "content"
        }
    }
}
```

- Output:

```
{
  "acknowledged": true
}
```

Let's attempt to query both the old and new fields

- Run:

```
GET /reviews/_search
{
    "query": {
        "match": {
          "content": "outstanding"
        }
    }
}
```

- You'll see the `Outstanding course! Bo really taught me a lot about Elasticsearch!` result

- Run:

```
GET /reviews/_search
{
    "query": {
        "match": {
          "comment": "outstanding"
        }
    }
}
```

- You'll see the `Outstanding course! Bo really taught me a lot about Elasticsearch!` result

## Updating field aliases

- Field aliases can actually be updated
  - Only its target field, though. Not the original field
- Simply perform a mapping update with a new `path` value
- Possible because aliases don't affect indexing
  - It's a query-construct

ES does the translation for us internally.

Example #1

- This query:

```
POST /reviews/_doc
{
  "rating": 5.0,
  "comment": "Outstanding course!",
  ...
}
```

- Is translated to:

```
POST /reviews/_doc
{
  "rating": 5.0,
  "content": "Outstanding course!",
  ...
}
```

Example #2:

- This query:

```
POST /reviews/_doc
{
  "rating": 5.0,
  "content": "Outstanding course!",
  ...
}
```

- Is translated to:

```
GET /reviews/_search
{
  "query": {
    "match": {
      "content": "outstanding"
    }
  }
}
```

## Index Aliases

- Similary to field Aliases, ES also supports index aliases.
- Typically used when dealing with large data volumes
- We won't cover that now, though

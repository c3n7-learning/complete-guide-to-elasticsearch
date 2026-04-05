# Query Execution Contexts

There are two contexts in which a query can be executed

1. Query Context
2. Filter execution context

## 1. Query Context

- What you've seen thus far (with one exception)
- Answers two questions
  - Does this document match? (yes/no)
  - How well does this document match (`_score` metadata field)
- Query results are sorted by `_score` descendingly
  - The most relevant documents appear on top

### Identifying the query context

- They have the key name `query`. Any thing below that are executed int the `query` context

```
GET /products/_search
{
  "query": {            <---- query context start
    "match": {
      "name": "pasta"
    }
  }                     <---- query context end
}
```

## 2.Filter execution context

- Only answers one question; "Does this document match?" (yes/no)
  - No relevance scores are calculated
- Used to filter data, typically on structured data (dates, numbers, `keyword`)
  - Relevance scoring is irrelevant if we just want to filter out documents
- Improves performance
  - No resources are spent calculating relevance scores
  - Query results can be cached

```
GET /products/_search
{
  "query": { (1)
    "bool": {
      "must": [
        { "match": { "name": "Beer" } }
      ],
      "must_not": [ (2)
        { "term": { "tags.keyword": "Wine" } }
      ],
      "filter": [ (3)
        { "term": { "tags.keyword": "Alcohol" } }
      ],
      "should": [
        { "match": { "description": "Beer" } }
      ]
    }
  }
}

(1) Query execution context
(2), (3) Filter execution context
```

## Changing the execution context

- It's sometimes possible to change the execution context
  - Only a few queries support it, though
- Typically done with the `bool` query and `filter` aggregation.
- Queries that support this generally have a `filter` parameter.

## Lecture Summary

- The query execution context calculates relevance scores
- The filter execution context ignores relevance scoring
  - Used for filtering data, typically with unstructured data (e.g. by date)
- The filter execution context improves performance
  - No resources spent calculating relevance scores
  - Queries can be cached
- Only a few queries support changing the execution context
  - Look for a `filter` parameter

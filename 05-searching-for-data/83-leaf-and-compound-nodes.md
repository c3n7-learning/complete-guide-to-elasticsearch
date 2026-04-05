# Leaf vs Compound Queries

- We have only covered leaf queries thus far
- Next up: compound queries
- Leaf queries seach for values and are independent queries
  - e.g. `term` and `match` queries
- Compound queries wrap `other` queries to produce a result

If we want to search for products with the keyword "Alcohol" that have stock less than 5, those would be two queries:

- This:

```
GET /products/_search
{
  "query": {
    "term": {
      "tags.keyword": "Alcohol"
    }
  }
}
```

- And:

```
GET /products/_search
{
  "query": {
    "range": {
      "in_stock": {
        "lte": 5
      }
    }
  }
}
```

In ES, we can only define a single query at the top level of our request. To have these two queries as single block, we need to wrap them with a compound query.

```
GET /products/_search
{
  "query": {
    [only a single query here]
  }
}
```

We could even wrap a compound query in another compound query. We can wrap however we like:

- SQL Equivalent `WHERE tag = "Alcohol" AND (in_stock = 0 OR is_active IS FALSE)`

```
                    AND
                     |
                    / \
                   /   \
                  /     \
                 /       \
    tags.keyword = "Alcohol"
                            OR
                           / \
                          /   \
                         /     \
                        /       \
               in_stock = 0    is_active = false
```

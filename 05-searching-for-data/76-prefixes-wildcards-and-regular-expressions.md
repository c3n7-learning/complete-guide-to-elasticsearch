# Prefixes, Wildcards and Regular Expressions

- Term level queries are used for exact matching
  - That's because we are querying non-analyzed values with queries that are not analyzed
- There are a few exceptions; we'll cover three of them for now
  - Querying by prefix, wildcards, and regular expressions
  - Remember to still query `keyword` fields

## `prefix` query

Run:

```
GET /products/_search
{
    "query": {
        "prefix": {
          "name.keyword": {
            "value": "Past"
          }
        }
    }
}
```

Output:

- We see results with name that begins with "Past"

```json
{
  "took": 40,
  "timed_out": false,
  "_shards": {
    "total": 2,
    "successful": 2,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 20,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "products",
        "_id": "94",
        "_score": 1,
        "_source": {
          "name": "Pastry - Banana Muffin - Mini"
        }
      },
      {
        "_index": "products",
        "_id": "277",
        "_score": 1,
        "_ignored": ["description.keyword"],
        "_source": {
          "name": "Pasta - Fettuccine Egg Fresh"
        }
      },
      {
        "_index": "products",
        "_id": "290",
        "_score": 1,
        "_ignored": ["description.keyword"],
        "_source": {
          "name": "Pasta - Lasagna Dry"
        }
      }
    ]
  }
}
```

| Match? | Indexed term                   |
| ------ | ------------------------------ |
| ✓      | "Pasta - Linguini Dry"         |
| ✓      | "Paste - Black Olive"          |
| ✓      | "Pastry - Baked Scones - Mini" |
| ✗      | "Linguini Pasta"               |

We could do this search in the `tags` keyword

```
GET /products/_search
{
    "query": {
        "prefix": {
          "tags.keyword": {
            "value": "Past"
          }
        }
    }
}
```

## `wildcard` queries

> Note  
> ⚠️ Avoid placing wildcards at the beginning of a pattern.
>
> - If you have many documents, this might make the query slow!
> - Only do so after considering the performance implications

| Pattern   | Terms          |
| --------- | -------------- |
| Past?     | ✅ "Pasta"     |
|           | ✅ "Paste"     |
| Bee?      | ❌ "Bee"       |
|           | ✅ "Beer"      |
|           | ❌ "Beets"     |
|           | ✅ "Beef"      |
| Bee\*     | ✅ "Bee"       |
|           | ✅ "Beer"      |
|           | ✅ "Beets"     |
|           | ✅ "Beef"      |
|           | ❌ "Beverage"  |
| ⚠️ \*Beer | ✅ "Beer"      |
|           | ✅ "Root Beer" |

- `?` the term + any other `one` character
- `\*` the term + any other `one or more` characters

Example #1:

```
GET /products/_search
{
    "query": {
        "wildcard": {
          "tags.keyword": {
            "value": "Past?"
          }
        }
    }
}
```

Example #2:

```
GET /products/_search
{
    "query": {
        "wildcard": {
          "tags.keyword": {
            "value": "Bee*"
          }
        }
    }
}
```

## Regular expressions

- The `regexp` query matches terms that match a regular expression
- Regular expressions are patterns used for matching strings
- Allows more complex queries than the `wildcard` query.

| Pattern       | Terms                |
| ------------- | -------------------- |
| Bee(f\|r)+    | ✅ "Beef"            |
|               | ✅ "Beer"            |
|               | ❌ "Beers"           |
|               | ❌ "Beet"            |
|               | ❌ "Beets"           |
| Bee[a-zA-Z]+  | ✅ "Beef"            |
|               | ✅ "Beer"            |
|               | ✅ "Beers"           |
|               | ✅ "Beet"            |
|               | ✅ "Beets"           |
| Bee(r\|t){1}  | ✅ "Beet"            |
|               | ❌ "Beetroot"        |
| Beer          | ❌ "Heineken (Beer)" |
|               | ❌ "Beer - Heineken" |
| Beer.\*       | ❌ "Heineken - Beer" |
|               | ✅ "Beer - Heineken" |
| ⚠️ .\*Beer    | ✅ "Heineken - Beer" |
|               | ❌ "Beer - Heineken" |
| ⚠️ .\*Beer.\* | ✅ "Heineken - Beer" |
|               | ✅ "Beer - Heineken" |

> Note  
> ⚠️ Avoid placing wildcards at the beginning of a pattern.

Example #1:

```
GET /products/_search
{
    "query": {
        "regexp": {
          "tags.keyword": {
            "value": "Bee(f|r)+"
          }
        }
    }
}
```

### Engine comparison

- ES supports regexp values that Apache Lucene accepts
- Things like `^` (start of string) or `$` (end of string) are not supported

| Other engines | Apache Lucene   |
| ------------- | --------------- |
| ^Beer         | Beer.\*         |
| Beer$         | .\*Beer         |
| Bee[a-zA-Z]+$ | Bee[a-zA-Z]+.\* |

## Case sensitivity

- All queries are case-sensitive by default.
- We can enable case-insensitive queries though using the `case_insensitive: true` parameter

`prefix` Example:

```
GET /products/_search
{
  "query": {
    "prefix": {
      "name.keyword": {
        "value": "Past",
        "case_insensitive": true
      }
    }
  }
}
```

`regexp` Example:

```
GET /products/_search
{
  "query": {
    "regexp": {
      "tags.keyword": {
        "value": "Bee(f|r)+",
        "case_insensitive": true
      }
    }
  }
}
```

`wildcard` Example:

```
GET /products/_search
{
  "query": {
    "wildcard": {
      "tags.keyword": {
        "value": "Past?",
        "case_insensitive": true
      }
    }
  }
}
```

## Lecture Summary

- The `prefix` query matches terms that begin with a prefix
- The `wildcard` query enables us to use wildcards
  - `?` to match any single character
  - `*` to match any number of characters (0-N)
- The `regexp` query matches terms based on regular expressions
  - More flexible than the `wildcard` query
  - The whole term must be matched
  - Uses Apache Lucene regex engine (`^` and `$` anchors are not supported)
- Avoid placing wildcards at the beginning of patterns if possible
- Use the `case_insensitive` parameter to ignore letter casing.

# Querying by field existence

SQL equivalent:

```sql
SELECT *
FROM products
WHERE tags IS NOT NULL
```

ES query:

- The `tags` mapping could just as well have been used

```
GET /products/_search
{
    "query": {
        "exists": {
            "field": "tags.keyword"
        }
    }
}
```

- Output

```json
{
  "took": 32,
  "timed_out": false,
  "_shards": {
    "total": 2,
    "successful": 2,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 554,
      "relation": "eq"
    },
    "hits": [
      {
        "_index": "products",
        "_id": "1",
        "_score": 1,
        "_ignored": ["description.keyword"],
        "_source": {
          "name": "Wine - Maipo Valle Cabernet",
          "price": 152,
          "in_stock": 38,
          "sold": 47,
          "tags": ["Beverage", "Alcohol", "Wine"],
          "description": "Aliquam augue quam, sollicitudin vitae, consectetuer eget, rutrum at, lorem. Integer tincidunt ante vel ipsum. Praesent blandit lacinia erat. Vestibulum sed magna at nunc commodo placerat. Praesent blandit. Nam nulla. Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede. Morbi porttitor lorem id ligula.",
          "is_active": true,
          "created": "2004/05/13"
        }
      }
    ]
  }
}
```

We have only 554 results! Even though all documents have a value for the `tags` field.

- The query matches documents with an indexed value for a field

### How empty values are indexed

- Empty strings are not treated as empty, and are thus indexed

| Field value | Indexed value |
| ----------- | ------------- |
| NULL        | N/A           |
| []          | N/A           |
| ""          | ""            |

Document #1:

```
POST /products/_doc
{
  "name": "Protein Powder",
  "tags": ["Supplement"]
}
```

Document #2:

```
POST /products/_doc
{
  "name": "Toast",
  "tags": []
}
```

Inverted index for `name` mapping:

| TERM      | DOCUMENT #1 | DOCUMENT #2 |
| --------- | ----------- | ----------- |
| "protein" | X           |             |
| "powder"  | X           |             |
| "toast"   |             | X           |

Inverted index for `tags.keyword` mapping

| TERM         | DOCUMENT #1 | DOCUMENT #2 |
| ------------ | ----------- | ----------- |
| "Supplement" | X           |             |

## Reasons for no indexed value

- Empty value provided (NULL or `[]`)
  - The `null_value` parameter is an exception for `NULL` values
- No value was provided for the field
- The `index` mapping parameter is set to `false` for the field. Thus the field is not indexed
- The value's length is greater than the `ignore_above` parameter. It defaults to `256` for keyword fields when dynamic mapping is used.
- Malformed value with the `ignore_malformed` mapping parameter set to `true`
  - e.g. a string may be ignored if one attempts to index it to an integer field

## Inverting the query

We have to use the `bool` query for this:

- SQL Equivalent

```sql
SELECT * FROM products WHERE tags IS NULL
```

- ES Query:

```
GET /products/_search
{
  "query": {
    "bool": {
      "must_not": [
        {
          "exists": {
            "field": "tags.keyword"
          }
        }
      ]
    }
  }
}
```

- Output:

```json
{
  "took": 67,
  "timed_out": false,
  "_shards": {
    "total": 2,
    "successful": 2,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 446,
      "relation": "eq"
    },
    "max_score": 0
  }
}
```

## Lecture Summary

- The `exists` query matches fields that have an `indexed` value
- Field values are only indexed if they are considered non-empty
  - `NULL` and empty arrays (`[]`) are empty values - empty strings ("") are not
  - There are a few other cases when values are not indexed.
- The `exists` query can be inverted by using the `bool` query's `must-not` occurence type

# Overview of mapping parameters

## Introduction

- We will cover the most important parameters
- This is not an exhaustive list, though
  - Most other parameters are very specialized and/or rarely used

References:

- https://www.elastic.co/docs/reference/elasticsearch/mapping-reference/mapping-parameters
- https://www.elastic.co/docs/reference/elasticsearch/mapping-reference/mapping-date-format

## `format` parameter

- Used to customize the format for `date` fields
- It is recommended to use the default format whenever possible
  - `"strict_date_optional_time||epoch_millis"
  - ISO 8601

If working with old systems though might necessitate using non ISO8601 dates

- Using Java's `DateFormatter` syntax
  - e.g. `dd/MM/yyy`
- Using built-in frmats
  - e.g. `epoch_second`

Example #1:

```
PUT /sales
{
  "mappings": {
    "properties": {
      "purchased_at": {
        "type": "date",
        "format": "dd/MM/yyyy"
      }
    }
  }
}
```

Example #2:

```
PUT /sales
{
  "mappings": {
    "properties": {
      "purchased_at": {
        "type": "date",
        "format": "epoch_second"
      }
    }
  }
}
```

## `properties` parameter

Defines nested fields for `object` and `nested` fields

- There is no such `object` data type in ES
- Object types are mapped implicitly
  Example #1:

```
PUT /sales
{
  "mappings": {
    "properties": {
      "sold_by": {
        "properties": {
          "name": { "type": "text" }
        }
      }
    }
  }
}
```

Example #2:

```
PUT /sales
{
  "mappings": {
    "properties": {
      "products": {
        "type": "nested",
        "properties": {
          "name": { "type": "text" }
        }
      }
    }
  }
}
```

## `coerce` parameter

- Used to enable/disable coercion of values (enabled by default)

To disable coercion for a field:

```
PUT /sales
{
  "mappings": {
    "properties": {
      "amount": {
        "type": "float",
        "coerce": false
      }
    }
  }
}
```

To disable coercion for the entire index:

- You could still override on the field level

```
PUT /sales
{
  "settings": {
    "index.mapping.coerce": false
  },
  "mappings": {
    "properties": {
      "amount": {
        "type": "float",
        "coerce": true
      }
    }
  }
}
```

## Introduction to `doc_values`

This gets into the world of Apache Lucene

- ES makes use of several data structures
  - No single DS serves all purposes
- Inverted indices are excellent for searching text
  - They don't perform well for many other data access patterns
- "Doc values" is another DS used by Apache Lucene
  - Optimized for a different data access pattern (document -> terms)

So:

- Essentially an "uninverted" inverted index
- Used for sorting, aggregations and scripting
- An additional data structure, not a replacement
- ES automatically queries the appropriate DS

### Disabling `doc_values`

Why would you want to do this?

- Set the `doc_values` parameter to `false` to save disk space
  - Because essentially the data is stored "twice" in the disk, once inverted, once uninverted
  - This saves disk space but _also slightly increases the indexing throughput_, aka the speed of our queries.
- Only disable `doc_values` if you won't use aggregations, sorting or scripting
- Particularly useful for large indices; typically not worth it for small ones
- Cannot e changed without reindexing documents into new index
  - Use with caution, and try to anticipate how fields will be queried

> Note  
> Consider this an advanced paramter, and don't configure it unless you have to.

```
PUT /sales
{
  "mappings": {
    "properties": {
      "buyer_email": {
        "type": "keyword",
        "doc_values": false
      }
    }
  }
}
```

## `norms` parameter

- Normalization factors used for relevance sorting
- Often we don't just want to _filter_ results, but also rank them
- Norms can be disabled to save disk space
  - USeful for fields that won't used for relevance scoring
  - So if you won't do relevance sorting? you can disable `norms`

Example fields:

- E.g a field like tags can be used for filtering or aggregation, and VERY rarely, relevance scores, so we can disable `_norms_`
- A field like `name` though will almost always be used to search via full text search, and relevance sorting makes sense for it.

```
PUT /products
{
  "mappings": {
    "properties": {
      "tags": {
        "type": "text",
        "norms": false
      }
    }
  }
}
```

## `index` parameter

- Disables indexing for a field
- Values are still stored within `_source`, but won't be part of the data structures used for indexing
- Useful if you won't use a field for search queries
- Saves disk space and slightly improves indexing throughput
- Often used for time series data
- Fields with indexing enabled can still be used for aggregations

```
PUT /server-metrics
{
  "mappings": {
    "properties": {
      "server_id": {
        "type": "integer",
        "index": false
      }
    }
  }
}
```

## `null_value` parameter

- `NULL` values cannot be indexed or searched
  - Same applies for empty arrays or an array of null values
- Use this parameter to replace `NULL` values with another value
  - Whenever ES encounters a `NULL` value, it will override it with the value you configured

Note that

- Only works for explicit `NULL` values, not empty arrays for example
- The replacement value must be of the same data type as the field
- Does not affect the value stored within `_source`

```
PUT /sales
{
  "mappings": {
    "properties": {
      "partner_id": {
        "type": "keyword",
        "null_value": "NULL"
      }
    }
  }
}
```

## `copy_to` parameter

- Used to copy multiple field values into a "group field"
  - Consider a user object with `first_name` and `last_name`
- Simply specify the name of the target field as the value
- E.g. `first_name` and `last_name` -> `full_name`
- _Values_ are copied, not terms/tokens
  - The analyzer of the target field is used for the values
- The target field is not part of `_source`

```
PUT /sales
{
  "mappings": {
    "properties": {
      "first_name": {
        "type": "text",
        "copy_to": "full_name"
      },
      "last_name": {
        "type": "text",
        "copy_to": "full_name"
      },
      "full_name": {
        "type": "text"
      }
    }
  }
}
```

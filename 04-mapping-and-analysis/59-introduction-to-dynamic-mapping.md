# Introduction to Dynamic Mapping

- Is a way to make ES easier to use, by not making us to explicitly specify the field mappings

Suppose we idnex this document whose index doesn't exist

- This document

```
POST /my-index/_doc
{
  "tags": ["computer", "electronics"],
  "in_stock": 4,
  "created_at": "2020/01/01 00:00:00"
}
```

- Will be mapped to

```
{
  "created_at": {
    "type": "date",
    "format": "yyyy/MM/dd HH:mm:ss||yyyy/MM/dd||epoch_millis"
  },
  "in_stock": {
    "type": "long"
  },
  "tags": {
    "type": "text",
    "fields": {
      "keyword": {
        "type": "keyword",
        "ignore_above": 256
      }
    }
  }
}
```

- ES goes with type `long` for `in_stock`, because even if we've passed a four, ES doesn't know how large our numeric value would get
- `text` is mapped to `text` and a multi-field mapping of `keyword`, becaues ES doesn't know if we'll do full text search, sorting or aggregations
- These defaults are what ES think we would want to do, but the defaults could take lots of disk space, e.g. because of using an `int` instead of a `long`, etc.
- The defaults are sensible, but if you want to have the most optimal index, do you own field mappin

What are the rules that determine how a field is mapped automatically?

- `[ "strict_date_optional_time","yyyy/MM/dd HH:mm:ss Z||yyyy/MM/dd Z"]`

| JSON                    | ELASTICSEARCH                                                                                                        |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------- |
| string                  | Either: <br/> - **text** field with **keyword** mapping <br/> - **date** field <br/> - (**float** or **long** field) |
| integer                 | long                                                                                                                 |
| floating point number   | float                                                                                                                |
| boolean (true or false) | boolean                                                                                                              |
| object                  | object                                                                                                               |
| array                   | Depends on the first non-null value                                                                                  |

Let's explore the mapping for the documents we added before using dynamic mapping:

- Run:

```
GET /products/_mapping
```

- Output:

```
{
  "products": {
    "mappings": {
      "properties": {
        "created": {
          "type": "date",
          "format": "yyyy/MM/dd HH:mm:ss||yyyy/MM/dd||epoch_millis",
          "locale": ""
        },
        "description": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "doc": {
          "properties": {
            "in_stock": {
              "type": "long"
            },
            "name": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "price": {
              "type": "long"
            }
          }
        },
        "in_stock": {
          "type": "long"
        },
        "is_active": {
          "type": "boolean"
        },
        "name": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "price": {
          "type": "long"
        },
        "sold": {
          "type": "long"
        },
        "tags": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        }
      }
    }
  }
}
```

- You can see the `text` and `description` fields have both `text` and `keyword` fields. Not the most optimal in terms of disk space, but its the 'sensible' ES default.

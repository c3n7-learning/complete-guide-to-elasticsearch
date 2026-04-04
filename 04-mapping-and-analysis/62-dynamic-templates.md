# Dynamic Templates

| JSON value        | match_mapping_type |
| ----------------- | ------------------ |
| true or false     | boolean            |
| { ... } (objects) | object             |
| "string value"    | string             |
| "2020/01/01"      | date               |
| 123.4             | double             |
| 123               | long               |
| Any               | \*                 |

We create a dynamic_template by configuring the index as follows:

- Run:

```
PUT /dynamic_template_test
{
    "mappings": {
        "dynamic_templates": [
            {
                "integers": {
                    "match_mapping_type": "long",
                    "mapping": {
                        "type": "integer"
                    }
                }
            }
        ]
    }
}
```

- Output:

```json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "dynamic_template_test"
}
```

Let's index a document that matches the dynamic template mapping we did before

- Run:

```
POST /dynamic_template_test/_doc
{
    "in_stock": 123
}
```

Let's look at the mapping:

- Run:

```
GET /dynamic_template_test/_mapping
```

- Output:

```
{
  "dynamic_template_test": {
    "mappings": {
      "dynamic_templates": [
        {
          "integers": {
            "match_mapping_type": "long",
            "mapping": {
              "type": "integer"
            }
          }
        }
      ],
      "properties": {
        "in_stock": {
          "type": "integer"
        }
      }
    }
  }
}
```

## Modify default mapping for strings (set `ignore_above` to 512)

```
PUT /test_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "strings": {
          "match_mapping_type": "string",
          "mapping": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 512
              }
            }
          }
        }
      }
    ]
  }
}
```

## Using `match` and `unmatch`

- Used to specify conditions for field names
- Field names must match the condition specified by the `match` parameter
- `unmatch` is used to exclude fields that were matched by the `match` parameter
- Both parameters support patterns with wildcards (\*)
  - Hardcoding field names wouldn't make any sense

```
PUT /test_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "strings_only_text": {
          "match_mapping_type": "string",
          "match": "text_*",
          "unmatch": "*_keyword",
          "mapping": {
            "type": "text"
          }
        }
      },
      {
        "strings_only_keyword": {
          "match_mapping_type": "string",
          "match": "*_keyword",
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ]
  }
}
```

With the above, if we run this:

```
POST /test_index/_doc
{
  "text_product_description": "A description.",
  "text_product_id_keyword": "ABC-123"
}
```

We'll get this you this

```json
{
  "test_index": {
    "mappings": {
      "properties": {
        "text_product_description": {
          "type": "text"
        },
        "text_product_id_keyword": {
          "type": "keyword"
        }
      }
    }
  }
}
```

## Setting `match_pattern` to `regex`

```
PUT /test_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "names": {
          "match_mapping_type": "string",
          "match": "^[a-zA-Z]+_name$",
          "match_pattern": "regex",
          "mapping": {
            "type": "text"
          }
        }
      }
    ]
  }
}
```

With the above, if we run this:

```
POST /test_index/_doc
{
  "first_name": "John",
  "middle_name": "Edward",
  "last_name": "Doe"
}
```

We'll get this:

```
{
  "properties": {
    "first_name": {
      "type": "text"
    },
    "last_name": {
      "type": "text"
    },
    "middle_name": {
      "type": "text"
    }
  }
}
```

## Using `path_match` and `patch_unmatch` parameters

- These parameters evaluate the full field `path`
  - i.e. not just the field name
- This is the dot notation that you saw earlier
  - e.g. `name`.`first_name`
- Widlcards are also supported

```
PUT /test_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "copy_to_full_name": {
          "match_mapping_type": "string",
          "path_match": "employer.name.*",
          "mapping": {
            "type": "text",
            "copy_to": "full_name"
          }
        }
      }
    ]
  }
}
```

Adding the following document:

```
POST /test_index/_doc
{
  "employer": {
    "name": {
      "first_name": "John",
      "middle_name": "Edward",
      "last_name": "Doe"
    }
  }
}
```

Will lead to this mapping

```json
{
  "properties": {
    "employer": {
      "properties": {
        "name": {
          "properties": {
            "first_name": {
              "type": "text",
              "copy_to": ["full_name"]
            },
            "middle_name": {
              "type": "text",
              "copy_to": ["full_name"]
            },
            "last_name": {
              "type": "text",
              "copy_to": ["full_name"]
            }
          }
        }
      }
    }
  }
}
```

## Using placeholders

- Placeholders are enclosed in `{}`
- The `dynamic_type` placeholder is replaced with the data type that was detected by dynamic mapping.

This example could be used for timeseries data

- We never really need to filter on specific values, but aggregate on time intervals
- Another thing is to disable norms, which you could still do

```
PUT /test_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "no_doc_values": {
          "match_mapping_type": "*",
          "mapping": {
            "type": "{dynamic_type}",
            "index": false
          }
        }
      }
    ]
  }
}
```

Adding this document:

```
POST /test_index/_doc
{
  "name": "John Doe",
  "age": 26
}
```

Will lead to this mapping:

```json
{
  "properties": {
    "age": {
      "type": "long",
      "index": false
    },
    "name": {
      "type": "text",
      "index": false
    }
  }
}
```

## Index templates vs dynamic templates

- Index templates apply mapping and index settings for matching indices
  - This happens when indices are created and their names match a pattern
- Dynamic templates are evaluated when new fields are encountered
  - ... and dynamic mapping is enabled
  - The specified field mapping is added if the templates's conditions match
- Index templates define fixed mappings; dynamic templates are... dynamic

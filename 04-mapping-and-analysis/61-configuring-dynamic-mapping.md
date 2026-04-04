# Configuring dynamic mapping

To disable dynamic mapping, add `dynamic: false` to the mappings

Create a new index without dynamic mapping:

- Run:

```
PUT /people
{
    "mappings": {
        "dynamic": false,
        "properties": {
            "first_name": {
                "type": "text"
            }
        }
    }
}
```

Now let's try to do dynamic mapping

- Run:

```
POST /people/_doc
{
    "first_name": "Bo",
    "last_name": "Andersen"
}
```

- Output:

```json
{
  "_index": "people",
  "_id": "orpQV50Bv160fQzNyk-y",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```

- No error? Thought we disabled dynamic mapping!

Let's look at the mapping of the index:

- Run:

```
GET /people/_mapping
```

- Output:

```json
{
  "people": {
    "mappings": {
      "dynamic": "false",
      "properties": {
        "first_name": {
          "type": "text"
        }
      }
    }
  }
}
```

Okay, interesting. How does the `_source` document look like?

- Run:

```
GET /people/_search
{
    "query": {
        "match": {
          "first_name": "Bo"
        }
    }
}
```

- Output:

```json
{
  "took": 19,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "people",
        "_id": "orpQV50Bv160fQzNyk-y",
        "_score": 0.2876821,
        "_source": {
          "first_name": "Bo",
          "last_name": "Andersen"
        }
      }
    ]
  }
}
```

Okay, now let's try to search with the last name

- Run:

```
GET /people/_search
{
    "query": {
        "match": {
          "last_name": "Andersen"
        }
    }
}
```

- Output:

```json
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 0,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  }
}
```

## Setting `dynamic` to `false`

- We set `false`, not `reject`
- New fields are ingored
  - They are not indexed, but still part of `_source`
- No inverted index is created for the `last_name` field
  - Querying the field gives no results
- Fields cannot be indexed without a mapping
  - When enabled, dynamic mapping creates one before indexing values.
- New fields must be mapped explicitly

## There's a better way

- Setting `dynamic` to `strict`
- ES will reject unmapped fields
  - All fields must be mapped explicitly
  - Similar to the behavior of relational databases

Let's delete the index

```RUN
DELETE /people
```

Now let's set the dynamic mapping setting to strict:

- Run:

```
PUT /people
{
    "mappings": {
        "dynamic": "strict",
        "properties": {
            "first_name": {
                "type": "text"
            }
        }
    }
}
```

Now let's try to index a document

- Run:

```
POST /people/_doc
{
    "first_name": "Bo",
    "last_name": "Andersen"
}
```

- Output:

```
{
  "error": {
    "root_cause": [
      {
        "type": "strict_dynamic_mapping_exception",
        "reason": "[1:42] mapping set to strict, dynamic introduction of [last_name] within [_doc] is not allowed"
      }
    ],
    "type": "strict_dynamic_mapping_exception",
    "reason": "[1:42] mapping set to strict, dynamic introduction of [last_name] within [_doc] is not allowed"
  },
  "status": 400
}
```

Dynamic mapping config can be overriden in nested objects:

- For example

```
PUT /computers
{
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "name": {
        "type": "text"
      },
      "specifications": {
        "properties": {
          "cpu": {
            "properties": {
              "name": {
                "type": "text"
              }
            }
          }
        },
        "other": {
          "dynamic": true,
          "properties": { ... }
        }
      }
    }
  }
}
```

This would not work ❌

```
POST /computers/_doc
{
  "name": "Gamer PC",
  "specifications": {
    "cpu": {
      "name": "Intel Core i7-9700K",
      "frequency": 3.6
    }
  }
}
```

This would work ✅

```
POST /computers/_doc
{
  "name": "Gamer PC",
  "specifications": {
    "cpu": {
      "name": "Intel Core i7-9700K"
    },
    "other": {
      "security": "Kensington"
    }
  }
}
```

## Enabling numeric detection

When enabling numeric detection, Elasticsearch will check the contents of strings to see if they contain only numeric
values - and map the fields accordingly as either `float` or `long`.

### Mapping

```
PUT /computers
{
  "mappings": {
    "numeric_detection": true
  }
}
```

### Example document

```
POST /computers/_doc
{
  "specifications": {
    "other": {
      "max_ram_gb": "32", # long
      "bluetooth": "5.2" # float
    }
  }
}
```

## Date detection

### Default date detection formats

```json
["strict_date_optional_time", "yyyy/MM/dd HH:mm:ss Z||yyyy/MM/dd Z"]
```

### Disabling date detection

```
PUT /computers
{
  "mappings": {
    "date_detection": false
  }
}
```

### Configuring dynamic date formats

```
PUT /computers
{
  "mappings": {
    "dynamic_date_formats": ["dd-MM-yyyy"]
  }
}
```

## Clean up

```
DELETE /people
```

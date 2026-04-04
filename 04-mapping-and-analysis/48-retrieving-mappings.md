# Retrieving Mappings

To retrieve the mapping for an index:

- Run:

```
GET /reviews/_mapping
```

- Output:

```json
{
  "reviews": {
    "mappings": {
      "properties": {
        "author": {
          "properties": {
            "email": {
              "type": "keyword"
            },
            "first_name": {
              "type": "text"
            },
            "last_name": {
              "type": "text"
            }
          }
        },
        "content": {
          "type": "text"
        },
        "product_id": {
          "type": "integer"
        },
        "rating": {
          "type": "float"
        }
      }
    }
  }
}
```

To retrieve the mapping for a field

- Run

```
GET /reviews/_mapping/field/content
```

- Output:

```json
{
  "reviews": {
    "mappings": {
      "content": {
        "full_name": "content",
        "mapping": {
          "content": {
            "type": "text"
          }
        }
      }
    }
  }
}
```

To retrieve the mapping for an object, use the `.` notation

- Run

```
GET /reviews/_mapping/field/author.email
```

- Output:

```json
{
  "reviews": {
    "mappings": {
      "author.email": {
        "full_name": "author.email",
        "mapping": {
          "email": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

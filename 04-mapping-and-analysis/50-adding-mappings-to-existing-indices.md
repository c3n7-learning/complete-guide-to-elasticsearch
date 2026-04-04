# Adding Mappings to Existing Indices

What if we want to update an index? e.g. we want to add the `created_at` fields

- Run

```
PUT /reviews/_mapping
{
    "properties": {
        "created_at": {
            "type": "date"
        }
    }
}
```

Let's retrieve the mapping:

- Run

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
        "created_at": {
          "type": "date"
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

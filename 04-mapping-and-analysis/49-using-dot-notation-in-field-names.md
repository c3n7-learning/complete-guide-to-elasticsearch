# Using dot notation in field names

Previously, we added explicit mappings like this:

```
PUT /reviews
{
    "mappings": {
        "properties": {
            "rating": { "type": "float" },
            "content": { "type": "text" },
            "product_id": { "type": "integer" },
            "author": {
                "properties": {
                    "first_name": { "type": "text" },
                    "last_name": { "type": "text" },
                    "email": { "type": "keyword" }
                }
            }
        }
    }
}
```

We can use the dot-notation, by adding a `.` at each level of hierarchy

- Easier to visualize isn't it?

```
PUT /reviews_dot_notation
{
  "mappings": {
    "properties": {
      "rating": { "type": "float" },
      "content": { "type": "text" },
      "product_id": { "type": "integer" },
      "author.first_name": { "type": "text" },
      "author.last_name": { "type": "text" },
      "author.email": { "type": "keyword" }
    }
  }
}
```

Let's review the mapping

- Run:

```
GET /reviews_dot_notation/_mapping
```

- Output:

```json
{
  "reviews_dot_notation": {
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

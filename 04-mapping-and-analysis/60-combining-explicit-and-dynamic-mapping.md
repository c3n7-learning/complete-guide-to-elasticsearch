# Combining explicit and dynamic mapping

Let's run some queries

Create a new index:

- Run:

```
PUT /people
{
    "mappings": {
        "properties": {
            "first_name": {
                "type": "text"
            }
        }
    }
}
```

Add a new document

- Mapping for "last_name" will be done dynamically

```
POST /people/_doc
{
    "first_name": "Bo",
    "last_name": "Andersen"
}
```

Let's check the mappings

- Run

```
{
  "people": {
    "mappings": {
      "properties": {
        "first_name": {
          "type": "text"
        },
        "last_name": {
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

Now clean up

- Run

```
DELETE /people/_mapping
```

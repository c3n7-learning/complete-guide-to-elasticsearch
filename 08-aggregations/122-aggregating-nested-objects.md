# Aggregating nested objects

Run:

```http
GET /recipes/_search
{
  "size": 0,
  "aggs": {
    "employees": {
      "nested": {
        "path": "ingredients"
      }
    }
  }
}
```

Run:

````
```http
GET /recipes/_search
{
  "size": 0,
  "aggs": {
    "employees": {
      "nested": {
        "path": "ingredients"
      },
      "aggs": {
        "maximum_amount": {
          "max": {
            "field": "ingredients.amount"
          }
        }
      }
    }
  }
}
````

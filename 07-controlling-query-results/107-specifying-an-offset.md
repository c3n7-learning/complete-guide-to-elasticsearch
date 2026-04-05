# Specifying an offset

- We use the `from` parameter for this. The default value is 0.

```
GET /recipes/_search
{
  "_source": false,
  "size": 2,
  "from": 2,
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```

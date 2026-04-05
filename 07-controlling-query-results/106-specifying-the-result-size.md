# Specifying the result size

## Using a query parameter

```http
GET /recipes/_search?size=2
{
  "_source": false,
  "query": {
    "match": { "title": "pasta" }
  }
}
```

## Using a parameter within the request body

```http
GET /recipes/_search
{
  "_source": false,
  "size": 2,
  "query": {
    "match": { "title": "pasta" }
  }
}
```

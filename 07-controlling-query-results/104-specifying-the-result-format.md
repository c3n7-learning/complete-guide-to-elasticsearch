# Specifying the result format

## Returning results as YAML

```http
GET /recipes/_search?format=yaml
{
  "query": {
    "match": { "title": "pasta" }
  }
}
```

## Returning pretty JSON

- This is way more helpful when running in console, because kibana already does pretty formatting, but the REST API doesn't
- Only use this for debugging, as it is more resource intensive

```http
GET /recipes/_search?pretty
{
  "query": {
    "match": { "title": "pasta" }
  }
}
```

Without formatting:

```shell
curl -u elastic:zwDitFPJsDTqWgFTA7pn -H "Content-Type:application/json" -XGET "http://localhost:9200/recipes/_search" -d'
{
  "query": {
    "match": { "title": "pasta" }
  }
}'
```

With formatting:

```shell
curl -u elastic:zwDitFPJsDTqWgFTA7pn -H "Content-Type:application/json" -XGET "http://localhost:9200/recipes/_search?pretty" -d'
{
  "query": {
    "match": { "title": "pasta" }
  }
}'
```

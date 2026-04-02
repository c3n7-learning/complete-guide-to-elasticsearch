# Upsert

Basically, to create or update a document

- Run:
  - This will increment the stock if it exists, or just create with the configured values if it doesn't

```
POST /products/_update/101
{
    "script": {
        "source": "ctx._source.in_stock++"
    },
    "upsert": {
        "name": "Blender",
        "price": 399,
        "in_stock": 5
    }
}
```

- Output:

```json
{
  "_index": "products",
  "_id": "101",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 3,
    "successful": 3,
    "failed": 0
  },
  "_seq_no": 10,
  "_primary_term": 1
}
```

- Run:

```
GET /products/_doc/101
```

- Output:

```json
{
  "_index": "products",
  "_id": "101",
  "_version": 1,
  "_seq_no": 10,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "name": "Blender",
    "price": 399,
    "in_stock": 5
  }
}
```

Let's do the entire journey again, and see what happens

- Run:

```
POST /products/_update/101
{
    "script": {
        "source": "ctx._source.in_stock++"
    },
    "upsert": {
        "name": "Blender",
        "price": 399,
        "in_stock": 5
    }
}
```

- Output:

```json
{
  "_index": "products",
  "_id": "101",
  "_version": 2,
  "result": "updated",
  "_shards": {
    "total": 3,
    "successful": 3,
    "failed": 0
  },
  "_seq_no": 11,
  "_primary_term": 1
}
```

- Run:

```
GET /products/_doc/101
```

- Output:

```json
{
  "_index": "products",
  "_id": "101",
  "_version": 2,
  "_seq_no": 11,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "name": "Blender",
    "price": 399,
    "in_stock": 6
  }
}
```

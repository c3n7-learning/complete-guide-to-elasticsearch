# Replacing Documents

We use the `PUT` verb

- Verify the state of the current document:

```
GET /products/_doc/100
```

- Output:

```json
{
  "_index": "products",
  "_id": "100",
  "_version": 10,
  "_seq_no": 9,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "name": "Toaster",
    "price": 49,
    "in_stock": 6,
    "tags": ["electronics"]
  }
}
```

Replace the document

- Run:

```
PUT /products/_doc/100
{
    "name": "Toaster",
    "price": 79,
    "in_stock": 4
}
```

- Run:

```
GET /products/_doc/100
```

- Output:

```json
{
  "_index": "products",
  "_id": "100",
  "_version": 12,
  "_seq_no": 13,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "name": "Toaster",
    "price": 79,
    "in_stock": 4
  }
}
```

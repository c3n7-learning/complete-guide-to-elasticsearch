## Retreiving documents by ID

To do this:

- Run:

```
GET /products/_doc/100
```

- Output:

```json
{
  "_index": "products",
  "_id": "100",
  "_version": 1,
  "_seq_no": 0,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "name": "Toaster",
    "price": 49,
    "in_stock": 4
  }
}
```

- If we didn't have such an id, you would see `found: false`

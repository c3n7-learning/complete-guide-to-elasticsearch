# Deleting Documents

Use the `DELETE` http verb:

- Run:

```
DELETE /products/_doc/100
```

- Verify it has been deleted:

```
GET /products/_doc/100
```

- Output:

```json
{
  "_index": "products",
  "_id": "100",
  "found": false
}
```

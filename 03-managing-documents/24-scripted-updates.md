# Scripted Updates

What if we deduct the value of `in_stock` without having to do a

- Query to determine the current level
- Query to update the value

This can be achieved via `Scripted Updates`

- Run

```
POST /products/_update/100
{
    "script": {
        "source": "ctx._source.in_stock--"
    }
}
```

- `ctx` is shortform for `context`.
- We can access the source document with the `_source` property
- We then deduct one item via `--`.
- You can have multiline strings via `""" """`

Let's see if we were successful:

- Run

```
GET /products/_doc/100
```

- Output

```json
{
  "_index": "products",
  "_id": "100",
  "_version": 8,
  "_seq_no": 7,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "name": "Toaster",
    "price": 49,
    "in_stock": 2,
    "tags": ["electronics"]
  }
}
```

We could also do assignment

- Run

```
POST /products/_update/100
{
    "script": {
        "source": "ctx._source.in_stock = 10"
    }
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
  "_version": 9,
  "_seq_no": 8,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "name": "Toaster",
    "price": 49,
    "in_stock": 10,
    "tags": ["electronics"]
  }
}
```

We could also accept parameters:

- Run

```
POST /products/_update/100
{
    "script": {
        "source": "ctx._source.in_stock -= params.quantity",
        "params": {
            "quantity": 4
        }
    }
}
```

- Run

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

We previously said the value `results: updated` or `results: noop` for updates.

- However, for scripted updates, we will always get a `results: updated`, even if the operation resulted in no changes.
- We can handle this by using an if statement and setting the `ctx.op` value.

The below explicitly sets the value of `ctx.op` to `noop` conditionally:

```
POST /products/_update/100
{
  "script": {
    "source": """
        if (ctx._source.in_stock == 0) {
          ctx.op = 'noop';
        }
        ctx._source.in_stock--;
      """
  }
}
```

The below always yields a value of `updated`, even if the operation yielded no update:

```
POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.in_stock > 0) {
        ctx._source.in_stock--;
      }
    """
  }
}
```

We could also tell ES to delete the document by setting the operation to `delete` by setting the value of the result key:

- This should only be used in situations where you want to use scripting to determine whether the document should be deleted

```
POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.in_stock <= 1) {
        ctx.op = 'delete';
      }

      ctx._source.in_stock--;
    """
  }
}
```

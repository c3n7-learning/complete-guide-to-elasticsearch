# Creating & Deleting Indices

Lets delete the `pages` index we created earlier

```
DELETE /pages
```

Let's create a new index `products`:

- For production, just use the defaults unless you have a good reason not to
- Run:

```
PUT /products
{
    "settings": {
        "number_of_shards": 2,
        "number_of_replicas": 2
    }
}
```

- Output:

```
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "products"
}
```

- `"acknowledged": true` whether the index was created
- `"shards_acknowledged": true` whether the shards were created without timing out. By default, this refers to the primary shard

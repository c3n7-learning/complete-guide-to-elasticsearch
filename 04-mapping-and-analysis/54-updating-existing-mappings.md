# Updating existing mappings

Assume:

- Suppose that product IDs may now include letters
- We need to change the `product_id` field's data type to either `text` or `keyword`
  - We won't use the field for full-text searches
  - We will use it for filtering, so the `keyword` data type is ideal

What if we just try to run this?

- Run:

```json
PUT /reviews/_mapping
{
  "properties": {
    "product_id": {
      "type": "keyword"
    }
  }
}
```

- Output:

```
{
  "error": {
    "root_cause": [
      {
        "type": "illegal_argument_exception",
        "reason": "mapper [product_id] cannot be changed from type [integer] to [keyword]"
      }
    ],
    "type": "illegal_argument_exception",
    "reason": "mapper [product_id] cannot be changed from type [integer] to [keyword]"
  },
  "status": 400
}
```

## Limitations for updating mappings

- Generally, ES field mappings **cannot be changed**
- We can add _new_ field mappings, but that's about it

There are exceptions:

- A few mapping parameters can be updated for existing mappings
- Let's add `"ignore_above": 256`. Strings longer than that will be ignored and not indexed

```
PUT /reviews/_mapping
{
  "properties": {
    "author": {
      "properties": {
        "email": {
          "type": "keyword"
          "ignore_above": 256
        }
      }
    }
  }
}
```

- Being able to update mappings would be problematic for existing documets
  - Text values may have already been analyzed, for instance
  - Changing between some data types would require rebuilding the whole data structure
  - If you have indexed millions of documents, this could take a while
- Even for an empty index, we cannot update a mapping
- Field mappings also cannot be removed, can't delete them
  - Just leave out the field when indexing documents
  - This is if you no longer need data in a field
- The Update By Query API can be used to reclaim disk space
- The solution is to reindex documents into a new index (sorry :-( )

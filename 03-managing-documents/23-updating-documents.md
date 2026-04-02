# Updating Documents

You just use the POST verb, and add the ID

- Run

```
POST /products/_update/100
{
    "doc": {
        "in_stock": 3
    }
}
```

- Output:

```json
{
  "_index": "products",
  "_id": "100",
  "_version": 2,
  "result": "updated",
  "_shards": {
    "total": 3,
    "successful": 3,
    "failed": 0
  },
  "_seq_no": 1,
  "_primary_term": 1
}
```

- You'll see `"result": "updated"`. It could also be `noop` if the was no update needed based on the provided data

Let's retreive it to see if the update picked

- Run

```
GET /products/_doc/100
```

- Output

```json
{
  "_index": "products",
  "_id": "100",
  "_version": 6,
  "_seq_no": 5,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "name": "Toaster",
    "price": 49,
    "in_stock": 3
  }
}
```

TO add new fields to a document

- Run

```
POST /products/_update/100
{
    "doc": {
        "tags": ["electronics"]
    }
}
```

- Then run:

```
GET /products/_doc/100
```

- You'll see

```json
{
  "_index": "products",
  "_id": "100",
  "_version": 7,
  "_seq_no": 6,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "name": "Toaster",
    "price": 49,
    "in_stock": 3,
    "tags": ["electronics"]
  }
}
```

## Documents are immutable

- ES Document are immutable (!)
  - But, didn't we just change an existing document?
- We actually _replaced_ documents in this lecture
- The Update API did some things for us, making it look like we updated documents.

### How the Update API Works

- The current document is retrieved
- The field fvalues are changes
- The existing document is replaced with the modified document
- We could do the exact same thing at the application level
  - The Updates API however does this without requiring the extra network roundtrips

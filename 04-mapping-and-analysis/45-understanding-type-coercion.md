# Understanding type coercion

- Data types are inspected when indexing documents
  - They are validated, and some invalid values are rejected
  - e.g. trying to index an object for a text field
- Coercion: Inspect mapping and coerce into field data type if possible, throw an exception if not.
- Sometimes, providing the wrong data type is okay
  - Let's look at an example...

Let's insert data into the `coercion_test` index. It will be created Implicitly, because we didn't create it before:

Example #1

- Run:

```
PUT /coercion_test/_doc/2
{
    "price": "7.4"
}
```

- Output:

```json
{
  "_index": "coercion_test",
  "_id": "1",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```

Example #2

- Run:

```
PUT /coercion_test/_doc/2
{
    "price": "7.4"
}
```

- Output:
  - This is where type coercion comes into play. The string is coerced into a float

```json
{
  "_index": "coercion_test",
  "_id": "2",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "_seq_no": 1,
  "_primary_term": 1
}
```

Example #3

- Run:

```
PUT /coercion_test/_doc/3
{
    "price": "7.4m"
}
```

- Output:

```json
{
  "error": {
    "root_cause": [
      {
        "type": "document_parsing_exception",
        "reason": "[1:15] failed to parse field [price] of type [float] in document with id '3'. Preview of field's value: '7.4m'"
      }
    ],
    "type": "document_parsing_exception",
    "reason": "[1:15] failed to parse field [price] of type [float] in document with id '3'. Preview of field's value: '7.4m'",
    "caused_by": {
      "type": "number_format_exception",
      "reason": "For input string: \"7.4m\""
    }
  },
  "status": 400
}
```

Example #4:

- Run:

```
GET /coercion_test/_doc/2
```

- Output:

```json
{
  "_index": "coercion_test",
  "_id": "2",
  "_version": 1,
  "_seq_no": 1,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "price": "7.4"
  }
}
```

- Wait, the price is a string!, we thought we coerced it into a float?

## Understanding the `_source` object

- Contains the values that were supplied at index time ("7.4")
  - Not the values that are indexed (7.4)
- Search queries use indexed values, not `_source`
  - ES searches BKD trees, inverted indices, etc
- `_source` does not reflect how values are indexed
  - Keep coercion in mind if you use values from `_source`
  - In this example, it might be either a string or floating point

## A few more things

- Supplying a floating point for an `integer` field will truncate it to an integer
- Coercion is **not** used for dynamic mapping
  - Supplying "7.4" for a new field will create a `text` mapping
- Always try to use the correct data type
  - Especially the first time you index a field
- Disabling coercion is a matter of preference
  - It is enabled by default
  - This course's instructor likes to disable it

## Clean-Up

Delete the index we just created

```
DELETE /coercion_test
```

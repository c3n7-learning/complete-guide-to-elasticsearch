# Understanding arrays

- There is no such thing as an `array` data type
  - It doesn't exist!
- Any field may contain zero or more values
  - No configuration or mapping needed
  - Simply supply an array when indexing a document
- We did this for the `tags` field for the `products` index

Example #1:

```
POST /products/_doc
{
    "tags": "Smartphone"
}
```

Example #2:

```
POST /products/_doc
{
    "tags": [ "Smartphone", "Electronics" ]
}
```

Both of the above are mapped to:

```json
{
  "products": {
    "mappings": {
      "properties": {
        "tags": {
          "type": "text"
        }
      }
    }
  }
}
```

So how is this stored internally? They are simply concatenated before they are analyzed:

- Run:

```
POST /_analyze
{
    "text": ["Strings are simply", "merged together."],
    "analyzer": "standard"
}
```

- Output:

```json
{
  "tokens": [
    {
      "token": "strings",
      "start_offset": 0,
      "end_offset": 7,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "are",
      "start_offset": 8,
      "end_offset": 11,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "simply",
      "start_offset": 12,
      "end_offset": 18,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "merged",
      "start_offset": 19,
      "end_offset": 25,
      "type": "<ALPHANUM>",
      "position": 103
    },
    {
      "token": "together",
      "start_offset": 26,
      "end_offset": 34,
      "type": "<ALPHANUM>",
      "position": 104
    }
  ]
}
```

- Note that the offsets for `merged` and `together` don't start from zero, rather, they continue from the last string:

```
"Strings are simply merged t".length - 1
26
```

- The strings are concatenated with a space between

## Constraints

- Array values should of the same data type
- You cannot mix strings and integers in an one array value
  - You can mix however if the types can be coerced into the correct data types
- Array values `should` be of the same data type
- Coercion only works for fields that are already mapped
  - If creating a field mapping with dynamic mapping, an array **must** contain the same data type
- I don't recommend coercion, _atleast on purpose_

Correct data types

```
✅ [ "electronics", "expensive", "popular" ]
✅ [ 37, 45, 9 ]
✅ [ true, false, true ]
✅ [ { "name": "Coffee Maker" }, { "name": "Toaster" }, { "name": "Blender" } ]
```

With Coercion:

```
⚠️ [ true, false, "true" ]
⚠️ [ "electronics", "expensive", 47 ]
⚠️ [ 37, 45, "9" ]
⚠️ [ true, false, "true" ]
```

Cannot coerce:

```
❌ [ { "name": "Coffee Maker" }, { "name": "Toaster" }, false ]
```

## Nested arrays

- Arrays may contain nested arrays
- Arrays are flattened during indexing
- `[ 1, [2, 3] ]` becomes `[ 1, 2, 3 ]`

## A small reminder

- Remember to use the `nested` data type for arrays of objects if you need to query the objects independently

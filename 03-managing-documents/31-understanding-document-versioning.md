# Understanding document versioning

## Introduction to versioning

- Is a VERY simple versioning system
- Not a revision history of documents.
- ES stores a `_version` metadata field with every document
  - The value is an integer
  - Is incremented by one when modifying a document
  - The value is retained for 60 seconds when deleting a document
    - Configured with the `index.gc_deletes` setting
  - The `_version` field is returned when retrieving documents

## Types of versioning

- The default versioning type is called _internal_ versioning.
- There is also an _external_ versioning type
  - Useful when versions are maintained outside of ES
  - E.g. when documents are also stored in a RDBMS

```
PUT /products/_doc/123?version=521&version_type=external
{
  "name": "Coffee Maker",
  "price": 64,
  "in_stock": 10
}
```

- External versioning is done by specifying the `version` and `version_type` query params
  - The `version` must be a natural number

## What's the point of versioning?

- You can tell how many times a document has been modified
  - Probably not that useful
- Versioning is hardly used anymore, and is mostly a thing from the past
- It was previously the way to do optimistic concurrency control
  - Now there is a better way, though
- You might see this field being used for clusters running old versions.

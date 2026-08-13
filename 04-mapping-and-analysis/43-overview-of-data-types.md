# Overview of data types

Reference:

- https://www.elastic.co/docs/reference/elasticsearch/mapping-reference/field-data-types

They include:

- object
- boolean
- double
- float
- integer
- short
- text
- long
- date
- ... and more

## `object` data type

- Used for any JSON object
- Objects may be nested
- Mapped using the `properties` parameter

This json object:
```json
{
  "name": "Coffee Maker",
  "price": 64.2,
  "in_stock": 10,
  "is_active": true,
  "manufacturer": {
    "name": "Nespresso",
    "country": "Switzerland"
    "owned_by": {
      "name": "Nestle Group",
      "country": "Switzerland"
    }
  }
}
```

Can be mapped like:
```
PUT /products
{
  "mappings": {
    "...": "...",
    "properties": {
      "manufacturer": {
        "name": { "type": "text" },
        "country": { "type": "text" },
        "owned_by": {
          "properties": {
            "name": { "type": "text" },
            "country": { "type": "text" }
          }
        }
      }
    }
  }
}
```

- Objects are **not** stored as objects in Apache Lucene
  - Objects are transformed to ensure that we can index any valid JSON
  - In particulary, objects are flattened

From:

```json
{
  "name": "Coffee Maker",
  "price": 64.2,
  "in_stock": 10,
  "is_active": true,
  "manufacturer": {
    "name": "Nespresso",
    "country": "Switzerland"
  }
}
```

To:

```json
{
  "name": "Coffee Maker",
  "price": 64.2,
  "in_stock": 10,
  "is_active": true,
  "manufacturer.name": "Nespresso",
  "manufacturer.country": "Switzerland"
}
```

What if we try to index an array of objects?

- From:

```json
{
  "name": "Coffee Maker",
  "reviews": [
    {
      "rating": 5.0,
      "author": "Average Joe",
      "description": "Haven't slept for days... Amazing!"
    },
    {
      "rating": 3.5,
      "author": "John Doe",
      "description": "Could be better :)"
    }
  ]
}
```

- To:

```json
{
  "name": "Coffee Maker",
  "reviews.rating": [5.0, 3.5],
  "reviews.author": ["Average Joe", "John Doe"],
  "reviews.description": [
    "Haven't slept for days... Amazing!",
    "Could be better :)"
  ]
}
```

- This leads to problems though, e.g. if we try to to this query

```
QUERY: MATCH products WHERE reviews.author == "John Doe" AND reviews.rating >= 4.0
```

- With this, the document is returned as a match, even though John Doe never rated anything over 4.0

How do we solve this? with the `nested` data type

## `nested` data type

- Similar to the `object` data type, but maintains object reltionships
  - Useful when indexing arrays of objects
- Enables us to query objects independently
  - Must use the `nested` query.
- Apache Lucene knows nothing about this though
  - `nested` objects are stored as hidden documents.

We define the schema as:

```
PUT /products
{
    "mappings": {
        "properties": {
            "name": { "type": "text" },
            "reviews": { "type": "nested" }
        }
    }
}
```

Then when we store the data, ES stores is like:

- With this we end up with 3 documents. 1 for the coffee maker, 2 for the reviews.

```json
{
  "name": "Coffee Maker",
  "reviews": [
    {
      "rating": 5.0,
      "author": "Average Joe",
      "description": "Haven't slept for days... Amazing!"
    },
    {
      "rating": 3.5,
      "author": "John Doe",
      "description": "Could be better :)"
    }
  ]
}
```

Now this query (only a Pseudo-query), will behave as we expect

```
QUERY: MATCH products WHERE reviews.author == "John Doe" AND reviews.rating >= 4.0
```

## `keyword` data type

- Used for exact matching of values
- Typically used for filtering, aggregations, and sorting
- E.g. searching for articles with a status of `PUBLISHED`
- For full-test searched, use the `text` data type instead
  - e.g. searching the body text of an article

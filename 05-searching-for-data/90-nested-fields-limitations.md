# Nested Fields Limitations

## Performance

- Indexing and querying `nested` fields is more expensive than for other data types
- An Apache Lucene document is created for each nested object.
  - If you index 1m documents with 10 nested objects, you end up with 11m internal Apache Lucene documents
  - Increased storage & query costs
- Important to remember this when dealing with large datasets
- ES provides safeguards to reduce the risk of performance bottlenecks

## Limitaitons

- We need to use a specialized data type (`nested`) and query (`nested`)
  - They want us to be explicit
- Max 50 `nested` fields per index (the default)
  - Can be increased with the `index.mapping.nested_fields.limit` setting (not recommended)
- 10,000 nested objects per document (across all `nested` fields).
  - Can be increased with `index.mapping.nested_objects.limit` setting (not recommended)

Assume we are running a service like shopify where we provide a webshop for companies

- Bad Mapping:
  - We have two nested fields `orders` and `lines`. This could get nasty
  - It's only a matter of time before the store receives 10k orders or even less, since the `lines` object is nested

```
{
  "store_id": "ob78s35qd",
  "name": "Pizza Shack",
  "website": "https://pizza-shack.com",
  "orders": [
    {
      "order_id": "order-6821",
      "customer": "Bo Andersen",
      "ordered_at": "2022-01-26T23:51:51Z",
      "comment": "Late night snack ;-)",
      "lines": [
        {
          "product": "Veggie Deluxe",
          "quantity": 1
        },
        {
          "product": "Meat Lover",
          "quantity": 1
        },
        {
          "product": "French Fries",
          "quantity": 1
        },
        ...
      ]
    },
    ...
  ]
}
```

Good Mapping:

- Split the store and orders into two distinct things
- Store

```
{
  "store_id": "ob78s35qd",
  "name": "Pizza Shack",
  "website": "https://pizza-shack.com",
  ...
}
```

- Order

```
{
  "order_id": "order-6821",
  "store_id": "ob78s35qd",
  "customer": "Bo Andersen",
  "ordered_at": "2023-01-01T23:51:51Z",
  "comment": "Soooo hungry!!!",
  "lines": [
    {
      "product": "Veggie Deluxe",
      "quantity": 1
    },
    {
      "product": "Meat Lover",
      "quantity": 1
    },
    {
      "product": "French Fries",
      "quantity": 1
    },
    ...
  ]
}
```

## Lecture Summary

- Keep performance in mind when using `nested` fields; they are not free
  - If you map documents well, you should be all good, though
  - Denormalizing data is often a good idea
  - ES has a few settings to prevent things from going wrong
- An index can have up to 50 `nested` fields by default
- A document can have up to 10,000 nested objects by default
  - This includes all `nested` fields

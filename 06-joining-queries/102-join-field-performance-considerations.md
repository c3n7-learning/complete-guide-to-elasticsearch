# Join Field Performance Considerations

## Tl; DR;

Using join fields is slow!

## More details

- Avoid join fields whenever possible, except for a few scenarios
- That's a slight simplification, as it depends on the data you're dealing with

In more detail:

- The more the child documents pointing to unique parents, the slower the `has_child` query is
  - Basically, the more the documents, the slower the query
  - You might not notice this before its a problem
    - You'll start to see this when you get more and more documents
- The number fo parent documents slows down the `has_parent` query
- Each level of document relations adds an overhead to queries (multi-level queries)

In some scenarios, the performance of join fields won't be bad:

- A one-to-many relationship between two document types, where one type has many more documents than the other
  - e.g. recipes as parent documents and ingredients as child documents. As there would be significantly more ingredients than recipes

Two questions remain:

1. If using join fields is generally a bad idea, why did we spend an hour talking about it?
   - In some cases, using join fields is completely find
     - If you don't have lots of documents, and won't have many documents anytime soon, performance will be fine
   - Join fields are paret of ES, regardless of how often they are used or not
   - You need to decide if the tool is right for your particular job
2. If join fields are bad, how do we map document relationships _properly_?
   - Consider using the nested data type

## Summary

- Generally speaking, just don't ;-)
- ES searches loads of data fast
- ES does this differently from relational databases
- Store data in ways optimized for quickly searching through it

Consider mapping employees and their addresses:

- instead of addresses being documents, make them nested objects or regular fields
- denormalize the data instead of mapping document relationships
- unless you have a good reason to use join fields, try to denormalize your data instead

```json
{
  "firstName": "Bo",
  "lastName": "Andersen",
  "street": "Wall Street",
  "postalCode": 1234
}
```

# Document Counts are Approximate

- Document counts are approximate and not guaranteed to be accurate

Why?

- Because of the distributed nature of ES
- An index may be distributed across shards
- If we provide a size of 3, each shard is asked to return it's most relevant top 3 results
- The coordinating node then adds these values up,and gives us a total

Assume we have these orders distributed across the shards, and each order has a product.

- Assume then we aggregate on the product name

|     | SHARD A        | SHARD B        | SHARD C        |
| --- | -------------- | -------------- | -------------- |
| 1   | PRODUCT A (50) | PRODUCT A (50) | PRODUCT A (50) |
| 2   | PRODUCT B (40) | PRODUCT B (40) | PRODUCT E (40) |
| 3   | PRODUCT C (30) | PRODUCT F (30) | PRODUCT F (30) |
| 4   | PRODUCT F (20) | PRODUCT C (20) | PRODUCT B (20) |
| 5   | PRODUCT D (10) | PRODUCT E (10) | PRODUCT C (10) |

Assume we query for the top 3 products:

- Each shard will then take the products with the 3 highest product count, and send this to the coordinating node
- The coordinating node then sums the values and orders them following the default ordering direction, provided we don't override the sorting order
- But when we merge the values, we see the sums are off!
- The coordinating node only sees the values it gets from the shards, not the values in the shards themselves
- The accuracy goes higher, the larger the `size` of the document is, the higher the computational burden. The vice versa is also true.
  - The default value is 10, so you get a relatively higher accuracy
  - For low values e.g. 3, you get a relatively lower accuracy

|     | ACTUAL RESULT   | CORRECT RESULT  |
| --- | --------------- | --------------- |
| 1   | PRODUCT A (150) | PRODUCT A (150) |
| 2   | PRODUCT B (80)  | PRODUCT B (100) |
| 3   | PRODUCT F (60)  | PRODUCT F (80)  |

If all the data for an index is in a single shard, then this problem won't apply for you

## `doc_count_error_upper_bound`

- This key contains a number representing the maximum possible document count for a term that was not part of the final results

For the table above?

- ES takes the document count from the last term that was returned from each shard. In our case, it would be:

```
90 = PRODUCT C (30) + PRODUCT F (30) + PRODUCT F (30)
```

- This error provides us with an error margin
  - if we use it, maybe it could give us some nice insights into behind-the-scene things

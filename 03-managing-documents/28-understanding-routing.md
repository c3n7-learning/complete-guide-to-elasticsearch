# Introduction to Routing

- How does ES know where to store documents?
- How are documents found once they have been indexed?
- The answer is routing
- Routing is the process of resolving a shard for a document

ES uses the following formula to determine on which shard to store the document

```
shard_num = hash(_routing) % num_primary_shards
```

- The same formula also uses the same formula to get/update/deleting documents.
- This formula works when searching by ID
- The formula changes when using other searching parameters

## Custom Routing

- Routing is 100% transparent when using ES
- This makes ES easier to use and learn
- It is possible to customize routing, though, for various purposes
  - That's quite complicated, so let's leave that for later
- The default routing strategy ensures that documents are distributed evenly across shards.
- If we customize the routing strategy, we might end up with shards larger than other shards.

Good-to-know fact:

- Remember when we said the number of shards can not be changed once an index has been created?
- It is because the number of shards is used in the formula (`num_primary_shards`)
  - New documents would be okay
  - Existing documents would be more difficult for ES to locate
- ES might end up trying to retreive documents from the wrong shard, and lead to problems

```
shard_num = hash(_routing) % num_primary_shards
```

- Modifying the number of shards, you have to delete an index and recreate it.

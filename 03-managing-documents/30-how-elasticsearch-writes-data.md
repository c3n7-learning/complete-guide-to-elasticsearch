# How Elasticsearch writes data

```
PUT /products/_doc/100
           |
           ↓
      +---------+
      |  Node X |
      |(coordin-|
      | ating   |
      | node)   |
      +---------+
           |
           ↓
      +-----------+
      |Pri Shard B|        Routing
      +-----------+
           |
          /
         /
        ↓
+--------------------------------------------------+
| +----------+ +----------+ +----------+ +--------+ |
| | Primary  | | Replica  | | Replica  | |Replica | |
| | Shard B  | |    B1    | |    B2    | |   B3   | |
| |   [✓]    | |   [✓]    | |   [✓]    | |  [✓]   | |
| +----------+ +----------+ +----------+ +--------+ |
+--------------------------------------------------+
      ↑             ↑             ↑             ↑
       \___________/ \___________/ \___________/
                   Replication
```

1. A given node receives a read request. This is called the `coordinating node`
   - This involves figuring on which shard the document is stored, via routing.
   - This yields the primary shard
2. The write request is always routed to the `primary shard`
3. The primary shard validates the request
4. The primary shard then performes the write operation locally
5. The primary shard forwards the operation to the replica shard in parallel.
   - The operation will succeed on the primary shard even if the operation can not be done successfully in the replica shards.

### How does ES handle failures?

Lots of things could go wrong, e.g. hardware failures.
Suppose:

- We are doing a write operation
- The write operation happens on the primary replica
- The primary replica sends the operation to the other replicas R1 and R2
- R1 picks the operation, but the shard for the primary replica then unfortunately dies before R2 can pick it

ES goes through a recovery process

- One of the replica shards is promoted to be the new primary shard. This could be R2, even though it is not up to date, though it thinks it is
- This makes it weird, as R1 has a more recent update than R2

ES Resolves this via

- **Primary terms and sequence numbers**

### Primary terms

- A way to distinguish between old and new primary shards.
- Essentially a counter for how many times the primary shard has changed.
- Is a counter that increments whenever a new primary shard is assigned
  \_ Every the primary shard changes, the `_primary_term` value is incremented e.g.
  - `_primary_term: 1`
  - `_primary_term: 2`
- A Sequence number is also given for each operation
  - `_seq_no: 491`

### Sequence numbers

- Appended to write operations together with the primary term
- Essentially a counter that is incremented for each write operation
- The primary shard incrases the sequence number
- Enable ES to order write operations

### Recovering when a primary shard fails

- Primary terms and sequence numbers are key when ES needs to recover from a primary shard failure
  - Enables ES to more efficiently figure out which write operations need to be applied
- For large indices, this process is really expensive
  - To speed thins up, ES uses checkpoints

### Global and local checkpoints

- Essentially sequence numbers
- Each replication group has a _global_ checkpoint
- Each replication shard has a _local_ checkpoint
- _Global checkpoints_
  - The sequence number that all active shards within a replication group have been aligned at least up to
- Local checkpoints
  - The sequence number for the last write operation that was performed.

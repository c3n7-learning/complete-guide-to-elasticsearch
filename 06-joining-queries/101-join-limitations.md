# Join Limitations

- Documents we're joining must be stored within the same index
- Parent & child documents must be on the same shard
- There can only be one join field per index
  - A join field can have as many relations as you want
- New relations can be added after creating the index
- Child relations can only be added to existing parents
- A document can only have one parent
  - e.g An employee can belong to only one department
  - But multiple children is possible, i.e. many-to-one

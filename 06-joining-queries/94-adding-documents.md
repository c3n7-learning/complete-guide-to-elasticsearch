# Adding Documents

## Adding departments

- When adding documents for a relation, we have to specify which relation this document is a part of
- Since our relation ideally is a "Department" has many "Employees", for departmeent, we just have to specify the name of the relation

```http
PUT /department/_doc/1
{
  "name": "Development",
  "join_field": "department"
}
```

```http
PUT /department/_doc/2
{
  "name": "Marketing",
  "join_field": "department"
}
```

- We are using the _lean_ syntax. If you wanted to be verbose you could:

```json
{
  "join_field": {
    "name": "department"
  }
}
```

## Adding employees for departments

- Since this is a child, we have to specify the details of the parent
  - `name: employee` tells ES that the document is an employee
  - `parent: 1`: id of the parent document

Run:

```
PUT /department/_doc/3
{
  "name": "Bo Andersen",
  "age": 28,
  "gender": "M",
  "join_field": {
    "name": "employee",
    "parent": 1
  }
}
```

Output:

```json
{
  "error": {
    "root_cause": [
      {
        "type": "document_parsing_exception",
        "reason": "[1:111] failed to parse: [routing] is missing for join field [join_field]"
      }
    ],
    "type": "document_parsing_exception",
    "reason": "[1:111] failed to parse: [routing] is missing for join field [join_field]",
    "caused_by": {
      "type": "illegal_argument_exception",
      "reason": "[routing] is missing for join field [join_field]"
    }
  },
  "status": 400
}
```

- Routing is a way for ES to tell on which child a document with a given ID is stored
- The default behavior is to use a document id
- We need to add a query parameter matching the parent's ID.
  - Why? the parent and child documents must be stored on the same shard
- Since we haven't explicitly specified the routing, ES will attempt to store the shard to a parent document with id 3 (the child document's id), which is wrong!

Run:

```
PUT /department/_doc/3?routing=1
{
  "name": "Bo Andersen",
  "age": 28,
  "gender": "M",
  "join_field": {
    "name": "employee",
    "parent": 1
  }
}
```

```
PUT /department/_doc/4?routing=2
{
  "name": "John Doe",
  "age": 44,
  "gender": "M",
  "join_field": {
    "name": "employee",
    "parent": 2
  }
}
```

```
PUT /department/_doc/5?routing=1
{
  "name": "James Evans",
  "age": 32,
  "gender": "M",
  "join_field": {
    "name": "employee",
    "parent": 1
  }
}
```

```
PUT /department/_doc/6?routing=1
{
  "name": "Daniel Harris",
  "age": 52,
  "gender": "M",
  "join_field": {
    "name": "employee",
    "parent": 1
  }
}
```

```
PUT /department/_doc/7?routing=2
{
  "name": "Jane Park",
  "age": 23,
  "gender": "F",
  "join_field": {
    "name": "employee",
    "parent": 2
  }
}
```

```
PUT /department/_doc/8?routing=1
{
  "name": "Christina Parker",
  "age": 29,
  "gender": "F",
  "join_field": {
    "name": "employee",
    "parent": 1
  }
}
```

# Multi-Level Relations

We'll create this multi-level relationship below

- `company->department->employee`: (multi-level relationship)
- `company->supplier`: (single-level relationship)

```
                    Company
                       ↑
                       |
           ←-----------+-----------→
           |                       |
           ↓                       ↓
      Department                Supplier
           ↑
           |
           |
           ↓
       Employee
```

## Creating the index with mapping

- We specify the company relation type, which will have `department` and `supplier`
  - `company` -> `has_many` -> `departments`, `supplier`
  - `department` -> `has_many` -> `departments`

```http
PUT /company
{
  "mappings": {
    "properties": {
      "join_field": {
        "type": "join",
        "relations": {
          "company": ["department", "supplier"],
          "department": "employee"
        }
      }
    }
  }
}
```

## Test Data

### Adding a company

```
PUT /company/_doc/1
{
  "name": "My Company Inc.",
  "join_field": "company"
}
```

### Adding a department

```
PUT /company/_doc/2?routing=1
{
  "name": "Development",
  "join_field": {
    "name": "department",
    "parent": 1
  }
}
```

### Adding an employee

- An employee is the grandchild of a company, so `routing=1` maps this employee to the root parent in the highest level of the hierarchy.
- This is key in

```
PUT /company/_doc/3?routing=1
{
  "name": "Bo Andersen",
  "join_field": {
    "name": "employee",
    "parent": 2
  }
}
```

### Adding some more test data

```
PUT /company/_doc/4
{
  "name": "Another Company, Inc.",
  "join_field": "company"
}
```

```
PUT /company/_doc/5?routing=4
{
  "name": "Marketing",
  "join_field": {
    "name": "department",
    "parent": 4
  }
}
```

```
PUT /company/_doc/6?routing=4
{
  "name": "John Doe",
  "join_field": {
    "name": "employee",
    "parent": 5
  }
}
```

## Example of querying multi-level relations

We want a company with a department with an employee called "John Doe"

- match departments with the child of type `department`
  - this `department` must match the query nested under it, namely `has_child`
    - `department.has_child` matches relation of type `employee` which has name `John Doe`

- Run:

```
GET /company/_search
{
  "query": {
    "has_child": {
      "type": "department",
      "query": {
        "has_child": {
          "type": "employee",
          "query": {
            "term": {
              "name.keyword": "John Doe"
            }
          }
        }
      }
    }
  }
}
```

- Output:

```json
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "company",
        "_id": "4",
        "_score": 1,
        "_source": {
          "name": "Another Company Inc.",
          "join_field": "company"
        }
      }
    ]
  }
}
```

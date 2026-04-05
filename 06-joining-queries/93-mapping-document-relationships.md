# Mapping Document Relationships

We use the `join_field` for this:

- `type`: the type of the relationship
- `relations`: the possible relations betweent the different types of documents

```http
PUT /department/_mapping
{
  "properties": {
    "join_field": {
      "type": "join",
      "relations": {
        "department": "employee"
      }
    }
  }
}
```

- We can now take advantage of this relation when adding documents
- We could have more than one relation if we wanted

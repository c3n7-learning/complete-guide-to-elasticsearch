# Introduction to mapping

- Defines the structure of documents, (e.g fields, and their data types)
  - Also used to configure how values are indexed
- Similar to a table's schema in a relational database
- MySQL:

```sql
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(255) NOT NULL,
    last_name VARCHAR(255) NOT NULL,
    dob DATE,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

- Elasticsearch:

```
PUT /employees
{
  "mappings": {
    "properties": {
      "id": { "type": "integer" },
      "first_name": { "type": "text" },
      "last_name": { "type": "text" },
      "dob": { "type": "date" },
      "description": { "type": "text" },
      "created_at": { "type": "date" }
    }
  }
}
```

Mapping is pretty cool and not as boring as it sounds. There are two approaches

- Explicit mapping: shown in the diagram shown above.
  - We define field mappings ourselves
- Dynamic mapping: ES generates field mappings for us

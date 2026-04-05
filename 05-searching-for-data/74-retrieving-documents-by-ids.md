# Retrieving documents by IDs

Let's query for multiple documents by ids:

- SQL Equivalent: `SELECT * FROM products WHERE _id IN ("100", "200", "300");`

```
GET /products/_search
{
    "query": {
        "ids": {
            "values": ["100", "200", "300"]
        }
    }
}
```

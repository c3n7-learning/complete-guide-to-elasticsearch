# Sorting by multi-value fields

We can sort by fields containing more than one value

- Remember, fields can contain 0 or more values by default

## Sorting by the average rating (descending)

- Ratings with the highest average rating
- we have `avg`, `max`, `min`, `median` and `sum`
  - `median` is for numeric data types
  - `max` and `min` works for dates as wel

```
GET /recipes/_search
{
  "_source": "ratings",
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "ratings": {
        "order": "desc",
        "mode": "avg"
      }
    }
  ]
}
```

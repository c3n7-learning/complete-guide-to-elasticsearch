# Introduction to this section

- Assume an address has many employees

```
┌─────────────────────┐                                    ┌─────────────────────┐
│      employee       │                                    │       address       │
├─────────────────────┤                                    ├─────────────────────┤
│ id                  │                                    │ id                  │
│ first_name          │                                    │ street_name         │
│ last_name           │ ─────────────────────────────────> │ postal_code         │
│ phone_number        │                                    │ city                │
│ address_id          │                                    │ ...                 │
│ ...                 │                                    │                     │
└─────────────────────┘                                    └─────────────────────┘
```

- Or a city has many departments

```
┌─────────────────────┐                                    ┌─────────────────────┐
│     department      │                                    │        city         │
├─────────────────────┤                                    ├─────────────────────┤
│ id                  │                                    │ id                  │
│ name                │ ─────────────────────────────────> │ name                │
│ city_id             │                                    │ ...                 │
└─────────────────────┘                                    └─────────────────────┘
```

This is NOT how we store data in elasticsearch.

- We tend to denormalize data in ES. This leads to better performance
- We store data as we would in NoSQL

```json
{
  "firstName": "text",
  "lastName": "text",
  "phoneNumber": "text",
  "address": {
    "streetName": "text",
    "postalCode": "text",
    "city": "text"
  }
}
```

Isn't this inefficient, disk wise?

- Yes it is! Storing a city for every department?
- Remember though, **Don't use ES as a primary data store**
- This allows us to store data however we want:
  - **Optimize search performance by denormalizing data**
  - Performance > disk space
- ES ony supports simple joins
- **Joins are expensive!**
  - They are VERY inefficient

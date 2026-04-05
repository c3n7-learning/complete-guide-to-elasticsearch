# Range Searches

## Introduction

- The range query is used to perform range searches
- e.g. `in_stock >= 1` and `in_stock <= 5`
- e.g. `created_at >= 2020/01/01` and `created_at <= 2020/01/31`

## Demos

### Querying numeric ranges

#### Products that are almost sold out

SQL Equivalent:

- `WHERE in_stock >= 1 AND in_stock <= 5`

Run:

```
GET /products/_search
{
    "query": {
        "range": {
          "in_stock": {
            "gte": 1,
            "lte": 5
          }
        }
    }
}
```

Boundaries not included:

- SQL equivalent: `WHERE in_stock > 1 AND in_stock < 5`

```
GET /products/_search
{
    "query": {
        "range": {
          "in_stock": {
            "gt": 1,
            "lt": 5
          }
        }
    }
}
```

#### Parameters

| Parameter | Math symbol | SQL equivalent | Description              |
| --------- | ----------- | -------------- | ------------------------ |
| gt        | >           | >              | Greater than             |
| gte       | ≥           | >=             | Greater than or equal to |
| lt        | <           | <              | Less than                |
| lte       | ≤           | <=             | Less than or equal to    |

### Querying dates

#### Dates without time

ES would use sensible defaults when working with dates

- E.g for `gte`, it would use midnight for the timestamp
- e.g. for `lte`, it would use 1 sec before midnight

```
GET /products/_search
{
    "query": {
        "range": {
          "created": {
            "gte": "2010/01/01",
            "lte": "2020/01/31"
          }
        }
    }
}
```

#### Dates with time (timestamps)

It would be advisable to specify timestamps though explicitly

```
GET /products/_search
{
    "query": {
        "range": {
          "created": {
            "gte": "2010/01/01 00:00:00",
            "lte": "2020/01/31 23:59:59"
          }
        }
    }
}
```

#### Specifying a date forma

The default date format is

```
yyyy/MM/dd HH:mm:ss||yyyy/MM/dd||epoch_millis
```

If we want to override that, use the `format` parameter

```
GET /products/_search
{
    "query": {
        "range": {
          "created": {
            "format": "dd/MM/yyyy",
            "gte": "01/01/2010",
            "lte": "31/01/2020"
          }
        }
    }
}
```

#### Specifying a UTC offset

By default, ES assumes the provided dates are in the UTC timezone.

- To override this, use the `time_zone` parameter:

```
GET /products/_search
{
    "query": {
        "range": {
          "created": {
            "time_zone": "+01:00",
            "gte": "2010/01/01 01:00:00",
            "lte": "2020/02/01 00:59:59"
          }
        }
    }
}
```

- Behind the scenes, ES will convert the dates to UTC

```
"gte": "2010/01/01 00:00:00",
"lte": "2020/01/31 23:59:59"
```

## Lecture Summary

- Use the `range` query to perform range searches
- Specify one or more of `gt`, `gte`, `lt`, or `lte` parameters
- Supports both numbers and dates
- Dates are automatically handled for `date` fields
  - Specifying the time is optional, but recommended if possible
  - Custom formats are supported through the format parameter
  - Time zones are handled with the `time_zone` parameter (UTC offset)

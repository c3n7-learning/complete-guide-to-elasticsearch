# Searching Multiple Fields

Reference

- https://www.elastic.co/docs/reference/query-languages/query-dsl/query-dsl-multi-match-query

## Basic Usage

This searches `vegetable` in either of the `name` and `tags` fields.

- Run:

```
GET /products/_search
{
    "query": {
        "multi_match": {
          "query": "vegetable",
          "fields": ["name", "tags"]
        }
    }
}
```

## Per-field relevance boosting

What if we want to boost one field?

- We can do that with `^` then passing a number. In the example below, name will be boosted by a factor of 2.
- Documents with `vegetable` in the name field will have the relevance score multiplied by two.

Run:

```
GET /products/_search
{
    "query": {
        "multi_match": {
          "query": "vegetable",
          "fields": ["name^2", "tags"]
        }
    }
}
```

## How the query works

Behind the scenes, this query:

```
GET /products/_search
{
  "query": {
    "multi_match": {
      "query": "vegetable",
      "fields": ["name", "description"]
    }
  }
}
```

Is translated into this:

- If atleast one of the query matches, the document is added to the document

```
GET /products/_search
{
  "query": {
    "match": {
      "name": "vegetable"
    }
  }
}

GET /products/_search
{
  "query": {
    "match": {
      "description": "vegetable"
    }
  }
}
```

## Default relevance scoring

This is pretty advanced stuff, we'll just look at a basic version of how default relevance works

Assume we search for two terms.

- This query:

```
GET /products/_search
{
  "query": {
    "multi_match": {
      "query": "vegetable broth",
      "fields": ["name", "description"]
    }
  }
}
```

- Is translated into:

```
GET /products/_search
{
  "query": {
    "match": {
      "name": "vegetable broth"
    }
  }
}


GET /products/_search
{
  "query": {
    "match": {
      "description": "vegetable broth"
    }
  }
}
```

- When the query is run and we might get the documents below:

```
{
  "name": "Vegetable Broth", 12.698752
  "description": "Can be used to make vegetable soup." 8.510115
}

{
  "name": "Vegetable Soup (Spicy)", 10.623691
  "description": "Spicy stuff. Careful!" N/A
}

{
  "name": "Vegetable Spinach Lasagne", 9.9819723
  "description": "Spinach is very healthy." N/A
}
```

- Documents with vegetable in both the name and description get a higher score
- The multimatch query will evaluate the score for each of the matched field, and pick the higher score
- Documents are then sorted by the highest score

## Specifying a tie breaker

- By default, one field is used for calculating a document's relevance score
- We can "reward" documents where multiple fields match with the `tie_breaker` parameter
  - Each matching field affects the relevance score

Consider this query:

```
GET /products/_search
{
  "query": {
    "multi_match": {
      "query": "vegetable broth",
      "fields": ["name", "description"],
      "tie_breaker": 0.3
    }
  }
}
```

And this document:

```
{
  "name": "Vegetable Broth", 12.698752
  "description": "Can be used to make vegetable soup." 8.510115
}
```

Score calculation:

- By default, ES would pick the highest score: (12.699)
- With the `tie_breaker` param, the calculation would be:

```
12.698752
+ 2.5530345 (8.510115 * 0.3)
---------------------------
= 15.251787
```

- The relevance score is then made to be `15.251`

For more than two fields, it would look like:

- The query:

```
GET /products/_search
{
  "query": {
    "multi_match": {
      "query": "vegetable broth",
      "fields": ["name", "description", "tags", "ingredients"],
      "tie_breaker": 0.3
    }
  }
}
```

- The document:

```
{
  "name": "Vegetable Broth", 12.698752
  "description": "Can be used to make vegetable soup." 8.510115
  "tags": ["Broth", "Cooking"], 11.671279
  "ingredients": ["Water", "Carrot", "Celery", "Onion", "Salt"] N/A
}
```

- The relevance score:

```
12.698752
+ 2.5530345 (8.510115 * 0.3)
+ 3.5013837 (11.671279 * 0.3)
--------------------------------
= 18.7531707
```

## Lecture Summary

- The `multi_match` query performs full text searches on multiple fields.
  - A document matches if atleast one field is matched
- Individual fields can be relevance boosted by modifying the field name (^)
- Internally, ES rewrites the query by modifying the field name (^)
- By default, the best matching field's relevance score is used for the document
  - Can be configured with the `type` parameter (see attached link)
- A tie breaker can be added to factor in every matching field's relevance score

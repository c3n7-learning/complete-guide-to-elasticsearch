# Introduction to term level queries

## Introduction

- One group of ES queries is called _term level_ queries
- Used to search structured data for exact values (filtering)
  - e.g. finding products where the brand equals "Nike"
- **Term level queries are not analyzed**
  - The search value is used exactly as is for inverted index lookups

```
# Document #1

{
    "name": "Nike SB Charge SLR",
    "color.keyword": ["Black", "White"],
    "brand.keyword": "Nike",
    "price": 84.0
}

            |
            |
# Inverted index for brand.keyword

TERM                    DOCUMENT #1
"Nike"                  X
```

Assume we search for this:

```
GET /products/_search
{
  "query": {
    "term": {
      "brand.keyword": "Nike"
    }
  }
}
```

We will get these results for the respective searches:

- ✅ "Nike" equals "Nike"
- ❌ "nike" does not equal "Nike"
- ❌ "Nike" does not equal "Nike, Inc."

Can be used with data types such as `keyword`, numbers, dates, etc

- Just **don't** use them with text fields!
- Assume we have the following document

```
# Document #1
{
  "name": "Nike SB Charge SLR",
  "color.keyword": ["Black", "White"],
  "brand.keyword": "Nike",
  "price": 84.0
}
        |
        |
        ↓
# Inverted index for name
TERM                    DOCUMENT #1
"nike"                  X
"sb"                    X
"charge"                X
"slr"                   X
```

And we attempt to do the following query:

```
GET /products/_search
{
  "query": {
    "term": {
      "name": "Nike"
    }
  }
}
```

- ❌ "Nike" will not be found, our inverted index has only `nike`
- ✅ "nike" equals "nike"

What if we try to search the product name?

```
GET /products/_search
{
  "query": {
    "term": {
      "name": "Nike SB Charge SLR"
    }
  }
}
```

- This won't be found, as our inverted index doesn't have a single term with all those characters combined

| TERM     | DOCUMENT #1 |
| -------- | ----------- |
| "nike"   | X           |
| "sb"     | X           |
| "charge" | X           |
| "slr"    | X           |

- ❌ The term `Nike SB Charge SLR` is not found

## So what's the big deal

- Results can seem strange and unpredictable (but of course they aren't, if you know how things work)
- Queries don't fail, they just behave as expected
- A query for "nike" works fine, but "Nike" doesn't match anaything
- Can be hard to debug if you don't understand how the query works
- _Many_ people have run into this - don't be the next one
- **TL;DR:** - Don't use term level queries on `text` fields. That's it!

## Lecture summary

- Term level queries are used for exact matching (filtering)
- Term level queries are not analyzed
  - The seach value is used exactly as is for inverted index lookups
  - Searches are therefore case sensitive
  - The entire term must match (`Nike` doesn't match `Nike, Inc.`)
- Can be used with multiple data types
  - **Don't** use them for the `text` data type (!!!)

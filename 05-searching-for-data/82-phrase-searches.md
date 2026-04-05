# Phrase Searches

In the previous section, we saw how to use multimatch queries.

- If we search "Fanta Zero" or "Zero Fanta", any document with "Fanta" and/or "Zero" is matched, regardless of the order in which we specify the words.
- The terms don't have to be adjacent.

## Basic Usage

- A phrase is sequence of one or more words
  - e.g. "browse the internet"
- We use the `match_phrase` to do phrase queries
- Here, the order of words matters!

For example:

This has no match:

```
GET /products/_search
{
  "query": {
    "match_phrase": {
      "name": "mango juice"
    }
  }
}
```

This has a match:

```
GET /products/_search
{
  "query": {
    "match_phrase": {
      "name": "juice mango"
    }
  }
}
```

This has a match:

```
GET /products/_search
{
  "query": {
    "match_phrase": {
      "description": "browse the internet"
    }
  }
}
```

## How does it work?

This query:

```
GET /products/_search
{
  "query": {
    "match_phrase": {
      "description": "Complete Guide to Elasticsearch"
    }
  }
}
```

Will be analyzed to:

```
["complete", "guide", "to", "elasticsearch"]
```

And the following documents will be matched/not matched as shown below:

- For a document to match, the terms have be found, with no other words between them, and in the correct order
- Note that the analyzer strips punctuation, so those don't count.

```
❌
{
  "description": "Very useful and complete guide! Great introduction to Elasticsearch!"
}


❌
{
  "description": "Learn Elasticsearch with this complete guide!"
}


❌
{
  "description": "Guide to Elasticsearch (complete)"
}


✅
{
  "description": "What a great and complete guide to Elasticsearch! Good stuff!"
}


✅
{
  "description": "Complete Guide — to Elasticsearch"
}
```

- Elastic search stores a bit more information in inverted indexes than we have previously seen.
- e.g., it also stores of the position of each term within documents. This is recorded during the analysis process by default
  - spcifically, this is handled by the analyzer's tokenizer
  - the inverted index canend up looking like this

Document #1:

```
{
  "description": "Learn Elasticsearch with this complete guide!"
}
                            |
                            |
["learn", "elasticsearch", "with", "this", "complete", "guide"]
     1           2          3       4         5          6
```

Document #2:

```
{
  "description": "Complete Guide - to Elasticsearch"
}
                    |
                    |
["complete", "guide", "to", "elasticsearch"]
     1          2       3         4
```

Inverted Index:

- This helps ES when doing the `match_phrase` query

| TERM            | DOCUMENT #1 | DOCUMENT #2 |
| --------------- | ----------- | ----------- |
| "complete"      | X (5)       | X (1)       |
| "elasticsearch" | X (2)       | X (4)       |
| "guide"         | X (6)       | X (2)       |
| "learn"         | X (1)       |             |
| "this"          | X (4)       |             |
| "to"            |             | X (3)       |
| "with"          | X (3)       |             |

- When we search for ["guide", "to", "elasticsearch"] for example, ES first matches all documents with either of those words.
- ES then checks whether those words are in a sequence of (n, n+1, n+2, ...)
  - e.g. ✅ matches for document #2: ["guide" (2), "to" (3), "elasticsearch" (4)] are in sequential order of `+1`
  - e.g. ❌ doesn't match for document #1 ["guide" (6), `____`, "elasticsearch" (2)] are not in sequential order, and there are missing terms

## Lecture Summary

- The `match_phrase` query is similar to the `match` query in some ways
- For the `match_phrase` query, the position (and thereby order) of terms matters
- Terms must appear in the correct order and with no other terms in-between
- The `standard` analyzer's tokenizer outputs term positions that are stored within the field's inverted index
  - These positions are then often used for phrase searches (among others)

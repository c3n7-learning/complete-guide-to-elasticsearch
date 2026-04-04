# Understanding inverted indices

## Introduction

- A field's values are stored in one of the several data structures
  - The data structure depends on the field's data type
- Ensures efficient data access - e.g. searches
- Handled by Apache Lucene, not Elasticsearch
- This lecture focuses on inverted indices

## Inverted indices

- Is a mapping between the terms and which documents contain them
- Outside the context of analyzers, we use the terminology `terms`
- Terms are sorted alphabetically
- Inverted indices contain more than just terms and document IDs
  - E.g. information for relevance scoring

Consider the text documents below:

1. `"2 guys walk into a bar, but the third... DUCKS! :-)"` -> `["2", "guys", "walk", "into", "a", "bar", "but", "the", "third", "ducks"]`
2. `"2 guys went into a bar"` -> `["2", "guys", "went", "into", "a", "bar"]`
3. `"2 ducks walk around the lake"` -> `["2", "ducks", "walk", "around", "the", "lake"]`

| TERM   | DOCUMENT #1 | DOCUMENT #2 | DOCUMENT #3 |
| ------ | ----------- | ----------- | ----------- |
| 2      | X           | X           | X           |
| a      | X           | X           |             |
| around |             |             | X           |
| bar    | X           | X           |             |
| but    | X           |             |             |
| ducks  | X           |             | X           |
| guys   | X           | X           |             |
| into   | X           | X           |             |
| lake   |             |             | X           |
| the    | X           |             | X           |
| third  | X           |             |             |
| walk   | X           |             | X           |
| went   |             | X           |             |

- when we search for `ducks`, we can easily tell which documents have that text
- The more logical mapping would be to map documents to the terms that we contain
- `Inverted Indices` maps terms to the documents that have them
- The above inverted indices are a simplified version of what Apache Lucene stores

## What happens when we store documents with a numbers of fields?

Document #1:

```json
{
  "name": "Coffee Maker",
  "description": "Makes coffee super fast!",
  "price": 64,
  "in_stock": 10,
  "created_at": "2009-11-08T14:21:51Z"
}
```

Document #2:

```json
{
  "name": "Toaster",
  "description": "Makes delicious toasts...",
  "price": 49,
  "in_stock": 4,
  "created_at": "2007-01-29T09:44:15Z"
}
```

An inverted index exists in the context of a field

- Each field will get it's own inverted index

- `name` field:

| TERM    | DOCUMENT #1 | DOCUMENT #2 |
| ------- | ----------- | ----------- |
| coffee  | X           |             |
| maker   | X           |             |
| toaster |             | X           |

- `description` field:

| TERM      | DOCUMENT #1 | DOCUMENT #2 |
| --------- | ----------- | ----------- |
| coffee    | X           |             |
| delicious |             | X           |
| fast      | X           |             |
| makes     | X           | X           |
| super     | X           |             |
| toasts    |             | X           |

### Cont.d

- One inverted index per text field
- Other data types use BKD trees, for instance

## Summary

- Values for a text field are analyzed and the results are stored within an inverted index
- Each field has a dedicated inverted index
- An inverted index is a mapping between terms and thich documents contain them
- Terms are stored alphabetically for performance reasons
- Created and maintained by Apache Lucene, _not_ Elasticsearch
- Inverted indices enable _fast_ searches
- Inverted indices contain other data as well
  - e.g. things used for relevance scoring (covered later)
- ES (technically Apache Lucene) uses other dat structures as well
  - e.g. BKD trees for numeric values, dates, and geospatial data

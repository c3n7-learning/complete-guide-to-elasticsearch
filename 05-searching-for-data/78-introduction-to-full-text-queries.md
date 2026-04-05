# Introduction to Full Text Queries

- Term level queries are used for exact matching on structured data
- Full text queries are used for searching unstructured data
  - e.g. website content, news articles, emails, chats, transcripts, etc
  - often used for long texts
  - We don't know which values a field may contain (hence "unstructured")

## Full text queries are analyzed

For example, this query:

```
GET /products/_search
{
    "query": {
        "match": {
            "body": "SHARDING"
        }
    }
}
```

- The text `"SHARDING"` is analyzed to `["sharding"]`, and the analyzed response is then compared against the inverted index

| TERM            | DOCUMENT #1 |
| --------------- | ----------- |
| "elasticsearch" | X           |
| "distributed"   | X           |
| "sharding"      | X           |
| "replication"   | X           |
| "nodes"         | X           |

## Full text queries vs term level queries

- The main difference is that full text queries are analyzed
  - Term level queries aren't and are there used for exact matching
- Don't use full text queries on `keyword` fields
  - That compares analyzed values with non-analyzed values

## Lecture Summary

- Use full text queries to search unstructured text values
  - e.g. blog posts, emails, chats, books, song titles, etc.
- Full text queries are not used for exact matching
  - They match values that include a term, often being one of many
- Full text queries are analyzed in the same way as the fields that are queried
  - Don't query `keyword` fields with full text queries because the field values were not analyzed during indexing.

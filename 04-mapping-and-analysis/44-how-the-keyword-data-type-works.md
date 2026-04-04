# How the "keyword" data type works

Introduction

- `keyword` fields are analyzed with the `keyword` analyzer
- The `keyword` analyzer is a no-op analyzer
  - It outputs the unmodified string as a single token

We can explore this via:

- Run:

```
POST /_analyze
{
    "text": "2 guys walk into   a bar, but the third... DUCKS! :-)",
    "analyzer": "keyword"
}
```

- Output:

```json
{
  "tokens": [
    {
      "token": "2 guys walk into   a bar, but the third... DUCKS! :-)",
      "start_offset": 0,
      "end_offset": 53,
      "type": "word",
      "position": 0
    }
  ]
}
```

- How it is stored in the inverted index

| TERM                                                | DOCUMENT #1 | DOCUMENT #2 | DOCUMENT #3 |
| --------------------------------------------------- | ----------- | ----------- | ----------- |
| 2 ducks walk around the lake                        | X           |             |             |
| 2 guys walk into a bar, but the third... DUCKS! :-) |             | X           |             |
| 2 guys went into a bar                              |             |             | X           |

For an example with emails:

| TERM                     | DOCUMENT #1 | DOCUMENT #2 | DOCUMENT #3 |
| ------------------------ | ----------- | ----------- | ----------- |
| info@codingexplained.com | X           |             |             |
| john@doe.com             |             | X           |             |
| AVERAGE@JOB.COM          |             |             | X           |

- We could configure the keyword analyzer to use the lowercase token filter, causing the inverted index to look like:

| TERM                     | DOCUMENT #1 | DOCUMENT #2 | DOCUMENT #3 |
| ------------------------ | ----------- | ----------- | ----------- |
| info@codingexplained.com | X           |             |             |
| john@doe.com             |             | X           |             |
| average@job.com          |             |             | X           |

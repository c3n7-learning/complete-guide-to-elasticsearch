# Using the Analyze API

Let's write a simple query to test the standard analyzer

```
POST /_analyze
{
    "text": "2 guys walk into   a bar, but the third... DUCKS! :-)"
}
```

If we run it, the standard analyzer will be used, but let's be explicit about it.

- Run:

```
POST /_analyze
{
    "text": "2 guys walk into   a bar, but the third... DUCKS! :-)",
    "analyzer": "standard"
}
```

- Output:

```json
{
  "tokens": [
    {
      "token": "2",
      "start_offset": 0,
      "end_offset": 1,
      "type": "<NUM>",
      "position": 0
    },
    {
      "token": "guys",
      "start_offset": 2,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "walk",
      "start_offset": 7,
      "end_offset": 11,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "into",
      "start_offset": 12,
      "end_offset": 16,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "a",
      "start_offset": 19,
      "end_offset": 20,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "bar",
      "start_offset": 21,
      "end_offset": 24,
      "type": "<ALPHANUM>",
      "position": 5
    },
    {
      "token": "but",
      "start_offset": 26,
      "end_offset": 29,
      "type": "<ALPHANUM>",
      "position": 6
    },
    {
      "token": "the",
      "start_offset": 30,
      "end_offset": 33,
      "type": "<ALPHANUM>",
      "position": 7
    },
    {
      "token": "third",
      "start_offset": 34,
      "end_offset": 39,
      "type": "<ALPHANUM>",
      "position": 8
    },
    {
      "token": "ducks",
      "start_offset": 43,
      "end_offset": 48,
      "type": "<ALPHANUM>",
      "position": 9
    }
  ]
}
```

- You can see `type` values like `<NUM>` or `<ALPHANUM>`.
- Punctuation and all whitespaces are stripped, because they offer no value when searching
- The `lowercase` character filter is applied on `DUCKS`

## Explicit speficiation of the analyzers

We could specify the analyzers for us. The standard analyzer could be specifed as

```
POST /_analyze
{
    "text": "2 guys walk into   a bar, but the third... DUCKS! :-)",
    "char_filter": [],
    "tokenizer": "standard",
    "filter": ["lowercase"]
}
```

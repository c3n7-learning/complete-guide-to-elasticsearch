# Built-in analyzers

## `standard` analyzer

- Splits text at word boundaries and removes punctuation
  - Done by the `standard` tokenizer
- Lowercases letters with the `lowercase` token filter
- Contains the `stop` token filter (disabled by default), for removing stop words

Example

```
"Is that Peter's cute-looking dog?"
                |
                |
["is", "that", "peter's", "cute", "looking", "dog"]
```

## `simple` analyzer

- Similar to the `standard` analyzer
  - Splits into tokens when encountering anything else than letters
- Lowercases letters with the `lowercase` tokenizer (not a token filter)
  - A tokenizer is not usually supposed to do that
  - Unusual and a performance hack

Example:

```
"Is that Peter's cute-looking dog?"
                |
                |
["is", "that", "peter", "s", "cute", "looking", "dog"]
```

## `whitespace` analyzer

- Splits text into tokens by whitespace
- Does not lowercase letters

Example:

```
"Is that Peter's cute-looking dog?"
                |
["Is", "that", "Peter's", "cute-looking", "dog?"]
```

## `keyword` analyzer

- No-op analyzer that leaves the input text intact
  - It simply outputs it as a single token
- Used for `keyword` fields by default
  - Used for exact matching

Example:

```
"Is that Peter's cute-looking dog?"
                |
["Is that Peter's cute-looking dog?"]
```

## `pattern` analyzer

- A regular expression is used to match token separators
  - It should match whatever should split the text into tokens
- This analyzer is very flexible
- The default pattern matches all non-word characters(`\W+`)
- Lowercases letters by default (can be disabled)

Example:

```
"Is that Peter's cute-looking dog?"
                |
["is", "that", "peter", "s", "cute", "looking", "dog"]
```

## There are other language-specific analyzers

- [Built-in analyzers](https://www.elastic.co/docs/reference/text-analysis/analyzer-reference)
- [Language analyzers](https://www.elastic.co/docs/reference/text-analysis/analysis-lang-analyzer)
- [English analyzers](https://www.elastic.co/docs/reference/text-analysis/analysis-lang-analyzer#english-analyzer)

You will see this under the english analyzer:

```
{
  "rebuilt_english": {
    "tokenizer": "standard",
    "filter": [
      "english_possessive_stemmer",
      "lowercase",
      "english_stop",
      "english_keywords",
      "english_stemmer"
    ]
  }
}
```

The `standard` tokenizer and a selection of filters are used:

- `english_possessive_stemmer` - will remove things like "peter's" or "john's"
- `lowercase` - lowercases all letters
- `english_stop` - removes english stop words
- `english_keywords` - tokens that match the keywords defined in this filter are assigned a key`word attribute of `true`. Subsequent stemmer filters will skip tokens that have the `keyword`attribute set to`true`.
- `english_stemmer` - stems words for the english language

We could add our own custom filters, e.g. to handle synonyms

## How to use them within field mappings

With this mapping:

```
PUT /products
{
  "mappings": {
    "properties": {
      "description": {
        "type": "text",
        "analyzer": "english"
      }
    }
  }
}
```

And this document:

```
POST /products/_doc
{
  "description": "Is that Peter's cute-looking dog?"
}
```

We get this:

- At both index time and search time

```
["peter", "cute", "look", "dog"]
```

## Configuring built-in analyzers

For example, the `english` analyzer is not configured to remove stop words by default. To do this, we can configure it

```
PUT /products
{
  "settings": {
    "analysis": {
      "analyzer": {
        "remove_english_stop_words": {
          "type": "standard",
          "stopwords": "_english_"
        }
      }
    }
  }
}
```

What options do we have? Let's check the docs!

- [Built-in analyzers](https://www.elastic.co/docs/reference/text-analysis/analyzer-reference)
- [Standard analyzer](https://www.elastic.co/docs/reference/text-analysis/analysis-standard-analyzer)

You will see:

- `max_token_length` - The maximum token length. If a token is seen that exceeds this length then it is split at `max_token_length` intervals. Defaults to `255`.
- `stopwords` - A pre-defined stop words list like `_english_` or an array containing a list of stop words. Defaults to `_none_`.
- `stopwords_path` - The path to a file containing stop words.

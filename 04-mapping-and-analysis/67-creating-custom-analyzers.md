# Creating Custom Analyzers

Reference

- https://www.elastic.co/docs/manage-data/data-store/text-analysis

If we want to analyze this text, we ideally wouldn't want the HTML stuff to be indexed:

- Run:

```
POST /_analyze
{
  "analyzer": "standard",
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}
```

- Output:
  - The html symbols are excluded, but the words in the tags are indexed
  - so do the `$apos` apostrophe

```json
{
  "tokens": [
    {
      "token": "i",
      "start_offset": 0,
      "end_offset": 1,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "apos",
      "start_offset": 2,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "m",
      "start_offset": 7,
      "end_offset": 8,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "in",
      "start_offset": 9,
      "end_offset": 11,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "a",
      "start_offset": 12,
      "end_offset": 13,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "em",
      "start_offset": 15,
      "end_offset": 17,
      "type": "<ALPHANUM>",
      "position": 5
    },
    {
      "token": "good",
      "start_offset": 18,
      "end_offset": 22,
      "type": "<ALPHANUM>",
      "position": 6
    },
    {
      "token": "em",
      "start_offset": 24,
      "end_offset": 26,
      "type": "<ALPHANUM>",
      "position": 7
    },
    {
      "token": "mood",
      "start_offset": 28,
      "end_offset": 32,
      "type": "<ALPHANUM>",
      "position": 8
    },
    {
      "token": "nbsp",
      "start_offset": 33,
      "end_offset": 37,
      "type": "<ALPHANUM>",
      "position": 9
    },
    {
      "token": "nbsp",
      "start_offset": 40,
      "end_offset": 44,
      "type": "<ALPHANUM>",
      "position": 10
    },
    {
      "token": "and",
      "start_offset": 45,
      "end_offset": 48,
      "type": "<ALPHANUM>",
      "position": 11
    },
    {
      "token": "i",
      "start_offset": 49,
      "end_offset": 50,
      "type": "<ALPHANUM>",
      "position": 12
    },
    {
      "token": "strong",
      "start_offset": 52,
      "end_offset": 58,
      "type": "<ALPHANUM>",
      "position": 13
    },
    {
      "token": "love",
      "start_offset": 59,
      "end_offset": 63,
      "type": "<ALPHANUM>",
      "position": 14
    },
    {
      "token": "strong",
      "start_offset": 65,
      "end_offset": 71,
      "type": "<ALPHANUM>",
      "position": 15
    },
    {
      "token": "açaí",
      "start_offset": 73,
      "end_offset": 77,
      "type": "<ALPHANUM>",
      "position": 16
    }
  ]
}
```

- Run:

```
{
  "tokens": [
    {
      "token": "I'm in a good mood - and I love açaí!",
      "start_offset": 0,
      "end_offset": 78,
      "type": "word",
      "position": 0
    }
  ]
}
```

- Output;
  - There is a filter called `html_strip`, let's try it out:
  - We haven't specified a tokenizer, hence the input text is emmitted as a single token.
  - The html tags have been removed
  - The html entities have been translated to apostrophes and white-spaces

```json
{
  "tokens": [
    {
      "token": "I'm in a good mood - and I love açaí!",
      "start_offset": 0,
      "end_offset": 78,
      "type": "word",
      "position": 0
    }
  ]
}
```

Let's build our own custom analyzer. The filters we'll use are

- `lowercase` - to convert all letters to lowercase
- `stop` - to remove all stop words
- `asciifolding` - we don't want our users to have to type the special characters `ç` and `í` in `açaí`. This filter converts those characters to their ascii equivalent

```
PUT /analyzer_test
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "char_filter": [
            "html_strip"
          ],
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "stop",
            "asciifolding"
          ]
        }
      }
    }
  }
}
```

Let's test out the analyzer:

- Since the analyzer exists under `analyze_test` index, we have to specify that

```
POST /analyzer_test/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}
```

- Output:

```
{
  "tokens": [
    {
      "token": "i'm",
      "start_offset": 0,
      "end_offset": 8,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "good",
      "start_offset": 18,
      "end_offset": 27,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "mood",
      "start_offset": 28,
      "end_offset": 32,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "i",
      "start_offset": 49,
      "end_offset": 50,
      "type": "<ALPHANUM>",
      "position": 6
    },
    {
      "token": "love",
      "start_offset": 59,
      "end_offset": 72,
      "type": "<ALPHANUM>",
      "position": 7
    },
    {
      "token": "acai",
      "start_offset": 73,
      "end_offset": 77,
      "type": "<ALPHANUM>",
      "position": 8
    }
  ]
}
```

## What if we want to build an analyzer for the danish language?

The configuration below takes the default configuration for th `stop` filter, and instructs it to remove `danish` stop words, instead of english ones

```
PUT /analyzer_test
{
  "settings": {
    "analysis": {
      "filter": {
        "danish_stop": {
          "type": "stop",
          "stopwords": "_danish_"
        }
      },
      "char_filter": { },
      "tokenizer": { },
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "char_filter": [
            "html_strip"
          ],
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "danish_stop",
            "asciifolding"
          ]
        }
      }
    }
  }
}
```

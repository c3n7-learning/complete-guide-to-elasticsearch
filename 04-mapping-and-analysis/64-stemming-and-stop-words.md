# Stemming and Stop Words

Consider the following sentence

- I
- loved : past tense
- drinking : gerund
- bottles : plural of "bottle"
- of
- wine
- on
- last
- year's : possession
- vacation

Suppose we index the following document using the standard analyzer:

```
POST /stemming_test/_doc
{
    "description": "I loved drinking bottles of wine on last year's vacation."
}
```

The standard analyzer will yield this inverted index

| TERM     | DOCUMENT #1 | ... |
| -------- | ----------- | --- |
| i        | X           |     |
| loved    | X           |     |
| drinking | X           |     |
| bottles  | X           |     |
| of       | X           |     |
| wine     | X           |     |
| on       | X           |     |
| last     | X           |     |
| year's   | X           |     |
| vacation | X           |     |

If we search for the text "loves", we won't get any matches

- This is not ideal, because the meaning is the same even if the tense is different

## Introduction to stemming

- Stemming reduces words to their root form
  - e.g. `loved` -> `love`
  - e.g. `drinking` -> `drink`

Stemming would make the following sentence

- "I loved drinking bottles of wine on last year's vacation."
  To be reduced to this
- "I love drink bottl of wine on last year vacat."

## Introduction to stop words

- Words that are filtered out during text analysis
  - Common words such as "a", "the", "at", "of", "on", etc
- They provide little to no value for relevance scoring.
- Fairly common to remove such words
  - Less common in ES today than in the past
    - The relevance algorithm has been improved significantly
- Not removed by default, and it's generally not recommended doing so

```
"I loved drinking bottles of wine on last year's vacation."
                        |
                        |
"I loved drinking bottles wine last year's vacation."
```

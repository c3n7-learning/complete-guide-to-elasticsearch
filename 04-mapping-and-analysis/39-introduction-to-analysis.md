# Introduction to Analysis

## Analysis

- Sometimes referred to as _text analysis_
- Applicable to text fields/values
- Text values are analyzed when indexing document
- The result is stored in data structures that are efficient for searching etc.
- The `_source` object is **not** used when searching for documents
  - It contains the exact values specified when indexing a document
  - It is not exactly efficient to search the original text of the document
- The original document has to have some analysis done on it before being stored to make it efficient for searching

```
+--------+           +----------+           +----------+
|        |           |    /\    |           | oo  ---- |
| ----   |  ------>  |   /  \   |  ------>  | oo  ---- |
| ----   |           |  | -- |  |           +----------+
| ----   |           |   \  /   |           | oo  ---- |
| ----   |           |    \/    |           | oo  ---- |
+--------+           +----------+           +----------+
                                            | oo  ---- |
Document              Analyzer              | oo  ---- |
                                            +----------+
                         |
                         |                    Storage
                    +----+----+
                    |    |    |
              Character  |  Token
               filters   |  filters
                         |
                    Tokenizer
```

### Character filters

- Adds, removes, or changes characters
- Analyzers contain zero or more character filters
- Character filters are applied in the order in which they are specified
- Example (`html_strip`) filter
  - Input: `"I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and`  
    `I <strong>love</strong> açaí!"`
- Output: `"I'm in a good mood - and I love it açaí!"`

### Tokenizer

- An analyzer contains atleast **one** tokenizer
- Tokenizes a string, i.e. splits it into tokens
- Characters may be stripped as part of tokenization
- Example:
  - Input: `"I REALLY like beer!"`
  - Output: `["I", "REALLY", "like", "beer"]`

### Token filters

- Receive the output of the tokenizer as input (i.e. the tokens)
- A token filter can add, remove, or modify tokens
- An analyzer contains zero or more token filters
- Token filters are applied in the order in which they are specified
- Example (`lowercase` filter):
  - Input: `["I", "REALLY", "like", "beer"]`
  - Output: `["i", "really", "like", "beer"]`

### Built-in and custom components

- Built-in analyzers, character filters, tokenizers, and token filters are available
- We can also build custom ones
  - You'll see how later in this section
- For now let's see how text values are analyzed by default

So, the standard analyzer can be summarized to look like this

1. No _Character filters_ are done by default
2. The standard _Tokenizer_ is run on the text
   - It is more complex than the below, but one can say it:
   - Basically one can say it splits the text into distinct "words"
   - It strips all punctuation
3. The tokens are then passed on to the _Token filters_ called `lowercase`

```
+------------------+           +------------------+           +------------------+
|  Character       |           |  Tokenizer       |           |  Token filters   |
|  filters         |           |                  |           |                  |
|                  |           |  standard        |           |  ["lowercase"]   |
|  (none)          |           |                  |           |                  |
+------------------+           +------------------+           +------------------+
|                  |           |                  |           |                  |
| Input: "I REALLY | --------> | Input: "I REALLY | --------> | Input: "I",      |
|       like beer!"|           |       like beer!"|           |        "REALLY", |
|                  |           |                  |           |        "like",   |
| Output: "I REALLY|           | Output: ["I",    |           |        "beer"    |
|       like beer!"|           |         "REALLY",|           |                  |
|                  |           |         "like",  |           | Output: "i",     |
|                  |           |         "beer"]  |           |         "really",|
|                  |           |                  |           |         "like",  |
|                  |           |                  |           |         "beer"   |
+------------------+           +------------------+           +------------------+
```

- There other analyzers, not just the standard analyzer, but we tend to default to the standard one.

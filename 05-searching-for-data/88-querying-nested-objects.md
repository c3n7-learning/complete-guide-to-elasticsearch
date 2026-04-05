# Querying Nested Objects

## The `nested` data type

- We covered the `nested` data type earlier in the course

- This lecture recaps on the concept of nested objects
  - Feel free to revisit [Overview of Data Types](../04-mapping-and-analysis/43-overview-of-data-types.md)

## Prerequisites

1. Let's download the `recipes` dataset from here: https://github.com/codingexplained/complete-guide-to-elasticsearch/blob/master/recipes-bulk.json
2. Import the dataset

```shell
curl -k -u elastic -H "Content-Type:application/x-ndjson" -XPOST "http://localhost:9200/recipes/_bulk" -
-data-binary "@05-searching-for-data/88-recipes-bulk.json"
```

Here's how a sample document looks like:

```json
{
  "title": "Fast and Easy Pasta With Blistered Cherry Tomato Sauce",
  "description": "Cherry tomatoes are almost always sweeter, riper, and higher in pectin than larger tomatoes at the supermarket. All of these factors mean that cherry tomatoes are fantastic for making a rich, thick, flavorful sauce. Even better: It takes only four ingredients and about 10 minutes, start to finish — less time than it takes to cook the pasta you're gonna serve it with.",
  "preparation_time_minutes": 12,
  "servings": { "min": 4, "max": 6 },
  "steps": [
    "Place pasta in a large skillet or sauté pan and cover with water and a big pinch of salt. Bring to a boil over high heat, stirring occasionally. Boil until just shy of al dente, about 1 minute less than the package instructions recommend.",
    "Meanwhile, heat garlic and 4 tablespoons (60ml) olive oil in a 12-inch skillet over medium heat, stirring frequently, until garlic is softened but not browned, about 3 minutes. Add tomatoes and cook, stirring, until tomatoes begin to burst. You can help them along by pressing on them with the back of a wooden spoon as they soften.",
    "Continue to cook until sauce is rich and creamy, about 5 minutes longer. Stir in basil and season to taste with salt and pepper.",
    "When pasta is cooked, drain, reserving 1 cup of pasta water. Add pasta to sauce and increase heat to medium-high. Cook, stirring and tossing constantly and adding reserved pasta water as necessary to adjust consistency to a nice, creamy flow. Remove from heat, stir in remaining 2 tablespoons (30ml) olive oil, and grate in a generous shower of Parmesan cheese. Serve immediately, passing extra Parmesan at the table."
  ],
  "ingredients": [
    { "name": "Dry pasta", "amount": 450, "unit": "grams" },
    { "name": "Kosher salt" },
    { "name": "Cloves garlic", "amount": 4, "unit": "pcs" },
    { "name": "Extra-virgin olive oil", "amount": 90, "unit": "ml" },
    { "name": "Cherry tomatoes", "amount": 750, "unit": "grams" },
    { "name": "Fresh basil leaves", "amount": 30, "unit": "grams" },
    { "name": "Freshly ground black pepper" },
    { "name": "Parmesan cheese" }
  ],
  "created": "2017-03-29T14:43:21Z",
  "ratings": [4.5, 5.0, 3.0, 4.5]
}
```

## Searching arrays of objects (the wrong way)

```http
GET /recipes/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        },
        {
          "range": {
            "ingredients.amount": {
              "gte": 100
            }
          }
        }
      ]
    }
  }
}
```

In the results, we get recipes with ingredients like:

```json
{
  "name": "Parmesan cheese"
}
{ "..." },
{
  "name": "Freshly grated Parmesan cheese",
  "amount": 55,
  "unit": "grams"
}
```

- But our query specifies we **dont** want ingredients with amounts less than 100!

Why are they included? Let's inspect the mapping:

```
GET /recipes/_mapping
```

- Output:

```
{
  "recipes": {
    "mappings": {
      "properties": {
        "created": {
          "type": "date"
        },
        "description": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "ingredients": {
          "properties": {
            "amount": {
              "type": "long"
            },
            "name": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "unit": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        },
        "preparation_time_minutes": {
          "type": "long"
        },
        "ratings": {
          "type": "float"
        },
        "servings": {
          "properties": {
            "max": {
              "type": "long"
            },
            "min": {
              "type": "long"
            }
          }
        },
        "steps": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "title": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        }
      }
    }
  }
}
```

- You can see the ingredients were specifed as an object (we can tell due to the nested `properties`; which is a tell-tale sign)
- Every ES value can contain 0 or more values, so we are dealing with an array of objects.

So, arrays! You remember the problems they had?

- This document:

```json
{
  "ingredients": [
    {
      "name": "Freshly grated Parmesan cheese",
      "amount": 55,
      "unit": "grams"
    },
    {
      "name": "Freshly ground black pepper"
    },
    {
      "name": "Mixed mushrooms",
      "amount": 225,
      "unit": "grams"
    },
    {
      "name": "Dried orecchiette",
      "amount": 450,
      "unit": "grams"
    },
    {
      "name": "Cloves garlic",
      "amount": 2,
      "unit": "pos"
    }
  ]
}
```

Is indexed as:

- Hence the unexpected query results!
- ES matches if `ingredients.name` is matched OR `ingredients.amount` is matched, then both are matched and returned. Doesn't matter whether the amount is not under the "parmesan recipe"
  - Is there an `ingredient.name` called "parmesan"?
  - Is there an `ingredient.amount` for any ingredient gte 100?

```json
{
  "ingredients.name": [
    "Freshly grated Parmesan cheese",
    "Freshly ground black pepper",
    "Mixed mushrooms",
    "Dried orecchiette",
    "Cloves garlic"
  ],
  "ingredients.amount": [55, 225, 450, 2],
  "ingredients.unit": ["grams", "grams", "grams", "pcs"]
}
```

## The problem & the solution

**Problem**:

- When indexing arrays of objects, the relationships between values are not maintained
- Queries can yield "unpredictable" results

**Solution**:

- Use the `nested` data type and the `nested` query
- Create a new index to update the field mapping and reindex documents

## Create the correct mapping (using the `nested` data type)

Delete the previous index:

```
DELETE /recipes
```

Create the new index:

```
PUT /recipes
{
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "description": { "type": "text" },
      "preparation_time_minutes": { "type": "integer" },
      "steps": { "type": "text" },
      "created": { "type": "date" },
      "ratings": { "type": "float" },
      "servings": {
        "properties": {
          "min": { "type": "integer" },
          "max": { "type": "integer" }
        }
      },
      "ingredients": {
        "type": "nested",
        "properties": {
          "name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword"
              }
            }
          },
          "amount": { "type": "integer" },
          "unit": { "type": "keyword" }
        }
      }
    }
  }
}
```

- The "secret" sauce is this section:

```
{
  "ingredients": {
    "type": "nested",
    "properties": {
      "name": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "amount": {
        "type": "integer"
      },
      "unit": {
        "type": "keyword"
      }
    }
  }
}
```

- Remember, ES stores nested documents internally as hidden documents

Import the test data again:

```shell
curl -k -u elastic -H "Content-Type:application/x-ndjson" -XPOST "http://localhost:9200/recipes/_bulk" -
-data-binary "@05-searching-for-data/88-recipes-bulk.json"
```

Now run the bool query again:

- Run:

```http
GET /recipes/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        },
        {
          "range": {
            "ingredients.amount": {
              "gte": 100
            }
          }
        }
      ]
    }
  }
}
```

- Output:
  - We can't query data of the nested data type like this

```json
{
  "took": 39,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 0,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  }
}
```

## Using the `nested` query

- If the `ingredients` nested object were nested further, we could use dot notation like `the.ingredients`:

Run:

```
GET /recipes/_search
{
  "query": {
    "nested": {
      "path": "ingredients",
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "ingredients.name": "parmesan"
              }
            },
            {
              "range": {
                "ingredients.amount": {
                  "gte": 100
                }
              }
            }
          ]
        }
      }
    }
  }
}
```

Output:

- Only one hit

```json
...
{
  "name": "Grated Parmesan cheese",
  "amount": 175,
  "unit": "grams"
}
...
```

## How documents are scored

- Matching child objects affect the parent document's relevance score
- Elasticsearch calculates a relevance score for each matching child object
  - This is because each nested object is a Lucene document
- Relevane scoring can be configured with the `score_mode` parameter

Adjusting relevance scores with `score_mode`

| score_mode    | Parent document's relevance score                              |
| ------------- | -------------------------------------------------------------- |
| avg (default) | The average relevance score of matching child objects.         |
| min           | The minimum relevance score of matching child objects.         |
| max           | The maximum relevance score of matching child objects.         |
| sum           | The sum of all matching child objects' relevance scores.       |
| none          | Ignore relevance scores for matching child objects (i.e. 0.0). |

## Lecture Summary

- Use the `nested` data type if you want to query objects independently
  - Otherwise the relationships between object values are not maintained
  - Each nested object is indexed as a hidden Lucene document
- Use the `nested` query on fields with the `nested` data type
  - Elasticsearch then handles everything automatically
- Use the `score_mode` parameter to adjust relevance scoring.

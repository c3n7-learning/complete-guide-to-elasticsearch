# Reindexing documents with the Reindex API

- https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-reindex

Let's get the current mappings:

- Run:

```
GET /reviews/_mapping
```

- Output:

```json
{
  "reviews": {
    "mappings": {
      "properties": {
        "author": {
          "properties": {
            "email": {
              "type": "keyword"
            },
            "first_name": {
              "type": "text"
            },
            "last_name": {
              "type": "text"
            }
          }
        },
        "content": {
          "type": "text"
        },
        "created_at": {
          "type": "date"
        },
        "product_id": {
          "type": "integer"
        },
        "rating": {
          "type": "float"
        }
      }
    }
  }
}
```

Let's create the new index:

- Let's update `product_id` from `integer` to `keyword`
- Run:

```
PUT /reviews_new
{
  "mappings": {
    "properties": {
      "author": {
        "properties": {
          "email": {
            "type": "keyword"
          },
          "first_name": {
            "type": "text"
          },
          "last_name": {
            "type": "text"
          }
        }
      },
      "content": {
        "type": "text"
      },
      "created_at": {
        "type": "date"
      },
      "product_id": {
        "type": "keyword"
      },
      "rating": {
        "type": "float"
      }
    }
  }
}
```

- Output:

```json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "reviews_new"
}
```

How do we move data to this new index? We could

- Build our own custom script that does this
- Use the `reindex` api, as ES has accounted for this

To copy over data to our new index:

- Run

```
POST /_reindex
{
    "source": {
        "index": "reviews"
    },
    "dest": {
        "index": "reviews_new"
    }
}
```

- Output:

```json
{
  "took": 178,
  "timed_out": false,
  "total": 5,
  "updated": 0,
  "created": 5,
  "deleted": 0,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1,
  "throttled_until_millis": 0,
  "failures": []
}
```

- We can see that 5 `"total": 5` were reindexed.

Let's query the new index:

- Run:

```
GET /reviews_new/_search
{
    "query": {
        "match_all": {}
    }
}
```

- Output:

```json
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 5,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "reviews_new",
        "_id": "1",
        "_score": 1,
        "_source": {
          "rating": 5,
          "content": "Outstanding course! Bo really taught me a lot about Elasticsearch!",
          "product_id": 123,
          "author": {
            "first_name": "John",
            "last_name": "Doe",
            "email": "johndoe123@example.com"
          }
        }
      },
      {
        "_index": "reviews_new",
        "_id": "2",
        "_score": 1,
        "_source": {
          "rating": 4.5,
          "content": "Not bad. Not bad at all!",
          "product_id": 123,
          "created_at": "2015-03-27",
          "author": {
            "first_name": "Average",
            "last_name": "Joe",
            "email": "avgjoe@example.com"
          }
        }
      },
      {
        "_index": "reviews_new",
        "_id": "3",
        "_score": 1,
        "_source": {
          "rating": 3.5,
          "content": "Could be better",
          "product_id": 123,
          "created_at": "2015-04-15T13:07:41Z",
          "author": {
            "first_name": "Spencer",
            "last_name": "Pearson",
            "email": "spearson@example.com"
          }
        }
      },
      {
        "_index": "reviews_new",
        "_id": "4",
        "_score": 1,
        "_source": {
          "rating": 5,
          "content": "Incredible!",
          "product_id": 123,
          "created_at": "2015-01-28T09:21:51+01:00",
          "author": {
            "first_name": "Adam",
            "last_name": "Jones",
            "email": "adam.jones@example.com"
          }
        }
      },
      {
        "_index": "reviews_new",
        "_id": "5",
        "_score": 1,
        "_source": {
          "rating": 4.5,
          "content": "Very useful",
          "product_id": 123,
          "created_at": 1436011284000,
          "author": {
            "first_name": "Taylor",
            "last_name": "West",
            "email": "twest@example.com"
          }
        }
      }
    ]
  }
}
```

- Note that the values of `product_id` still look like integers

## `_source` data types

- The data type doesn't reflect how the values are indexed
- `_source` contains the field values supplied at index time
- It's common to use `_source` values from search results
  - You would probably expect a string for a keyword field
- We can modify the `_source` value while reindexing
- Alternatively, this can be handled at the application level

## To modify the `_source` while reindexing:

Clear the previous data:

```
POST /reviews_new/_delete_by_query
{
    "query": {
        "match_all": {}
    }
}
```

Run re-indexing with the script that does the conversion:

- Run:

```
POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  },
  "script": {
    "source": """
        if (ctx._source.product_id != null) {
            ctx._source.product_id = ctx._source.product_id.toString();
        }
    """
  }
}
```

Retrieve the new documents:

- Run:

```
GET /reviews_new/_search
{
    "query": {
        "match_all": {}
    }
}
```

- Output:

```json
{
  "took": 27,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 5,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "reviews_new",
        "_id": "1",
        "_score": 1,
        "_source": {
          "author": {
            "last_name": "Doe",
            "first_name": "John",
            "email": "johndoe123@example.com"
          },
          "product_id": "123",
          "rating": 5,
          "content": "Outstanding course! Bo really taught me a lot about Elasticsearch!"
        }
      },
      {
        "_index": "reviews_new",
        "_id": "2",
        "_score": 1,
        "_source": {
          "author": {
            "last_name": "Joe",
            "first_name": "Average",
            "email": "avgjoe@example.com"
          },
          "product_id": "123",
          "rating": 4.5,
          "created_at": "2015-03-27",
          "content": "Not bad. Not bad at all!"
        }
      },
      {
        "_index": "reviews_new",
        "_id": "3",
        "_score": 1,
        "_source": {
          "author": {
            "last_name": "Pearson",
            "first_name": "Spencer",
            "email": "spearson@example.com"
          },
          "product_id": "123",
          "rating": 3.5,
          "created_at": "2015-04-15T13:07:41Z",
          "content": "Could be better"
        }
      },
      {
        "_index": "reviews_new",
        "_id": "4",
        "_score": 1,
        "_source": {
          "author": {
            "last_name": "Jones",
            "first_name": "Adam",
            "email": "adam.jones@example.com"
          },
          "product_id": "123",
          "rating": 5,
          "created_at": "2015-01-28T09:21:51+01:00",
          "content": "Incredible!"
        }
      },
      {
        "_index": "reviews_new",
        "_id": "5",
        "_score": 1,
        "_source": {
          "author": {
            "last_name": "West",
            "first_name": "Taylor",
            "email": "twest@example.com"
          },
          "product_id": "123",
          "rating": 4.5,
          "created_at": 1436011284000,
          "content": "Very useful"
        }
      }
    ]
  }
}
```

## Advanced reindexing

### Reindex documents matching a query

```
POST /_reindex
{
  "source": {
    "index": "reviews",
    "query": {
      "match_all": { }
    }
  },
  "dest": {
    "index": "reviews_new"
  }
}
```

We could make it more interesting

- Reindex only positive reviews

```json
POST /_reindex
{
  "source": {
    "index": "reviews",
    "query": {
      "range": {
        "rating": {
          "gte": 4.0
        }
      }
    }
  },
  "dest": {
    "index": "reviews_new"
  }
}
```

### Removing fields

- Field mappings cannot be deleted
- Fields can be left out when indexing documents
- Maybe though we want to reclai disk space used by a field
  - Already indexed values still take up disk space
  - For large data sets, this may be worthwhile
    - This assumes we no longer need the values
- We can use source filtering
- Any fields left out will not be reindexed

```
POST /_reindex
{
  "source": {
    "index": "reviews",
    "_source": ["content", "created_at", "rating"]
  },

  "dest": {
    "index": "reviews_new"
  }
}
```

### Changing a field's name

- Note: the `_source` is a java hash map
- `remove` returns the value then removes it from the hash map

```
POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  },
  "script": {
    "source": """
    # Rename "content" field to "comment"
    ctx._source.comment = ctx._source.remove("content");
    """
  }
}
```

### Using `ctx.op` within scripts

- In case of `noop`, the document will not be reindexed

Ignore reviews with ratings below `4.0`

```
POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  },
  "script": {
    "source": """
    if (ctx._source.rating < 4.0) {
      ctx.op = "noop"; # Can also be set to "delete"
    }
    """
  }
}
```

- Usually, using the `query` parameter is possible, you can prefer this
- For more advanced use cases, `ctx.op` can be used
- Using the `query` parameter is better performance wise and is preferred
- Specifying `delete` deletes the document within the destination index
  - The destination index might not be empty as in our example
  - So maybe we want to do this
  - The same can often be done with the Delete by Query API though

### Parameters for the Reindex API

- More parameters are available than the ones we covered
  - e.g. for handling version conflicts
- A snapshot is created before reindexing documents
- A version conflict causes the query to be aborted by default
- The destination index is not necessarily empty,
  - so version conflicts can occur and cause problems

### Batching and throttling

- The Reindex API performs operations in batches
  - Just like the Update by Query and Delete by Query APIs
  - It uses the Scroll API internally
  - This is how milions of documents can be reindexed efficiently
- Throttling can be configured to limit the performance impact
  - Useful for production clusters
- Check the documentation if you need to reindex lots of documents.

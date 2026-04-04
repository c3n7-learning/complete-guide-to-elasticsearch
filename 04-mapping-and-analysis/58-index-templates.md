# Index Templates

Reference

- https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-indices-put-index-template

## Introduction to index templates

- A way to automatically apply settings and mappings on index creation
  - Works by matching index names against an index pattern
- When creating a new index, ES checks if the new index's name matches any configured template, and applies the settings and mappings for the template

Here is the general structure

```
PUT /_index_template/my-index-template (1)
{
  "index_patterns": ["my-index-pattern*"], (2)
  "template": {
    "settings": { ... }, (3)
    "mappings": { ... } (4)
  }
}
```

1. The name of the index template
2. Pattern(s) determining when the index template is applied
3. Settings to apply to the new index
4. Field mappings to add to the new index

Access logs index template (for Apache)

```
PUT /_index_template/access-logs
{
  "index_patterns": ["access-logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 2,
      "index.mapping.coerce": false
    },
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },

        /* Requested URL */
        "url.original": { "type": "wildcard" },
        "url.path": { "type": "wildcard" },
        "url.scheme": { "type": "keyword" },
        "url.domain": { "type": "keyword" },

        /* Client geography */
        "client.geo.continent_name": { "type": "keyword" },
        "client.geo.country_name": { "type": "keyword" },
        "client.geo.region_name": { "type": "keyword" },
        "client.geo.city_name": { "type": "keyword" },

        /* User agent */
        "user_agent.original": { "type": "keyword" },
        "user_agent.name": { "type": "keyword" },
        "user_agent.version": { "type": "keyword" },
        "user_agent.device.name": { "type": "keyword" },
        "user_agent.os.name": { "type": "keyword" },
        "user_agent.os.version": { "type": "keyword" }
      }
    }
  }
}
```

Cont.d

- Typically used for data sets that are stored in multiple indices
  - E.g. time series data
    - We can store time-based indices by going with either Yearly, Monthly or Daily

Indexing documents into time-based indices

| Yearly indices   | Monthly indices     | Daily indices          |
| ---------------- | ------------------- | ---------------------- |
| access-logs-2018 | access-logs-2023-01 | access-logs-2023-01-01 |
| access-logs-2019 | access-logs-2023-02 | access-logs-2023-01-02 |
| access-logs-2020 | access-logs-2023-03 | access-logs-2023-01-03 |
| access-logs-2021 | access-logs-2023-04 | access-logs-2023-01-04 |
| access-logs-2022 | access-logs-2023-05 | access-logs-2023-01-05 |
| access-logs-2023 | access-logs-2023-06 | access-logs-2023-01-06 |
| ...              | ...                 | ...                    |

The daily logs above could be output by logic such as below:

```
from datetime import datetime, timezone
from elasticsearch import Elasticsearch

es = Elasticsearch('https://localhost:9200')

now = datetime.now(timezone.utc)
index_name = f"access-logs-{now.year}-{now.month:02d}"

doc = {
    '@timestamp': now.isoformat(timespec='seconds'),
    'url.original': 'https://example.com/products',
    'url.path': '/products',
    # ...
}

es.index(index=index_name, document=doc)
```

- ES will create an index implicitly if it doesnt exist, but maybe the default mappings and settings aren't what we want. Index templates can help solve this problem

## Indexing a document

```
┌─────────────────────────────────┐
│ POST /access-logs-2023-01/_doc  │
└─────────────────┬───────────────┘
                  │
                  v
            ┌─────────────┐
            │ Does index  │
            │   exist?    │
            └──┬─────────┬┘
               │         │
            Yes│         │No
               │         │
               v         v
    ┌──────────────┐  ┌─────────────┐
    │ Index into   │  │ Does index  │
    │ existing     │  │ template    │
    │ index        │  │ match?      │
    └──────────────┘  └──┬─────────┬┘
                         │         │
                       No│         │Yes
                         │         │
                         v         v
             ┌─────────────────┐ ┌─────────────────┐
             │ Create index    │ │ Create index    │
             │ with default    │ │ based on        │
             │ settings        │ │ template        │
             └────────┬────────┘ └────────┬────────┘
                      │                   │
                      v                   v
             ┌─────────────────┐ ┌─────────────────┐
             │ Index document  │ │ Index document  │
             │ into new index  │ │ into new index  │
             └─────────────────┘ └─────────────────┘
```

## Demo

Let us create an index template for access logs:

- Run:

```
PUT /_index_template/access-logs
{
  "index_patterns": ["access-logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 2,
      "index.mapping.coerce": false
    },
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },

        /* Requested URL */
        "url.original": { "type": "wildcard" },
        "url.path": { "type": "wildcard" },
        "url.scheme": { "type": "keyword" },
        "url.domain": { "type": "keyword" },

        /* Client geography */
        "client.geo.continent_name": { "type": "keyword" },
        "client.geo.country_name": { "type": "keyword" },
        "client.geo.region_name": { "type": "keyword" },
        "client.geo.city_name": { "type": "keyword" },

        /* User agent */
        "user_agent.original": { "type": "keyword" },
        "user_agent.name": { "type": "keyword" },
        "user_agent.version": { "type": "keyword" },
        "user_agent.device.name": { "type": "keyword" },
        "user_agent.os.name": { "type": "keyword" },
        "user_agent.os.version": { "type": "keyword" }
      }
    }
  }
}
```

Let us create a new index and add a coument to it

- Run:

```
POST /access-logs-2023-01/_doc
{
  "@timestamp": "2023-01-01T00:00:00Z",
  "url.original": "https://example.com/products",
  "url.path": "/products",
  "url.scheme": "https",
  "url.domain": "example.com",
  "client.geo.continent_name": "Europe",
  "client.geo.country_name": "Denmark",
  "client.geo.region_name": "Capital City Region",
  "client.geo.city_name": "Copenhagen",
  "user_agent.original": "Mozilla/5.0 (iPhone; CPU iPhone OS 12_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/12.0 Mobile/15E148 Safari/604.1",
  "user_agent.name": "Safari",
  "user_agent.version": "12.0",
  "user_agent.device.name": "iPhone",
  "user_agent.os.name": "iOS",
  "user_agent.os.version": "12.1.0"
}
```

- Output:

```json
{
  "_index": "access-logs-2023-01",
  "_id": "oLpSVZ0Bv160fQzNT0-2",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```

Let's look at the index we've just created

- Run

```
GET /access-logs-2023-01
```

- Output:

```json
{
  "access-logs-2023-01": {
    "aliases": {},
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },
        "client": {
          "properties": {
            "geo": {
              "properties": {
                "city_name": { "type": "keyword" },
                "continent_name": { "type": "keyword" },
                "country_name": { "type": "keyword" },
                "region_name": { "type": "keyword" }
              }
            }
          }
        },
        "url": {
          "properties": {
            "domain": { "type": "keyword" },
            "original": { "type": "wildcard" },
            "path": { "type": "wildcard" },
            "scheme": { "type": "keyword" }
          }
        },
        "user_agent": {
          "properties": {
            "device": {
              "properties": {
                "name": { "type": "keyword" }
              }
            },
            "name": { "type": "keyword" },
            "original": { "type": "keyword" },
            "os": {
              "properties": {
                "name": { "type": "keyword" },
                "version": { "type": "keyword" }
              }
            },
            "version": { "type": "keyword" }
          }
        }
      }
    },
    "settings": {
      "index": {
        "mapping": {
          "coerce": "false"
        },
        "number_of_shards": "2"
      }
    }
  }
}
```

## Creating an index manually

```
┌─────────────────────────────┐
│ PUT /access-logs-2023-01    │
└─────────────┬───────────────┘
              │
              v
        ┌─────────────┐
        │ Does index  │
        │   exist?    │
        └──┬─────────┬┘
           │         │
         Yes│         │No
           │         │
           v         v
┌─────────────────┐ ┌─────────────┐
│ Cannot create   │ │ Does index  │
│ index           │ │ template    │
└─────────────────┘ │ match?      │
                    └──┬─────────┬┘
                       │         │
                     No│         │Yes
                       │         │
                       v         v
           ┌─────────────────┐ ┌─────────────────────┐
           │ Create index    │ │ Create index based  │
           │ based on        │ │ on index template   │
           │ request         │ │ + request           │
           └─────────────────┘ └─────────────────────┘
```

Let us demo this:

- Run:

```
PUT /access-logs-2023-02
{
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "properties": {
      "url.query": {
        "type": "keyword"
      }
    }
  }
}
```

Then inspect the index, you'll the field mapping and the overriden shards

- Run:

```
GET /access-logs-2023-02
```

## Updating index templates

- `PUT` in http means to replace a resource

```
PUT /_index_template/access-logs
{
  "index_patterns": ["access-logs-*"],
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },

        /* Requested URL */
        "url.original": { "type": "wildcard" },
        "url.path": { "type": "wildcard" },
        "url.scheme": { "type": "keyword" },
        "url.domain": { "type": "keyword" },

        /* Client geography */
        "client.geo.continent_name": { "type": "keyword" },
        "client.geo.country_name": { "type": "keyword" },
        "client.geo.region_name": { "type": "keyword" },
        "client.geo.city_name": { "type": "keyword" },

        /* User agent */
        "user_agent.original": { "type": "keyword" },
        "user_agent.name": { "type": "keyword" },
        "user_agent.version": { "type": "keyword" },
        "user_agent.device.name": { "type": "keyword" },
        "user_agent.os.name": { "type": "keyword" },
        "user_agent.os.version": { "type": "keyword" }
      }
    }
  }
}
```

- The index template will be overriden with the new settings
- The existing indices wont be updated
- Only new ones going forward are going to use the new settings

### Retreiving an index template

```
GET /_index_template/access-logs
```

### Deleting an index template

```
DELETE /_index_template/access-logs
```

## Reserved index patterns

Avoid these

- `logs-*-*`
- `metrics-*-*`
- `synthetics-*-*`
- `profiling-*`

## Priorities

- Index patterns cannot overrlable by default
  - Can't have patterns that are overlapping
- Only a single index template can be applied to a new index
  - Overlapping index templates aren't merged, an error is thrown

This will work ✅

```
PUT /_index_template/monthly-logs
{
  "index_patterns": ["monthly-*"],
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },
        "month_name": { "type": "keyword" }
      }
    }
  }
}
```

This will lead to exceptions being thrown when new indexes are created, due to the overlapping ❌

```
PUT /_index_template/monthly-access-logs
{
  "index_patterns": ["monthly-*-logs"],
  "template": {
    "mappings": {
      "properties": {
        "url.original": { "type": "wildcard" },
        "url.path": { "type": "wildcard" },
        "url.scheme": { "type": "keyword" },
        "url.domain": { "type": "keyword" }
      }
    }
  }
}
```

### Specifying priorities

- Specifying a `priority` parameter to handle overlapping index patterns
- Defaults to zero
- The index template with highest priority "wins"

This will work ✅

```
PUT /_index_template/monthly-logs
{
  "index_patterns": ["monthly-*"],
  "priority": 0,
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },
        "month_name": { "type": "keyword" }
      }
    }
  }
}
```

This will work ✅

```
PUT /_index_template/monthly-access-logs
{
  "index_patterns": ["monthly-*-logs"],
  "priority": 1,
  "template": {
    "mappings": {
      "properties": {
        "url.original": { "type": "wildcard" },
        "url.path": { "type": "wildcard" },
        "url.scheme": { "type": "keyword" },
        "url.domain": { "type": "keyword" }
      }
    }
  }
}
```

- To reiterate, only one index template will be used, the one with the highest priority.

## Summary

- Index templates automatically apply settings and mappings to new indices
- Only a single index template can be applied to a new index
- Useful for data sets that store data in multiple indices, e.g. time series data
- Enables us to simply index documents into indices that don't already exit
- Indices can still be created manually
  - API request and index template are merged (the request takes precedence)
- Use priorities to allow overlapping index patterns.
  - The index template with the highest priority is used.

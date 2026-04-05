# Terms Lookup Mechanism

- The basic form looks documents that contain one or more documents in the query
- Some times we might want to look up so many terms, e.g. 500, which we don't want to include in the query definition

The terms query supports **terms lookup mechanism** (fetching the terms from a document)

- We can specify an index from which we want to retrieve the fields

## Adding test data

- Create two indices, users and stories
- Each user has `following`, which maintains the id of users the current user is following

Stories:

- The stories that a user is following

```
PUT /users/_doc/1
{
  "name": "John Roberts",
  "following" : [2, 3]
}

PUT /users/_doc/2
{
  "name": "Elizabeth Ross",
  "following" : []
}

PUT /users/_doc/3
{
  "name": "Jeremy Brooks",
  "following" : [1, 2]
}

PUT /users/_doc/4
{
  "name": "Diana Moore",
  "following" : [3, 1]
}

PUT /stories/_doc/1
{
  "user": 3,
  "content": "Wow look, a penguin!"
}

PUT /stories/_doc/2
{
  "user": 1,
  "content": "Just another day at the office... #coffee"
}

PUT /stories/_doc/3
{
  "user": 1,
  "content": "Making search great again! #elasticsearch #elk"
}

PUT /stories/_doc/4
{
  "user": 4,
  "content": "Had a blast today! #rollercoaster #amusementpark"
}

PUT /stories/_doc/5
{
  "user": 4,
  "content": "Yay, I just got hired as an Elasticsearch consultant - so excited!"
}

PUT /stories/_doc/6
{
  "user": 2,
  "content": "Chilling at the beach @ Greece #vacation #goodtimes"
}
```

## Querying stories from a user's follows

- Get the stories for users followed by user with id of 1

Run:

```
GET /stories/_search
{
  "query": {
    "terms": {
      "user": {
        "index": "users",
        "id": "1",
        "path": "following"
      }
    }
  }
}
```

Output:

```json
{
  "took": 51,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "stories",
        "_id": "1",
        "_score": 1,
        "_source": {
          "user": 3,
          "content": "Wow look, a penguin!"
        }
      },
      {
        "_index": "stories",
        "_id": "6",
        "_score": 1,
        "_source": {
          "user": 2,
          "content": "Chilling at the beach @ Greece #vacation #goodtimes"
        }
      }
    ]
  }
}
```

- We see stories followed by user `3` and user `2`, which makes sense because those are the users followed by user `#1`

What happens when we make the query?

```
┌─────────┐                                ┌─────────────────────┐                ┌─────────┐
│ CLIENT  │ ──────────────────────────────►│ COORDINATING NODE   │◄──────────────►│ User 1  │
└─────────┘                                └─────────────────────┘                └─────────┘
     ▲                                                │                          following: [2, 3]
     │                                                │
     │                                                ▼
     │                                    ╔═══════════════════════╗
     │                                    ║ GET /stories/_search  ║
     │                                    ║                       ║
     │ Search results                     ║ {                     ║
     │                                    ║   "query": {          ║
     └────────────────────────────────────╫     "terms": {        ║
                                          ║       "user": [2, 3]  ║
                                          ║     }                 ║
                                          ║   }                   ║
                                          ║ }                     ║
                                          ╚═══════════════════════╝
```

- The coordinating node sees that the query needs the terms to be filled,
  - so it makes a GET query first to retrieve the matching data, and feeds that to the `terms` query, then the query is executed

- We could do this at the application level, but we would do two queries and incur costs from
  - the additional network latency
  - data transfer
  - If we're transferring 1000 users, that's lots of data (if user ids were UUIDs for example).
- If we can, it's just better to do this from ES
  - Let ES do everything internally for best performance
  - Performance factors include the mapping and how much data there is
  - The more terms, the slower the query
  - Keep this in mind when looking up lots (thousands) of terms
  - Elasticsearch has a term limit limit of approximately 65,000 terms
  - Can be configured per index, but keep performance in mind

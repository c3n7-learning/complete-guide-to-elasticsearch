# Sending Queries with cURL

We previously used Kibana's Console because of its benefits:

- The easiest way to run Elasticsearch queries
- Formatting of query results
- Automatically sets things like `Content-Type` and authentication headers
- Auto completion when writing queries
- We'll be using it throughout the course.

But, we can use cURL, and we'll demo it for now.

First try, use elasticsearch's url:

```shell
$ curl -X GET http://localhost:9200
curl: (52) Empty reply from server
```

The output is empty because we haven't used HTTPs:

```shell
$ curl -X GET https://localhost:9200

curl: (60) SSL certificate problem: self signed certificate in certificate chain
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above
```

The certificate used by ES is self signed. So we could

1. Simply tell curl to not verify the tls certificate

```shell
curl --insecure -X GET https://localhost:9200 | jq
```

2. Or specify to curl the certificate it should verify against

```shell
cd elasticsearch
curl --cacert config/certs/http_ca.crt -X GET https://localhost:9200 | jq
```

Whichever we use, the output is:

```json
{
  "error": {
    "root_cause": [
      {
        "type": "security_exception",
        "reason": "missing authentication credentials for REST request [/]",
        "header": {
          "WWW-Authenticate": [
            "Basic realm=\"security\", charset=\"UTF-8\"",
            "Bearer realm=\"security\"",
            "ApiKey"
          ]
        }
      }
    ],
    "type": "security_exception",
    "reason": "missing authentication credentials for REST request [/]",
    "header": {
      "WWW-Authenticate": [
        "Basic realm=\"security\", charset=\"UTF-8\"",
        "Bearer realm=\"security\"",
        "ApiKey"
      ]
    }
  },
  "status": 401
}
```

The error we get is authentication deployment

- Run:

```shell
curl --cacert config/certs/http_ca.crt -u elastic -X GET https://localhost:9200 | jq
```

- Or (insecure):

```shell
curl --cacert config/certs/http_ca.crt -u elastic:zwDitFPJsDTqWgFTA7pn -X GET https://localhost:9200 | jq
```

- We get the output:

```json
{
  "name": "Nvs-MacBook-Pro.local",
  "cluster_name": "elasticsearch",
  "cluster_uuid": "B-__7ZfyTeKnSIQsFE_0Og",
  "version": {
    "number": "9.3.2",
    "build_flavor": "default",
    "build_type": "tar",
    "build_hash": "43a703737aab6baefa748bc7b69e4054926f2b2c",
    "build_date": "2026-03-16T13:12:56.143057855Z",
    "build_snapshot": false,
    "lucene_version": "10.3.2",
    "minimum_wire_compatibility_version": "8.19.0",
    "minimum_index_compatibility_version": "8.0.0"
  },
  "tagline": "You Know, for Search"
}
```

Let us try to search a `products` index, which doesn't exist yet. the `_search` api requires us to send some params. We'll talk more about that later:

- Run

```shell
curl --cacert config/certs/http_ca.crt -u elastic:zwDitFPJsDTqWgFTA7pn -X GET -H "Content-Type:application/json" https://localhost:9200/products/_search -d '{ "query": { "match_all" : {} }}' | jq
```

- We get this error, which we expect:

```shell
{
  "error": {
    "root_cause": [
      {
        "type": "index_not_found_exception",
        "reason": "no such index [products]",
        "resource.type": "index_or_alias",
        "resource.id": "products",
        "index_uuid": "_na_",
        "index": "products"
      }
    ],
    "type": "index_not_found_exception",
    "reason": "no such index [products]",
    "resource.type": "index_or_alias",
    "resource.id": "products",
    "index_uuid": "_na_",
    "index": "products"
  },
  "status": 404
}
```

# Importing data with cURL

- We will use the cURL HTTP client in this lecture
  - Should be already installed on macOS and Linux
  - Some versions of Windows should have this.
- Download the test data `products-bulk.json`, currently named in the directory as `36-products-bulk.json`

Now run the following commands:

- Enter the directory with the downloaded json

```shell
cd 03-managing-documents
```

- Run the curl command

```shell
curl --insecure -H "Content-Type: application/x-ndjson" -u elastic:zwDitFPJsDTqWgFTA7pn -X POST https://localhost:9200/products/_bulk --data-binary "@36-products-bulk.json" | jq
```

Let's examine the shards in our cluster

- Run:

```shell
GET /_cat/shards?v
```

- Output:

| Index    | Shard | Prirep | Docs | Node                  |
| -------- | ----- | ------ | ---- | --------------------- |
| products | 0     | r      | 530  | Nvs-MacBook-Pro.local |
| products | 0     | r      | 530  | second-node           |
| products | 0     | p      | 530  | third-node            |
| products | 1     | p      | 470  | Nvs-MacBook-Pro.local |
| products | 1     | r      | 470  | second-node           |
| products | 1     | r      | 470  | third-node            |

- The documents have been evenly distributed across the two shards

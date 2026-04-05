# Introductions to Aggregations

- Aggregations in ES are way more powerful than what we've come to expect in relational databases

For example:

- We could query the total/average number of sold products grouped by the name, for example

```
  ORDER A        ORDER B       ORDER C      ORDER D
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
 PRODUCT A     PRODUCT A     PRODUCT B     PRODUCT B
 AMOUNT: 50    AMOUNT: 20    AMOUNT: 10    AMOUNT: 75

    \             \         /             /
     \             \       /             /
      \             \     /             /
       \             \   /             /
        \             \ /             /
         \             V             /
          \           /|\           /
           \         / | \         /
            \       /  |  \       /
             \     /   |   \     /
              \   /    |    \   /
               \ /     |     \ /
                ▽      |      ▽
             ╔════════════════════╗
             ║                    ║
             ║        FUNNEL      ║
             ║                    ║
             ╚════════════════════╝
                        |
                        |
                        ▼

PRODUCT A                              PRODUCT B
    |                                      |
TOTAL AMOUNT (SUM): 70                TOTAL AMOUNT (SUM): 85
AVERAGE AMOUNT: 35                    AVERAGE AMOUNT: 42.5
```

## Adding `orders` index with field mappings

```
PUT /orders
{
  "mappings": {
    "properties": {
      "purchased_at": {
        "type": "date"
      },
      "lines": {
        "type": "nested",
        "properties": {
          "product_id": {
            "type": "integer"
          },
          "amount": {
            "type": "double"
          },
          "quantity": {
            "type": "short"
          }
        }
      },
      "total_amount": {
        "type": "double"
      },
      "status": {
        "type": "keyword"
      },
      "sales_channel": {
        "type": "keyword"
      },
      "salesman": {
        "type": "object",
        "properties": {
          "id": {
            "type": "integer"
          },
          "name": {
            "type": "text"
          }
        }
      }
    }
  }
}
```

## Populating the `orders` index with test data

- Download the data from here: https://github.com/codingexplained/complete-guide-to-elasticsearch/blob/master/orders-bulk.json
- If you are using a cloud hosted Elasticsearch deployment, remove the `--cacert` argument.

macOS & Linux:

```
curl --cacert config/certs/http_ca.crt -u elastic -H "Content-Type:application/x-ndjson" -X POST https://localhost:9200/orders/_bulk --data-binary "@orders-bulk.json"
```

Windows:

```shell
curl --cacert config\certs\http_ca.crt -u elastic -H "Content-Type:application/x-ndjson" -X POST https://localhost:9200/orders/_bulk --data-binary "@orders-bulk.json"
```

For my local (macOs without https):

```shell
curl -u elastic:zwDitFPJsDTqWgFTA7pn -H "Content-Type:application/x-ndjson" -X POST http://localhost:9200/orders/_bulk --data-binary "@08-aggregations/111-orders-bulk.json"
```

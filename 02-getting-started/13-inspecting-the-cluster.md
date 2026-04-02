# Inspecting the cluster

Go to the dev tools:

1. Kibana (http://localhost:5601)
2. Management -> Dev Tools

Kibana simply runs HTTP calls on our behalf, we could write our own REST API Client, but why do that when Kibana has done this for us?

### Our first query

Kibana works with HTTP verbs.

#### Let us test out retrieving data:

Run:

```
GET /_cluster/health
```

You will see:

```json
{
  "cluster_name": "elasticsearch",
  "status": "green",
  "timed_out": false,
  "number_of_nodes": 1,
  "number_of_data_nodes": 1,
  "active_primary_shards": 39,
  "active_shards": 39,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 0,
  "unassigned_primary_shards": 0,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 100
}
```

- Consider this url `/_cluster/health`
  - `_cluster` is the `API` we want to access
  - `health` is the command
- There are more commands that we can check, but that is beyond the scope of this lecture

Let's dig in to the results we've gotten
| Key/Value | Notes |
| :------- | :--- |
| `"cluster_name": "elasticsearch"` | The cluster name can be configured with `$ES_HOME` > `config` > `elasticsearch.yml` |
| `"status": "green"` | The status of the cluster is healthy/green |

#### Let us list out the nodes

Run:

```
GET /_cat/nodes?v
```

You will see a table which basically shows this data:

```json
{
  "ip": "127.0.0.1",
  "heap.percent": 7,
  "ram.percent": 98,
  "cpu": 33,
  "load_1m": 4.01,
  "load_5m": null,
  "load_15m": null,
  "node.role": "cdfhilmrstw",
  "master": "*",
  "name": "Nvs-MacBook-Pro.local"
}
```

- We can see that the node has role `*` and is the master node.

References

- https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-cat-nodes
- https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-nodes-info

#### Let us check the list of indices we have

Run:

```
RUN /_cat/indices
```

- We won't see any output. That's not to mean we don't have indices, as there are system indices. We can view them via:

```
GET /_cat/indices?v&expand_wildcards=all
```

You'll see output like:

```json
[
  {
    "health": "green",
    "status": "open",
    "index": ".internal.alerts-transform.health.alerts-default-000001",
    "uuid": "c2pBhO80RmKzmk25fqDHxg",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".slo-observability.sli-v3.6",
    "uuid": "mqrhpkE5TRe-C40hsmwEoQ",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".internal.alerts-ml.anomaly-detection.alerts-default-000001",
    "uuid": "UJFuFramTdioP7mgwGqbmA",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".internal.alerts-observability.slo.alerts-default-000001",
    "uuid": "EgKZcgEfR66EgyYK3EvKrw",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".kibana_security_session_1",
    "uuid": "eRZLRD5QSGGuf6JOS5xSfw",
    "pri": 1,
    "rep": 0,
    "docs.count": 1,
    "docs.deleted": 0,
    "store.size": "6.9kb",
    "pri.store.size": "6.9kb",
    "dataset.size": "6.9kb"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".internal.alerts-default.alerts-default-000001",
    "uuid": "mqI9NZmGQ7q-FdFR128Hpg",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".internal.alerts-observability.apm.alerts-default-000001",
    "uuid": "hzPBrg1zRCi1jF6qRMrCTw",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".internal.alerts-streams.alerts-default-000001",
    "uuid": "LHMlyr3vTvGXETR5PhT0vQ",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".kibana_ingest_9.3.2_001",
    "uuid": "kF58BCpVR6-P1L1P8aoe7A",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".internal.alerts-security.attack.discovery.alerts-default-000001",
    "uuid": "tI1oR3kjS72kPUq3UlylNw",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".internal.alerts-observability.metrics.alerts-default-000001",
    "uuid": "klIWNvi9QXGcMr3zWO3veA",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".kibana-siem-rule-migrations-integrations",
    "uuid": "Dld2xAYxSvysbshfhf0niw",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".apm-custom-link",
    "uuid": "Wo7Q2mBYQpSRHVCna8TrHw",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".kibana_usage_counters_9.3.2_001",
    "uuid": "y7G0UMp6REGzS9A4mxFrSA",
    "pri": 1,
    "rep": 0,
    "docs.count": 83,
    "docs.deleted": 4,
    "store.size": "104.1kb",
    "pri.store.size": "104.1kb",
    "dataset.size": "104.1kb"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".internal.alerts-ml.anomaly-detection-health.alerts-default-000001",
    "uuid": "r5QfY4c-QPiWYC2jl-IVsg",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".kibana_locks-000001",
    "uuid": "ZOlHZoYARHKCO_l0Q0Aayg",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 2,
    "store.size": "29.3kb",
    "pri.store.size": "29.3kb",
    "dataset.size": "29.3kb"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".security-profile-8",
    "uuid": "6o4it6LDSlenS3zkFKdtWw",
    "pri": 1,
    "rep": 0,
    "docs.count": 1,
    "docs.deleted": 0,
    "store.size": "8.8kb",
    "pri.store.size": "8.8kb",
    "dataset.size": "8.8kb"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".internal.alerts-security.alerts-default-000001",
    "uuid": "WUAimB03QkypQdFEp5it0A",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".ds-ilm-history-7-2026.04.02-000001",
    "uuid": "aaGrH-8HTs66OcZ6ifvYnQ",
    "pri": 1,
    "rep": 0,
    "docs.count": 45,
    "docs.deleted": 0,
    "store.size": "16.7kb",
    "pri.store.size": "16.7kb",
    "dataset.size": "16.7kb"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".internal.alerts-stack.alerts-default-000001",
    "uuid": "HX8h_yczQ_62txmGT7P00Q",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".kibana_alerting_cases_9.3.2_001",
    "uuid": "FUfD-pmmSsClvIm-JAYwTg",
    "pri": 1,
    "rep": 0,
    "docs.count": 1,
    "docs.deleted": 0,
    "store.size": "7.5kb",
    "pri.store.size": "7.5kb",
    "dataset.size": "7.5kb"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".kibana_search_solution_9.3.2_001",
    "uuid": "DkiGIIXoTO-dh1Lh2Bh_Ow",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".internal.alerts-observability.logs.alerts-default-000001",
    "uuid": "sJvr1N_MRVy3RntTfXxkBw",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".internal.alerts-observability.uptime.alerts-default-000001",
    "uuid": "UALXDpgOSKGnE4wdYWVfOg",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".kibana_task_manager_9.3.2_001",
    "uuid": "KGg2enYLQ-u6YTmyZCYWnw",
    "pri": 1,
    "rep": 0,
    "docs.count": 49,
    "docs.deleted": 6,
    "store.size": "167.1kb",
    "pri.store.size": "167.1kb",
    "dataset.size": "167.1kb"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".apm-agent-configuration",
    "uuid": "HL_A7APKTm2IaubaZJQu9A",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".apm-source-map",
    "uuid": "G0OKTuiBRr2xjtW3hszbIQ",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".ds-.kibana-event-log-ds-2026.04.02-000001",
    "uuid": "ono3K2lZRzKcjBrLQddKgg",
    "pri": 1,
    "rep": 0,
    "docs.count": 1,
    "docs.deleted": 0,
    "store.size": "6.3kb",
    "pri.store.size": "6.3kb",
    "dataset.size": "6.3kb"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".ds-.edr-workflow-insights-default-2026.04.02-000001",
    "uuid": "2LoVOUDBRiylNkcbpAQatw",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".kibana_analytics_9.3.2_001",
    "uuid": "n0TAhRfoT62YrAtSbY8t3w",
    "pri": 1,
    "rep": 0,
    "docs.count": 5,
    "docs.deleted": 0,
    "store.size": "1.8mb",
    "pri.store.size": "1.8mb",
    "dataset.size": "1.8mb"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".slo-observability.summary-v3.6.temp",
    "uuid": "tJcFXdfwS4CV5FMa3k-dnQ",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".slo-observability.summary-v3.6",
    "uuid": "_Q3--wcjQJuZX7YJUxdoTg",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".security-7",
    "uuid": "_OwbSxCUT6Gws3s2DBLWcg",
    "pri": 1,
    "rep": 0,
    "docs.count": 302,
    "docs.deleted": 0,
    "store.size": "985.4kb",
    "pri.store.size": "985.4kb",
    "dataset.size": "985.4kb"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".kibana_9.3.2_001",
    "uuid": "Q8s51mWLSsGYkXChkCLxbg",
    "pri": 1,
    "rep": 0,
    "docs.count": 27,
    "docs.deleted": 2,
    "store.size": "37.8kb",
    "pri.store.size": "37.8kb",
    "dataset.size": "37.8kb"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".kibana-siem-rule-migrations-prebuiltrules",
    "uuid": "WERCtyLcSqi7A_d01_xYiQ",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".internal.alerts-observability.threshold.alerts-default-000001",
    "uuid": "jMYhiC8bRHuekM7OiExsoQ",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".kibana_security_solution_9.3.2_001",
    "uuid": "Oiab82BeSMqFmJsjif-bzA",
    "pri": 1,
    "rep": 0,
    "docs.count": 4,
    "docs.deleted": 0,
    "store.size": "45.1kb",
    "pri.store.size": "45.1kb",
    "dataset.size": "45.1kb"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".internal.alerts-dataset.quality.alerts-default-000001",
    "uuid": "KxYK3xvwT86TG0O1VPWybQ",
    "pri": 1,
    "rep": 0,
    "docs.count": 0,
    "docs.deleted": 0,
    "store.size": "249b",
    "pri.store.size": "249b",
    "dataset.size": "249b"
  },
  {
    "health": "green",
    "status": "open",
    "index": ".ds-.logs-elasticsearch.deprecation-default-2026.04.02-000001",
    "uuid": "lPIehbk9T2Kv5b23f_Yo7w",
    "pri": 1,
    "rep": 0,
    "docs.count": 2,
    "docs.deleted": 0,
    "store.size": "24kb",
    "pri.store.size": "24kb",
    "dataset.size": "24kb"
  }
]
```

- Config data e.g. for kibana is stored in ES.
- These so-called _system indices_ have a leading `.` in the name. Following the linux convention, files with a preceding `.` are hidden by default.

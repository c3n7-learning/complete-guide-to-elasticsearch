# Overview of the Elastic Stack

## Kibana

### Introduction

- An analytics & visualization platform.
- You can use many built-in visualizations
  - e.g. charts for users, traffic
- Can configure machine learning in Kibana for change/anomaly detection
- Can manage parts of Elasticsearch and Logstash
  - e.g. authentication & authorization
- You can think of it as an interface to the data stored on Elasticsearch
- You can build dashboards for visualizations and metrics.
  - [You can find a demo of what Kibana can do here](https://demo.elastic.co/)

## Logstash

### Introduction

- Is a data processing pipeline which
- Processes data streams like events
  - e.g. logs, kafka streams,e tc
- A pipeline consists of 3 parts
  - Inputs (plugins that receive input)
  - Filters (plugins that process/enrich raw input)
  - Outputs (plugins that send data to stashes e.g. kafka, elasticsearch)

## X-Pack

- Is actually a pack of features
- Adds additional features to the Elasticsearch & Kibana

These features include:

- Security: Adds authentication and authorization
  - Can integrate with authentication providers e.g. LDAP
  - Control permissions with fine-grained authorization
- Monitoring: Gain insight into how the Elastic Stack is running.
  - e.g. CPU & memory usage
- Alerting: Get notified if something goes wrong
  - Can be used for lots of things, not just things go wrong
  - e.g. CPU usage exceeds 90%, or a user logs in via several IP addresses
  - Be notified by email, Slack, etc
- Reporting: Export Kibana visualizations and data
  - Reports can be generated on-demand or scheduled and sent to email
  - Can be triggered when certain conditions are fulfilled
  - Can customize look and feel of reports
  - Export data as CSV, e.g. for use within spreadsheets
- Machine Learning: Enables ML for Elasticsearch & Kibana
  - For abnormality detection and such.
- Graph: Analyze the relationships in your data
  - e.g. show related products or suggest songs based on what the listener likes.
  - Popular is not the same as relevant.
  - 10 random Google users are unlikely to have much in common.
  - 10 Stackoverflow users **do** have something in common.
  - We want the _uncommonlly_ _common_ signals relevance.
  - Graphs considers relevance with Elasticsearch
  - Exposes an API you use to Integrate into applications.
  - Provides a plugin wiht Kibana to visualize the relationships.
  - Graph works out of the box ("plug and play")
- Elasticsearch SQL: Query Elasticsearch with SQL
  - Elasticsearch queries are written in Query DSL
  - Query DSL is Flexible, but also a bit verbose at first
  - You can now send SQL over HTTP or through a JDBC driver
  - Elasticsearch translates the SQL internally
  - The translate API returns the corresponding Query DSL
  - This can be a great way to get started with a query
  - Perhaps most useful during development and getting started

## Beats

Is a collection of so called _data shippers_. Lightweight agents with a single purpose that collect and send data to ElasticSearch

- Filebeat: collects log files and sends the entries to Logstash or Elasticsearch
  - Useful for collecting error logs or access logs
  - Includes modules for common services (e.g. nginx & MySQL)
- Metricbeat: collects system and service metrics
  - e.g. memory and CPU usage
  - Includes modules for common services (e.g. nginx & MySQL)
- Packetbeat: Collects network data
  - e.g. HTTP requests or database transactions
- Winlogbeat: Collects Windows Event Logs
- Auditbeat: Collects audit data from Linux
- Heartbeat: Monitors service uptime

## Summary

1. Data Ingestion
   - Beats
   - Logstash
2. Search, Analyze & Store Data
   - Elasticsearch
3. Kibana - Visualize your data
   x. X-Pack: Adds featres to the Elastic Stack

You have heard of the ELK Stack:

- E: Elasticsearch
- L: Logstash
- K: Kibana

The term originates from a time before Beats and Logstash

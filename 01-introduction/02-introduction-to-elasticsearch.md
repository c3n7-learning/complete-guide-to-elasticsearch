# Introduction to Elasticsearch

It supports:

- Full-text search
- Query and analyze structured data for analytics.
  - It is not realy an analytics solution, but it can do lots of good.
- Analyze application logs and system metrics
  - e.g. errors and CPU/memory usage
  - Is referred to Application Performance Management (APM)
- Send events to Elasticsearch
  - Sales, website clicks, phone calls, etc.
  - We can do that with "aggregations"
  - Elastic search is great at analyzing lots of data
  - Forecast future values with machine learning.
- Anomality Detection
  - Find when there's an anomality from normal behavior
  - You don't have to configure that automatically

> Note  
> This course focuses primarily on searching.
> This is the **core** of Elasticsearch

### How does Elastic Search Work?

- Data is stored as documents
  - Similar to rows in relational databases (e.g. MySQL)
  - Is essentially just a JSON document
- Querying Elasticsearch
  - Over REST API
- Written in Java, built on Apache Lucerne
- Gained popularity due to **ease of use**, and it's **high scalability**.
- Used by large companies like:
  - Firefox
  - FAcebook
  - StackEschange
  - Netflix
  - Github
  - SoundCloud
  - FourSquare

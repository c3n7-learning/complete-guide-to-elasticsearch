# Elasticsearch vs OpenSearch

In summary? AWS vs Elastic the Company

- From version 7.11, Elasticsearch & Kibana changed from Apache License 2.0 to Service Side Public License (SSPL)
- The SSPL license ensures cloud providers collaborate with Elastic
- The source code is still publicly available
- The software is technically no longer considered "open source"
- Elasticsearch & Kibana is now distributed under the Elastic License
  - The core is still free to use. Some features require a paid license.

AWS Didn't like this:

- They had been maintained a parallel version called Open Distro for Elastcisearch
- They renamed it to Opensearch
- From 7.10, Opensearch moved to its own version `1.0` `1.1`, etc.

Elasticsearch vs Opensearch

- The Core APIs are the same:
  - Search Queries
  - Indexing Documents
  - Aggregations
  - Index Management
  - Basic Security
- There are differences:
  - Elastic:
    - SIEM
    - Observability
    - Enterprise Search
    - ML
    - etc
  - Open Search
    - Security Analytics
    - Observability
    - ML
    - etc

How do you want to host your cluster?

- For self hosting, Elasticsearch requires a license for some features
- Both can be hosted pretty much anywhere
- For managed services, check pricing and features
  - Also consider integrations with your existing cloud provider
- Elastic Search
  - Elastic Cloud
  - GCP
  - Azure
  - Alibaba Cloud
- Open Search
  - Bonsai: they have a free forever trial
  - AWS
  - Digital Ocean
  - Aven
  - Instaclustr
  - ExoScale
- Elastic Cloud is built on top of cloud providers
  - It includes AWS, GCP, Azure
  - But Elastic manages them. Cloud providers get paid for the infra, Elastic Cloud gets paid for managing the things
  - Some native cloud integrations may be unavailable, e.g. AWS Kinesis
  - Licenses are "baked into" Elastic Cloud & Elasticsearch cloud services.

> Note:  
> We will use ElasticSearch & Kibana for this Course

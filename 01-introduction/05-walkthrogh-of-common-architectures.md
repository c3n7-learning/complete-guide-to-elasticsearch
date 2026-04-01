# Walkthrough of common architectures

## E-Commerce

Normal Architecture

- User searches for product
- System searches the DB
- System responds to user
- _DBs are not known for being good at Full-text search_

Now introduce Elasticsearch

- User searches for product
- System search ElasticSearch for the product
- Communicating with Elasticsearch
  - Can be done with any HTTP Library
  - Using thee official client libraries is recommend

How does the data get to Elasticsearch

- The web app should keep the data updated on both the DB and Elasticsearch
- What if we have 100,000 products? won't the initial ingestion take forever?
  - You will need to write a script that imports data
  - From there, the web app will keep the data up to date
- This simple architecture is what's often used

Now the boss wants reports:

- We could build our own interface
- Use **Kibana** Instead
- Kibana can run on any machine

Over time, the web traffic increases significantly

- We need to monitor server resources
- We will do this with Metricbeat, install it on the webserver

How do we get the data into Elasticsearch?

- Configure Metricbeat to send data to Elasticsearch. (in practice, data is sent to the ingest node)
- We can now visualize this data with Kibana
- Metricbeat ships with default Kibana dashboards
- Kibana stores its configuration within Elasticsearch
- We can optionally configure alerting

The development team and code base is growing

- We want to monitor the access and error logs
- Response times can be monitored for each endpoint
- We can identify bad deployments (increased response times or 4xx/5xx HTTP status codes)
- We can accomplish all of this with Filebeat
- Filebeat ships with modules for common log file formats

6 months later

- Two more web servers have been added
- More kinds of data is added to Elasticsearch
- Now we need to do more advanced event processing
- We want to do data enrichment.
- We could do this in our logic, but introduces more code that is not business logic
  - Decentralized event processing/microservices complicates this
  - What if we could centralize event processing?
- We can do that with Logstash
- What about the data from Metricbeat and Filebeat?
  - For Metricbeat, we won't necessarily need to post-process
  - For filebeat, we can optionally send the data to logstash
- In a perfect world, all events go through Logstash

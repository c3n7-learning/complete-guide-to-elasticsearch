# A word on document types

I need to steal your attention for a moment to talk about `mapping types`. These were a thing for older versions of Elasticsearch, but will be removed in version 8.x. They are already deprecated and are only available when actively opting into using them. However, this means that pretty much all endpoints for the Elasticsearch API has changed. Whenever you see `default` as part of an endpoint, this refers to the mapping type.

We have been using queries that are compatible with Elasticsearch 7.0.0 and later until now, but you are about to see some queries that use a type of `default`. The reason for that is that those lectures were recorded before `_doc` was a thing. Some APIs now use `_doc` instead of default, while others completely got rid of that part of the endpoint. For the queries that contain `default`, please check the [GitHub repository](https://github.com/codingexplained/complete-guide-to-elasticsearch) for the updated queries. The difference is just semantics, so hopefully it won't be an issue for you.

For instance, when you see the following:

`GET /products/default/_search`

... you should replace it with:

`GET /products/_search`

I am gradually updating the course with the latest queries, but please understand that re-recording and editing 10+ hours of content takes a long time, so there will be a period in which you will see the old document type being specified for parts of the course. This is a work in progress, and this document will be pushed forward as I make progress. 🙂 If in doubt, please refer to the [GitHub repository](https://github.com/codingexplained/complete-guide-to-elasticsearch).

Thank you for your understanding, and happy searching!

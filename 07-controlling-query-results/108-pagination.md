# Pagination

Reference:

- https://www.elastic.co/docs/reference/elasticsearch/rest-apis/paginate-search-results

## Introduction

To have pagination on our platform

- We need to calculate the total number of pages

```
total_pages = ceil(total_hits / page_size)

14 = ceil(137 / 10)
```

- To go from one page to another

```
from = (page_size * (page_number - 1))

50 = (10 * (6 - 1))
```

Note that this approach is limited to 10,000 results

- When doing deep pagination, requests take up more and more heap memory, and requests take longer
- This limit helps us maintain stability
- Usually, users won't have to get to deeper than the first couple of pages

To work against this, [we have `search_after` that we can utilize](https://www.elastic.co/docs/reference/elasticsearch/rest-apis/paginate-search-results#search-after)

## How it works

- ES handles the search query, and when it returns results, it's completly done with it, so we can't use cursors. ES is stateless

Updates/deletes while queries happen:

- Suppose that a user searches for something and gets 100 hits
- Then suppose a document was added
- When the user navigates to the second page, the user will see a result they saw on the first pge

# Mapping Recommendations

- Most of these are the instructors opinions from their experience

## Use explicit mappings

- Dynamic mapping is convenient, but often not a good idea in production
- Save disk space with optimized mappings when storing many documents
- Set `dynamic` to `"strict"`, not `false`.
  - Avoids surprises and unexpected results

## Mapping of text fields

- Don't always map strings as both `text` and `keyword, unless you need to
  - Typically only one is needed
  - Each mapping requires disk space
- Do you need to perform full-text searches?
  - Add a `text` mapping
- Do you need to do aggregations, sorting, or filtering on exact values?
  - Add a `keyword` mapping

## Disable coercion

- Coercion forgives you for not doing the right thing
- Try to do the right thing instead
- Always use the correct data types whenever possible

## Use appropriate numeric data types

- For whole numbers, the `integer` data type might be enough
  - `long` can store larger numbers, but also uses more disk space
- For decimal numbers, the `float` data type might be precise enough
  - `double` stores numbers with a higher precision but uses 2x disk space
  - Usually, `float` provides enough precision

## Mapping parameters

- Set `doc_values` to `false` if you don't need sorting, aggregations, and scripting
- Set `norms` to `false` if you don't need relevance scoring
- Set `index` to `false` if you don't need to filter on values
  - You can still do aggregations, e.g. for time series data
- Probably only worth the effort when storing lots of documents
  - Otherwise it's probably an over complication
  - It's probably not worth it for less than a million documents
- Worst case scenario, you will need to reindex documents

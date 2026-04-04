# How Missing Fields are Handled

## Missing fields

- All fields in ES are optional
- You can leave out a field when indexing documents
- E.g., unlike relational databases where you need to allow NULL values
- Some integrity checks need to be done at the application level
  - e.g. having required fields
- Adding a field mapping does not make a field required
- Search automatically, ES will handle missing fields
  - You don't have to worry about missing fields

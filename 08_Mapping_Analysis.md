# Introduction to Analysis

When a document is indexed, it is processed by analyzer before storing it.
Analyzer may have : 
- Character filter
    - Ex : Remove html strips
- Tokenizer( only one)
    -  Splits text into token
- Token Filter
    - Add, remove or modify tokens

Character filter is not default, but basic tokenizer and token filter are default.

## Using the Analyze API

### Analyzing a string with the `standard` analyzer
```json
POST /_analyze
{
  "text": "2 guys walk into   a bar, but the third... DUCKS! :-)",
  "analyzer": "standard"
}
```
```json
// Result
    {
      "token": "walk",
      "start_offset": 7,
      "end_offset": 11,
      "type": "<ALPHANUM>",
      "position": 2
    },
```

### Building the equivalent of the `standard` analyzer
```json
POST /_analyze
{
  "text": "2 guys walk into   a bar, but the third... DUCKS! :-)",
  "char_filter": [],
  "tokenizer": "standard",
  "filter": ["lowercase"]
}
// The result is the same
```
# Understanding Inverted Indices
Inverted Indices is Mapping of terms(tokens) and documents containing them
- Searches and inverted indexing for text fields are done by Apache Luecene not Elastic Search
- Luecene Uses Inverted Index. 
  - Instead of: ```Document → terms```
  - Lucene stores: ```terms → List of Documents```
  - This is what makes search fast.

- Inverted indices store more than just terms and document IDs, such as relavence scoring etc.

- Inverted index is used for only text data types. Other uses different data structures like BKD, geospatial data etc.


# Mapping
Defines the structure of Documents ( eg : fields and data types) like a table schema in  RDB.

Types : 
- Explicit Mapping ( user defined)
- Dynamic Mapping  ( Elastic Search Generated)

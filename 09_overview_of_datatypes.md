
## Overview of datatypes
integer, long, text, boolean, object, float etc( you know them)
### Object
Used for any Json object, Mapped using the properties parameter again :
```json
PUT my-index-000001/_doc/1
{ //
  "region": "US",
  "manager": {
    "age":     30,
    "name": {
      "first": "John",
      "last":  "Smith"
    }
  }
}
```
 However, Apache Luecene flattens them while storing as inverted inex like author.first_name, author.email instead of objects.
```json
{
  "region":             "US",
  "manager.age":        30,
  "manager.name.first": "John",
  "manager.name.last":  "Smith"
}
```

Explicit Mapping : 
```json
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "region": {
        "type": "keyword"
      },
      "manager": {
        "properties": {
          "age":  { "type": "integer" },
          "name": {
            "properties": {
              "first": { "type": "text" },
              "last":  { "type": "text" }
            }
          }
        }
      }
    }
  }
}
```

#### Problem : 
```json 
PUT my-index-000001/_doc/1
{
  "group" : "fans",
  "user" : [
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]
}

// This is stored like 
{
  "group" :        "fans",
  "user.first" : [ "alice", "john" ],
  "user.last" :  [ "smith", "white" ]
}
// The user.first and user.last fields are flattened into multi-value fields, and the association between alice and white is lost. This document would incorrectly match a query for alice AND smith:
GET my-index-000001/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "user.first": "Alice" }},
        { "match": { "user.last":  "Smith" }}
      ]
    }
  }
}
```
Solution is Using **nested** data type. 

```json
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "user": {
        "type": "nested"
      }
    }
  }
}
// Now the flatteing will not happen, because it is stored as nested datatype instead of object
// They are stored as hidden documents, appears only for searches
```

### Keyword
- keyword, which is used for structured content such as IDs, email addresses, hostnames, status codes, zip codes, or tags.
- Keyword fields are often used in sorting, aggregations
- Avoid using keyword fields for full-text search. Use the text field type instead.

```json
 PUT my-index-000001
{
  "mappings": {
    "properties": {
      "tags": {
        "type":  "keyword"
      }
    }
  }
}
```
- Keywords are analyzed using keyword analyzer, which returns noops, unmodified string.
- Consider mapping a numeric identifier as a keyword if:

    - You don’t plan to search for the identifier data using range queries.

#### Testing the `keyword` analyzer
```json
POST /_analyze
{
  "text": "2 guys walk into   a bar, but the third... DUCKS! :-)",
  "analyzer": "keyword"
}
// ouptut is just untouched and returned as a single token
```

### Coercion

#### Supplying a floating point
```json
PUT /coercion_test/_doc/1
{
  "price": 7.4
}
// Supplying a floating point within a string
PUT /coercion_test/_doc/2
{
  "price": "7.4"
}

// string is converted into float ( this is coercion)
```
```json
PUT /coercion_test/_doc/3
{
  "price": "7.4m"
}
// query failed
```

#### Retrieve document
```json
GET /coercion_test/_doc/2
// result
 "_source": {
    "price": "7.4"
  }
```
**Note** : Elastic search uses BKD trees and Inverted index to search not _source.
#### Clean up
```
DELETE /coercion_test
```

### Arrays
In Elasticsearch, there is no dedicated array data type. 
- Any field can contain zero or more values by default, however, all values in the array must be of the same data type
- Arrays with a mixture of data types are not supported: [ 10, "some string" ]

Arrays of strings are concatenated when analyzed
```
POST /_analyze
{
  "text": ["Strings are simply", "merged together."],
  "analyzer": "standard"
}
```

Example 2 : 
```json
PUT my-index-000001/_doc/1
{
  "message": "some arrays in this document...",
  "tags":  [ "elasticsearch", "wow" ],
  "lists": [
    {
      "name": "prog_list",
      "description": "programming list"
    },
    {
      "name": "cool_list",
      "description": "cool stuff list"
    }
  ]
}
PUT my-index-000001/_doc/2
{
  "message": "no arrays in this document...",
  "tags":  "elasticsearch",
  "lists": {
    "name": "prog_list",
    "description": "programming list"
  }
}
GET my-index-000001/_search
{
  "query": {
    "match": {
      "tags": "elasticsearch"
    }
  }
}
```

# How dates work in Elasticsearch
Dates are stored as milliseconds since epoch ( long datatype).

Every format you send is converted UTC timezone milliseconds ( long datatype)
## Supplying only a date
```json
PUT /reviews/_doc/2
{
  "rating": 4.5,
  "content": "Not bad. Not bad at all!",
  "product_id": 123,
  "created_at": "2015-03-27",
  "author": {
    "first_name": "Average",
    "last_name": "Joe",
    "email": "avgjoe@example.com"
  }
}
```

## Supplying both a date and time
```json
PUT /reviews/_doc/3
{
  "rating": 3.5,
  "content": "Could be better",
  "product_id": 123,
  "created_at": "2015-04-15T13:07:41Z",
  "author": {
    "first_name": "Spencer",
    "last_name": "Pearson",
    "email": "spearson@example.com"
  }
}
```

## Specifying the UTC offset
```json
PUT /reviews/_doc/4
{
  "rating": 5.0,
  "content": "Incredible!",
  "product_id": 123,
  "created_at": "2015-01-28T09:21:51+01:00",
  "author": {
    "first_name": "Adam",
    "last_name": "Jones",
    "email": "adam.jones@example.com"
  }
}
```

## Supplying a timestamp (milliseconds since the epoch)
```json
# Equivalent to 2015-07-04T12:01:24Z
PUT /reviews/_doc/5
{
  "rating": 4.5,
  "content": "Very useful",
  "product_id": 123,
  "created_at": 1436011284000,
  "author": {
    "first_name": "Taylor",
    "last_name": "West",
    "email": "twest@example.com"
  }
}
```

## Retrieving documents
```json
GET /reviews/_search
{
  "query": {
    "match_all": {}
  }
}
```
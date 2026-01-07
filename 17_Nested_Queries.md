# Querying nested objects

## Importing test data

Follow these instructions ( `07_Importing_Curl_Data.md` )and specify `recipes-bulk.json` as the file name.

## Searching arrays of objects (the wrong way)

```json
GET /recipes/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        },
        {
          "range": {
            "ingredients.amount": {
              "gte": 100
            }
          }
        }
      ]
    }
  }
}

//Results Example
            // {
            //   "name": "Finely grated Parmesan cheese",
            //   "amount": 60,
            //   "unit": "grams"
            // },
```

- No we see, there are some examples where the `name` is `parmesan` but the amount is less than 100;
- Since we doesnt mention any explicit mappings, the field ingredients is mapped as `object` dynamically.
- While Indexing the objects are flattened as field values.
- Go see the problem of Object datatype in `09_overview_of_datatypes.md`
- Solution 
    - `nested` datatype and `nested` query
    - Create new index to update the field mapping & reindex documents

```
DELETE /recipes
```
```json
PUT /recipes
{
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "description": { "type": "text" },
      "preparation_time_minutes": { "type": "integer" },
      "steps": { "type": "text" },
      "created": { "type": "date" },
      "ratings": { "type": "float" },
      "servings": {
        "properties": {
          "min": { "type": "integer" },
          "max": { "type": "integer" }
        }
      },
      "ingredients": {
        "type": "nested",
        "properties": {
          "name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword"
              }
            }
          },
          "amount": { "type": "integer" },
          "unit": { "type": "keyword" }
        }
      }
    }
  }
}
```

[Import the test data again](#importing-test-data).

### Using the `nested` query is different from previous 

```json
GET /recipes/_search
{
  "query": {
    "nested": {
      "path": "ingredients",
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "ingredients.name": "parmesan"
              }
            },
            {
              "range": {
                "ingredients.amount": {
                  "gte": 100
                }
              }
            }
          ]
        }
      }
    }
  }
}
```

# Nested Inner Hits.
- With `nested` query, matches are root documents.. and `hits.total.value` is shown for whole documents.
- If you want to know which nested object caused the hit, we can use `inner_hits`.

```json
// inner_hits
GET /recipes/_search
{
  "query": {
    "nested": {
      "path": "ingredients",
      "inner_hits": {},
      "query": {
        "bool": {
          "must": [
            { "match": { "ingredients.name": "parmesan" } },
            { "range": { "ingredients.amount": { "gte": 100 }} }
          ]
        }
      }
    }
  }
}
```


### Specifying custom key and/or number of inner hits

```json
GET /recipes/_search
{
  "query": {
    "nested": {
      "path": "ingredients",
      "inner_hits": {
        "name": "my_hits",
        "size": 10
      }, 
      "query": {
        "bool": {
          "must": [
            { "match": { "ingredients.name": "parmesan" } },
            { "range": { "ingredients.amount": { "gte": 100 } } }
          ]
        }
      }
    }
  }
}
```

## Limitation of Nested fields
- Indexing and querying nested fields can be expensive
    - Because apache luecene creates new document for each nested object.
- We need to use specialized query with name "`nested`" and "`path`" 
- 10000, nested objects per document..
    - to prevent OOM exception
- If there are too many nested fields, its better to split that index into two indices like in DBMS Normalization


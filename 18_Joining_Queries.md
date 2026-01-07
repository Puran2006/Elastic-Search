# Joining Queries
- Its a good practice to `normalize` data and maintain relations in Relational DataBases 
- BUT, in elastic we `denormalize` data, becuase that leads to best performance.

`Question` : It is inefficient right, because we can reduce some duplication and save disk space?

`Ans`: 
 - Never use elastic search as primary datasource, it is not designed to.
- It doesnt support complex joins as Relation Databases. They support only simple joins( BUT Expensive)

### Add departments test data using `add-departments-test-data.md`

## Mapping document relationships

```json
PUT /department/_mapping
{
  "properties": {
    "join_field": { // This  can be of any name
      "type": "join",
      "relations": {
        "department": "employee" //department is parent of employee
      }
    }
  }
}
```

### Add more departments docs data using `adding-documents.md`

## Querying by Parent ID
```json
GET /department/_search
{
  "query": {
    "parent_id" :  {
      "type" : "employee",
      "id" : 1
    }
  }
}
```

## Querying child documents by parent

### Matching child documents by parent criteria

```json
GET /department/_search
{
  "query": {
    "has_parent": {
      "parent_type": "department",
      //  "score": true,
      "query": {
        "term": {
          "name.keyword": "Development"
        }
      }
    }
  }
}
```


## Querying parent by child documents

### Finding parents with child documents matching a `bool` query

```json
GET /department/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      // "score_mode": "sum",
      // "min_children": 2,
      // "max_children": 5,
      "query": {
        "bool": {
          "must": [
            {
              "range": {
                "age": {
                  "gte": 50
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "gender.keyword": "M"
              }
            }
          ]
        }
      }
    }
  }
}
```
## Multi-level relations

### Creating the index with mapping

```json
PUT /company
{
  "mappings": {
    "properties": {
      "join_field": { 
        "type": "join",
        "relations": {
          "company": ["department", "supplier"],
          "department": "employee"
        }
      }
    }
  }
}
```
### Add company data docs data using `adding-company-docs.md`


### Example of querying multi-level relations

```json
// Get the Company Name that John_Doe works
GET /company/_search
{
  "query": {
    "has_child": {
      "type": "department",
      "query": {
        "has_child": {
          "type": "employee",
          "query": {
            "term": {
              "name.keyword": "John Doe"
            }
          }
        }
      }
    }
  }
}
```
### Parent/child inner hits

### Including inner hits for the `has_child` query

```json
GET /department/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      "inner_hits": {},
      "query": {
        "bool": {
          "must": [
            {
              "range": {
                "age": {
                  "gte": 50
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "gender.keyword": "M"
              }
            }
          ]
        }
      }
    }
  }
}
```
## Terms Lookup Mechamism

Add the data regarding this using `adding-user-storeis-docs.md` 

### Querying stories from a user's followers
We are querying fron index to another index.
```json
GET /stories/_search
{
  "query": {
    "terms": {
      "user": {
        "index": "users",
        "id": "1",
        "path": "following"
      }
    }
  }
}
```

## Join Limitations
- Documents must be stored within the same index..
- Parent and Child documents must be indexed on same Shard...( Done via routing)
- A document can have only one parent, but can have multiple children..
- Join fields are expensive and must be used when only neccesary..









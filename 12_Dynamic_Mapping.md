# Dynamic Mapping

If you dont Mention any mapping explicitly Dynamic Mapping is done by Elastic Search. 

In this case the string can be mapped as text + keyword or date or float or long depends on it number 

Example : 
```json
POST /my-index/_doc 
{
    "cpu" : {
        "name" : "INTEL CORE 7",
        "frequency" : 3.6,
        "cores" : 8
    }
}

GET my-index/_mapping
// Observe the mappings. 
```

## Combining Explicit and Dynamic Mappings

```json
// Create index with one field mapping
PUT /people
{
  "mappings": {
    "properties": {
      "first_name": {
        "type": "text"
      }
    }
  }
}

// Index a test document with an unmapped field
POST /people/_doc
{
  "first_name": "Bo",
  "last_name": "Andersen"
}

// Retrieve mapping
GET /people/_mapping

// Clean up
DELETE /people
```

## Configuring Dynamic Mappings
-  If We set `dynamic : false`  : 
    - Fields that are not in explicit mapping are **ignored** while indexing, but they are part of `_source`
    - No inverted index is created, Hence No results while searching them.
-  If We set `dynamic : strict`  : 
    - Fields that are not in explicit mapping are **rejected** while indexing and gives error.

```json
// Disable dynamic mapping
PUT /people
{
  "mappings": {
    "dynamic": false,
    "properties": {
      "first_name": {
        "type": "text"
      }
    }
  }
}

// Index a test document
POST /people/_doc
{
  "first_name": "Bo",
  "last_name": "Andersen"
}

// Retrieve mapping
GET /people/_mapping

//Observe the Mapping
```

```json
// Search `first_name` field
GET /people/_search
{
  "query": {
    "match": {
      "first_name": "Bo"
    }
  }
}

// Search `last_name` field
GET /people/_search
{
  "query": {
    "match": {
      "last_name": "Andersen"
    }
  }
}
// second search will not work

DELETE /people
```


## Inheritance for the `dynamic` parameter
The following example sets the `dynamic` parameter to `"strict"` at the root level, but overrides it with a value of 
`true` for the `specifications.other` field mapping.

#### Mapping
```json
PUT /computers
{
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "name": {
        "type": "text"
      },
      "specifications": {
        "properties": {
          "cpu": {
            "properties": {
              "name": {
                "type": "text"
              }
            }
          },
          "other": {
            "dynamic": true,
            "properties": { ... }
          }
        }
      }
    }
  }
}
```

#### Example document (invalid)
```json
POST /computers/_doc
{
  "name": "Gamer PC",
  "specifications": {
    "cpu": {
      "name": "Intel Core i7-9700K",
      "frequency": 3.6
    }
  }
}
```

#### Example document (OK)
```json
POST /computers/_doc
{
  "name": "Gamer PC",
  "specifications": {
    "cpu": {
      "name": "Intel Core i7-9700K"
    },
    "other": {
      "security": "Kensington"
    }
  }
}
```

### Enabling numeric detection
When enabling numeric detection, Elasticsearch will check the contents of strings to see if they contain only numeric 
values - and map the fields accordingly as either `float` or `long`.

#### Mapping
```json
PUT /computers
{
  "mappings": {
    "numeric_detection": true
  }
}
```

#### Example document
```json
POST /computers/_doc
{
  "specifications": {
    "other": {
      "max_ram_gb": "32", // long
      "bluetooth": "5.2" // float
    }
  }
}
```

### Date detection

#### Disabling date detection
```json
PUT /computers
{
  "mappings": {
    "date_detection": false
  }
}
```

#### Configuring dynamic date formats
```json
PUT /computers
{
  "mappings": {
    "dynamic_date_formats": ["dd-MM-yyyy"]
  }
}
```

### Clean up
```json
DELETE /people
```


# Mapping Recommendations

- Use explicit Mappings instead of Dynamic Mappings to save disk space while storing so Many docs
  - Set `dynamic:strict` to false
- Dont always map text fields as both `text` and `keywords`
  - Full Text - text
  - filtering aggregation - keyword
- Disable Coercion and try to do the right thing and be responsible to use correct datatypes
- Use `integer` instead of `long` for small numeric data types. 
- Set these Mapping Parameters to false to save disk space:
    - `doc_values : false` if you dont need sorting, aggregarion etc
    - `norms : false` if you dont need relavence scoring
    - `index : false` if you dont need to filter values


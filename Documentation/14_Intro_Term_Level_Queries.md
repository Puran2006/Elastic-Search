# Introduction to Searching
It took a quite a time to get to searchin, but believe me its worth the effort

Now we will start learning QUERY DSL( Domain Specific Language)
Instead of other lanuages like adding in query parameter in url. Here we will add in the request body in json format.

# Term level queries
- These queries are search for exact values( Same Case, Exact Match).
```json
GET /products/_search
{
  "query": {
    "term": {
      "name": "Vegetable"
    }
  }
}
// name = vegetable ( ❌)
// name = Vegetables ( ❌)
// name = Vegetable (✅)
```
- Term level queries are not analyzed, search value = Value used for index lookups
- Dont Use term level queries for text

## Searching terms
```json
GET /products/_search
{
  "query": {
    "term": {
      "tags.keyword": "Vegetable"
    }
  }
}
// The reason we use tags.keyword is tags has a type text and also mapped as keyword. 
        //   "tags": {
        //     "type": "text",
        //     "fields": {
        //       "keyword": {
        //         "type": "keyword",
        //         "ignore_above": 256
        //       }
        //     }
        //   }


// A more Explicit syntax 

GET /products/_search
{
  "query": {
    "term": {
      "tags.keyword": {
        "value": "Vegetable"
      }
    }
  }
}


// Case insensitive search

GET /products/_search
{
  "query": {
    "term": {
      "tags.keyword": {
        "value": "vegetable",
        "case_insensitive": true
      }
    }
  }
}

// Searching for multiple terms
GET /products/_search
{
  "query": {
    "terms": {
      "tags.keyword": ["Soup", "Meat"]
    }
  }
}

// Searching for booleans
GET /products/_search
{
  "query": {
    "term": {
      "is_active": true
    }
  }
}

// Searching for numbers
GET /products/_search
{
  "query": {
    "term": {
      "in_stock": 1
    }
  }
}

// Searching for dates
GET /products/_search
{
  "query": {
    "term": {
      "created": "2007/10/14"
    }
  }
}

// Searching for timestamps

GET /products/_search
{
  "query": {
    "term": {
      "created": "2007/10/14 12:34:56"
    }
  }
}
```

## Retrieve Documents by IDS

```json
GET /products/_search
{
  "query": {
    "ids": {
      "values": ["100", "200", "101"]
    }
  }
}
```

## Range Search Queries

```json
GET /products/_search
{
  "query": {
    "range": {
      "in_stock": {
        "gte": 5,
        "lte": 20
      }
    }
  }
}

// Querying dates

GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2020/01/01",
        "lte": "2020/01/31"
      }
    }
  }
}

//Specifying Data Format
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "format": "dd/MM/yyyy",
        "gte": "01/01/2020",
        "lte": "31/01/2020"
      }
    }
  }
}
```

## Prefixes, wildcards & regular expressions
- Some times if we search for Vegetable and the value is Vegetables we expect a result in term level queries too...

So Prefixes, wildcards etc will help here...

- Still Use these queries with keyword fields not text... 

### Prefix
```json
GET /products/_search
{
  "query": {
    "prefix": {
      "name.keyword": "Win"
    }
  }
}

// name : Wine - Maipo Valle Cabernet
```

### Wildcard
```json
GET /products/_search
{
  "query": {
    "wildcard": {
      "tags.keyword": "Past?" // Past*
    }
  }
}

// Avoid Placing wildcard beginning of value to optimize performance
```

### Regular Expressions
```json
GET /products/_search
{
  "query": {
    "regexp": {
      "tags.keyword":  "Bee(f|r)+"
    }
  }
}

// ^ and $ are not supported because they are not present in Apache Luecene
```


### Case insensitive searches

All of the above queries can be made case insensitive by adding the `case_insensitive` parameter, e.g.:

```json
GET /products/_search
{
  "query": {
    "prefix": {
      "name.keyword": {
        "value": "Past",
        "case_insensitive": true
      }
    }
  }
}
```

# Querying by field existence


**SQL:** `SELECT * FROM products WHERE tags IS NOT NULL`
```json       
GET /products/_search
{
  "query": {
    "exists": {
      "field": "tags.keyword"
    }
  }
}
```


**SQL:** `SELECT * FROM products WHERE tags IS NULL`

There is no dedicated query for this, so we do it with the `bool` query.

```json
GET /products/_search
{
  "query": {
    "bool": {
      "must_not": [
        {
          "exists": {
            "field": "tags.keyword"
          }
        }
      ]
    }
  }
}

// If Tags = [], NULL
// "" --> (empty string is ind
// exed, but the [] or empty value or NULL are not)
```

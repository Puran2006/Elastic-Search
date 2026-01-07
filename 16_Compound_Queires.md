# Compoud Queries
Compound Queires wrap other queries to produce a result..
- For example we need a product 
  -  `name` contain `Wine` ( match query)
  -  `in_stock` gte `5`  ( range query)
- For these type we use Compound query.. There are many ways...

For now we will start with Boolean Queries

# Boolean Queries: 
#### Query clauses added within the `must` occurrence type are required to match. 
**SQL** : `WHERE name LIKE "%Wine%" AND in_stock >=5 : `
```json
GET /products/_search
{
  "query": {
    "bool" : {
      
      "must": [
        {
          "range": {
            "in_stock": {
              "gte": 5,
              "lte": 10
            }
          }
        },
        {
          "match": {
            "name": "Wine"
          }
        }
      ]
    }
  }
}
```

#### Query clauses added within the `must_not` occurrence type are required to _not_ match.
**SQL:** `SELECT * FROM products WHERE tags IN ("Alcohol") AND tags NOT IN ("Wine")`

```json

GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "tags.keyword": "Alcohol"
          }
        }
      ],
      "must_not": [
        {
          "term": {
            "tags.keyword": "Wine"
          }
        }
      ],
      "should": [
        {
          "term": {
            "tags.keyword": "Beer"
          }
        },
        {
          "match": {
            "name": "beer"
          }
        },
        {
          "match": {
            "description": "beer"
          }
        }
      ]
    }
  }
}
```


Matching query clauses within the `should` occurrence type boost a matching document's relevance score.

```json
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "tags.keyword": "Alcohol"
          }
        }
      ],
      "must_not": [
        {
          "term": {
            "tags.keyword": "Wine"
          }
        }
      ],
      "should": [
        {
          "term": {
            "tags.keyword": "Beer"
          }
        },
        {
          "match": {
            "name": "beer"
          }
        },
        {
          "match": {
            "description": "beer"
          }
        },
       // "minimum_should_match": 1 // one of the should queries to match
      ]
    }
  }
}
```


Query clauses defined within the `filter` occurrence type must match. 
This is similar to the `must` occurrence type. The difference is that 
`filter` query clauses do not affect relevance scores and may be cached.

```json
GET /products/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "tags.keyword": "Alcohol"
          }
        }
      ]
    }
  }
}
```
NOTES
- If a query contains only `should` clause, atleat one query must match

- If a query clause have `must, must_not, filter` it is not necessary for `should` to be satified..

- Use `filter` for better performace if you dont need relevance_score in results
### Example
 
**SQL:** `SELECT * FROM products WHERE (tags IN ("Beer") OR name LIKE '%Beer%') AND in_stock <= 100`


```json
GET /products/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "range": {
            "in_stock": {
              "lte": 100
            }
          }
        }
      ],
      "should": [
        { "term": { "tags.keyword": "Beer" } },
        { "match": { "name": "Beer" } }
      ],
      "minimum_should_match": 1
    }
  }
}

```

# Boosting Query
- The `bool` query with `should` will help us increase the relavance score..
- What if we want to decrease or control the relavacne ? 
    - Here comes the `Boosting` Query

```json
GET /products/_search
{
  "size": 20,
  "query": {
    "match": {
      "name": "juice"
    }
  }
}
// results : Apple Juice, Cider Apple Juice, Orange Juice etc..
```

Now we want the juices with Apple to be at the bottom of the search results

```json
GET /products/_search
{
  "size": 20,
  "query": {
    "boosting": {
      "positive": {
        "match": {
          "name": "juice"
        }
      },
      "negative": {
        "match": {
          "name": "Apple"
        }
      },
      "negative_boost": 0.5
    }
  }
}
```

### Examples

```json
// "I don't like bacon"
GET /recipes/_search
{
  "query": {
    "boosting": {
      "positive": {
        "match_all": {}
      },
      "negative": {
        "term": {
          "ingredients.name.keyword": "Bacon"
        }
      },
      "negative_boost": 0.5
    }
  }
}

// Pasta products, preferably without bacon
GET /recipes/_search
{
  "query": {
    "boosting": {
      "positive": {
        "term": {
          "ingredients.name.keyword": "Pasta"
        }
      },
      "negative": {
        "term": {
          "ingredients.name.keyword": "Bacon"
        }
      },
      "negative_boost": 0.5
    }
  }
}

// "All products, I like pasta, but not bacon"
GET /recipes/_search
{
  "query": {
    "boosting": {
      "positive": {
        "bool": {
          "must": [
            { "match_all": {} }
          ],
          "should": [
            {
              "term": {
                "ingredients.name.keyword": "Pasta"
              }
            }
          ]
        }
      },
      "negative": {
        "term": {
          "ingredients.name.keyword": "Bacon"
        }
      },
      "negative_boost": 0.5
    }
  }
}
```

# Disjunction max (`dis_max`)

This is similar to `multi_match`
```
GET /products/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "name": "vegetable" } },
        { "match": { "tags": "vegetable" } }
      ],
      "tie_breaker": 0.3
    }
  }
}
```


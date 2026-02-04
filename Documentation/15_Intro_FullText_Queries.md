# Introduction to Full Text Queries

- Full Text Queries are used for searching Unstructured Data.
    - Ex : Website, emails, chats etc
- Full Text queries are analyzed.
    - Vegetable = vegetable = Vegetables ( depends on the analyzer we used.)
- Dont use on `keyword` fields

### `match` Query
```json
GET /products/_search
{
  "query": {
    "match": {
      "name": "wine mapio capio"
    //"operator" : "AND" //Default is "OR"
    }
  }
}
//  "name": "Wine - Maipo Valle Cabernet",
```
## Introducntion to Relavence Scoring
- If you have observed for the last search, ` "_score": 1.8359637`
- For Term Level Queries Mostly `_score = 1`, whether it matches or not.
- Search results are sorted based on the `_score` metafield so that related results are at the top.

### `multi-match` Query
```json
GET /products/_search
{
  "query": {
    "multi_match": {
      "query" : "vegetable broth",
      "fields" : ["name", "tags"]
    }
  }
}

// EQUIVALENT TO 
GET /products/_search
{
  "query": {
    "match": {
      "name" : "vegetable broth"
    }
  }
}
// sc
GET /products/_search
{
  "query": {
    "match": {
      "tags" : "vegetable broth"
    }
  }
}
```

- By default, one filed i used for calculating a documments relevance score
  - vegetable is in both name and tags with different `_score`, the one with the highest relavance score is considered..
- But with `tie_breaker`, Each matching field after highest score will affect the `_score`.

```json
GET /products/_search
{
  "query": {
    "multi_match": {
      "query": "vegetable broth",
      "fields": ["name", "description"],
      "tie_breaker": 0.3
    }
  }
}
// _score( "name" ) = 10
// _score ("descriptiion") = 5

// Total = 10 + 5*0.3 = 11.5
```

## Phrase Searches
Lets consider you search `"name" : "Zero Fanta"` and `"name" : "Fanta Zero"`. Here the order does not matter in the `match` query.

But in the Phrase query, the order matters - `match_phrase`
```json
GET /products/_search
{
  "query": {
    "match_phrase": {
      "name": "Mango Juice"
    }
  }
}
```
- The value is also analyzed here so, `tokens = ["mango" , "juice"]`
- But Order and Adjancency Matters here: 
  - `"Mango Almond Juice"` --> Wrong
  - `"Mango - Jucie"`      ---> Correct

```json
GET /products/_search
{
  "query": {
    "match_phrase": {
      "name": "juice mango"
    }
  }
}
GET /products/_search
{
  "query": {
    "match_phrase": {
      "name": "juice (mango)"
    }
  }
}

GET /products/_search
{
  "query": {
    "match_phrase": {
      "description": "browse the internet"
    }
  }
}
```


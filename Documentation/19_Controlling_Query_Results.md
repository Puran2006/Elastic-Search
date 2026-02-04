# Controlling QUery Results
- In Small Systems it may not matter, but the project of CDS Data Search Engine, where millions of rows are present it is useful I think.


## Source filtering
It is like selecting columns in a Relation Database.
### Excluding the `_source` field altogether

```json
GET /recipes/_search
{
  "_source": false,
  "query": {
    "match": { "title": "pasta" }
  }
}
```

### Only returning the `created` field

```json
GET /recipes/_search
{
  "_source": "created",
  "query": {
    "match": { "title": "pasta" }
  }
}
```

### Only returning an object's key

```json
GET /recipes/_search
{
  "_source": "ingredients.name",
  "query": {
    "match": { "title": "pasta" }
  }
}
```

### Returning all of an object's keys

```json
GET /recipes/_search
{
  "_source": "ingredients.*",
  "query": {
    "match": { "title": "pasta" }
  }
}
```

### Returning the `ingredients` object with all keys, __and__ the `servings` field

```json
GET /recipes/_search
{
  "_source": [ "ingredients.*", "servings" ],
  "query": {
    "match": { "title": "pasta" }
  }
}
```

### Including all of the `ingredients` object's keys, except the `name` key

```json
GET /recipes/_search
{
  "_source": {
    "includes": "ingredients.*",
    "excludes": "ingredients.name"
  },
  "query": {
    "match": { "title": "pasta" }
  }
}
```

## Specifying the result size and offset

### Using a query parameter

```json
GET /recipes/_search?size=2
{
  "_source": false,
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```

### Using a parameter within the request body

```json
GET /recipes/_search
{
  "_source": false,
  "size": 3,
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```

### Specifying an offset with the `from` parameter

```json
GET /recipes/_search
{
  "_source": false,
  "size": 3,
  "from": 3,
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```

## Sorting results

### Sorting by ascending order (implicitly)

```json
GET /recipes/_search
{
  "_source": false,
  "query": {
    "match_all": {}
  },
  "sort": [
    "preparation_time_minutes"
  ]
}
```

### Sorting by descending order

```json
GET /recipes/_search
{
  "_source": "created",
  "query": {
    "match_all": {}
  },
  "sort": [
    { "created": "desc" }
  ]
}
```

### Sorting by multiple fields

```json
GET /recipes/_search
{
  "_source": [ "preparation_time_minutes", "created" ],
  "query": {
    "match_all": {}
  },
  "sort": [
    { "preparation_time_minutes": "asc" },
    { "created": "desc" }
  ]
}
```


### Sorting by the average rating (descending)

```json
// This is like a basic aggregation
GET /recipes/_search
{
  "_source": "ratings",
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "ratings": {
        "order": "desc",
        "mode": "avg"
      }
    }
  ]
}
```


# Manging Mappings
## Adding explicit mappings

Add field mappings for `reviews` index
```json
PUT /reviews
{
  "mappings": {
    "properties": {
      "rating": { "type": "float" },
      "content": { "type": "text" },
      "product_id": { "type": "integer" },
      "author": {
        "properties": {
          "first_name": { "type": "text" },
          "last_name": { "type": "text" },
          "email": { "type": "keyword" }
        }
      }
    }
  }
}
// OR
// adding mapping to existing index 
PUT /reviews/_mappings
{
  "properties": {
    "rating": { "type": "float" },
    "content": { "type": "text" },
    "product_id": { "type": "integer" },
    "author": {
      "properties": {
        "first_name": { "type": "text" },
        "last_name": { "type": "text" },
        "email": { "type": "keyword" }
      }
    }
  }

} 
```

Index a test document
```json
PUT /reviews/_doc/1
{
  "rating": 5.0,
  "content": "Outstanding course! Bo really taught me a lot about Elasticsearch!",
  "product_id": 123,
  "author": {
    "first_name": "John",
    "last_name": "Doe",
    "email": "johndoe123@example.com"
  }
}
```

## Retrieving mappings
Retrieving mappings for the `reviews` index
```
GET /reviews/_mapping
```

Retrieving mapping for the `content` field
```
GET /reviews/_mapping/field/content
```

Retrieving mapping for the `author.email` field
```
GET /reviews/_mapping/field/author.email
```
### Using dot notation for the `author` object
```json
PUT /reviews_dot_notation
{
  "mappings": {
    "properties": {
      "rating": { "type": "float" },
      "content": { "type": "text" },
      "product_id": { "type": "integer" },
      "author.first_name": { "type": "text" },
      "author.last_name": { "type": "text" },
      "author.email": { "type": "keyword" }
    }
  }
}

GET /reviews_dot_notation/_mapping
```

## Adding mappings to existing indices

 Add new field mapping to existing index
```
PUT /reviews/_mapping
{
  "properties": {
    "created_at": {
      "type": "date"
    }
  }
}
```

Retrieve the mapping
```
GET /reviews/_mapping
```

## Updating existing mappings

- Generally, field mappings cannot be updated. Changind datatypes would take rebuilding the whole data structures

This query won't work.
```json
PUT /reviews/_mapping
{
  "properties": {
    "product_id": {
      "type": "keyword"
    }
  }
}
```

- We cannot also remove the field mappings
- Some mapping parameters can be changed
Exception : The `ignore_above` mapping parameter _can_ be updated, for instance.
```json
PUT /reviews/_mapping
{
  "properties": {
    "author": {
      "properties": {
        "email": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    }
  }
}
```
### The solution is to reindex documents into a new index ( This is the only and Optimal way)

# Reindexing Documents
### Add new index with new mapping
```json
PUT /reviews_new
{
  "mappings" : {
    "properties" : {
      "author" : {
        "properties" : {
          "email" : {
            "type" : "keyword",
            "ignore_above" : 256
          },
          "first_name" : {
            "type" : "text"
          },
          "last_name" : {
            "type" : "text"
          }
        }
      },
      "content" : {
        "type" : "text"
      },
      "created_at" : {
        "type" : "date"
      },
      "product_id" : {
        "type" : "keyword"
      },
      "rating" : {
        "type" : "float"
      }
    }
  }
}
// only the product_id is changed from integer to keyword
```
### Reindex documents from `reviews` to `reviews_new`
```json
POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  }
}
GET /reviews_new/_search
{
  "query": {
    "match_all": {}
  }
}
// "product_id": 123 ( is still a integer, _source remains same from the start of index)
```

### To solve this Delete all documents and Reindex with script
```json
POST /reviews_new/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}
// Convert `product_id` values to strings
POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  },
  "script": {
    "source": """
      if (ctx._source.product_id != null) {
        ctx._source.product_id = ctx._source.product_id.toString();
      }
    """
  }
}

// Retireive Documents again 
GET /reviews_new/_search
{
  "query": {
    "match_all": {}
  }
}

// "product_id": "123" ( Now its a string )
```

### Bonus Tips

```json
// Reindex only positive reviews
POST /_reindex
{
  "source": {
    "index": "reviews",
    "query": {
      "range": {
        "rating": {
          "gte": 4.0
        }
      }
    }
  },
  "dest": {
    "index": "reviews_new"
  }
}

// Removing fields (source filtering)
POST /_reindex
{
  "source": {
    "index": "reviews",
    "_source": ["content", "created_at", "rating"]
  },
  "dest": {
    "index": "reviews_new"
  }
}

 // Changing a field's name
POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  },
  "script": {
    "source": """
      # Rename "content" field to "comment"
      ctx._source.comment = ctx._source.remove("content");
    """
  }
}

// Ignore reviews with ratings below 4.0 with ctx.op ( not preffered)

POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  },
  "script": {
    "source": """
      if (ctx._source.rating < 4.0) {
        ctx.op = "noop"; # Can also be set to "delete"
      }
    """
  }
}
```

# Field Aliases
If there are many documents which needs to renindexed, its not worth using the REINDEX API.

Anothre Method is field aliases just like alias in another systems like linux. Lets see the example of renaming content to comment.

###  Add `comment` alias pointing to the `content` field
```json
PUT /reviews/_mapping
{
  "properties": {
    "comment": {
      "type": "alias",
      "path": "content"
    }
  }
}
// Using the field alias
GET /reviews/_search
{
  "query": {
    "match": {
      "comment": "outstanding"
    }
  }
}

// Using the "original" field name still works

GET /reviews/_search
{
  "query": {
    "match": {
      "content": "outstanding"
    }
  }
}
```

# Multi Field Mappings

### Add a new "fields" key and set the `keyword` mapping to a `text` field
```json
PUT /multi_field_test
{
  "mappings": {
    "properties": {
      "description": {
        "type": "text"
      },
      "ingredients": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      }
    }
  }
}

//Index a test document

POST /multi_field_test/_doc
{
  "description": "To make this spaghetti carbonara, you first need to...",
  "ingredients": ["Spaghetti", "Bacon", "Eggs"]
}


// Retrieve documents

GET /multi_field_test/_search
{
  "query": {
    "match_all": {}
  }
}

// Querying the `text` mapping ( match query )

GET /multi_field_test/_search
{
  "query": {
    "match": {
      "ingredients": "Spaghetti"
    }
  }
}
// Querying the `keyword` mapping ( term level)

GET /multi_field_test/_search
{
  "query": {
    "term": {
      "ingredients.keyword": "Spaghetti"
    }
  }
}


GET /multi_field_test/_analyze
{
  "field": "ingredients",
  "text": "Spaghetti Bacon Eggs"
}

GET /multi_field_test/_analyze
{
  "field": "ingredients.keyword",
  "text": "Spaghetti"
}



// Clean up
DELETE /multi_field_test
```
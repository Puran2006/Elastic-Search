# Creating and Managing Indexes
```
PUT /products
{
  "settings": {
    "number_of_shards": 2, 
    "number_of_replicas": 2
  }
}
```
### Adding Documents

```
POST /products/_doc
{
  "name" : "Coffe Mamker", 
  "price": 64,
  "in_stock": 10
}
```


```
PUT /products/_doc/100
{
  "name" : "Toaster", 
  "price": 49,
  "in_stock": 4
}
```

In this latest one id is 100 instead of random id

### Retrieving Documents
```
GET /products/_doc/100
```

### Updating Documents
```
POST /products/_update/100
{
    "doc" : {
        "in_stock" : 20
    }
}



POST /products/_update/100
{
    "doc" : {
        "tags" : ["electronics"]
    }
}


```


The Document is immutable, but we have changed how is it possible ? 

The POST(update) API : 
- Retrives current document.
- Field values are changed
- Existing document is replaced with modified document

But it just appeared as it was updated directly.



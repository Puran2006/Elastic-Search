# Creating Indiciesand Managing Documents
```json
PUT /products
{
  "settings": {
    "number_of_shards": 2, 
    "number_of_replicas": 2
  }
}
```
## Adding Documents

```json
POST /products/_doc
{
  "name" : "Coffe Mamker", 
  "price": 64,
  "in_stock": 10
}
```


```json
PUT /products/_doc/100
{
  "name" : "Toaster", 
  "price": 49,
  "in_stock": 4
}
```

In this latest one id is 100 instead of random id

## Retrieving Documents
```json
GET /products/_doc/100
```

## Updating Documents
```json
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

## Scripted Updates
We can update the value like increasing or decresing instock values directly by -1 or +1. It would be cool right..

Its like a custom code with logic and updating documents

```json
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock--"
  }
}
```
- ctx - context
- _source - containst the doc data
- in_stock - Field

Now if do a 
```json
GET /products/_doc/100
```
The value would have been updated

Now for another example, we can pass in parameters: 
```json
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock += params.quantity",
    "params": {
      "quantity" : 4
    }
  }
}
```
If else Example: 
```

POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.in_stock > 0) {
        ctx._source.in_stock--
      }
    """"
  }
}
```
The Scripting is useful when we are using it through APIs

## Upserts
Updating Document or Insert Document conditionally.

```json
POST /products/_update/101
{
  "script": {
    "source": "ctx._source.in_stock--"
  },
  "upsert": {
    "name" : "Blender", 
    "price": 399,
    "in_stock": 5
  }
}
// "result" : created ( 1st time)
// "result" : updated ( 2nd time)
```
Now this creates a new document with id 101
```
GET /products/_doc/101
```
Now if you rerun the update again, only script is exceuted not upsert, because it is already created last item.

## Relplacing Documents 
```json
PUT /products/_doc/100
{
  "name" : "Toaster",
  "price" : 40,
  "in_stock" : 10
}
// "result": "updated",
```
Now this removes the tag : ["electronics"]

## Deleting Documents

```json
DELETE /products/_doc/101
// "result": "deleted",
```
```json
GET /products/_doc/101
// "found": false
```



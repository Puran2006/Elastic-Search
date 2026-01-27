## Understanding Routing 
1) How does ElasticSearch know where to store documents ? 
2) How are documents found once they have been indexed ? 

Ans : Routing ( Process of resolving a shard for a document)
```
shard_num = hash(_routing) % num_primary_shards
```
The same applies while we ask to retrieve documents by id. 

For example if id = 100 in our case, we will get a shard number and elastic search will go and look for them.

Elastic Search routing is evenly distrbuted and open source, everyone change it according to them. lets not deep dive there...

Lets get our minds a quick intersting fact...

- Imagina a scenario:
  - We have two primary shards and we a index a document with id =100 into the index..
  - Based on routing formula, the document is stored in Shard B.
  - Now we add 3 more shards after sometime and Now we have 5 primary shards..
  - Now if we try to retrieve the GET/products/_doc/100. This might come empty.., even if the document is present..
  - Because, the routing formula might yield different results, due to change in primary_shards..

**Thats why index shards cannot be changed normally.**

We have to create a new index with more shards and reindex every document from old to new index.. This is done easily through split and shrink APIs.

## How Elastic Search Reads Data
```
Get /products/_doc/100
        |
Coordinating Node
        |
Routing find Shard or replica 
        |
Shard is selected from replication group ( Primary Shard + Replicas)--> AdaptiveReplicsSelection
        | 
Coordinating Node Reads and send data to the request
```

## How Elastic Search Writes data

```
PUT /products/_doc/100
        |
Coordinating Node
        |
Routing find Primary Shard
        |
Primary Shard is selected from Replication group for write
        | 
Validates Request, field values, structure
        |
Perform Writes Operation and Succeed
        |
Sends the update to all replica shards paralley

```
```json
GET /prooducts/_doc/100

// Ouput 
{
  "_index": "products",
  "_id": "100",
  "_version": 6,
  "_seq_no": 8,
  "_primary_term": 2,
  "found": true,
  "_source": {
    "name": "Toaster",
    "price": 40,
    "in_stock": 10
  }
}
```

- Primary_Term tells How many times primary Shard has been changed..
- Sequence_No tells How many times write operation is performed
- They are useful while recovery of primary shard failure.
- But not for very large data sets with millions of data and thousand s of write operations.
- For these elastic search uses concept of **checkpoints** 

## Versioning 
**_version** field, increments by modifying document everytime.

Types of versioning : 
- internal
- external ( useful when documents are stored outside like RDBMS)

```json
PUT /products/_doc/123?version=5?version_type=external
{
    "name" : "Knife",
    ...
    ...
}
```
Probably not that useful, not a best practice this way...

## Update by Query Multiple Documents

This includes using concepts of pri_term and seq_term.

```json
POST /products/_update_by_query
{
  "script": {
    "source": "ctx._source.in_stock--"
  },
  
  "query": {
    "match_all": {}
  }
}

GET /prodcuts/_search
{
  "query": {
    "match_all": {}
  }
}


```
Bulk request are used to update all the docs that are matched by query.
```json
{
  "took": 940,
  "timed_out": false,
  "total": 3,
  "updated": 3,
  "deleted": 0,
  "batches": 1, // No of batches the bulk operation performed
  "version_conflicts": 0,
  "noops": 0,
  "retries": { // no of retries while errors
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1,
  "throttled_until_millis": 0,
  "failures": [] // failures
}
```
In Failure, the query is aborted not rolled back, means some might be updated and some might not....

## Delete by Query Multiple Documents
```json
POST /products/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}
```
Delete all the documents..
Currently we havenot specified any particular pattern or sql inside query we will learn later.

## Batch Processing 
We can update many documents with a single query with bulk API.

Bulk API expects the NDJSON format
```
action_and_metadata\n
optional_source\n
action_and_metadata\n
optional_source\n
```
Create Index
```json
POST /_bulk
{ "index":{ "_index":"products", "_id" : 102 } }
{ "name" : "Espresso Machine", "price":199, "in_stock":9 }
{ "create": { "_index" : "products", "_id" : 103} }
{ "name" : "Milk Frother", "price":299, "in_stock":5 }
// index creates again even if document exists and create gives error if document exists
```
Update and Delete Index
```json
POST /_bulk
{ "update":{ "_index":"products", "_id" : 201 } }
{ "doc" : { "price" : 200 } }
{ "delete":{ "_index":"products", "_id" : 201 } }
```

If we want to perform all operations on same index
 Update and Delete Index
```json
POST /products/_bulk
{ "update":{ "_id" : 201 } }
{ "doc" : { "price" : 200 } }
{ "delete":{ "_id" : 201 } }
```

- Note : 
    1) From curl or program, content-type should be : **application/x-ndjson**
    2) line must end with \n or \r\n ( dont type ) and last line should be empty
    3)  A failed action will not affect other actions. Details are in response json


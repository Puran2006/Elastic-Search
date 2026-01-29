# Ingest Pipelines
Elasticsearch ingest pipelines let you perform common transformations on your data before indexing.

A pipeline consists of a series of configurable tasks called processors
```
Incoming Docs -->| lowercase dissect | --> Indexing
                 |--Ingest pipeline--|
```
Example of creating a ingest pipeline with two set `processors` followed by `lowercase` processor.
```json
PUT _ingest/pipeline/my-pipeline
{
  "description": "My optional pipeline description",
  "version" : 1,
  "processors": [
    {
      "set": {
        "description": "My optional processor description",
        "field": "instock.price",
        "value": 10
      }
    },
    {
      "set": {
        "description": "Set 'my-boolean-field' to true",
        "field": "instock.available",
        "value": true
      }
    },
    {
      "lowercase": {
        "field": "title"
      }
    }
  ]
}
```

You can test the pipeline in the Kibana or the Shell console using `simulateAPI`

```json
POST _ingest/pipeline/my-pipeline/_simulate
{
    "docs" : [
    {
        "_source" : {        
            "title" : "Pasta Sauce",
            "instock": {
                "price" : 20
            }
        }
    },
    {
        "_source" : {      
            "title" : "Vegetable Sauce",
            "instock": {
                "available" : false
            }        
        }
    },
   {
    "_source" :  {
        "title" : "Wine Sauce",
        "instock": {
            "price" : 30,
            "available" : true
            }        
        }
    },
    {
    "_source" : {
        "title" : "Soya Sauce"
        }
    }
    ]
}

// or 
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "processors": [
      {
        "lowercase": {
          "field": "text"
        }
      }
    ]
  },
    "docs" : [
    {
        "_source" : {        
            "title" : "Pasta Sauce",
            "instock": {
                "price" : 20
            }
        }
    },
    {
        "_source" : {      
            "title" : "Vegetable Sauce",
            "instock": {
                "available" : false
            }        
        }
    },
   {
    "_source" :  {
        "title" : "Wine Sauce",
        "instock": {
            "price" : 30,
            "available" : true
            }        
        }
    },
    {
    "_source" : {
        "title" : "Soya Sauce"
        }
    }
    ]
}
```

## Add a pipeline to an indexing request
```json
POST my_index/_doc?pipeline=my-pipeline
    {        
        "title" : "Soya Sauce"    
    }
```
You can also use the pipeline parameter with the update by query or reindex APIs.

```json
POST my_index/_update_by_query?pipeline=my-pipeline
POST _reindex
{
  "source": {
    "index": "my_index"
  },
  "dest": {
    "index": "my_new_index",
    "op_type": "create",
    "pipeline": "my-pipeline"
  }
}
```

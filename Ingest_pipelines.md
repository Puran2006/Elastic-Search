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

## Handling Failures in Pipeline
Now if we provide a parameter that is not in the pipeline lowercase processors like the below example we will get error.

```json
POST my_index/_doc?pipeline=my-pipeline
    {        
        "context" : "Soya Sauce"    
    }
```

To handle failures there are two options `ignore_failure` and `on_failure`

### ignore_failure
```json
PUT _ingest/pipeline/my-pipeline
{
  "processors": [
    {
      "lowercase": {
        "description": "Process to lowerase the incoming titles",
        "field": "title",
        "ignore_failure": true
      }
    }
  ]
}
// This time there wont be no error becaue it ignores
POST my_index/_doc?pipeline=my-pipeline
    {        
        "context" : "Tomato Sauce"    
    }
```

### on_failure
Use the on_failure parameter to specify a list of processors to run immediately after a processor failure. If on_failure is specified, Elasticsearch afterward runs the pipeline’s remaining processors, even if the on_failure configuration is empty.

```json
PUT _ingest/pipeline/my-pipeline
{
  "processors": [
    {
      "lowercase": {
        "description": "Process to lowerase the incoming titles",
        "field": "title",
        "on_failure": [
            {
                "set" : {
                    "field" : "title",
                    "value" : "on failure title"
                }
            }
        ]
      }
    }
  ]
}

// This time there wont be no error becaue it ignores
POST my_index/_doc?pipeline=my-pipeline
    {        
        "context" : "Tomato Sauce"    
    }

If you look at the source it also contains title + context
```

You can also specify `on_failure` for a `pipeline`. If a processor without an on_failure value fails, Elasticsearch uses this pipeline-level parameter as a fallback. Elasticsearch will not attempt to run the pipeline’s remaining processors.

```json
PUT _ingest/pipeline/my-pipeline
{
  "processors": [ ... ],
  "on_failure": [
    {
      "set": {
        "description": "Index document to 'failed-<index>'",
        "field": "_index",
        "value": "failed-{{{ _index }}}"
      }
    }
    {
      "set": {
        "description": "Record error information",
        "field": "error_information",
        "value": "Processor {{ _ingest.on_failure_processor_type }} with tag {{ _ingest.on_failure_processor_tag }} in pipeline {{ _ingest.on_failure_pipeline }} failed with message {{ _ingest.on_failure_message }}"
      }
    }
  ]
}
```

### Most Commonly used processors

Here’s a look at some frequently used ingest processors:

1. **Convert**: Changes the data type of a field.
2. **Rename**: Changes the name of a field.
3. **Set**: Assigns a specified value to a field.
4. **HTML Strip**: Strips HTML tags from a field's content.
5. **Lowercase**: Transforms the text in a field to lowercase.
6. **Uppercase**: Transforms the text in a field to uppercase.
7. **Trim**: Removes whitespace from the beginning and end of a field's value.
8. **Split**: Divides the field content into an array, using a comma `,` as the delimiter.
9. **Remove**: Deletes a field from the document.
10. **Append**: Adds a value to an array field.
    
```json
PUT _ingest/pipeline/my-pipeline
{
 "description": "Pipeline to demonstrate various ingest processors",
    "processors": [
        {
            "convert": {
                "field": "price",
                "type": "float",
                "ignore_missing": true
            }
        },
        {
            "rename": {
                "field": "old_name",
                "target_field": "new_name"
            }
        },
        {
            "set": {
                "field": "status",
                "value": "active"
            }
        },
        {
            "html_strip": {
                "field": "description"
            }
        },
        {
            "lowercase": {
                "field": "username"
            }
        },
        {
            "uppercase": {
                "field": "category"
            }
        },
        {
            "trim": {
                "field": "title"
            }
        },
        {
            "split": {
                "field": "tags",
                "separator": ","
            }
        },
        {
            "remove": {
                "field": "temporary_field"
            }
        },
        {
            "append": {
                "field": "tags",
                "value": ["new_tag"]
            }
        }
    ]
}

POST my_index/_doc?pipeline=my-pipeline
{
    "price": "100.50",
    "old_name": "old_value",
    "description": "<p>This is a description with HTML.</p>",
    "username": "UserNAME",
    "category": "books",
    "title": "   Example Title with Whitespace   ",
    "tags": "tag1,tag2,tag3",
    "temporary_field": "This field should be removed"
}

GET /my_index/_search
{
    "query": {
        "match_all": {}
    }
}
```

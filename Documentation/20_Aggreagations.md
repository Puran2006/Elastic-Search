# Introduction Aggregations

They are way of grouping and performing group operations on documents.

### Adding `orders` index with field mappings

```
PUT /orders
{
  "mappings": {
    "properties": {
      "purchased_at": {
        "type": "date"
      },
      "lines": {
        "type": "nested",
        "properties": {
          "product_id": {
            "type": "integer"
          },
          "amount": {
            "type": "double"
          },
          "quantity": {
            "type": "short"
          }
        }
      },
      "total_amount": {
        "type": "double"
      },
      "status": {
        "type": "keyword"
      },
      "sales_channel": {
        "type": "keyword"
      },
      "salesman": {
        "type": "object",
        "properties": {
          "id": {
            "type": "integer"
          },
          "name": {
            "type": "text"
          }
        }
      }
    }
  }
}
```

### Populating the `orders` index with test data
```bash
curl -k -H "Content-Type:application/x-ndjson" -X POST https://localhost:9200/orders/_bulk --data-binary "@orders-bulk.json"
```

# Metric Aggregations
Types: 
- Single Value Numeric Metric Aggregations
- Multi Value Numeric Metric Aggregations


### Calculating statistics with `sum`, `avg`, `min`, and `max` aggregations

```json
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "total_sales": {
      "sum": {
        "field": "total_amount"
      }
    },
    "avg_sale": {
      "avg": {
        "field": "total_amount"
      }
    },
    "min_sale": {
      "min": {
        "field": "total_amount"
      }
    },
    "max_sale": {
      "max": {
        "field": "total_amount"
      }
    }
  }
}
```


### Retrieving the number of distinct values using `cardinatlity`

```json
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "total_salesmen": {
      "cardinality": {
        "field": "salesman.id"
      }
    }
  }
}
```

### Retrieving the number of values or rows using `value_count`

```json
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "values_count": {
      "value_count": {
        "field": "total_amount"
      }
    }
  }
}
```

### Using `stats` aggregation for common statistics
stats give `sum`, `avg`, `min`, `max` and `count`
```json
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "amount_stats": {
      "stats": {
        "field": "total_amount"
      }
    }
  }
}
```

# Bucket Aggregations  
## Introduction to bucket aggregations

### Creating a bucket for each `status` value
Gives all the distinct values and how many times they have repeated
```json
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status"
      }
    }
  }
}
// "doc_count_error_upper_bound": 0,
//       "sum_other_doc_count": 0,
//       "buckets": [
//         {
//           "key": "processed",
//           "doc_count": 209 
//           } // processed is there in over 209 documents

```
But what if we dont have a status value for a document..

### Aggregating documents with missing field (or `NULL`)

```json
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status",
        "missing": "N/A"
      }
    }
  }
}
```


### Changing the minimum document count for a bucket to be created and show even there are not any N/A values

```json
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status",
        "size": 20,
        "missing": "N/A",
        "min_doc_count": 0
      }
    }
  }
}
```

### Ordering the buckets

```json
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status",
        "size": 20,
        "missing": "N/A",
        "min_doc_count": 0,
        "order": {
          "_key": "asc"
        }
      }
    }
  }
}
```

### Document counts in `term` query of bucket aggregations are approximate

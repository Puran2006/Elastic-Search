# Nested Aggregations
We can use Aggs inside of the aggs..

### Retrieving statistics for each status

```json
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status"
      },
      "aggs": {
        "status_stats": {
          "stats": {
            "field": "total_amount"
          }
        }
      }
    }
  }
}
```

### Narrowing down the aggregation context using some `query` command

```json
GET /orders/_search
{
  "size": 0,
  "query": {
    "range": {
      "total_amount": {
        "gte": 100
      }
    }
  },
  "aggs": { // run in the context of query
    "status_terms": {
      "terms": {
        "field": "status"
      },
      "aggs": {  // run in the context of parent aggs
        "status_stats": {
          "stats": {
            "field": "total_amount"
          }
        }
      }
    }
  }
}
```
# Filtering out documents

### Filtering out documents with low `total_amount`

```
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "low_value": {
      "filter": {
        "range": {
          "total_amount": {
            "lt": 50
          }
        }
      }
    }
  }
}
```

### Aggregating on the bucket of remaining documents

```
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "low_value": {
      "filter": {
        "range": {
          "total_amount": {
            "lt": 50
          }
        }
      },
      "aggs": {
        "avg_amount": {
          "avg": {
            "field": "total_amount"
          }
        }
      }
    }
  }
}
```

## Defining bucket rules with filters

### Placing documents into buckets based on criteria

```json
GET /recipes/_search
{
  "size": 0,
  "aggs": {
    "my_filter": {
      "filters": { 
        "filters": { // filter is again repeated for no use, but it is what it is
          "pasta": {
            "match": {
              "title": "pasta"
            }
          },
          "spaghetti": {
            "match": {
              "title": "spaghetti"
            }
          }
        }
      }
    }
  }
}
```

# Range aggregations

### `range` aggregation

```json
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "amount_distribution": {
      "range": {
        "field": "total_amount",
        "ranges": [
          {
            "to": 50
          },
          {
            "from": 50,
            "to": 100
          },
          {
            "from": 100
          }
        ]
      }
    }
  }
}
```

### `date_range`

```json
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "purchased_ranges": {
      "date_range": {
        "field": "purchased_at",
        "format": "yyyy-MM-dd",
        // keyed : true
        "ranges": [
          {
            "from": "2016-01-01",
            "to": "2016-01-01||+6M",
            //  "key": "first_half"
          },
          {
            "from": "2016-01-01||+6M",
            "to": "2016-01-01||+1y",
            // "key": "first_half"
          }
        ]
      },
      "aggs": {
        "bucket_stats": {
          "stats": {
            "field": "total_amount"
          }
        }
      }
    }
  }
}
```

# HistoGram Aggregation
In range aggregations we mentioned from and to for `range` parameter. 

But what if we want all the data to be covered with a gap of certain number like 0-25, 25-50, 50-75,...

### Distribution of `total_amount` with interval `25`
```json
GET orders/_search
{
  "size": 0,
  "aggs": {
    "distribution_amount": {
      "histogram": {
        "field": "total_amount",
        "interval": 25,
        "min_doc_count": 1 // only include if atleast one doc is in interval
      }
    }
  }
}
```

### Specifying fixed bucket boundaries

```json
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "amount_distribution": {
      "histogram": {
        "field": "total_amount",
        "interval": 25,
        "min_doc_count": 0,
        "extended_bounds": {
          "min": 0,
          "max": 200
        }
      }
    }
  }
}
```

### Aggregating by month with the `date_histogram` aggregation

```json
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "orders_over_time": {
      "date_histogram": {
        "format": "yyyy-MM-dd", 
        "field": "purchased_at",
        "calendar_interval": "month"
      }
    }
  }
}
```

# `global` aggregation

### Break out of the aggregation context
Here the all_orders is not in context of the `query gte > 100` and the stats_expensive is depended on the context of the query.
```json

GET /orders/_search
{
  "query": {
    "range": {
      "total_amount": {
        "gte": 100
      }
    }
  },
  "size": 0,
  "aggs": {
    "all_orders": {
      "global": { },
      "aggs": {
        "stats_amount": {
          "stats": {
            "field": "total_amount"
          }
        }
      }
    },
    "stats_expensive": {
      "stats": {
        "field": "total_amount"
      }
    }
  }
}
```

# Missing Field Values:

### Adding test documents

```json
PUT /orders/_doc/1001
{
  "total_amount": 100
}

PUT /orders/_doc/1002
{
  "total_amount": 200,
  "status": null
}
```

### Aggregating documents with `missing` field value

```json
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "orders_without_status": {
      "missing": {
        "field": "status"
      },
      "aggs": {
        "missing_sum": {
          "sum": {
            "field": "total_amount"
          }
        }
      }
    }
  }
}
```

```json
DELETE /orders/_doc/1001
DELETE /orders/_doc/1002
```


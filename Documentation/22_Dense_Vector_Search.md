# Dense vector field type
- The dense_vector field type stores dense vectors of numeric values.
-  Dense vector fields are primarily used for k-nearest neighbor (kNN) search.
- The dense_vector type does not support aggregations or sorting.
- You add a dense_vector field as an array of numeric values based on element_type with float by default:

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "my_vector": {
        "type": "dense_vector",
        "dims": 3
      },
      "my_text" : {
        "type" : "keyword"
      }
    }
  }
}
PUT my-index/_doc/1
{
  "my_text" : "text1",
  "my_vector" : [0.5, 10, 6]
}
PUT my-index/_doc/2
{
  "my_text" : "text2",
  "my_vector" : [-0.5, 10, 10]
}


// This is not the same
PUT my_index/_doc
{
    "my_vector" : [5, 5,4 , 3],
    "text" : 4
}

GET my_index/_mapping

```
`Unlike most other data types, dense vectors are always single-valued. It is not possible to store multiple values in one dense_vector field.`

A k-nearest neighbor (kNN) search finds the k nearest vectors to a query vector, as measured by a similarity metric.
- dense vector fields are always indexed as int8_hnsw.

When indexing is enabled, you can define the vector similarity to use in kNN search:
- Default value is cosine similarity
```json
PUT my-index-2
{
  "mappings": {
    "properties": {
      "my_vector": {
        "type": "dense_vector",
        "dims": 3,
        "similarity": "dot_product"
      }
    }
  }
}
```
Note
- Indexing vectors for approximate kNN search is an expensive process.
- It can take substantial time to ingest documents that contain vector fields with index enabled.

You can disable indexing by setting the index parameter to false:
```json

PUT my-index-2
{
  "mappings": {
    "properties": {
      "my_vector": {
        "type": "dense_vector",
        "dims": 3,
        "index": false
      }
    }
  }
}

```
### Accessing dense_vector fields in search responses

By default, dense_vector fields are not included in _source in responses from the _search. This helps reduce response size and improve performance, especially in scenarios where vectors are used solely for similarity scoring and not required in the output.

To retrieve vector values explicitly, you can use:

```json
// Try this You should not get the "my_vector" in the _source
GET my_index/_search
{   
    "query": {
        "match_all": {}
    }
}

// The fields option to request specific vector fields directly:
POST my-index-2/_search
{
  "fields": ["my_vector"]
}

// The _source.exclude_vectors flag to re-enable vector inclusion in _source responses:
POST my-index-2/_search
{
  "_source": {
    "exclude_vectors": false
  }
}
```


### Automatically quantize vectors for kNN search

The dense_vector type supports quantization to reduce the memory footprint required when searching float vectors. The three following quantization strategies are supported:

- int8 - Quantizes each dimension of the vector to 1-byte integers. This reduces the memory footprint by 75% (or 4x) at the cost of some accuracy.
- int4 - Quantizes each dimension of the vector to half-byte integers. This reduces the memory footprint by 87% (or 8x) at the cost of accuracy.
- bbq - Better binary quantization which reduces each dimension to a single bit precision. This reduces the memory footprint by 96% (or 32x) at a larger cost of accuracy. Generally, oversampling during query time and reranking can help mitigate the accuracy loss.

Here is an example of how to create a binary quantized index:
```json
PUT my-byte-quantized-index
{
  "mappings": {
    "properties": {
      "my_vector": {
        "type": "dense_vector",
        "dims": 64,
        "index": true,
        "index_options": {
          "type": "bbq_hnsw"
        }
      }
    }
  }
}
```
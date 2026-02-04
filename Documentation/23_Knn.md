# KNN 
In the full-text search `query` option passed to the search() method of the Elasticsearch client. When searching vectors, the `knn` option is used instead.

```python
results = es.search(
        knn={
            'field': 'embedding',
            'query_vector': es.get_embedding(parsed_query),
            'num_candidates': 50,
            'k': 10,
        },
        size=5,
        from_=from_
    )
```
# Hybrid search
Elasticsearch integrates the RRF algorithm into the search query. Consider the following example, which has query and knn sections to request full-text and vector searches respectively, and a rrf section that combines them into a single result list.
```python
es.search(
    query={
        # full-text search query here
    },
    knn={
        # vector search query here
    },
    rank={
        "rrf": {}
    }
)
```

## Python code for basic KNN search
```python
from pprint import pprint
from elasticsearch import Elasticsearch

es = Elasticsearch('http://localhost:9200')
client_info = es.info()
print('Connected to Elasticsearch!')
pprint(client_info.body)
es.indices.delete(index='my_index', ignore_unavailable=True)
es.indices.create(index='my_index', mappings={
            'properties': {
                'embedding': {
                    'type': 'dense_vector',
                }
            }
        })
from sentence_transformers import SentenceTransformer
model = SentenceTransformer('all-MiniLM-L6-v2')

import json
with open('../data/astronomy.json', 'r') as f:
    documents = json.load(f)

documents[0]

operations = []
for doc in documents:
    operations.append(
        {'index': {'_index': 'my_index'}}
        )
    operations.append(
        {
            **doc,
            'embedding': model.encode(doc['content'])
        }
    )

response = es.bulk(operations=operations)
pprint(response.body)
response = es.search(
    index='my_index',
    body={
        'query':
            {
                'match_all': {}
            }
    }
)

pprint(response["hits"]["total"])
# Since the dense_vector are not returned by default, we need to explicitly request them
response = es.search(
    index='my_index',
    body={
        "fields" : ["embedding"]
    }
)

response.body['hits']['hits'][0]['fields']
response = es.indices.get_mapping(index='my_index')
pprint(response.body)
query = "What is a black hole?"
query_vector = model.encode(query)
results = es.search(
    index='my_index',
    body={
        "size": 3,
        "knn" : {
            "field" : "embedding",
            "query_vector" : query_vector,
            "k" : 3, # Top K similar vectors
            "num_candidates": 5 # Number of candidates to consider for KNN search
        }
    }
)

results.body['hits']['hits']
query = "How do we find exoplanets?"
query_vector = model.encode(query)
result = es.search(
    index='my_index',
    knn={
        "field": "embedding",
        "query_vector": query_vector,
        "num_candidates": 5,
        "k": 1,
    }
)

result.body['hits']['hits']
```
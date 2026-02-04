# Importing data with cURL
```bash
sudo apt install curl
```
## Navigating to bulk file directory

```bash
cd /search-learning
touch products-bulk.json
# copy some index and data fro products-bulk.json into wsl
```

## Importing data into local clusters

```bash
# Without CA certificate validation. This is fine for development clusters, but don't do this in production!
curl -k -H "Content-Type:application/x-ndjson" -XPOST http://localhost:9200/products/_bulk --data-binary "@products-bulk.json"

# With CA certificate validation. The certificate is located at $ES_HOME/config/certs/http_ca.crt
curl --cacert /path/to/http_ca.crt -H "Content-Type:application/x-ndjson" -XPOST https://localhost:9200/products/_bulk --data-binary "@products-bulk.json"
```

```json
{
    "errors": false,
    "took": 1794,
    "items": [
        {
            "index": {
                "_index": "products",
                "_id": "1",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 3,
                    "successful": 3,
                    "failed": 0
                },
                "_seq_no": 14,
                "_primary_term": 2,
                "status": 201
            }
        },
        {
            "index": {
                "_index": "products",
                "_id": "2",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 3,
                    "successful": 3,
                    "failed": 0
                },
                "_seq_no": 15,
                "_primary_term": 2,
                "status": 201
            }
        },
        {
            "index": {
                "_index": "products",
                "_id": "3",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 3,
                    "successful": 3,
                    "failed": 0
                },
                "_seq_no": 16,
                "_primary_term": 2,
                "status": 201
            }
        },
        {
            "index": {
                "_index": "products",
                "_id": "4",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 3,
                    "successful": 3,
                    "failed": 0
                },
                "_seq_no": 3,
                "_primary_term": 2,
                "status": 201
            }
        },
        {
            "index": {
                "_index": "products",
                "_id": "5",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 3,
                    "successful": 3,
                    "failed": 0
                },
                "_seq_no": 17,
                "_primary_term": 2,
                "status": 201
            }
        },
        {
            "index": {
                "_index": "products",
                "_id": "6",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 3,
                    "successful": 3,
                    "failed": 0
                },
                "_seq_no": 18,
                "_primary_term": 2,
                "status": 201
            }
        },
        {
            "index": {
                "_index": "products",
                "_id": "7",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 3,
                    "successful": 3,
                    "failed": 0
                },
                "_seq_no": 19,
                "_primary_term": 2,
                "status": 201
            }
        },
        {
            "index": {
                "_index": "products",
                "_id": "8",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 3,
                    "successful": 3,
                    "failed": 0
                },
                "_seq_no": 4,
                "_primary_term": 2,
                "status": 201
            }
        },
        {
            "index": {
                "_index": "products",
                "_id": "9",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 3,
                    "successful": 3,
                    "failed": 0
                },
                "_seq_no": 20,
                "_primary_term": 2,
                "status": 201
            }
        },
        {
            "index": {
                "_index": "products",
                "_id": "10",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 3,
                    "successful": 3,
                    "failed": 0
                },
                "_seq_no": 21,
                "_primary_term": 2,
                "status": 201
            }
        },
        {
            "index": {
                "_index": "products",
                "_id": "11",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 3,
                    "successful": 3,
                    "failed": 0
                },
                "_seq_no": 22,
                "_primary_term": 2,
                "status": 201
            }
        },
        {
            "index": {
                "_index": "products",
                "_id": "12",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 3,
                    "successful": 3,
                    "failed": 0
                },
                "_seq_no": 5,
                "_primary_term": 2,
                "status": 201
            }
        }
    ]
}
```

Now you can go into kibana console and check the shards, 
```
GET /_cat/shards?v
```
```
index                                                          shard prirep state   docs   store dataset ip         node
products                                                       0     r      STARTED   13  25.6kb  25.6kb 172.18.0.4 es01
products                                                       0     p      STARTED   13  25.5kb  25.5kb 172.18.0.2 es03
products                                                       0     r      STARTED   13  25.5kb  25.5kb 172.18.0.3 es02
products                                                       1     p      STARTED    3  10.3kb  10.3kb 172.18.0.4 es01
products                                                       1     r      STARTED    3  10.3kb  10.3kb 172.18.0.2 es03
products                                                       1     r      STARTED    3  10.3kb  10.3kb 172.18.0.3 es02
```
# Working with multiple Nodes 

Until Now we have worked with a single node, Now to see how the shards and replicas
 are stored and distribute we need atleast 3 nodes.
```yml
services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.1
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=elastic-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - xpack.security.enabled=false
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    ports:
      - 9200:9200
    networks:
      - elastic

  es02:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.1
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=elastic-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - xpack.security.enabled=false
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    networks:
      - elastic

  es03:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.1
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=elastic-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - xpack.security.enabled=false
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    networks:
      - elastic

networks:
  elastic:
    driver: bridge

```

```bash
docker compose up -d
```

```bash
curl http://localhost:9200/_cat/nodes?v
```

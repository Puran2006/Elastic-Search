
# Elastic Search Concepts

- **In MySQL** : you have a database → table → rows → columns

- **In Elasticsearch** : you have a cluster → index → documents → fields



# INDEX

- An index is like a database table in SQL.

- It stores documents (JSON objects) and defines their fields via mapping.

- Elasticsearch uses indexes to organize and search data efficiently.

Analogy : 
- Imagine a library section dedicated to "Books".

    - The section Books = index

    - Each book = document

    - Book properties like title, author, year = fields

Example : 
```json 
# Index: parts 
{
  "part_name": "Brake Pad",
  "brand": "Bosch",
  "price": 1200,
  "in_stock": true
}
{
  "part_name": "Brake Pad",
  "brand": "Bosch",
  "price": 1200,
  "in_stock": true
}

```

A document is a **single JSON object** stored in an index.
 Equivalent to a row in SQL.


A field is a property of a document, like a column in SQL.

Each field has a type: text, keyword, boolean etc


# Shards

Elasticsearch automatically splits an index into shards. A shard is a smaller, self-contained index.
- Shards allow Elasticsearch to:

- Scale horizontally (distribute data across multiple nodes)

- Search in parallel → faster queries

A big library section (index) is split into shelves (shards).

# Replica
A replica is a copy of a shard.

Used for:

- High availability → if a node fails, replica shard keeps data safe

- Performance → searches can be distributed across replicas

Each primary shard can have N replicas (default 1)

- Replicas are read-only (writes go to primary shards)


Order : 
```scss
Cluster
 └── Node(s)
     └── Index
         └── Shard(s)
             └── Segment(s)
                 └── Document(s)
                     └── Field(s)
```
- Cluster is the entire Elasticsearch system. One or more nodes working together
- Node is One running Elasticsearch instance . Usually one Docker container / VM / server
- Index is a logical collection of documents. Similar to a table in MySQL
- Shard is a physical partition of an index. Allows data to be split across nodes
- Segment is A Lucene index. Luecene Uses Inverted Index. 
  - Instead of: ```Document → Words```
  - Lucene stores: ```Word → List of Documents```
  - This is what makes search fast.
- Document is One JSON object. Equivalent to a row in MySQL 
- Field is a key-value pair inside a document
- Mappings Defines field types

Elasticsearch uses an in-memory indexing buffer to temporarily hold newly indexed documents before they are made searchable. When a refresh occurs, the contents of this buffer are written into a new Lucene segment, which is first placed in the filesystem cache (managed by the operating system and backed by RAM), not immediately on disk. Because files in the filesystem cache can be read as if they were on disk, Elasticsearch can open these segments and make the documents searchable very quickly. This design avoids expensive disk I/O on every write, allows frequent refreshes without performance loss, and is the key reason Elasticsearch provides near real-time search rather than instant real-time visibility.

```bash
PUT /pages

GET _cat/indices?v

GET _cat/shards?v   

# observe the primary and th replica shards where primary is # started and replica is unassigned, thats  why our index is in yellow state

```
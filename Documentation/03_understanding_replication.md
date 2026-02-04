
# Elasticsearch Node Roles & Shard Rebalancing
One node = one container

Replicas are placed on different nodes

You need multiple nodes for replicas to work

3 nodes is the minimum realistic cluster

```bash
PUT /pages

GET _cat/indices?v

GET _cat/shards?v   

# observe the primary and the replica shards where primary is started and replica is unassigned, thats  why our index is in yellow state

```

## Master-Eligible Nodes

Master-eligible nodes are responsible for cluster coordination.

### Responsibilities
- Elect the master node
- Maintain cluster state
- Manage index metadata
- Control shard allocation and rebalancing
- Handle node join and leave events
---

## Data Nodes

Data nodes store and process actual Elasticsearch data.

### Responsibilities
- Store primary and replica shards
- Handle indexing operations
- Execute search queries
- Perform aggregations
---

## Shard Rebalancing

Shard rebalancing is Elasticsearch’s automatic process of redistributing shards across nodes.

### When Rebalancing Happens
- A node joins the cluster
- A node leaves or fails
- A new index is created
- Shard or replica settings change

### How Rebalancing Works
1. Master node detects a cluster change
2. Master calculates optimal shard placement
3. Shards are moved between nodes
4. Cluster remains available during movement
5. Cluster health returns to green when balanced

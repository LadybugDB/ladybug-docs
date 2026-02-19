---
title: CREATE GRAPH
description: Create subgraph statements
---

## Create a subgraph

Subgraphs provide isolation and configuration separate from the rest of the system, similar to
`CREATE DATABASE` and `USE DATABASE` in relational databases. Each subgraph has its own schema, data, and
configuration.

### Strictly typed subgraphs (default)

By default, all subgraphs are strictly typed, meaning you must create node and relationship tables
before inserting data.

```cypher
CREATE GRAPH my_graph;
USE my_graph;
CREATE NODE TABLE User(name STRING PRIMARY KEY);
CREATE (u:User {name: 'Alice'});
```

### Open type graphs

Open type graphs allow you to create nodes with labels without first defining the node table.
This provides compatibility for users migrating from GQL or Neo4j.

```cypher
CREATE GRAPH my_graph ANY;
USE my_graph;
CREATE (u:User {name: 'Alice'});
```

In open type graphs, you can still use `CREATE NODE TABLE` if you want to define a strict schema.

## Switch between subgraphs

Use `USE` to switch between subgraphs:

```cypher
USE my_graph;
```

To list all available subgraphs:

```cypher
CALL SHOW_GRAPHS() RETURN *;
```

## Drop a subgraph

To drop a subgraph and all its contents:

```cypher
DROP GRAPH my_graph;
```

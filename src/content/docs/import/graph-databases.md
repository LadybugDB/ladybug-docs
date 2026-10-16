---
title: "Graph database interoperability"
description: "Import data from other graph databases into Ladybug"
---

Ladybug supports interoperability with various graph databases, allowing you to migrate data from external graph systems into Ladybug for querying and analysis.

## Neo4j

Ladybug provides a Neo4j extension that enables direct migration of data from Neo4j databases. This includes node labels, relationship types, and properties.

For detailed instructions on how to migrate data from Neo4j, see the [Neo4j extension](/extensions/neo4j) documentation.

### Quick example

```cypher
CALL NEO4J_MIGRATE(
    "http://localhost:7474",
    "neo4j",
    "password",
    ["User", "Product"],
    ["FOLLOWS", "PURCHASED"]
);
```

This imports:
- Nodes with labels `User` and `Product` as node tables
- Relationships with types `FOLLOWS` and `PURCHASED` as relationship tables

## GraphAr

[Apache GraphAr](https://graphar.apache.org/) is an open source, standard data file format for graph data storage and retrieval. It is designed for efficient storage and out-of-core querying of large-scale graph data, using chunking, columnar storage, and CSR/CSC semantics.

To use GraphAr data in Ladybug, convert it to Icebug format first:

```bash
uvx icebug-format --graphar <path to graphar archive>
```

This generates a directory of Parquet files plus a Cypher schema file that can be loaded directly with `lbug -i`. See the [Icebug format](/import/icebug) section for details.

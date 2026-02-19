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

Ladybug can read GraphAr archives directly. GraphAr data can be generated from various sources including Neo4j, PySpark, and other graph systems that support GraphAr export.

### Loading GraphAr data

Use the `READ_PARQUET` function to read GraphAr archives:

```cypher
-- Read GraphAr vertex files
LOAD FROM PARQUET('/path/to/graphar/vertices/*.parquet');

-- Read GraphAr edge files
LOAD FROM PARQUET('/path/to/graphar/edges/*.parquet');
```

## Icebug format

[Icebug](https://github.com/Ladybug-Memory/icebug-format) is a Ladybug-native graph-aware Parquet format designed for ingestion-free graph analytics. Unlike general-purpose Parquet files, Icebug preserves graph structure (node and relationship tables) and enables direct querying without preprocessing.

### Generating Icebug files

Use the `icebug-format` tool to generate Icebug files from existing databases:

```bash
# From a DuckDB database
uvx icebug-format --source-db demo-db.duckdb --schema schema.cypher

# From a GraphAr archive
uvx icebug-format --graphar <path to archive>
```

This generates a directory of Parquet files (for nodes and relationships) plus a Cypher schema file.

### Using Icebug files

Start Ladybug with the generated schema file using the `-i` flag:

```bash
lbug -i csr_graph/schema.cypher
```

Then query the graph directly:

```cypher
MATCH (a:User)-[b:LivesIn]->(c:City)
RETURN a.*, b.*, c.*;
```

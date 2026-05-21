---
title: "Icebug format"
description: Read data from Icebug format
---

Icebug format comes in two flavours: icebug-disk and icebug-memory

## icebug-disk

[icebug-disk](https://github.com/Ladybug-Memory/icebug-format) is a Ladybug-native graph-aware Parquet format designed for ingestion-free graph analytics. Unlike general-purpose Parquet files, Icebug preserves graph structure (node and relationship tables) and enables direct querying without preprocessing.

### Generating Icebug files

Use the `icebug-format` tool to generate icebug-disk files from existing databases:

```bash
# From a DuckDB database
uvx icebug-format --source-db demo-db.duckdb --schema schema.cypher

# From a GraphAr archive
uvx icebug-format --graphar <path to archive>
```

This generates a directory of Parquet files (for nodes and relationships) plus a Cypher schema file.

schema.cypher example:

```cypher
CREATE NODE TABLE city(id INT32, name STRING, population INT64, PRIMARY KEY(id)) WITH (storage = '<path-to-dir>', format = 'icebug-disk');
CREATE NODE TABLE user(id INT32, name STRING, age INT64, PRIMARY KEY(id)) WITH (storage = '<path-to-dir>', format = 'icebug-disk');
CREATE REL TABLE follows(FROM user TO user, since INT32) WITH (storage = '<path-to-dir>', format = 'icebug-disk');
CREATE REL TABLE livesin(FROM user TO city) WITH (storage = '<path-to-dir>', format = 'icebug-disk');
```

### Using Icebug files

Start Ladybug with the generated schema file using the `-i` flag:

```bash
lbug -i csr_graph/schema.cypher
```

or Run the DDL queries yourself in a Ladybug instance. We also support parquet files on remote storage (e.g. S3)

Then query the graph directly:

```cypher
MATCH (a:User)-[b:LivesIn]->(c:City)
RETURN a.*, b.*, c.*;
```
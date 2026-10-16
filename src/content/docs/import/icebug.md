---
title: "Icebug format"
description: Read data from Icebug format
---

[Icebug format](https://github.com/Ladybug-Memory/icebug-format) comes in two flavours: icebug-disk and icebug-memory. Both store graph data in CSR format. You can bring your own node and rel tables(with `from` and `to` columns) and generate icebug files with the [icebug-format](https://pypi.org/project/icebug-format/) tool

## icebug-disk

[icebug-disk](https://github.com/Ladybug-Memory/icebug-format) is a Ladybug-native graph-aware Parquet format designed for ingestion-free graph analytics. Unlike general-purpose Parquet files, Icebug preserves graph structure (node and relationship tables) and enables direct querying without preprocessing.

### Generating Icebug files

Use the `icebug-format` tool to generate icebug-disk files from existing databases:

```bash
# From a DuckDB database
uvx icebug-format --source-db demo-db.duckdb --schema input_schema.cypher

# From a GraphAr archive
uvx icebug-format --graphar <path to archive>
```

This generates a directory of Parquet files (for nodes and relationships) plus a Cypher schema file.

output schema.cypher:

```cypher
CREATE NODE TABLE city(id INT32, name STRING, population INT64, PRIMARY KEY(id)) WITH (storage = '<path-to-dir>', format = 'icebug-disk');
CREATE NODE TABLE user(id INT32, name STRING, age INT64, PRIMARY KEY(id)) WITH (storage = '<path-to-dir>', format = 'icebug-disk');
CREATE REL TABLE follows(FROM user TO user, since INT32) WITH (storage = '<path-to-dir>', format = 'icebug-disk');
CREATE REL TABLE livesin(FROM user TO city) WITH (storage = '<path-to-dir>', format = 'icebug-disk');
```

:::note[Note]
For node tables, you can directly pass the parquet file path as the storage location
:::

### Using Icebug files

Start Ladybug with the generated schema file using the `-i` flag:

```bash
lbug -i csr_graph/schema.cypher
```

or Run the DDL queries yourself in a Ladybug instance.

Then query the graph directly:

```cypher
MATCH (a:User)-[b:LivesIn]->(c:City)
RETURN a.*, b.*, c.*;
```

If the ladybug instance is created backed by a file `graph.lbdb`, you can move it around or export it, along with the data, and query it without needing to re-create the graph. You can also attach the same db to another instance of ladybug using

```cypher
ATTACH 'graph.lbdb' AS mygraph (dbtype lbug);
```

For more details about attaching databases, see the [attach documentation](/extensions/attach/lbug).

### Remote storage

Icebug-disk supports Parquet files on remote storage. The `storage` path in your schema.cypher can be any URI supported by Ladybug's file system extensions.

| Storage type | Example URI | Extension required |
|---|---|---|
| Amazon S3 | `s3://my-bucket/graphs/mygraph/` | `httpfs` |
| Google Cloud Storage | `gcs://my-bucket/graphs/mygraph/` | `httpfs` |
| Azure Blob Storage | `az://my-container/mygraph/` | `azure` |
| Huggingface Hub | `xet://huggingface.co/mygraph/` | `httpfs` |
| HTTPS | `https://host/path/mygraph/` | `httpfs` |

#### Example: S3

First, install and configure the `httpfs` extension if not already done:

```cypher
INSTALL httpfs;
LOAD httpfs;
CALL s3_credential(
  key_id='YOUR_KEY_ID',
  secret='YOUR_SECRET',
  region='us-east-1'
);
```

Then use S3 URIs directly in your schema:

```cypher
CREATE NODE TABLE city(id INT32, name STRING, population INT64, PRIMARY KEY(id))
  WITH (storage = 's3://my-bucket/mygraph/', format = 'icebug-disk');
CREATE REL TABLE livesin(FROM user TO city)
  WITH (storage = 's3://my-bucket/mygraph/', format = 'icebug-disk');
```

:::note[Note]
Remote Parquet files are read lazily — only the row groups needed for a query are fetched. This makes remote icebug-disk datasets practical even over high-latency links, though local storage will always be faster for repeated queries.
:::

## icebug-memory

[icebug-memory](https://github.com/Ladybug-Memory/icebug-format) is a Ladybug-native graph-aware Arrow format designed for ingestion-free graph analytics. Unlike general-purpose Arrow tables, Icebug preserves graph structure (node and relationship tables) and enables direct querying without preprocessing.

### Generating Icebug tables

Use the `icebug-format` tool to generate icebug-memory tables from existing arrow tables:

```python
from icebug_format import IcebugMemGraph

graph: IcebugMemGraph = IcebugMemGraph.from_arrow_tables(
    from_node_arrow_table=users,   # pa.Table, first column is the primary key
    rel_arrow_table=livesin,       # pa.Table with 'source' / 'from' and 'target' / 'to' columns
    to_node_arrow_table=cities,    # pa.Table, first column is the primary key
)
```

:::note[Note]
The conversion utility is only available in Python for now.
:::

### Using Icebug tables

Ladybug Python, Node.js, Rust, and C++ bindings expose create APIs for node and rel tables. For example, in Python:

```python
import ladybug as lb

# get icebug graph from earlier step

db = lb.Database()
conn = lb.Connection(db)

# Create node table
conn.create_arrow_table(
    table_name="users",         # node table name to be used in ladybug
    dataframe=graph.src         # node table as a pa.Table
)

# create rel table
conn.create_arrow_rel_table(
    table_name="livesin",       # rel table name to be used in ladybug
    src_table_name="users",     # src node table name from table creation earlier
    dst_table_name="cities",    # dst node table name from table creation earlier
    layout="CSR",
    dataframe=graph.indices,    # rel table with 'source' and 'target' columns
    dst_col_name="to",          # dst col name in the indices table
    indptr=graph.indptr,        # row pointers for indices table
)

conn.execute("MATCH (a:users)-[b:livesin]->(c:cities) RETURN a.*, b.*, c.*")
```

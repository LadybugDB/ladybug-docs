---
title: Indexes
description: Create and manage primary key indexes on node tables using HASH or ART index types
---

Ladybug automatically creates a primary key index(hash) on every node table to enforce uniqueness
and accelerate primary-key lookups. Ladybug also supports ART indexes for faster range queries on primary keys. Ladybug also maintains **zone maps** (min/max indexes) on all columns automatically — these are used to skip irrelevant node groups during scans and to answer `COUNT(*)` queries without reading column data.

## Default HASH index

When you create a node table, Ladybug automatically builds a hash-based primary-key index.
No extra DDL is required

### Space amplification

The hash index stores one entry per node and adds roughly **15–25 bytes per row** on top of the column data, depending on the primary key type:

| Primary key type | Index overhead |
|---|---|
| `INT32` | ~14 bytes/row |
| `INT64` | ~18 bytes/row |
| `STRING` | ~18 bytes/row + key length |

The column data is stored with compression (Zstandard by default) and is typically similar in size to the source Parquet file. So the total on-disk footprint of a node table is roughly:

```
total size ≈ compressed column data + (num_rows × ~15–25 bytes)
```

**Example**: a 300 MB Parquet file resulted in a **1.2 GB** `.lbdb` database with the default hash index enabled. Disabling the hash index brought it down to **1 GB** — roughly 16% smaller.

If you want to disable the default HASH index to save space, you can do so by setting the `enable_default_hash_index` property to `false` before creating any node tables:

```cypher
CALL enable_default_hash_index = false;
```

Note: The config resets on close, so you need to run this command every time you start a new session if you want to keep the default index disabled

## Creating indexes manually

If you want to create an index on a node table when the `enable_default_hash_index` config is set to false, you can run one of the index creation commands:

To create the inbuilt HASH index:
```cypher
CREATE HASH INDEX <index_name> FOR (<alias>:<NodeTable>) ON (<alias>.<property>);
```

```cypher
CREATE INDEX <index_name> FOR (<alias>:<NodeTable>) ON (<alias>.<property>);
```

To create the ART index:
```cypher
CREATE ART INDEX <index_name> FOR (<alias>:<NodeTable>) ON (<alias>.<property>);
```

Note: At a time, only one primary key index can be created per node table

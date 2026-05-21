---
title: Indexes
description: Create and manage primary key indexes on node tables using HASH or ART index types
---

Ladybug automatically creates a primary key index(hash) on every node table to enforce uniqueness
and accelerate primary-key lookups. Ladybug also supports ART indexes for faster range queries on primary keys

## Default HASH index

When you create a node table, Ladybug automatically builds a hash-based primary-key index.
No extra DDL is required

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

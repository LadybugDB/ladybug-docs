---
title: Alter
description: Alter DDL statements
---

## Add column

`ADD COLUMN` allows you to add a new column to a node or relationship table. If you don't specify a default value, the newly added column is filled with `NULL` values.

Column names must be unique within a node or relationship table.

For example, consider that you try to run the following command to add a column `age`, but it
already exists in the `User` table:
```cypher
ALTER TABLE User ADD age INT64;
```
The query will raise the following exception:
```
"Binder exception: Property: age already exists."
```

The following query adds a new column with the default value `NULL` to the User table.
```cypher
ALTER TABLE User ADD grade INT64;
```

You can also specify the default value of the added column.
```cypher
ALTER TABLE User ADD grade INT64 DEFAULT 40;
```

#### Add column if not exists

If the given column name already exists in the table, Ladybug throws an exception when you try to create it.
To avoid the exception being raised, use the `IF NOT EXISTS` modifier. This tells Ladybug to do nothing when
the given column name already exists in the table.

Example:
```cypher
ALTER TABLE User ADD IF NOT EXISTS grade INT64;
```
This query tells Ladybug to only create the `grade` column if it doesn't exist.

The same applies to relationship tables.

## Drop column

`DROP COLUMN` allows you to remove a column from a table.

The following query drops the age column from the User table.
```cypher
ALTER TABLE User DROP age;
```

#### Drop column if exists

If the given column name does not exist in the table, Ladybug throws an exception when you try to drop it.
To avoid the exception being raised, use the `IF EXISTS` modifier. This tells Ladybug to do nothing when
the given column name does not exist in the table.

Example:
```cypher
ALTER TABLE User DROP IF EXISTS grade;
```
This query tells Ladybug to only drop the `grade` column if it exists.

The same applies to relationship tables.

## Add connection to relationship table

`ADD FROM <node_table_name> TO <node_table_name>` allows you to add a connection between two node tables into an existing relationship table.

The following example creates a node table `Celebrity` and adds `User` follows `Celebrity` into `Follows` relationship table.
```cypher
CREATE NODE TABLE Celebrity(name STRING PRIMARY KEY);
ALTER TABLE Follows ADD FROM User TO Celebrity;
```

#### Add connection if not exists

Use the `IF NOT EXISTS` modifier to do nothing if the given connection already exists.

Example:
```cypher
ALTER TABLE Follows ADD IF NOT EXISTS FROM User TO Celebrity;
```

## Drop connection from relationship table

`DROP FROM <node_table_name> TO <node_table_name>` allows you to drop a connection between two node tables from an existing relationship table.

The following example drops the connection between `User` and `Celebrity` from `Follows` relationship table.
```cypher
ALTER TABLE Follows DROP FROM User TO Celebrity;
```

#### Drop connection if exists

Use the `IF  EXISTS` modifier to do nothing if the given connection does not exist.

Example:
```cypher
ALTER TABLE Follows DROP IF EXISTS FROM User TO Celebrity;
```

## Rename table

`RENAME TABLE` allows you to rename a table.

The following query renames table User to Student.
```cypher
ALTER TABLE User RENAME TO Student;
```

## Rename column

`RENAME COLUMN` allows you to rename a column of a table.<br />

The following query renames the age column to grade.
```cypher
ALTER TABLE User RENAME age TO grade;
```

## Set sorted by

`SET SORTED BY` records sorted-by metadata on a table.

```cypher
ALTER TABLE User SET SORTED BY (age ASC);
```

Syntax (node tables):

```cypher
ALTER TABLE <node-table> SET SORTED BY (<property> ASC|DESC[, ...]) [CSR];
```

Each item names an existing column of the table followed by a required
direction (`ASC` or `DESC`). Single-column and composite lists are allowed, and
directions may be mixed (e.g. `(age DESC, name ASC)`). Declaring unknown columns
is rejected:

```cypher
ALTER TABLE User SET SORTED BY (height ASC);
-- Binder exception: Column height does not exist in table User.
```

A new declaration replaces any previous sorted-by metadata on the table, and a
plain declaration (without `CSR`) clears a previously declared `CSR` flag.
`RENAME COLUMN` follows renames inside the stored list. The declaration is
visible in `EXPLAIN` as `Set Table <table> Sorted By <col> ASC|DESC[, ...]
[CSR]`.

Without `CSR`, the list is metadata only: Ladybug neither verifies nor
establishes the ordering in storage, records no change-epoch watermark, and the
optimizer does not consume it. It is persisted across checkpoints and carried
over by subsequent `ALTER` operations. Only the `CSR` forms below have optimizer
or storage effects.

| Node-table form | Example | Result |
| --- | --- | --- |
| Single column, either direction | `(age ASC)`, `(age DESC)` | Recorded as metadata. |
| Composite, mixed directions | `(age DESC, name ASC)` | Recorded as metadata. |
| Single ascending primary key + `CSR` | `(id ASC) CSR` | CSR constraint declared (see below). |
| `CSR` with `DESC`, composite, or non-PK | `(id DESC) CSR`, `(id ASC, name ASC) CSR`, `(name ASC) CSR` | Rejected (see below). |

### Node-table CSR constraint

Appending the `CSR` keyword declares a CSR (compressed sparse row) constraint on a
node table:

```cypher
ALTER TABLE User SET SORTED BY (id ASC) CSR;
```

`CSR` asserts that the table's primary key is a `csr_index` interchangeable with
the rel table's `table_offset` (i.e. `primary_key == rowid`). This is a user
assertion: Ladybug does not verify it from the data, so only declare it when the
invariant actually holds.

Validation rules (node tables):

- Exactly one `SORTED BY` property.
- It must be ascending (`ASC`). `DESC` is rejected.
- It must be the table's primary key. Any other column is rejected.
- Composite `SORTED BY` lists with `CSR` are rejected.

```cypher
ALTER TABLE User SET SORTED BY (id DESC) CSR;
-- Binder exception: CSR requires exactly one SORTED BY property in ascending order.
ALTER TABLE User SET SORTED BY (id ASC, name ASC) CSR;
-- Binder exception: CSR requires exactly one SORTED BY property in ascending order.
ALTER TABLE User SET SORTED BY (name ASC) CSR;
-- Binder exception: CSR requires the SORTED BY property to be the primary key.
```

Invalidation: Ladybug records the node table's change epoch at declaration time.
Any subsequent mutation of the node table invalidates the invariant and the
optimizer silently disregards the `CSR` flag.

Optimizer behavior: when the `CSR` flag is set and the node table is unmutated
since declaration, the optimizer may use the offset-count degree rewrite
(`RelDegreeTableMode::OFFSET_COUNT`). It may also rewrite
`COUNT(DISTINCT nbr)` over a forward (`FWD`) `WALK` variable-length path
(`MATCH (a:User {id: ...})-[r:Friend*1..5]->(b:User) RETURN count(distinct b)`)
with a fixed source node, no node predicate, a single rel group, and single-labeled
bound/neighbor nodes into a bounded per-depth BFS that counts distinct reachable
offsets (`LogicalReachableCount`/`PhysicalReachableCount`). If any of these
conditions is not met, the original plan is used.

### Rel-table CSR sorted-by-dest

For relationship tables, `CSR` declares that the CSR adjacency lists are physically
stored with neighbors in non-decreasing order within each `FROM` row:

```cypher
ALTER TABLE Follows SET SORTED BY (FROM ASC, TO ASC) CSR;
```

`FROM` and `TO` are the user-facing names for the rel table's source and
destination endpoints. They are reserved structural names, not rel properties.
`bound` and `neighbor` remain internal concepts.

Validation rules (rel tables):

- The `CSR` keyword is required. Plain `SET SORTED BY` on a rel table is rejected.
- The property list must be exactly `(FROM ASC, TO ASC)`. Any other shape
(including `DESC`, single-column, or rel-property lists) is rejected.

```cypher
ALTER TABLE Follows SET SORTED BY (FROM ASC, TO ASC);
-- Binder exception: SORTED BY on rel table Follows requires the CSR clause.
ALTER TABLE Follows SET SORTED BY (FROM ASC, TO DESC) CSR;
-- Binder exception: CSR on rel table Follows requires SORTED BY (FROM ASC, TO ASC).
```

This is a user assertion: storage does not establish the ordering at checkpoint
time, so only declare it on data that is actually sorted (e.g. loaded from a
pre-sorted source). Declaring it on unsorted data produces wrong results on the
fast path.

Effect and invalidation: the flag lets `CSRArrowArrays::symmetrize()` (which
computes `A + A.T`, the symmetric/undirected view used for Arrow CSR results)
skip the `O(m log max-degree)` per-row sort and merge directly against the raw
indices. Ladybug records a `csrChangeEpoch` watermark at declaration time; if the
rel table is mutated afterwards, the fast path is invalidated and `symmetrize()`
falls back to the safe per-row sort.

### Node vs rel at a glance

| Table kind | Without `CSR` | With `CSR` |
| --- | --- | --- |
| Node | Any existing column(s), `ASC`/`DESC`, single or composite. Metadata only: no ordering enforced, no watermark, no optimizer effect. | Exactly one property, `ASC`, and it must be the primary key. Sets the `CSR` flag + change-epoch watermark; mutation invalidates it. |
| Rel | Rejected (`CSR` required). | Only `(FROM ASC, TO ASC)`. Declares sorted CSR adjacency; sets the watermark; mutation falls back to the safe path. |

## Comment on a table

`COMMENT ON` allows you to add comments to a table.

The following query adds a comment to `User` table.
```cypher
COMMENT ON TABLE User IS 'User information';
```
Comments can be extracted through the `SHOW_TABLES()` function. See [CALL](/cypher/query-clauses/call) for more information.
```cypher
CALL SHOW_TABLES() RETURN *;
```
```table
┌───────────┬───────────┬──────────────────┐
│ TableName │ TableType │ TableComment     │
├───────────┼───────────┼──────────────────┤
│ User      │ NODE      │ User information │
│ City      │ NODE      │                  │
└───────────┴───────────┴──────────────────┘
```


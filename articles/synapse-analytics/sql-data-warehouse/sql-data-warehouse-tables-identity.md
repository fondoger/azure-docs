---
title: Use IDENTITY to create surrogate keys
description: Learn about recommendations and examples for using the IDENTITY property to create surrogate keys on tables in dedicated SQL pool.
author: mstehrani
ms.author: emtehran

ms.date: 01/22/2025
ms.service: azure-synapse-analytics
ms.subservice: sql-dw
ms.topic: concept-article
ms.custom: azure-synapse
---

# Use IDENTITY to create surrogate keys in dedicated SQL pool

In this article, you find recommendations and examples for using the `IDENTITY` property to create surrogate keys on tables in dedicated SQL pool.

## What is a surrogate key?

A surrogate key on a table is a column with a unique identifier for each row. The key isn't generated from the table data. Data modelers like to create surrogate keys on their tables when they design data warehouse models. You can use the `IDENTITY` property to achieve this goal simply and effectively without affecting load performance.
> [!NOTE]
> In Azure Synapse Analytics:
> - The IDENTITY value increases on its own in each distribution and doesn't overlap with IDENTITY values in other distributions. The IDENTITY value in Synapse isn't guaranteed to be unique if the user explicitly inserts a duplicate value with `SET IDENTITY_INSERT ON` or reseeds IDENTITY. For details, see [CREATE TABLE (Transact-SQL) IDENTITY (Property)](/sql/t-sql/statements/create-table-transact-sql-identity-property?view=azure-sqldw-latest&preserve-view=true).
> - UPDATE on distribution column doesn't guarantee the IDENTITY value is unique. Use [DBCC CHECKIDENT (Transact-SQL)](/sql/t-sql/database-console-commands/dbcc-checkident-transact-sql?view=azure-sqldw-latest&preserve-view=true) after UPDATE on distribution column to verify uniqueness.

## Create a table with an IDENTITY column

The `IDENTITY` property is designed to scale out across all the distributions in the dedicated SQL pool without affecting load performance. Therefore, the implementation of `IDENTITY` is oriented toward achieving these goals.

You can define a table as having the `IDENTITY` property when you first create the table by using syntax that's similar to the following statement:

```sql
CREATE TABLE dbo.T1
(    C1 INT IDENTITY(1,1) NOT NULL,
     C2 INT NULL
)
WITH
(   DISTRIBUTION = HASH(C2),
    CLUSTERED COLUMNSTORE INDEX
);
```

You can then use `INSERT..SELECT` to populate the table.

The remainder of this section highlights the nuances of the implementation to help you understand them more fully.  

### Allocation of values

The `IDENTITY` property doesn't guarantee the order in which the surrogate values are allocated due to the distributed architecture of the data warehouse. The `IDENTITY` property is designed to scale out across all the distributions in the dedicated SQL pool without affecting load performance.

The following example is an illustration:

```sql
CREATE TABLE dbo.T1
(    C1 INT IDENTITY(1,1) NOT NULL,
     C2 VARCHAR(30) NULL
)
WITH
(   DISTRIBUTION = HASH(C2),
    CLUSTERED COLUMNSTORE INDEX
);

INSERT INTO dbo.T1
VALUES (NULL);

INSERT INTO dbo.T1
VALUES (NULL);

SELECT *
FROM dbo.T1;

DBCC PDW_SHOWSPACEUSED('dbo.T1');
```

In the preceding example, two rows landed in distribution 1. The first row has the surrogate value of 1 in column `C1`, and the second row has the surrogate value of 61. Both of these values were generated by the `IDENTITY` property. However, the allocation of the values isn't contiguous. This behavior is by design.

### Skewed data

The range of values for the data type are spread evenly across the distributions. If a distributed table suffers from skewed data, then the range of values available to the datatype can be exhausted prematurely. For example, if all the data ends up in a single distribution, then effectively the table has access to only one-sixtieth of the values of the data type. For this reason, the `IDENTITY` property is limited to `INT` and `BIGINT` data types only.

### SELECT..INTO

When an existing `IDENTITY` column is selected into a new table, the new column inherits the `IDENTITY` property, unless one of the following conditions is true:

- The `SELECT` statement contains a join.
- Multiple `SELECT` statements are joined by using `UNION`.
- The `IDENTITY` column is listed more than one time in the `SELECT` list.
- The `IDENTITY` column is part of an expression.

If any one of these conditions is true, the column is created `NOT NULL` instead of inheriting the `IDENTITY` property.

### CREATE TABLE AS SELECT

`CREATE TABLE AS SELECT` (CTAS) follows the same SQL Server behavior that's documented for `SELECT..INTO`. However, you can't specify an `IDENTITY` property in the column definition of the `CREATE TABLE` part of the statement. You also can't use the `IDENTITY` function in the `SELECT` part of the CTAS. To populate a table, you need to use `CREATE TABLE` to define the table followed by `INSERT..SELECT` to populate it.

## Insert explicit values into an IDENTITY column

Dedicated SQL pool supports `SET IDENTITY_INSERT <your table> ON|OFF` syntax. You can use this syntax to explicitly insert values into the `IDENTITY` column.

Many data modelers like to use predefined negative values for certain rows in their dimensions. An example is the -1 or *unknown member* row.

The next script shows how to explicitly add this row by using `SET IDENTITY_INSERT`:

```sql
SET IDENTITY_INSERT dbo.T1 ON;

INSERT INTO dbo.T1
(   C1,
    C2
)
VALUES (-1,'UNKNOWN');

SET IDENTITY_INSERT dbo.T1 OFF;

SELECT     *
FROM    dbo.T1;
```

## Load data

The presence of the `IDENTITY` property has some implications to your data-loading code. This section highlights some basic patterns for loading data into tables by using `IDENTITY`.

To load data into a table and generate a surrogate key by using `IDENTITY`, create the table and then use `INSERT..SELECT` or `INSERT..VALUES` to perform the load.

The following example highlights the basic pattern:

```sql
--CREATE TABLE with IDENTITY
CREATE TABLE dbo.T1
(    C1 INT IDENTITY(1,1),
     C2 VARCHAR(30)
)
WITH
(   DISTRIBUTION = HASH(C2),
    CLUSTERED COLUMNSTORE INDEX
);

--Use INSERT..SELECT to populate the table from an external table
INSERT INTO dbo.T1
(C2)
SELECT     C2
FROM    ext.T1;

SELECT *
FROM   dbo.T1;

DBCC PDW_SHOWSPACEUSED('dbo.T1');
```

> [!NOTE]
> It's not possible to use `CREATE TABLE AS SELECT` currently when loading data into a table with an `IDENTITY` column.

For more information on loading data, see [Designing Extract, Load, and Transform (ELT) for dedicated SQL pool](design-elt-data-loading.md) and [Loading best practices](../sql/data-loading-best-practices.md).

## System views

You can use the [sys.identity_columns](/sql/relational-databases/system-catalog-views/sys-identity-columns-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) catalog view to identify a column that has the `IDENTITY` property.

To help you better understand the database schema, this example shows how to integrate `sys.identity_columns` with other system catalog views:

```sql
SELECT  sm.name
,       tb.name
,       co.name
,       CASE WHEN ic.column_id IS NOT NULL
             THEN 1
        ELSE 0
        END AS is_identity
FROM        sys.schemas AS sm
JOIN        sys.tables  AS tb           ON  sm.schema_id = tb.schema_id
JOIN        sys.columns AS co           ON  tb.object_id = co.object_id
LEFT JOIN   sys.identity_columns AS ic  ON  co.object_id = ic.object_id
                                        AND co.column_id = ic.column_id
WHERE   sm.name = 'dbo'
AND     tb.name = 'T1'
;
```

## Limitations

The `IDENTITY` property can't be used:

- When the column data type isn't `INT` or `BIGINT`
- When the column is also the distribution key
- When the table is an external table

The following related functions aren't supported in dedicated SQL pool:

- [IDENTITY()](/sql/t-sql/functions/identity-function-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [@@IDENTITY](/sql/t-sql/functions/identity-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [SCOPE_IDENTITY](/sql/t-sql/functions/scope-identity-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [IDENT_CURRENT](/sql/t-sql/functions/ident-current-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [IDENT_INCR](/sql/t-sql/functions/ident-incr-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [IDENT_SEED](/sql/t-sql/functions/ident-seed-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)

## Common tasks

You can use the following sample code to perform common tasks when you work with `IDENTITY` columns.

Column C1 is the `IDENTITY` in all the following tasks.

### Find the highest allocated value for a table

Use the `MAX()` function to determine the highest value allocated for a distributed table:

```sql
SELECT MAX(C1)
FROM dbo.T1
```

### Find the seed and increment for the IDENTITY property

You can use the catalog views to discover the identity increment and seed configuration values for a table by using the following query:

```sql
SELECT  sm.name
,       tb.name
,       co.name
,       ic.seed_value
,       ic.increment_value
FROM        sys.schemas AS sm
JOIN        sys.tables  AS tb           ON  sm.schema_id = tb.schema_id
JOIN        sys.columns AS co           ON  tb.object_id = co.object_id
JOIN        sys.identity_columns AS ic  ON  co.object_id = ic.object_id
                                        AND co.column_id = ic.column_id
WHERE   sm.name = 'dbo'
AND     tb.name = 'T1'
;
```

## Related content

- [Design tables using dedicated SQL pool](sql-data-warehouse-tables-overview.md)
- [CREATE TABLE (Transact-SQL) IDENTITY (Property)](/sql/t-sql/statements/create-table-transact-sql-identity-property?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [DBCC CHECKIDENT (Transact-SQL)](/sql/t-sql/database-console-commands/dbcc-checkident-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)

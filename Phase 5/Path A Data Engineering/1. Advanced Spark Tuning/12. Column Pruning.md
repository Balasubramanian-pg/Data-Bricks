# 012 Column Pruning

Canonical documentation for 012 Column Pruning. This document defines concepts, terminology, and standard usage.

## Purpose
012 Column Pruning exists to optimize data retrieval and processing by ensuring that only the data attributes strictly necessary for a given operation are loaded into memory or transmitted across a network. In modern data systems—where tables may contain hundreds or thousands of columns—reading entire rows (horizontal records) when only a few fields are required creates significant I/O, memory, and compute overhead. 

This topic addresses the problem of resource exhaustion and latency in data-intensive environments by defining the mechanisms for identifying and discarding unused attributes as early as possible in the data lifecycle.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Logical Analysis:** The process of determining which columns are required by downstream operations (projections, filters, joins).
* **Physical Optimization:** The interaction between the query optimizer and storage layers (e.g., columnar storage formats).
* **Data Flow:** The propagation of required-column metadata through a transformation graph.

**Out of scope:**
* **Row Filtering (Predicate Pushdown):** While related, the removal of horizontal records is a distinct optimization.
* **Specific Vendor Implementations:** Syntax or configuration specific to engines like Apache Spark, Snowflake, or PostgreSQL.
* **Data Compression Algorithms:** Techniques for shrinking data size within a column.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Projection** | The operation of selecting a specific subset of columns from a dataset. |
| **Upstream** | Operations or stages closer to the data source. |
| **Downstream** | Operations or stages closer to the final output or consumer. |
| **Columnar Storage** | A data storage architecture that organizes data by column rather than by row, facilitating efficient pruning. |
| **Materialization** | The process of loading data from storage into memory or writing intermediate results to disk. |
| **Schema Evolution** | The change in the structure of data over time, which pruning must account for to avoid data loss. |

## Core Concepts

### 1. Dependency Tracking
Column pruning relies on a "bottom-up" or "backward" analysis of a data pipeline. The optimizer starts at the final output (the sink) and identifies which columns are requested. It then traverses the execution plan toward the source, marking every column required for calculations, joins, or filters as "active."

### 2. Early Discard
The fundamental principle of 012 Column Pruning is to discard unreferenced columns at the earliest possible stage. Ideally, this occurs at the storage layer (I/O level), preventing the data from ever entering the compute environment.

### 3. Metadata-Only Operations
In advanced pruning scenarios, if a query only requires metadata (e.g., `COUNT(*)`, column names, or data types), the pruning engine may bypass the data blocks entirely, fulfilling the request using only the file headers or system catalogs.

## Standard Model

The standard model for 012 Column Pruning follows a three-tier architecture:

1.  **Requirement Analysis:** The execution engine parses the request and builds a "Required Column Set" (RCS).
2.  **Optimizer Propagation:** The RCS is pushed back through the logical plan. If a transformation (like a join) requires additional columns for its own internal logic, those are added to the RCS for all upstream nodes.
3.  **Physical Pushdown:** The final RCS is passed to the Data Source Provider. If the storage format supports it (e.g., Parquet, ORC), the provider only reads the byte-offsets corresponding to the columns in the RCS.

## Common Patterns

### Projection Pushdown
The most common pattern where the `SELECT` or `Project` operator is moved as close to the data source as possible. This reduces the width of the "tuple" or "row" early in the execution.

### Nested Field Pruning
In complex data types (JSON, Structs, Maps), pruning is applied to sub-fields. If a table has a `User` column with `Name`, `Address`, and `ID`, but only `ID` is used, the engine prunes the `Name` and `Address` sub-fields within the complex object.

### Join-Key Retention
A pattern where columns not present in the final output are retained temporarily because they serve as the join keys or partitioning keys for intermediate shuffle operations.

## Anti-Patterns

### "Select Star" (`SELECT *`)
The most prevalent anti-pattern. Requesting all columns bypasses the benefits of pruning, forcing the system to materialize and transport data that will ultimately be ignored by the application logic.

### Late Projection
Performing heavy transformations or joins on wide rows and only selecting the required columns at the very end of the pipeline. This leads to excessive memory pressure and unnecessary serialization/deserialization costs.

### Opaque Transformations
Using User-Defined Functions (UDFs) or "Black Box" operators that do not expose which columns they access. This often forces the optimizer to disable pruning and pass the entire row to ensure the function has the data it needs.

## Edge Cases

*   **Dynamic Schema/Schema-on-Read:** In systems where the schema is not known until the data is read, pruning must be performed dynamically during the scan, which is less efficient than static pruning.
*   **Virtual/Computed Columns:** When a column is derived from other columns, pruning the derived column requires the system to "un-prune" or retain the underlying source columns.
*   **Triggers and Logging:** System-level requirements (like audit logging or database triggers) may implicitly require columns that are not explicitly requested in the user query.
*   **Empty Projections:** Queries that do not require any columns (e.g., `EXISTS` checks or `COUNT` constants). The system must decide whether to read a "dummy" column or use metadata.

## Related Topics
*   **011 Predicate Pushdown:** The horizontal counterpart to column pruning (filtering rows).
*   **015 Partition Pruning:** Skipping entire directories or files based on partition keys.
*   **022 Vectorized Execution:** Processing batches of pruned columnar data for CPU efficiency.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
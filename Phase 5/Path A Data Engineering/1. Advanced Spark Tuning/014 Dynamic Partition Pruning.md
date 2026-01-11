# 014 Dynamic Partition Pruning

Canonical documentation for 014 Dynamic Partition Pruning. This document defines concepts, terminology, and standard usage.

## Purpose
Dynamic Partition Pruning (DPP) addresses the performance bottleneck inherent in large-scale data processing where static analysis is insufficient to determine the minimal set of data required for a query. In distributed systems, data is often partitioned by specific columns to optimize retrieval. While static pruning can eliminate partitions based on constant filters in a query, it cannot optimize scenarios where the filter criteria are derived from another data source at runtime.

DPP exists to bridge this gap by allowing the engine to use the results of one side of a join (typically a smaller, filtered dimension table) to dynamically prune partitions from the other side (typically a large fact table) during execution. This minimizes I/O overhead, reduces network traffic, and optimizes resource utilization.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* The mechanism of runtime partition elimination based on join predicates.
* The relationship between dimension-side filtering and fact-side scanning.
* Optimization logic involving broadcast mechanisms and subqueries.
* Theoretical performance implications for star-schema architectures.

**Out of scope:**
* Specific vendor-specific configuration parameters (e.g., Spark, Trino, or Hive-specific flags).
* Physical storage formats (Parquet, ORC, Avro) except where they relate to partitioning metadata.
* Static partition pruning (compile-time optimization).

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Partition Pruning** | The process of skipping irrelevant data files or directories based on partition metadata. |
| **Static Pruning** | Pruning that occurs at query compilation time based on literal values in the `WHERE` clause. |
| **Dynamic Pruning** | Pruning that occurs at runtime by injecting filters derived from the result of a join operand. |
| **Dimension Table** | A smaller table containing descriptive attributes, often used as the source of the pruning filter. |
| **Fact Table** | A large, partitioned table containing quantitative data, typically the target of the pruning operation. |
| **Broadcast Exchange** | A mechanism where a small dataset is distributed to all worker nodes to facilitate a join or filter. |
| **Pruning Predicate** | The specific condition or set of values used to identify which partitions must be scanned. |

## Core Concepts

### The Runtime Filter
Unlike static filters, which are known before the query starts, a dynamic filter is generated during the execution phase. When a query joins a large partitioned table with a smaller filtered table, the engine executes the filter on the smaller table first. The resulting unique keys are then packaged as a dynamic predicate and pushed down to the scan operation of the large table.

### Inter-node Communication
In distributed environments, DPP relies on the ability of the query coordinator or engine to pass information between different branches of the execution plan. This is often achieved by intercepting the results of a "Build" side of a join and propagating those results to the "Probe" side before the probe scan initiates.

### Metadata-Level Skipping
DPP operates at the metadata level. Instead of reading a file and then discarding rows (row-level filtering), the engine uses the dynamic predicate to entirely exclude file paths or partition identifiers from the scan task list.

## Standard Model
The standard model for Dynamic Partition Pruning follows a specific execution flow:

1.  **Filter Application:** A filter is applied to the Dimension Table (e.g., `Date > '2023-01-01'`).
2.  **Result Materialization:** The engine identifies the unique values of the join key from the filtered Dimension Table.
3.  **Predicate Injection:** These unique values are converted into a dynamic filter (e.g., `FactTable.date_id IN (val1, val2, ...)`).
4.  **Partition Elimination:** The Fact Table scan operator receives this dynamic filter and compares it against the partition metadata.
5.  **Optimized Scan:** Only the directories/files corresponding to the matching partitions are read into memory.

## Common Patterns

### Star Schema Optimization
The most frequent application of DPP is in star schemas where a central fact table is joined with multiple dimension tables. DPP allows the engine to prune the fact table based on attributes in any of the dimension tables, even if those attributes are not part of the fact table's schema.

### Multi-Level Partitioning
In systems with hierarchical partitioning (e.g., Year/Month/Day), DPP can be applied at multiple levels simultaneously. If the join key corresponds to the "Month" partition, the engine can skip all "Year" and "Day" folders that do not contain the relevant months.

### Broadcast-Based DPP
When the filtered dimension table is small enough to fit in memory, it is broadcast to all executors. Each executor then uses the local copy of the dimension keys to prune the partitions of the fact table data it is responsible for processing.

## Anti-Patterns

### Over-Partitioning
Creating too many small partitions (the "small files problem") can negate the benefits of DPP. The overhead of the engine processing the dynamic metadata for thousands of tiny partitions may exceed the time saved by skipping them.

### Pruning on Non-Partitioned Columns
Attempting to apply DPP logic to columns that are not defined as partition keys. While engines may apply row-level filtering, this is not "Partition Pruning" and does not provide the I/O benefits of skipping entire data structures.

### High-Cardinality Dynamic Filters
If the filtered dimension table produces a massive set of unique keys (e.g., millions of IDs), the cost of transmitting and processing the dynamic predicate can become a bottleneck, potentially leading to memory exhaustion or increased latency.

## Edge Cases

### Empty Dimension Results
If the filter on the dimension table returns zero rows, the DPP mechanism should ideally short-circuit the fact table scan entirely, resulting in a zero-scan operation. However, some implementations may still perform a metadata check.

### Non-Equi Joins
DPP is primarily designed for equi-joins (where `TableA.key = TableB.key`). In cases of range joins or inequality joins, the ability to generate a discrete list of partitions to prune is significantly diminished or impossible.

### Data Skew
If the dimension values are heavily skewed toward a single partition, DPP will successfully prune other partitions, but the remaining scan will still encounter a bottleneck. DPP optimizes I/O volume but does not inherently solve compute skew.

## Related Topics
* **008 Predicate Pushdown:** The general concept of moving filters as close to the data source as possible.
* **022 Star Schema Design:** The architectural pattern that most frequently benefits from DPP.
* **045 Broadcast Joins:** A common execution strategy used to facilitate dynamic pruning.
* **012 Partitioning Strategies:** The foundational data organization required for pruning to function.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
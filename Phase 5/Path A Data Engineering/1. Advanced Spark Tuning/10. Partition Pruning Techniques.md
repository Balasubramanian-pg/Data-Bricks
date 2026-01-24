# 010 Partition Pruning Techniques

Canonical documentation for 010 Partition Pruning Techniques. This document defines concepts, terminology, and standard usage.

## Purpose
Partition pruning exists to optimize query performance and resource utilization in large-scale data systems. As datasets grow, scanning an entire table (a "full table scan") becomes computationally expensive and time-prohibitive. Partition pruning addresses this by enabling the query engine to programmatically exclude data segments (partitions) that are guaranteed not to contain records matching the query criteria. This reduces I/O overhead, minimizes memory consumption, and accelerates execution times.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Core logic of partition elimination during query planning and execution.
* Theoretical mechanisms for static and dynamic pruning.
* Relationship between data distribution strategies and pruning efficacy.
* Metadata requirements for pruning operations.

**Out of scope:**
* Specific vendor-specific SQL syntax or "hints."
* Physical storage formats (e.g., Parquet vs. ORC) except where they impact metadata availability.
* Hardware-level optimizations (e.g., SSD vs. HDD seek times).

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Partition** | A logical or physical division of a dataset based on the values of one or more specific columns (partition keys). |
| **Partition Key** | The specific attribute(s) used to determine the assignment of a record to a partition. |
| **Pruning** | The process by which a query optimizer ignores partitions that do not satisfy the query's filter predicates. |
| **Predicate** | A logical expression in a query (e.g., `WHERE date > '2023-01-01'`) used to filter data. |
| **Static Pruning** | Pruning that occurs during the query compilation or planning phase when filter values are known constants. |
| **Dynamic Pruning** | Pruning that occurs during query execution, often when filter values depend on the results of another operation (e.g., a subquery or join). |
| **Metadata Store** | A repository containing the min/max values, distinct counts, or specific identifiers associated with each partition. |

## Core Concepts
The fundamental principle of partition pruning is **Predicate Evaluation against Metadata**. 

1.  **Data Locality:** Data is organized such that records with similar partition key values are stored together.
2.  **Metadata Awareness:** The system maintains a high-level map of which values reside in which partitions without reading the actual data records.
3.  **Search Space Reduction:** By comparing query predicates against the metadata map, the engine creates a "pruned" execution plan that only targets relevant storage locations.

## Standard Model
The standard model for partition pruning involves a three-tier interaction:

1.  **The Parser/Analyzer:** Identifies predicates involving columns designated as partition keys.
2.  **The Optimizer:** Compares the identified predicates against the partition catalog.
    *   If a predicate is a constant (e.g., `year = 2024`), the optimizer performs **Static Pruning**.
    *   If a predicate involves a join or a subquery, the optimizer prepares a mechanism for **Dynamic Pruning** (e.g., a bloom filter or a runtime lookup).
3.  **The Execution Engine:** Accesses only the storage addresses of the non-pruned partitions.

## Common Patterns
*   **Range Pruning:** Used when data is partitioned by continuous values (e.g., timestamps). The engine skips partitions whose range does not overlap with the query range.
*   **List/Discrete Pruning:** Used when data is partitioned by specific categories (e.g., `RegionID`). The engine only accesses partitions matching the specific IDs in the `IN` clause.
*   **Hash Pruning:** While hash partitioning is primarily for distribution, pruning can occur if the query specifies an exact match for the hash key, allowing the engine to calculate the specific bucket and ignore all others.
*   **Multi-level Pruning:** In systems with composite partitioning (e.g., partitioned by `Year` then sub-partitioned by `Region`), pruning can occur at multiple hierarchical levels.

## Anti-Patterns
*   **Function Wrapping:** Applying a function to a partition key in a predicate (e.g., `WHERE YEAR(transaction_date) = 2023`). This often "blinds" the optimizer, preventing it from matching the predicate to the raw partition metadata.
*   **Type Mismatch (Implicit Casting):** Comparing a string literal to a numeric partition key. The resulting implicit cast can disable pruning logic.
*   **Opaque Predicates:** Using non-deterministic functions (e.g., `RAND()`) or complex expressions that the optimizer cannot resolve against the partition map at plan time.
*   **Over-Partitioning:** Creating too many small partitions. While this allows for granular pruning, the overhead of managing and checking metadata for thousands of partitions can outweigh the benefits of skipping data.

## Edge Cases
*   **Null Values:** Handling of `NULL` in partition keys varies. If a partition is not explicitly defined for `NULL`, records may be placed in a "default" or "overflow" partition, which must be scanned if the predicate is ambiguous.
*   **Late-Binding Parameters:** In some prepared statements, the optimizer may not be able to prune until the moment of execution, leading to a "generic" plan that is less efficient than a "custom" plan.
*   **Overlapping Ranges:** In complex systems where partition boundaries might overlap (rare in standard RDBMS but possible in some distributed systems), pruning logic must be conservative to ensure data integrity, potentially reducing pruning efficiency.
*   **Join-Based Pruning:** When pruning depends on a dimension table join, the "small" side of the join must be processed first to generate the keys used to prune the "large" fact table.

## Related Topics
*   **008 Data Sharding Strategies:** The physical distribution of data across nodes.
*   **012 Indexing vs. Partitioning:** Comparing local/global indexes to partition-level metadata.
*   **015 Statistics Collection:** The process of gathering the metadata that enables pruning.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
# 011 Predicate Pushdown

Canonical documentation for 011 Predicate Pushdown. This document defines concepts, terminology, and standard usage.

## Purpose
Predicate Pushdown is an optimization strategy in data processing designed to minimize data movement and resource consumption. It addresses the inefficiency of retrieving large datasets from storage only to discard a significant portion during the application of filters at the processing layer.

By "pushing" the filtering logic (the predicate) down to the data source or storage layer, the system ensures that only the relevant subset of data is read, transferred across the network, and loaded into memory. This reduces I/O overhead, decreases network congestion, and improves overall query performance.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* The logical mechanism of moving filter operations toward the data source.
* Interaction between query optimizers and storage engines.
* Impact on I/O and computational efficiency.
* Theoretical requirements for pushdown compatibility.

**Out of scope:**
* Specific SQL syntax for various database vendors.
* Implementation details of specific file formats (e.g., Parquet, ORC) beyond their general structural support for pushdown.
* Hardware-level optimizations (e.g., FPGA-based filtering).

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Predicate** | A logical expression that evaluates to a Boolean value (TRUE, FALSE, or UNKNOWN) for each record in a dataset. |
| **Pushdown** | The process of delegating the evaluation of a predicate from a high-level execution engine to a lower-level storage or retrieval component. |
| **Selectivity** | A measure of the proportion of rows from a dataset that satisfy a predicate. High selectivity means few rows are returned. |
| **Projection** | The selection of specific columns from a dataset, often used in conjunction with pushdown to further reduce I/O. |
| **Storage Layer** | The component responsible for persisting and retrieving raw data (e.g., a filesystem, a block store, or a remote database). |
| **Query Optimizer** | The software component that determines the most efficient way to execute a data request, including whether pushdown is possible. |

## Core Concepts

### The Early Filtering Principle
The fundamental axiom of Predicate Pushdown is that filtering should occur as early as possible in the data pipeline. The closer the filter is to the physical bits on the disk, the less work the rest of the system must perform.

### Logical vs. Physical Pushdown
*   **Logical Pushdown:** The query optimizer identifies that a filter can be moved before a join or an aggregation in the execution plan.
*   **Physical Pushdown:** The execution engine communicates the filter requirements to the storage interface (e.g., an S3 Select call or a scan of Parquet metadata) so that the storage layer only returns matching blocks or rows.

### Metadata-Based Pruning
Modern storage formats often store summary statistics (min/max values, null counts) for blocks of data. Predicate pushdown leverages this metadata to skip entire files or blocks without reading the actual records, a process often called "block skipping" or "coarse-grained filtering."

## Standard Model
The standard model for Predicate Pushdown involves three distinct layers:

1.  **The Requestor (Application/User):** Submits a query containing a filter (e.g., `WHERE temperature > 30`).
2.  **The Optimizer (Execution Engine):** Analyzes the query. It recognizes the predicate and determines if the underlying storage layer can interpret and execute that specific logic.
3.  **The Provider (Storage Layer):** Receives the predicate along with the data request. It applies the filter during the scan process and returns only the qualifying records to the Optimizer.

In this model, the "Contract" between the Optimizer and the Provider is critical. The Provider must be able to understand the data types and operators used in the predicate.

## Common Patterns

### Columnar Storage Pushdown
In columnar formats, pushdown is highly effective because the engine can read the metadata for a specific column and skip the corresponding data for all other columns if the predicate is not met.

### Partition Pruning
A specialized form of pushdown where the predicate applies to the "partition key" (e.g., `date` or `region`). The system avoids scanning entire directories or partitions that do not match the predicate.

### Join-Predicate Pushdown
In distributed systems, predicates are often pushed through join operations. If a filter exists on one side of a join, it can sometimes be pushed to the other side (semi-join optimization) to reduce the amount of data shuffled across the network.

## Anti-Patterns

### Opaque Predicates (Black Box Functions)
Using User-Defined Functions (UDFs) or complex transformations within a predicate often prevents pushdown. If the storage layer cannot "see" inside the function to understand its logic, it cannot apply the filter during the scan.

### Type Mismatches
If the predicate compares a column to a value of a different data type (e.g., comparing a string-formatted date to a timestamp object), the optimizer may fail to push the predicate down due to the need for explicit type casting at the processing layer.

### Late Filtering
Applying filters after an expensive operation (like a wide shuffle or a complex aggregation) when the filter could have been applied to the source data.

## Edge Cases

### Non-Deterministic Functions
Predicates involving non-deterministic functions (e.g., `WHERE event_time > CURRENT_TIMESTAMP`) can be difficult to push down if the storage layer and the execution engine have different views of "now" or if the value changes during execution.

### Negation and Inequality
While `EQUALS` and `GREATER THAN` are easily pushed down, complex negations (`NOT IN`, `NOT LIKE`) or inequality operators may not be supported by all storage layer metadata structures, leading to full scans.

### Nested Data Structures
Pushing predicates into complex nested types (JSON, Maps, Arrays) requires the storage layer to have deep schema awareness. Many implementations default to reading the entire nested object and filtering in memory.

## Related Topics
*   **012 Columnar Projection:** The practice of reading only required columns.
*   **045 Query Optimization:** The broader field of improving execution plans.
*   **089 Partitioning Strategies:** How data layout affects pushdown efficiency.
*   **102 Data Shuffling:** The movement of data between nodes that pushdown seeks to minimize.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
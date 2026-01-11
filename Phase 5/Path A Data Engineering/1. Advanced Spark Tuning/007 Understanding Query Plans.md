# 007 Understanding Query Plans

Canonical documentation for 007 Understanding Query Plans. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of a query plan is to provide a deterministic execution strategy for a declarative request. In relational and non-relational data systems, users specify *what* data they want (declarative) rather than *how* to retrieve it (imperative). The query plan serves as the bridge between these two states, translating high-level logic into a sequence of physical operations.

Understanding query plans is essential for diagnosing performance bottlenecks, ensuring resource efficiency, and predicting how data volume changes will impact system latency.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* The lifecycle of a query from parsing to execution.
* Logical vs. Physical operators.
* Cost-based optimization (CBO) principles.
* Common access methods and join strategies.
* Factors influencing plan selection (statistics, cardinality, and constraints).

**Out of scope:**
* Specific vendor-specific syntax (e.g., `EXPLAIN ANALYZE`, `SET SHOWPLAN_XML`).
* Hardware-level execution details (e.g., CPU cache line optimization).
* Specific storage engine file formats.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Optimizer** | The component of a database engine that evaluates multiple execution strategies and selects the most efficient one based on cost or rules. |
| **Cardinality** | The estimated or actual number of rows/documents processed by a specific operator in a plan. |
| **Cost** | An abstract unit of measurement representing the estimated resource consumption (CPU, I/O, Memory) required to execute a plan. |
| **Operator** | A discrete functional unit in a query plan (e.g., a Join, a Filter, or a Sort) that performs a specific action on data. |
| **Selectivity** | A measure of the proportion of rows from a dataset that satisfy a specific predicate (filter). |
| **SARGable** | (Search ARGumentable) A condition in a query that allows the engine to use an index to speed up the search. |
| **Predicate** | A logical expression that evaluates to true, false, or unknown, used to filter data. |

## Core Concepts

### 1. The Query Lifecycle
A query plan is the output of a multi-stage process:
1.  **Parsing:** Validating syntax and converting the query into an initial logical tree.
2.  **Binding/Normalization:** Validating object names (tables, columns) and types against the system catalog.
3.  **Optimization:** The engine generates multiple candidate plans and uses a cost model to select the "cheapest" one.
4.  **Execution:** The engine follows the chosen plan to retrieve or modify data.

### 2. The Tree Structure
Query plans are typically represented as a tree of operators. Data flows from the "leaves" (bottom or right) toward the "root" (top or left). Each operator consumes the output of its children, processes it, and passes the result to its parent.

### 3. Logical vs. Physical Plans
*   **Logical Plan:** An abstract representation of the algebraic operations required (e.g., "Join Table A and Table B").
*   **Physical Plan:** The specific implementation of those operations (e.g., "Perform a Hash Join between Table A and Table B using an Index Scan on Table A").

## Standard Model

### Cost-Based Optimization (CBO)
The modern standard for query planning is the Cost-Based Optimizer. The CBO relies on **Statistics** (histograms, row counts, and density) to estimate the cost of various execution paths. The goal is not necessarily to find the *perfect* plan, but to find a *good* plan in a reasonable amount of time.

### Access Methods
The standard model defines how data is initially retrieved:
*   **Full Scan:** Reading every record in a collection or table.
*   **Range Scan:** Reading a subset of records based on a sorted index.
*   **Point Lookup:** Retrieving a single record via a unique identifier or index key.

### Join Strategies
When combining datasets, three primary physical strategies are utilized:
1.  **Nested Loops:** For every row in the outer table, search the inner table. Efficient for small datasets or when the inner side is indexed.
2.  **Hash Join:** Build a hash table in memory from the smaller dataset and probe it with the larger dataset. Efficient for large, unsorted datasets.
3.  **Merge Join:** Both datasets are sorted by the join key and merged. Efficient for large datasets that are already sorted.

## Common Patterns

### Filter Pushdown
The optimizer moves filter predicates as close to the data source as possible to reduce the number of rows processed by subsequent operators (e.g., Joins or Sorts).

### Projection Pruning
The optimizer ensures that only the columns explicitly requested (or required for intermediate steps) are retrieved from storage, reducing I/O and memory overhead.

### Parallel Execution
The engine splits a single operator's work across multiple threads or nodes, aggregating the results at a "Gather" or "Exchange" node.

## Anti-Patterns

### Cartesian Products (Cross Joins)
Occurs when two datasets are joined without a joining predicate, resulting in a result set size equal to the product of the two inputs. This often indicates a missing join condition.

### Non-SARGable Predicates
Using functions or calculations on an indexed column in a `WHERE` clause (e.g., `WHERE YEAR(date_column) = 2023`). This prevents the optimizer from using an index, forcing a full scan.

### Implicit Type Conversion
When a query compares two different data types, the engine may be forced to convert every value in a column to match the constant, disabling index usage.

## Edge Cases

### Parameter Sniffing
In systems using prepared statements or stored procedures, the optimizer may generate a plan based on the parameters provided during the first execution. If subsequent parameters have vastly different data distributions, the cached plan may become highly inefficient.

### Data Skew
Statistics usually represent an average distribution. If a specific value appears in 90% of rows while others appear in 0.1%, the optimizer may choose a plan that is optimal for the average but catastrophic for the skewed value.

### Empty or Small Sets
In very small tables (e.g., < 10 rows), an optimizer may choose a Full Scan over an Index Seek because the overhead of initializing the index read exceeds the cost of reading the entire table into memory.

## Related Topics
*   **008 Database Indexing Strategies:** How physical structures support query plan operators.
*   **012 Statistics and Histograms:** The data inputs that drive Cost-Based Optimization.
*   **015 Distributed Query Execution:** How plans are partitioned across multiple nodes.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
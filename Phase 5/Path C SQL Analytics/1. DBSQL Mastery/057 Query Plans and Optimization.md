# 057 Query Plans and Optimization

Canonical documentation for 057 Query Plans and Optimization. This document defines concepts, terminology, and standard usage.

## Purpose
Query Plans and Optimization represent the mechanism by which a database management system (DBMS) translates a declarative request for data into an imperative sequence of physical operations. Because declarative languages (such as SQL) specify *what* data is required rather than *how* to retrieve it, the optimization layer is necessary to evaluate the vast search space of potential execution strategies and select the most efficient path based on resource constraints and data distribution.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* The lifecycle of a query from parsing to execution plan generation.
* Theoretical frameworks for cost-based and rule-based optimization.
* Logical vs. physical plan transformations.
* Cardinality estimation and the role of data statistics.
* Common join algorithms and access methods.

**Out of scope:**
* Specific syntax for vendor-specific "EXPLAIN" commands.
* Hardware-level tuning (e.g., BIOS settings, disk RAID configurations).
* Application-level caching strategies.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Access Path** | The method used by the engine to retrieve data from a specific table or index (e.g., Sequential Scan, Index Seek). |
| **Cardinality** | The estimated or actual number of rows returned by a specific operation or query fragment. |
| **Cost Model** | A mathematical framework used by the optimizer to assign a numeric value to an execution plan based on estimated I/O, CPU, and memory usage. |
| **Heuristics** | Rule-based optimization techniques applied regardless of data distribution (e.g., "always perform projections as early as possible"). |
| **Join Reordering** | The process of determining the most efficient sequence in which to combine multiple relations to minimize intermediate result sets. |
| **SARGable** | (Search Argumentable) A condition in a query that allows the engine to utilize an index effectively. |
| **Selectivity** | A measure of how many rows a filter or predicate will return relative to the total number of rows (expressed as a fraction or percentage). |
| **Search Space** | The set of all possible valid execution plans for a given query. |

## Core Concepts

### The Optimization Pipeline
The transformation of a query typically follows a linear progression:
1.  **Parsing:** Validating syntax and converting the query into a parse tree.
2.  **Binding/Normalization:** Resolving object names (tables, columns) and checking permissions.
3.  **Rewriting:** Applying logical transformations (e.g., flattening subqueries, constant folding).
4.  **Optimization:** Generating and costing multiple physical plans to select the "cheapest" candidate.
5.  **Execution:** Converting the chosen plan into machine-executable instructions.

### Logical vs. Physical Plans
*   **Logical Plan:** An algebraic representation of the query (Relational Algebra). It describes *what* operations are performed (e.g., Join, Filter, Project) without specifying the algorithm.
*   **Physical Plan:** An actionable blueprint that specifies the exact algorithms used (e.g., Hash Join, Index Scan) and the order of execution.

### Cost-Based Optimization (CBO)
Modern optimizers primarily use CBO. The engine uses metadata (statistics) about the data—such as histograms, null counts, and distinct values—to predict the cost of various paths. The goal is not necessarily to find the *perfect* plan, but to avoid a catastrophic plan and find one that is "good enough" within a reasonable optimization time window.

## Standard Model

### The Volcano/Iterator Model
The standard model for query execution is the **Volcano Iterator Model**. In this model, each operator in the execution plan implements a standard interface (usually `Open()`, `Next()`, and `Close()`). 
*   Operators are arranged in a tree.
*   Data flows upward from the leaves (data sources) to the root (the final result).
*   Each `Next()` call pulls a single tuple (or a batch of tuples) from the child operator, allowing for pipelined execution and reduced memory overhead.

### Join Algorithms
The standard model recognizes three primary physical join strategies:
1.  **Nested Loop Join:** For every row in the outer table, scan the inner table. Best for small datasets or when the inner side is indexed.
2.  **Merge Join:** Requires both inputs to be sorted on the join key. It steps through both inputs simultaneously. Highly efficient for large, sorted datasets.
3.  **Hash Join:** Builds a hash table of the smaller input in memory and probes it with the larger input. Ideal for large, unsorted datasets where memory is sufficient.

## Common Patterns

### Predicate Pushdown
Moving filters as close to the data source as possible. By filtering rows early, the engine reduces the volume of data that must be processed by subsequent joins and aggregations.

### Projection Pruning
Selecting only the columns required for the final output or intermediate calculations. This reduces I/O (especially in columnar formats) and memory consumption.

### Index-Only Access
A pattern where the query can be satisfied entirely by the data stored within an index, bypassing the need to fetch the full row from the primary data storage (the "heap" or "clustered index").

## Anti-Patterns

### Non-SARGable Predicates
Applying functions or calculations to indexed columns in a `WHERE` clause (e.g., `WHERE YEAR(date_column) = 2023`). This prevents the optimizer from using an index seek, forcing a full scan.

### Over-Hinting
Manually forcing the optimizer to use a specific index or join type. While useful for immediate fixes, hints can become technical debt as data distributions change, preventing the optimizer from adapting to new realities.

### Cartesian Products (Cross Joins)
Joining tables without a join condition. This results in an exponential explosion of the result set ($N \times M$), often leading to resource exhaustion.

### Stale Statistics
Failing to update the metadata used by the optimizer. If the optimizer believes a table has 10 rows when it actually has 10 million, it will likely choose an inappropriate execution plan.

## Edge Cases

### Correlation and Skew
Standard optimizers often assume data is uniformly distributed and that columns are independent. **Data Skew** (e.g., 90% of orders belonging to one "Guest" ID) or **Column Correlation** (e.g., City="Seattle" and Zip="98101") can lead to significant cardinality estimation errors.

### Combinatorial Explosion in Join Reordering
As the number of tables in a join increases, the number of possible join orders grows factorially. For queries involving a large number of tables (typically >12), optimizers may switch from exhaustive search to genetic algorithms or heuristic-based pruning to keep optimization time under control.

### Parameter Sniffing
In systems using prepared statements or stored procedures, the optimizer may generate a plan based on the parameters provided during the first compilation. If subsequent executions use parameters with vastly different selectivities, the cached plan may be highly inefficient.

## Related Topics
*   **012 Relational Algebra:** The mathematical foundation for logical query plans.
*   **045 Indexing Strategies:** The physical structures that provide access paths.
*   **088 Distributed Query Processing:** Optimization across multiple physical nodes.
*   **102 Transaction Isolation:** How locking and versioning affect plan selection.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
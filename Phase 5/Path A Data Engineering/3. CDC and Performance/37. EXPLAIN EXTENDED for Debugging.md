# 037 EXPLAIN EXTENDED for Debugging

Canonical documentation for 037 EXPLAIN EXTENDED for Debugging. This document defines concepts, terminology, and standard usage.

## Purpose
The 037 EXPLAIN EXTENDED protocol exists to provide deep transparency into the database query optimizer's decision-making process. While standard execution plans outline the basic path of data retrieval, the "Extended" diagnostic layer reveals the internal transformations, cost estimations, and filtering statistics that occur before and during execution.

This topic addresses the "Black Box" problem in query processing, where a developer's intent (the declarative SQL) is decoupled from the engine's action (the imperative execution). It serves as the primary diagnostic tool for identifying performance bottlenecks, incorrect index selection, and unexpected query rewrites.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative, focusing on the theoretical framework of extended query diagnostics.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Optimizer Transformations:** Documentation of how the engine rewrites queries for efficiency.
* **Selectivity and Filtering:** Analysis of row-set reduction at various stages of the execution pipeline.
* **Diagnostic Metadata:** The interpretation of warnings and auxiliary information generated during the optimization phase.
* **Execution Plan Validation:** Comparing predicted costs against actual data distribution.

**Out of scope:**
* **Vendor-specific Syntax:** Specific command variations (e.g., `EXPLAIN ANALYZE` vs. `EXPLAIN EXTENDED`).
* **Hardware-level Debugging:** CPU/RAM hardware interrupts or disk I/O latency outside the context of the query plan.
* **Application-level Code:** Debugging the ORM or client-side logic that generates the SQL.

## Definitions
| Term | Definition |
|------|------------|
| **Query Execution Plan (QEP)** | The sequence of operations used by the database engine to access data and produce a result set. |
| **Optimizer** | The internal component that evaluates multiple potential QEPs and selects the one with the lowest estimated cost. |
| **Rewritten Query** | The canonical, transformed version of a SQL statement after the optimizer has applied rules like constant folding or subquery flattening. |
| **Selectivity** | A measure of how much a predicate (WHERE clause) filters a dataset; high selectivity returns few rows. |
| **Filtered Percentage** | An estimate of the percentage of rows that will be filtered by a condition not covered by an index. |
| **Cost Model** | The mathematical framework used to assign "weights" to operations (e.g., sequential scan vs. index seek). |

## Core Concepts

### 1. The Transformation Phase
Before execution, the optimizer transforms the user's SQL into a more efficient internal representation. The 037 EXPLAIN EXTENDED protocol captures this "Rewritten Query." This is critical for debugging because it reveals if the optimizer has converted a `SUBQUERY` into a `JOIN` or if it has eliminated "dead code" (predicates that are always true/false).

### 2. Predictive Filtering
Extended diagnostics provide a `filtered` metric. This represents the estimated percentage of rows that will be filtered by the table condition. A low percentage indicates that the engine is fetching many rows only to discard them later, signaling a need for better indexing or more specific predicates.

### 3. Diagnostic Warnings
The "Extended" state triggers the generation of auxiliary messages. These messages often contain information about:
* Implicit type conversions (which can disable index usage).
* Deprecated syntax.
* Optimization notes (e.g., "Impossible WHERE noticed after reading const tables").

## Standard Model

The standard model for 037 EXPLAIN EXTENDED follows a three-stage diagnostic workflow:

1.  **Request Generation:** The SQL statement is submitted with the extended diagnostic flag.
2.  **Plan Generation & Transformation:** The optimizer parses the statement, applies algebraic transformations, and calculates costs based on available statistics.
3.  **Metadata Retrieval:** The system returns the tabular execution plan along with a secondary "Note" or "Warning" result set containing the rewritten query and optimization remarks.

### The Diagnostic Output Structure
The output must provide:
* **Access Type:** How the table is accessed (e.g., All, Index, Range, Ref).
* **Key Usage:** Which index was chosen and why.
* **Row Estimates:** The number of rows the optimizer expects to examine.
* **Filtering Estimates:** The percentage of rows remaining after the filter is applied.

## Common Patterns

### The "Full Table Scan" Identification
Using extended diagnostics to identify where the `type` is `ALL` and the `filtered` percentage is low. This pattern indicates that the engine is scanning the entire dataset but only utilizing a small fraction of it.

### Subquery Flattening
Observing the rewritten query to see if the optimizer successfully converted a dependent subquery into a semi-join. This is a common pattern for verifying that the engine is handling complex logic efficiently.

### Index Merge
Identifying when the optimizer decides to use multiple indexes for a single table and intersect or union the results.

## Anti-Patterns

*   **Ignoring the Rewritten Query:** Focusing only on the tabular output without reviewing how the optimizer modified the SQL logic.
*   **Stale Statistics Reliance:** Performing extended debugging on a system with outdated table statistics, leading to "hallucinated" execution plans that do not reflect reality.
*   **Over-Optimization for Small Sets:** Applying complex indexing strategies based on EXPLAIN output for tables with fewer than 1,000 rows, where a full table scan is often more efficient.
*   **Type Mismatch Neglect:** Ignoring warnings about implicit casting (e.g., comparing a string column to an integer), which often causes the optimizer to ignore valid indexes.

## Edge Cases

*   **Zero-Row Tables:** When a table is empty, the optimizer may produce a "const" or "impossible" plan that does not reflect how the query will perform once the table is populated.
*   **Highly Correlated Data:** Statistics may suggest high selectivity, but if data is heavily correlated, the `filtered` estimate may be wildly inaccurate.
*   **Virtual/Generated Columns:** Debugging queries against virtual columns requires checking if the optimizer can "see through" the expression to an underlying index.
*   **Partition Pruning:** In partitioned environments, the extended plan must be checked to ensure the optimizer is only accessing relevant partitions (Pruning).

## Related Topics
*   **Cost-Based Optimization (CBO):** The underlying theory of how databases choose plans.
*   **Index Cardinality:** The uniqueness of data in a column, which dictates optimizer choices.
*   **Predicate Pushdown:** The optimization where filtering is performed as early as possible in the data pipeline.
*   **Join Buffer Management:** How the engine handles memory during multi-table operations.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
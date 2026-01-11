# 013 Adaptive Query Execution Deep Dive

Canonical documentation for 013 Adaptive Query Execution Deep Dive. This document defines concepts, terminology, and standard usage.

## Purpose
Adaptive Query Execution (AQE) exists to bridge the gap between static query optimization and the unpredictable nature of data at scale. Traditional query optimizers rely on pre-execution statistics (cardinality, histograms) which are often stale, inaccurate, or non-existent. 

AQE addresses the "optimization-execution disconnect" by enabling the query engine to re-optimize and adjust the physical execution plan mid-stream, based on actual runtime statistics gathered during the execution of preceding query stages. This ensures that the execution strategy remains optimal even when initial estimates are incorrect.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative, focusing on the architectural patterns of dynamic optimization rather than specific software configurations.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* The lifecycle of a dynamic query plan.
* Mechanisms for runtime statistics collection.
* Re-optimization triggers (partitioning, join strategies, skew handling).
* The relationship between query stages and materialization boundaries.

**Out of scope:**
* Specific vendor-specific configuration parameters (e.g., Spark-specific `spark.sql.adaptive.enabled`).
* Hardware-level optimizations (CPU/GPU instruction sets).
* Static cost-based optimization (CBO) theory, except where it serves as the input for AQE.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Query Stage** | A segment of a query plan that can be executed independently until a data exchange (shuffle) boundary is reached. |
| **Materialization Point** | A boundary where data is written to intermediate storage (memory or disk), allowing the engine to calculate exact statistics before proceeding. |
| **Shuffle/Exchange** | The process of redistributing data across a cluster, serving as the primary trigger point for AQE re-evaluation. |
| **Data Skew** | A condition where a disproportionately large amount of data is concentrated in a small number of partitions, leading to "straggler" tasks. |
| **Runtime Statistics** | Actual metrics (size in bytes, row count) collected during the execution of a query stage. |
| **Partition Coalescing** | The process of merging small, fragmented partitions into larger, more efficient units based on runtime size. |

## Core Concepts

### The Feedback Loop
The fundamental principle of AQE is the transition from a linear "Plan -> Execute" model to a cyclical "Plan -> Execute Stage -> Observe -> Re-plan" model. 

1.  **Observation:** As a query stage completes, the engine captures the exact size and distribution of the output data.
2.  **Analysis:** The engine compares these actuals against the original estimates.
3.  **Adjustment:** If the delta exceeds a defined threshold, the engine modifies the downstream physical plan.

### Materialization Boundaries
AQE relies on the concept of "blocking" operations. In distributed systems, a shuffle (or exchange) acts as a natural barrier. Because the engine must wait for all map-side tasks to complete before the reduce-side can begin, this pause provides a "window of opportunity" to inspect the data and change the plan for the reduce-side.

## Standard Model

The standard model for AQE involves three primary optimization pillars:

### 1. Dynamically Coalescing Shuffle Partitions
Static plans often use a fixed number of partitions. If the data is small, this leads to excessive overhead (too many small tasks). If the data is large, it leads to resource exhaustion. AQE observes the shuffle output and merges small partitions into a target size, ensuring optimal task granularity.

### 2. Dynamically Switching Join Strategies
A static optimizer might choose a Sort-Merge Join based on an estimated table size. If AQE discovers at runtime that one side of the join is small enough to fit in memory, it can dynamically convert the Sort-Merge Join into a Replicated (Broadcast) Join, eliminating the need for a costly shuffle of the larger table.

### 3. Dynamically Handling Skew Joins
When data skew is detected at a materialization point (e.g., one partition is 10x larger than the average), AQE splits the skewed partition into smaller sub-partitions. These sub-partitions are then processed in parallel and joined against the corresponding data from the other side, preventing a single task from bottlenecking the entire stage.

## Common Patterns

*   **The "Small Table" Discovery:** A query joins a filtered fact table with a dimension table. The filter is highly selective, but the optimizer doesn't know this. AQE detects the small post-filter size and converts the join to a broadcast.
*   **Post-Aggregation Compression:** After a `GROUP BY` operation, the resulting data is significantly smaller than the input. AQE coalesces the output partitions to ensure the subsequent `ORDER BY` or `JOIN` doesn't launch thousands of empty tasks.
*   **Iterative Refinement:** In complex multi-stage DAGs (Directed Acyclic Graphs), AQE applies optimizations at every shuffle boundary, compounding the performance gains as the query progresses.

## Anti-Patterns

*   **Over-Instrumentation:** Collecting too many granular statistics (e.g., full histograms for every column) during execution can introduce more latency than the optimization saves.
*   **Premature Materialization:** Forcing shuffle boundaries solely to trigger AQE in scenarios where the data is already well-distributed and statistics are known.
*   **Ignoring Initial Statistics:** Relying entirely on AQE and neglecting the Cost-Based Optimizer (CBO). AQE is a corrective measure, not a replacement for a good initial plan.
*   **Small Data Overhead:** Enabling aggressive AQE on sub-second queries where the planning overhead exceeds the execution time.

## Edge Cases

*   **Empty Partitions:** When a filter results in zero rows, AQE must handle the "empty set" gracefully without crashing downstream joins or aggregations.
*   **UDF Uncertainty:** User-Defined Functions (UDFs) often act as "black boxes." AQE is particularly valuable here because it can measure the output of a UDF where a static optimizer would have to guess.
*   **Broadcast Timeouts:** If AQE decides to switch to a broadcast join, but the "small" table is still large enough to cause network congestion or memory pressure, it may trigger a timeout or Out-Of-Memory (OOM) error.
*   **Non-Deterministic Functions:** If a query uses non-deterministic functions (e.g., `current_timestamp()`), re-planning mid-query must ensure consistency across different stages.

## Related Topics

*   **Cost-Based Optimization (CBO):** The static precursor to AQE.
*   **Shuffle Service Architecture:** The underlying mechanism that enables data redistribution.
*   **Vectorized Execution:** A complementary technique for improving per-node processing speed.
*   **Data Skipping and Pruning:** Techniques used in conjunction with AQE to reduce the initial data volume.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
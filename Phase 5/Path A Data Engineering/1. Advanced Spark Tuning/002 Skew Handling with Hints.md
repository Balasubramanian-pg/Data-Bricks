# 002 Skew Handling with Hints

Canonical documentation for 002 Skew Handling with Hints. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of Skew Handling with Hints is to provide a mechanism for manual intervention in distributed data processing when data distribution is non-uniform. In distributed systems, data is partitioned across multiple nodes; when a specific key or set of keys contains significantly more records than others, it creates a "skew." This results in "straggler" tasks that extend the total execution time of a job, as the system must wait for the overloaded node to complete. 

Hints allow the practitioner to inform the query optimizer or execution engine about known imbalances that the engine's automated statistics or heuristics may have failed to detect or mitigate. This section addresses the problem of resource underutilization and execution bottlenecks caused by data variance.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Theoretical foundations of data skew in distributed joins and aggregations.
* The logic of hint-driven execution plan modification.
* Strategies for rebalancing skewed partitions (e.g., sub-partitioning, salting).
* Metadata requirements for effective skew hinting.

**Out of scope:**
* Specific SQL syntax or API calls for individual vendors (e.g., Spark, Hive, Oracle, Snowflake).
* Automatic skew detection algorithms (Adaptive Query Execution).
* Hardware-level load balancing.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Data Skew** | A condition where data is distributed unevenly across partitions, leading to a disproportionate amount of work for a subset of processing nodes. |
| **Hint** | A directive or metadata provided to the execution engine to override or influence the default execution plan. |
| **Straggler** | A task within a distributed job that takes significantly longer to complete than others due to the volume of data it must process. |
| **Salting** | A technique where a random prefix or suffix is added to a skewed key to force it into multiple different partitions. |
| **Sub-partitioning** | The process of further dividing a single logical partition into smaller physical units to parallelize processing. |
| **Broadcast Join** | A join strategy where a small table is replicated to all nodes to avoid shuffling the larger, potentially skewed table. |

## Core Concepts
### The Skew Problem
In a distributed "Shuffle-and-Sort" or "Hash-Join" architecture, the engine determines the destination node for a record based on the hash of its join/grouping key. If 90% of the records share the same key, 90% of the data will be sent to a single node. This negates the benefits of parallelism.

### Hint-Driven Optimization
While modern engines attempt to detect skew dynamically, they often rely on stale or coarse-grained statistics. Hints serve as a "Source of Truth" provided by the developer, who has domain knowledge of the data distribution. A hint typically triggers a specific transformation in the logical plan, such as:
1.  **Splitting the Skewed Key:** Identifying specific values to be handled separately.
2.  **Replication:** Replicating the matching rows from the non-skewed side of a join to all partitions where the skewed key exists.

## Standard Model
The standard model for Skew Handling with Hints involves three primary components:

1.  **Identification:** The user identifies the skewed relation (table) and, optionally, the specific column(s) and values causing the skew.
2.  **Directive Injection:** The user attaches a hint to the query or transformation. This hint specifies the target (e.g., Table A) and the skewed values (e.g., "ID: 001, 999").
3.  **Execution Plan Alteration:**
    *   The engine reads the hint.
    *   For the skewed values, the engine ignores the standard hash-partitioning.
    *   The engine "salts" the skewed keys on the left side and "replicates" the corresponding keys on the right side (or vice versa) to ensure a correct join result across multiple nodes.

## Common Patterns
### Key-Specific Hinting
The most common pattern where the user explicitly lists the values known to be skewed. This is highly effective for "null" values or default IDs that appear frequently.

### Table-Level Hinting
When the specific keys are unknown or too numerous to list, a table-level hint instructs the engine to apply skew-mitigation logic (like increased sub-partitioning) to the entire join operation for that table.

### Partial Broadcast
A pattern where only the skewed portion of a dataset is handled via a broadcast-like mechanism, while the remainder of the data is processed using standard shuffle-joins.

## Anti-Patterns
*   **Over-Hinting:** Applying skew hints to every join regardless of data distribution. This introduces unnecessary overhead and complexity into the execution plan.
*   **Hinting Non-Skewed Data:** Providing hints for keys that are actually uniformly distributed can lead to "negative scaling," where the overhead of the skew-handling logic exceeds the processing time of a standard join.
*   **Static Hinting on Dynamic Data:** Hardcoding skewed values in a hint when the data distribution changes frequently. This leads to "hint rot," where the optimization becomes obsolete or detrimental as new skewed keys emerge.
*   **Ignoring Statistics:** Using hints as a substitute for maintaining up-to-date table statistics. Hints should be a secondary measure when statistics are insufficient.

## Edge Cases
*   **Multi-Key Skew:** When a join is performed on multiple columns, and the skew only exists in the *combination* of those columns. Hints must be able to address composite keys.
*   **Cross-Join Skew:** Skew occurring in a Cartesian product, which can lead to exponential data explosion. Standard salting techniques may not suffice.
*   **Skew in Aggregations:** While most hints focus on joins, skew in `GROUP BY` operations requires different handling, such as two-stage aggregation (partial aggregation followed by final aggregation).
*   **Memory Pressure:** If the "non-skewed" side of a join is too large to be replicated/broadcasted to handle the skewed keys, the system may encounter Out-of-Memory (OOM) errors.

## Related Topics
*   **001 Data Partitioning Strategies:** The foundational method of distributing data.
*   **003 Adaptive Query Execution (AQE):** The automated counterpart to manual skew hinting.
*   **004 Join Strategies:** Understanding Hash, Sort-Merge, and Broadcast joins is essential for applying skew hints correctly.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
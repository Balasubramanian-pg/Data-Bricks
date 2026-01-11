# 003 Salting for Skew Mitigation

Canonical documentation for 003 Salting for Skew Mitigation. This document defines concepts, terminology, and standard usage.

## Purpose
Salting for skew mitigation addresses the "Hot Key" problem inherent in distributed computing and storage systems. In parallel processing environments, data is typically partitioned across multiple nodes based on a hash of a specific key. When the distribution of these keys is non-uniform (data skew), a disproportionate volume of data or processing load is assigned to a single partition. This results in "stragglers"—nodes that take significantly longer to complete their tasks than others—thereby bottlenecking the entire system and limiting the benefits of horizontal scaling.

Salting introduces artificial entropy into the partitioning key to force a more uniform distribution of data across the available processing units.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* The mathematical and logical foundations of key salting.
* Strategies for mitigating skew in join operations and aggregations.
* Impact of salting on data cardinality and partition distribution.
* Theoretical trade-offs between distribution uniformity and computational overhead.

**Out of scope:**
* Specific syntax for SQL engines (e.g., Hive, Spark SQL, BigQuery).
* Vendor-specific "auto-skew" features or proprietary optimization algorithms.
* Hardware-level load balancing (L4/L7).

## Definitions
| Term | Definition |
|------|------------|
| **Data Skew** | A condition where data is distributed unevenly across partitions, leading to resource imbalance. |
| **Hot Key** | A specific key value that appears with significantly higher frequency than others in a dataset. |
| **Salt** | A random or deterministic suffix or prefix added to a key to alter its hash value. |
| **Salt Range** | The set of possible values (e.g., 0 to N-1) from which a salt is selected. |
| **Fan-out** | The process of replicating a smaller dataset to match the salted keys of a larger, skewed dataset during a join. |
| **Partitioning Key** | The attribute used by a distribution function to determine the destination node for a record. |

## Core Concepts

### The Mechanism of Distribution
In distributed systems, the destination partition $P$ for a record is typically determined by:
$P = \text{hash}(\text{key}) \mod \text{number\_of\_partitions}$

When a key is "hot," the hash function consistently maps it to the same partition. Salting modifies the input:
$P_{salted} = \text{hash}(\text{key} + \text{salt}) \mod \text{number\_of\_partitions}$

### Entropy Injection
By appending a salt, a single logical key is transformed into $N$ physical keys. This allows the system to distribute the processing of a single logical key across $N$ different workers.

### The Cost of Salting
Salting is not a "free" operation. It increases the complexity of the execution plan and, in the case of joins, requires the replication of data (fan-out) on the opposing side of the operation to ensure correctness.

## Standard Model

The standard model for salting involves three distinct phases:

1.  **Salt Generation:** A salt value is generated based on a defined range $[0, N-1]$. For aggregations, this is often a random integer. For joins, it is often a deterministic value or a cross-product.
2.  **Key Transformation:** The original key $K$ is transformed into $K'$.
3.  **Operation Execution:**
    *   **For Aggregations:** Perform a partial aggregation on the salted key, then a final aggregation on the original key (de-salting).
    *   **For Joins:** The skewed "left" table is salted. The "right" table (usually the smaller dimension) is replicated $N$ times, with each replica receiving one of the possible salt values to ensure all potential matches are covered.

## Common Patterns

### 1. Random Salting (Aggregation Pattern)
Used when performing `GROUP BY` operations on skewed data. Each record for a hot key is assigned a random salt.
*   **Step A:** Group by `Key + RandomSalt`.
*   **Step B:** Group the results of Step A by `Key` to produce the final result.
*   **Benefit:** Parallelizes the initial heavy lifting of the aggregation.

### 2. Deterministic Fan-out (Join Pattern)
Used when joining a large, skewed fact table with a smaller dimension table.
*   **Fact Table:** Append a random salt $s \in [0, N-1]$ to the join key.
*   **Dimension Table:** Explode (cross-join) each row by the entire range of salts $[0, N-1]$.
*   **Join:** Perform the join on the salted keys.
*   **Benefit:** Prevents a single join-node from being overwhelmed by the hot key.

### 3. Conditional Salting
A more advanced pattern where only known hot keys are salted, while uniform keys remain untouched. This minimizes the overhead of data replication for keys that do not require skew mitigation.

## Anti-Patterns

*   **Over-Salting:** Setting the salt range $N$ significantly higher than the number of available processing cores. This creates excessive small tasks and metadata overhead without improving parallelism.
*   **Salting Uniform Data:** Applying salting to keys that are already evenly distributed. This introduces unnecessary computational complexity and data inflation (in joins) with no performance gain.
*   **Static Salt Ranges:** Using a hardcoded salt range that does not account for future data growth. As the volume of a hot key grows, a once-sufficient salt range may become a new bottleneck.
*   **Ignoring Memory Limits:** In join fan-outs, replicating the dimension table $N$ times can lead to Out-of-Memory (OOM) errors if the dimension table is not sufficiently small.

## Edge Cases

*   **Skew on Both Sides:** If both tables in a join operation are heavily skewed on the same key, simple salting and fan-out may be insufficient. This often requires a "skew-join" strategy where both sides are salted and handled via a complex matrix-style join.
*   **Low Cardinality Keys:** If the total number of unique keys is less than the number of partitions, salting is mandatory to utilize the full cluster, even if the data isn't technically "skewed" in the traditional sense.
*   **Non-Integer Salts:** While integers are standard, using strings or binary salts can lead to unexpected hash collisions depending on the underlying distribution algorithm.

## Related Topics
*   **001 Partitioning Strategies:** The foundational method of dividing data.
*   **002 Broadcast Joins:** An alternative to salting when one side of a join is small enough to fit in memory.
*   **008 Combiner Functions:** Used in map-reduce frameworks to perform partial aggregations before the shuffle phase.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
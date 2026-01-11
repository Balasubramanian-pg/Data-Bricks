# 004 Repartitioning Strategies

Canonical documentation for 004 Repartitioning Strategies. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of repartitioning strategies is to manage the distribution of data across a distributed system to ensure optimal resource utilization, minimize computational bottlenecks, and maintain system stability. In distributed computing and database management, data is divided into discrete units called partitions. As data grows or access patterns change, the initial distribution often becomes inefficient. Repartitioning addresses the problem of "data skew" (uneven distribution) and "resource starvation" by re-aligning data boundaries with the underlying physical or logical compute resources.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Logical and physical redistribution of data sets.
* Algorithmic approaches to data placement.
* Performance trade-offs between different distribution models.
* Impact of repartitioning on downstream processing (joins, aggregations).

**Out of scope:**
* Specific syntax for vendor-specific frameworks (e.g., Apache Spark, Kafka, or Snowflake).
* Hardware-level disk partitioning or filesystem-level formatting.
* Network protocol specifications for data transfer.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Partition** | A logical subset of a larger dataset, typically processed as a single unit of work. |
| **Data Skew** | A condition where data is unevenly distributed across partitions, leading to "hot spots" where one node does significantly more work than others. |
| **Shuffle** | The process of redistributing data across different partitions, often involving network I/O to move data between physical nodes. |
| **Cardinality** | The uniqueness of data values in a column; high cardinality is often required for effective repartitioning. |
| **Fan-out** | The process of taking data from one partition and distributing it to many, or the ratio of input to output partitions. |
| **Locality** | The principle of keeping data and the computation that requires it on the same physical resource to minimize latency. |

## Core Concepts
The fundamental goal of repartitioning is to achieve **Maximum Parallelism** while minimizing **Overhead**.

1.  **Parallelism vs. Overhead:** Increasing the number of partitions allows more tasks to run simultaneously. However, every partition introduces metadata overhead and scheduling latency. The optimal strategy finds the "Goldilocks zone" where the cost of the shuffle is outweighed by the gain in processing speed.
2.  **Deterministic vs. Non-deterministic:** Some strategies ensure that a specific record always ends up in a specific partition (essential for joins), while others prioritize even distribution without regard for record content.
3.  **The Shuffle Boundary:** Repartitioning almost always triggers a shuffle. This is the most expensive operation in distributed systems because it involves serializing data, moving it over the network, and deserializing it on a new node.

## Standard Model
The standard model for repartitioning follows a **Map-Shuffle-Reduce** flow:
1.  **Identification:** The system evaluates the current distribution and determines the target number of partitions ($N$).
2.  **Mapping Function:** A function $f(x)$ is applied to each record to determine its destination partition index $i$, where $0 \le i < N$.
3.  **Exchange:** Data is buffered and transmitted to the nodes responsible for the new partition indices.
4.  **Materialization:** The new partitions are written to memory or disk for the next stage of computation.

## Common Patterns

### 1. Hash-Based Repartitioning
A hash function is applied to a specific key (or set of keys) within the data. 
*   **Usage:** Used when subsequent operations require records with the same key to be co-located (e.g., Group-By or Joins).
*   **Pros:** Deterministic; ensures data affinity.
*   **Cons:** Can lead to skew if the key distribution is not uniform.

### 2. Round-Robin Repartitioning
Records are distributed across partitions in a rotating sequence, regardless of their content.
*   **Usage:** Used for simple load balancing when no specific key-based grouping is required.
*   **Pros:** Guarantees nearly perfectly equal partition sizes.
*   **Cons:** Destroys any existing data locality or ordering.

### 3. Range-Based Repartitioning
Data is partitioned based on the sorted range of a specific value (e.g., Date ranges or Alphabetical ranges).
*   **Usage:** Ideal for range-based queries or when the output must be globally sorted.
*   **Pros:** Efficient for filtering specific segments of data.
*   **Cons:** Requires a "sampling" phase to determine range boundaries to avoid skew.

### 4. Broadcast (Small-to-Large)
Technically a form of redistribution where a small dataset is copied to every single node processing a larger dataset.
*   **Usage:** Used to avoid shuffing a massive table by instead moving a tiny table to all locations.
*   **Pros:** Eliminates the need to shuffle the large table.
*   **Cons:** Limited by the memory capacity of the individual nodes.

## Anti-Patterns

*   **Over-Partitioning:** Creating thousands of tiny partitions for a small dataset. This leads to the "Small File Problem," where the overhead of managing partitions exceeds the time spent processing data.
*   **Fixed Partitioning:** Hard-coding partition counts in an environment with fluctuating data volumes. This leads to under-utilization during low volume and system crashes during high volume.
*   **Partitioning on Low-Cardinality Keys:** Repartitioning by a "Gender" or "Boolean" column. This results in only two active partitions regardless of how many nodes are available, creating massive bottlenecks.
*   **Unnecessary Shuffling:** Triggering a repartition immediately before another operation that will trigger its own repartition (e.g., repartitioning by Key A and then immediately joining on Key B).

## Edge Cases

*   **The "Null" Sink:** In hash-based repartitioning, null values often hash to the same value. If a dataset has a high percentage of nulls, one partition will become a "hot spot" containing all null records.
*   **Empty Partitions:** In range-based partitioning, if the sampling is inaccurate, the system may create partitions that contain zero records, wasting scheduling resources.
*   **Monolithic Keys:** A single key (e.g., a "Default" ID) that represents 50% of the data. No repartitioning strategy based on that key can ever balance the load; such cases require "salting" (adding a random suffix to the key to break it up).

## Related Topics
*   **001 Data Sharding:** The physical storage equivalent of repartitioning.
*   **015 Distributed Joins:** Operations that are heavily dependent on repartitioning strategies.
*   **022 Load Balancing:** The broader architectural concept of distributing work.
*   **030 Consistent Hashing:** A specific type of hashing used to minimize redistribution when nodes are added or removed.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
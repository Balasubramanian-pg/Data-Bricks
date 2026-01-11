# 008 Broadcast Hash Joins Deep Dive

Canonical documentation for 008 Broadcast Hash Joins Deep Dive. This document defines concepts, terminology, and standard usage.

## Purpose
The Broadcast Hash Join (BHJ) exists to optimize join operations in distributed computing environments by eliminating the need for an all-to-all data shuffle. In large-scale data processing, joining two distributed datasets typically requires repartitioning both datasets across the network based on the join key (a Shuffle Hash Join). This process is resource-intensive and often becomes a bottleneck.

The BHJ addresses this by leveraging the asymmetry in dataset sizes. By distributing the entirety of a smaller dataset to every compute node, the system can perform the join locally against the partitions of the larger dataset, significantly reducing network latency and I/O overhead.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative. While various distributed engines (e.g., Spark, Flink, Trino) implement this pattern, the underlying logic remains consistent across the industry.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Mechanics of the Build and Probe phases.
* Data distribution requirements and constraints.
* Performance trade-offs regarding memory and network bandwidth.
* Logical requirements for applicability.

**Out of scope:**
* Specific vendor-specific configuration parameters (e.g., `spark.sql.autoBroadcastJoinThreshold`).
* Non-hash-based join strategies (e.g., Sort-Merge Join, Nested Loop Join).
* Hardware-specific optimizations (e.g., GPU-accelerated joins).

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Build Side** | The smaller dataset that is collected and transformed into an in-memory hash table. |
| **Probe Side** | The larger dataset that remains partitioned across the cluster and is iterated over to find matches in the hash table. |
| **Broadcast** | The act of sending a complete copy of a dataset to every worker node in a distributed cluster. |
| **Hash Table** | A data structure that maps keys to values, used on the Build Side to provide $O(1)$ average time complexity for lookups. |
| **Shuffle** | The process of redistributing data across a network so that records with the same key end up on the same node. |
| **Driver/Coordinator** | The central node responsible for collecting the Build Side data before broadcasting it to workers. |

## Core Concepts

### The Asymmetry Principle
The fundamental requirement for a BHJ is a significant size disparity between the two datasets. The "Small" side must be small enough to fit entirely within the memory of a single worker node (and often the coordinator node).

### The Two-Phase Execution
1.  **Build Phase:** The system scans the smaller dataset, collects it to a central point (or replicates it), and distributes it to all active workers. Each worker builds an internal hash table where the join key is the hash key.
2.  **Probe Phase:** Each worker scans its local partition of the "Large" dataset. For every row in the large dataset, the system calculates the hash of the join key and "probes" the local hash table for a match.

### Data Locality
Unlike Shuffle Joins, which move both datasets to new locations, the BHJ preserves the locality of the Probe Side. Only the Build Side moves. This is particularly effective when the Probe Side is already partitioned or indexed in a way that minimizes disk I/O.

## Standard Model

The standard model for a Broadcast Hash Join follows a specific workflow:

1.  **Requirement Validation:** The optimizer evaluates the size of the Build Side. If it falls below a defined threshold, BHJ is selected.
2.  **Materialization:** The Build Side is fully materialized (computed) and transferred to the Coordinator.
3.  **Serialization and Transfer:** The Coordinator serializes the Build Side data and broadcasts it via a peer-to-peer or star-topology network distribution to all Workers.
4.  **Local Hash Map Creation:** Each Worker deserializes the data and populates an in-memory hash map.
5.  **Streaming Probe:** The Worker streams the Probe Side from its local storage or network partition.
6.  **Join Logic:**
    *   If a match is found in the hash map, the joined row is emitted.
    *   In an **Inner Join**, non-matches are discarded.
    *   In a **Left Outer Join** (where Probe is Left), non-matches are emitted with nulls for the Build Side columns.

## Common Patterns

### Star Schema Joins
In data warehousing, joining a massive "Fact" table with multiple small "Dimension" tables is a classic use case. Each dimension table is broadcast to all nodes, allowing the Fact table to be processed in a single pass without shuffling.

### Filtering and Projection Pushdown
To make a dataset eligible for broadcasting, it is common to apply aggressive filters and column projections (selecting only necessary columns) before the join. This reduces the memory footprint of the resulting hash table.

### Broadcast-Side Deduplication
If the Build Side contains duplicate keys, the hash table must store a list of values for each key. To optimize memory, a common pattern is to aggregate or deduplicate the Build Side prior to the broadcast if the business logic allows.

## Anti-Patterns

### Broadcasting Large Datasets
Attempting to broadcast a dataset that exceeds the available memory of the Worker nodes will result in `OutOfMemory` (OOM) errors. This is the most common failure mode for BHJs.

### The "Double-Sided" Large Join
Using BHJ when both tables are large is an anti-pattern. Even if the system allows it, the network saturation caused by broadcasting a multi-gigabyte table to hundreds of nodes can be more expensive than a standard Shuffle Sort-Merge Join.

### Broadcasting Volatile Streams
In stream processing, broadcasting a side that updates frequently can lead to consistency issues and excessive network churn as the hash table must be rebuilt and re-broadcasted repeatedly.

## Edge Cases

### Empty Build Side
If the Build Side is empty, an **Inner Join** can be optimized to return an empty result immediately without scanning the Probe Side. However, an **Outer Join** must still scan the entire Probe Side to return rows with null-extensions.

### Memory Fragmentation
Even if the Build Side data is technically smaller than the available heap, the overhead of the Hash Map data structure (pointers, object headers) can be 2x-5x the raw data size. This can trigger aggressive Garbage Collection (GC) cycles.

### Skewed Probe Side
While BHJ handles skew in the *Join Key* better than Shuffle Joins (because the Probe Side isn't repartitioned), it does not solve skew in the *Data Distribution*. If one worker holds 90% of the Probe Side data, that worker will still be a bottleneck, even if the hash table is available locally.

## Related Topics
* **Shuffle Hash Joins:** The alternative when both tables are too large to broadcast.
* **Sort-Merge Joins:** A join strategy used for large datasets that avoids hash table memory overhead by pre-sorting data.
* **Cost-Based Optimization (CBO):** The logic used by engines to decide whether to use BHJ based on statistics.
* **Bloom Filters:** Often used in conjunction with joins to skip unnecessary probes.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
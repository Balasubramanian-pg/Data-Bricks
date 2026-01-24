# 033 Data Skipping with Statistics

Canonical documentation for 033 Data Skipping with Statistics. This document defines concepts, terminology, and standard usage.

## Purpose
The primary purpose of Data Skipping with Statistics is to optimize query performance and reduce resource consumption by avoiding the retrieval and processing of irrelevant data. In large-scale distributed systems, I/O (Input/Output) is frequently the primary bottleneck. By leveraging pre-computed metadata about data segments, a query engine can mathematically prove that certain segments do not contain records satisfying a query's predicates, thereby "skipping" those segments entirely.

This mechanism shifts the burden of filtering from the execution phase (reading every row) to the planning phase (evaluating metadata), leading to significant reductions in latency and operational costs.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
**In scope:**
* Core mechanisms of metadata-driven pruning.
* Statistical types used for skipping (Min/Max, Null counts, etc.).
* The relationship between data organization (clustering/sorting) and skipping efficiency.
* Theoretical boundaries of predicate evaluation against metadata.

**Out of scope:**
* Specific vendor implementations (e.g., Snowflake Micro-partitions, Delta Lake Checkpoints, Apache Parquet Footers).
* Indexing structures that require separate pointer-based lookups (e.g., B-Trees).
* Hardware-level skipping (e.g., SSD-level filtering).

## Definitions
| Term | Definition |
|------|------------|
| **Data Segment** | The atomic unit of data storage evaluated for skipping (e.g., a file, a block, or a page). |
| **Metadata** | Statistical summaries stored alongside or within data segments describing the range and distribution of values. |
| **Predicate** | A logical expression (e.g., `WHERE age > 25`) used to filter data. |
| **Pruning** | The act of excluding a data segment from the scan set based on statistical contradiction. |
| **Granularity** | The size of the data segment relative to the total dataset; determines the precision of skipping. |
| **Zone Map** | A common structure for storing min/max values for specific "zones" or segments of a column. |
| **False Positive** | A scenario where metadata suggests a segment *might* contain relevant data, but the actual records do not satisfy the predicate. |

## Core Concepts

### 1. Predicate Pushdown
Data skipping relies on the ability of the query optimizer to "push" filters down to the storage layer. Instead of reading all data and filtering in memory, the engine evaluates the filter against the segment metadata first.

### 2. Statistical Summarization
For every data segment, the system maintains a set of statistics for each column. The most common statistics include:
*   **Minimum Value:** The lowest value in the segment.
*   **Maximum Value:** The highest value in the segment.
*   **Null Count:** The number of empty values, allowing for skipping when predicates involve `IS NOT NULL`.

### 3. Range Contradiction
The fundamental logic of skipping is based on contradiction. If a query seeks `Value X`, and the metadata for `Segment A` shows `Min: 10, Max: 20`, and `X = 25`, the engine can prove `X` is not in `Segment A` without opening the file.

### 4. Data Locality and Clustering
The effectiveness of data skipping is directly proportional to how data is physically organized. If values are randomly distributed, every segment's Min/Max range will likely cover the entire domain of the data, rendering skipping impossible. If data is sorted or clustered, ranges become narrow and non-overlapping, maximizing skipping potential.

## Standard Model
The standard model for Data Skipping with Statistics involves three distinct layers:

1.  **The Metadata Store:** A persistent storage area (often headers within files or a centralized catalog) containing the Min/Max/Null stats for every segment.
2.  **The Optimizer/Planner:** During query compilation, the optimizer compares the query constants against the Metadata Store.
3.  **The Execution Engine:** Receives a "pruned" list of segments. It only initializes I/O streams for segments that could potentially contain the requested data.

## Common Patterns

### Temporal Ordering
Data is naturally ingested and stored by time. Queries filtering on timestamps (e.g., `last 24 hours`) achieve near-perfect skipping because the time ranges of segments do not overlap.

### Z-Ordering / Space-Filling Curves
A technique used to map multi-dimensional data into one dimension while preserving locality. This allows for effective data skipping on multiple columns simultaneously, rather than just the primary sort column.

### Bucket Pruning
Data is distributed into "buckets" based on a hash of a specific column. While different from range-based skipping, it allows the engine to skip all buckets that do not correspond to the hash of the predicate value.

## Anti-Patterns

### High-Cardinality Random Distribution
Storing high-cardinality data (like UUIDs) in a random order. This results in every data segment having a Min/Max range that spans nearly the entire possible value space, forcing a full scan for every query.

### Over-fragmentation (Small File Problem)
Creating segments that are too small. While this increases the precision of skipping, the overhead of managing and reading the metadata for millions of tiny segments outweighs the benefits of skipping the data itself.

### Overlapping Ranges
Frequent updates or poorly managed merges that lead to segments having heavily overlapping Min/Max ranges. This reduces the "discriminatory power" of the statistics.

## Edge Cases

### Null Value Handling
Predicates like `column > 10` often implicitly exclude Nulls. If a segment contains only Nulls, and the engine tracks `Null Count == Total Rows`, the entire segment can be skipped even if the Min/Max values are technically undefined.

### Type Coercion and Collations
If a query compares a string to a numeric value, or uses a different collation (case-sensitivity) than what was used to calculate the Min/Max statistics, the engine must often default to a full scan to ensure correctness, as the statistical boundaries may no longer be valid.

### Schema Evolution
If a column type is changed (e.g., Integer to Long), existing statistics may become incompatible. The system must handle the transition by either re-calculating statistics or disabling skipping for affected segments until they are rewritten.

## Related Topics
*   **012 Columnar Storage:** The physical format that typically enables efficient statistical tracking.
*   **045 Partitioning Strategies:** A higher-level form of data skipping based on directory structures.
*   **088 Bloom Filters:** A probabilistic metadata structure used for set-membership skipping (distinct from range-based skipping).

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
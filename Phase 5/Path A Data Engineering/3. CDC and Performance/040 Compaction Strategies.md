# 040 Compaction Strategies

Canonical documentation for 040 Compaction Strategies. This document defines concepts, terminology, and standard usage.

## Purpose
Compaction strategies address the inherent inefficiencies of append-only and log-structured storage systems. In these systems, data is not updated in place; instead, new versions of data (including deletions) are written as new entries. Over time, this leads to data fragmentation, redundant entries, and increased storage consumption.

Compaction is the background process of merging data segments, resolving duplicate keys, and removing deleted records (tombstones) to optimize read performance and reclaim storage space. This topic defines the logic governing *when* and *how* these data segments are selected for merging.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Theoretical models of data merging and reclamation.
* Trade-offs between Write, Read, and Space amplification (RUM Conjecture).
* Categorization of compaction algorithms.
* Selection criteria for data segments.

**Out of scope:**
* Specific vendor-specific configuration parameters (e.g., specific Cassandra or RocksDB properties).
* Hardware-level optimization (e.g., SSD TRIM commands).
* Memory-level garbage collection (GC) in managed runtimes.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **LSM Tree** | Log-Structured Merge-tree; a data structure optimized for high-throughput writes that relies on periodic compaction. |
| **SSTable** | Sorted String Table; an immutable data file containing sorted key-value pairs. |
| **Tombstone** | A marker placed in a data segment to indicate that a specific key has been deleted. |
| **Write Amplification** | The ratio of total bytes written to the storage device versus the bytes of actual data requested by the application. |
| **Read Amplification** | The number of storage I/O operations required to satisfy a single logical read request. |
| **Space Amplification** | The ratio of the total size of files on disk to the size of the logical data set. |
| **Compaction Window** | The timeframe or threshold-based trigger that initiates a compaction task. |

## Core Concepts

### The RUM Conjecture
Compaction strategies are governed by the RUM Conjecture, which states that storage systems can optimize for only two of the following three overheads at the expense of the third:
1.  **Read Overhead:** Reducing the number of segments checked during a query.
2.  **Update (Write) Overhead:** Reducing the frequency of rewriting data.
3.  **Memory/Space Overhead:** Reducing the footprint of redundant or deleted data.

### Merging Logic
At its core, compaction involves:
*   **Selection:** Identifying a subset of data segments based on size, age, or overlap.
*   **Merging:** Reading the selected segments and performing a multi-way merge-sort.
*   **Resolution:** Keeping only the most recent version of a key and discarding tombstones (if the compaction level allows).
*   **Replacement:** Writing the merged data to a new segment and atomically deleting the old segments.

## Standard Model
The standard model for compaction categorizes strategies based on how they organize data segments into "tiers" or "levels."

### 1. Size-Tiered Compaction (STCS)
Data segments are grouped into tiers based on size. When a tier reaches a threshold number of segments, they are merged into a single larger segment in the next tier.
*   **Primary Benefit:** Low write amplification.
*   **Primary Drawback:** High space amplification and high read amplification (as the same key may exist in multiple tiers).

### 2. Leveled Compaction (LCS)
Data is organized into fixed-size levels. Each level (except Level 0) represents a non-overlapping range of keys. When a level exceeds its capacity, a segment is merged with overlapping segments in the next level.
*   **Primary Benefit:** Low read and space amplification.
*   **Primary Drawback:** High write amplification due to frequent rewriting of data across levels.

### 3. Time-Windowed / Date-Tiered Compaction
Segments are grouped based on the timestamp of the data they contain. Compaction occurs primarily within specific time windows.
*   **Primary Benefit:** Ideal for time-series data where older data is rarely updated or is deleted in bulk.
*   **Primary Drawback:** Inefficient if data is frequently updated across different time ranges.

## Common Patterns

### Major vs. Minor Compaction
*   **Minor Compaction:** Frequent, localized merging of small, newly written segments (often from memory to disk).
*   **Major Compaction:** A comprehensive merge of all data segments into a single (or few) segments. This is the only way to fully guarantee the removal of all expired tombstones.

### Sub-leveling
In leveled strategies, Level 0 often acts as a "buffer" where segments are allowed to overlap (similar to size-tiered) before being promoted to the strictly non-overlapping Level 1.

### Tiered-Leveled Hybrid
A strategy that uses size-tiered logic for the first few levels of data (to handle high write bursts) and leveled logic for the bulk of the data (to optimize long-term read performance).

## Anti-Patterns

*   **Manual-Only Compaction:** Relying solely on manual triggers for compaction. This leads to unpredictable "cliffs" in performance and storage exhaustion.
*   **Tombstone Saturation:** Failing to compact frequently enough to clear tombstones, leading to "tombstone scans" where a read must process thousands of delete markers before finding a live record.
*   **Over-Compaction:** Setting thresholds too low, causing the system to spend more I/O on background compaction than on serving application traffic.
*   **Ignoring Disk Headroom:** Running compaction on a disk that is >70% full. Since compaction requires writing new segments before deleting old ones, it can trigger out-of-disk-space (OOD) errors.

## Edge Cases

*   **The "Oldest Tombstone" Problem:** A tombstone cannot be safely removed until the system is certain that no older versions of the key exist in any other segment. In size-tiered systems, a tombstone in a small, new segment might be blocked from deletion by a very old segment that is rarely compacted.
*   **Skewed Key Distribution:** If a small subset of keys is updated constantly, leveled compaction may result in "heavy" segments that are constantly being rewritten, while the rest of the levels remain static.
*   **Bootstrap/Bulk Load:** During massive data ingestion, standard compaction strategies may struggle to keep up. Specialized "bulk load" modes often bypass standard compaction logic to build the hierarchy in one pass.

## Related Topics
*   **010 LSM Tree Fundamentals:** The underlying data structure that necessitates compaction.
*   **025 Write-Ahead Logs (WAL):** How data is persisted before it enters the compaction-eligible segments.
*   **055 Bloom Filters:** A mechanism used alongside compaction to reduce read amplification by skipping segments that do not contain a key.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
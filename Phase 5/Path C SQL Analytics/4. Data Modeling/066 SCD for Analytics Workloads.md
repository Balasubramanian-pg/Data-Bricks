# 066 SCD for Analytics Workloads

Canonical documentation for 066 SCD for Analytics Workloads. This document defines concepts, terminology, and standard usage.

## Purpose
The 066 SCD (Slowly Changing Dimension) standard addresses the requirement for maintaining historical accuracy within analytical data environments. In modern analytics, business entities (e.g., customers, products, or locations) change over time. Without a structured approach to managing these changes, historical reporting becomes inaccurate, as current attributes are incorrectly applied to past events.

This topic exists to provide a framework for capturing, storing, and querying state changes in a way that preserves the integrity of point-in-time analysis while optimizing for the high-read, high-volume nature of analytical workloads.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
**In scope:**
* Theoretical frameworks for tracking attribute changes over time.
* Temporal logic and validity interval management.
* Strategies for maintaining referential integrity between facts and dimensions.
* Metadata requirements for auditing and lineage.

**Out of scope:**
* Specific vendor implementations (e.g., specific SQL syntax for Snowflake, BigQuery, or Databricks).
* Real-time streaming ingestion mechanics (focus is on the state management logic).
* Transactional database normalization (3NF).

## Definitions
| Term | Definition |
|------|------------|
| **SCD (Slowly Changing Dimension)** | A design pattern used in data warehousing to manage and store both current and historical data over time. |
| **Natural Key (Business Key)** | The identifier assigned to an entity in the source system (e.g., EmployeeID, SKU). |
| **Surrogate Key** | A unique, system-generated identifier used to distinguish between different versions of the same natural key. |
| **Effective Date** | The timestamp or date marking the beginning of a record's validity. |
| **End Date (Expiry Date)** | The timestamp or date marking the conclusion of a record's validity. |
| **Current Flag** | A boolean or binary indicator used to quickly identify the most recent version of a record. |
| **Point-in-Time (PIT) Analysis** | The process of joining facts to the specific version of a dimension that was valid at the time the fact occurred. |

## Core Concepts
### Temporal Validity
The foundation of 066 SCD is the concept of "validity intervals." Every record in an analytical dimension must define the window of time during which its attributes were true. This is typically achieved through a closed-open interval logic: `[Effective_Timestamp, End_Timestamp)`.

### Idempotency
In analytical workloads, the process of updating SCD tables must be idempotent. Re-running a data pipeline for a specific period should result in the same state, ensuring that historical records are not duplicated or corrupted by late-arriving data.

### Grain and Granularity
The grain of an SCD table is defined by the combination of the Natural Key and the Effective Date. In 066-compliant systems, the granularity must be sufficient to capture all business-relevant changes without creating "noise" from transient system states.

## Standard Model
The standard model for 066 SCD for Analytics Workloads prioritizes **Type 2** logic as the primary mechanism for historical preservation, supplemented by metadata for performance.

1.  **Identity Management:** Every record must possess a Surrogate Key to allow Fact tables to join to specific historical states.
2.  **Version Control:** Each change to a monitored attribute triggers the creation of a new row.
3.  **Temporal Metadata:**
    *   `valid_from`: The start of the record's validity.
    *   `valid_to`: The end of the record's validity (standardized to a high-water mark like `9999-12-31` for active records).
    *   `is_current`: A flag for filtering the latest state.
    *   `hash_diff`: A deterministic hash of all monitored attributes used to detect changes during ingestion.

## Common Patterns
### Type 0: Retain Original
Attributes are never updated. This is used for "Fixed" data such as "Date of Birth" or "Original Order Date."

### Type 1: Overwrite
The current value overwrites the old value. No history is kept. This is used for correcting errors or for attributes where history has no analytical value (e.g., a "Last Synced" timestamp).

### Type 2: Add New Row
The standard for analytics. A new row is added with a new surrogate key, and the previous row is "expired" by updating its `valid_to` date.

### Type 4: History Table
The main table holds only the current state (Type 1), while a separate "shadow" or "history" table captures every change in a Type 2 format. This optimizes performance for users who only care about the current state.

## Anti-Patterns
*   **Using Natural Keys in Fact Tables:** Joining facts directly to natural keys in a Type 2 environment leads to "fan-out" or incorrect attribute association.
*   **Null End Dates:** Using `NULL` to represent the current record's `valid_to` date. This often breaks indexing and requires complex `COALESCE` logic in join conditions.
*   **Over-Versioning:** Tracking changes on high-frequency, volatile fields (like "Last Login Time") within a Type 2 dimension, leading to massive table bloat.
*   **Overwriting History for Corrections:** Changing a `valid_from` date of a historical record without a clear audit trail, which invalidates previously generated financial or compliance reports.

## Edge Cases
*   **Late-Arriving Data:** When a change event from three days ago arrives today. The system must "interject" the record into the timeline, potentially expiring a record that was previously considered current and re-calculating the validity chain.
*   **Source Deletions:** When a record is deleted in the source system. The analytical workload must decide whether to "Hard Delete" (removing history), "Soft Delete" (marking a flag), or "Expire" (setting the `valid_to` to the deletion time).
*   **Micro-second Collisions:** Multiple changes to the same natural key within the same processing batch. Standard practice requires ordering by a source-provided sequence number or timestamp to determine the final state.

## Related Topics
*   **Dimensional Modeling:** The broader framework of Star and Snowflake schemas.
*   **Bitemporal Modeling:** Tracking both when an event happened in the real world (valid time) and when it was recorded in the database (transaction time).
*   **Data Lineage:** Tracking the movement and transformation of data from source to the SCD target.
*   **Surrogate Key Management:** Strategies for generating and maintaining unique identifiers across distributed systems.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
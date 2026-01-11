# 031 Watermarking for Incremental Loads

Canonical documentation for 031 Watermarking for Incremental Loads. This document defines concepts, terminology, and standard usage.

## Purpose
Watermarking for incremental loads addresses the challenge of efficiently synchronizing data between a source and a target system without re-processing the entire dataset. As data volumes grow, "full-load" strategies become computationally expensive and time-prohibitive. Watermarking provides a mechanism to identify, isolate, and extract only the records that have been created or modified since the last successful synchronization event.

This topic establishes the framework for stateful data ingestion, ensuring data integrity, minimizing resource consumption, and enabling near-real-time data availability.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
**In scope:**
* Logical frameworks for tracking data progression.
* State management and persistence of markers.
* Selection criteria for watermark columns.
* Strategies for handling updates and inserts (CDC-lite).

**Out of scope:**
* Specific vendor implementations (e.g., Azure Data Factory, Apache Spark, AWS Glue).
* Physical storage formats (e.g., Parquet vs. Avro).
* Real-time streaming windowing (though concepts overlap, this focuses on batch/micro-batch incremental loads).

## Definitions
| Term | Definition |
|------|------------|
| **Watermark** | A metadata value (typically a timestamp or sequence ID) that represents the point of progress in a data synchronization task. |
| **High-Water Mark (HWM)** | The maximum value of the watermark column processed during the most recent successful load. |
| **Low-Water Mark (LWM)** | The starting point for the current load, usually equivalent to the previous HWM. |
| **Watermark Column** | The specific attribute in the source data (e.g., `updated_at`, `row_version`) used to determine if a record is new or changed. |
| **State Store** | A persistent storage mechanism (table, file, or metadata service) used to track the HWM between executions. |
| **Monotonicity** | The property of a sequence where values strictly increase or decrease; essential for reliable watermarking. |

## Core Concepts

### Statefulness
Unlike stateless operations, incremental loading requires the system to "remember" its previous state. The system must persist the High-Water Mark in a reliable State Store. If the state is lost, the system typically defaults to a full load to ensure no data is missed.

### Monotonicity and Reliability
For a column to be a valid candidate for watermarking, it should ideally be monotonically increasing. If a watermark column can be updated to a value lower than the current HWM, or if records can be inserted with a value lower than the HWM (late-arriving data), the incremental logic will fail to capture those records.

### Idempotency
An incremental load process should be idempotent. If a process fails mid-execution, re-running the process with the same Low-Water Mark should result in the same final state without creating duplicate records or corrupting the target.

## Standard Model

The standard model for watermarking follows a cyclical four-step process:

1.  **Lookup:** Retrieve the current High-Water Mark from the State Store. This becomes the **Low-Water Mark** for the current session.
2.  **Extraction:** Query the source system for all records where the Watermark Column is greater than (or equal to, depending on logic) the Low-Water Mark.
3.  **Transformation/Load:** Process the extracted records and commit them to the target system.
4.  **Update:** Upon successful completion of the load, identify the new maximum value from the processed records and update the State Store with this new **High-Water Mark**.

## Common Patterns

### Timestamp-based Watermarking
The most common pattern, using a `last_modified_dt` or `created_at` column. It is effective for capturing both new records and updates, provided the source system updates this timestamp on every change.

### Identity/Sequence-based Watermarking
Uses an auto-incrementing primary key or a sequence ID. This is highly reliable for append-only datasets but cannot detect updates to existing records unless a separate versioning column exists.

### Look-back Windows
To account for potential clock drift or long-running transactions in the source system, a "look-back" or "overlap" is introduced. The Low-Water Mark is calculated as `HWM - Buffer_Interval`. This requires the target system to handle deduplication (Upsert logic).

### Multi-Column Watermarking
In complex scenarios, a combination of columns (e.g., a `TransactionDate` and a `SequenceID`) is used to ensure uniqueness and ordering when a single column is insufficient to guarantee monotonicity.

## Anti-Patterns

*   **Using Non-Indexed Columns:** Selecting a watermark column that is not indexed in the source database leads to full table scans, defeating the performance benefits of incremental loading.
*   **Hardcoding Watermarks:** Storing the watermark within the pipeline code or manual configuration rather than a dynamic State Store.
*   **Ignoring Late-Arriving Data:** Assuming that data always arrives in chronological order. Without a look-back window or a "soft-delete" awareness, late records are permanently missed.
*   **Updating State Before Success:** Updating the High-Water Mark in the State Store before the data has been successfully committed to the target. This leads to data loss if the pipeline fails.

## Edge Cases

*   **Clock Drift:** When the source system and the orchestration engine have unsynchronized system clocks, leading to gaps in data capture.
*   **Deletions:** Standard watermarking cannot detect physical deletes in the source system. Handling deletes requires "Soft Deletes" (an `is_deleted` flag) or a full-key comparison.
*   **Duplicate Watermarks:** Multiple records sharing the exact same timestamp or ID. If the load cuts off exactly at that value, some records might be processed while others are left for the next run, potentially causing duplicates if "greater than or equal to" logic is used.
*   **Schema Evolution:** If the watermark column is renamed, dropped, or its data type is changed in the source, the incremental logic will break.

## Related Topics
*   **Change Data Capture (CDC):** A more robust but complex method of tracking changes using database logs.
*   **Idempotent Consumers:** Design patterns for ensuring target systems handle duplicate messages gracefully.
*   **Slowly Changing Dimensions (SCD):** Methods for managing historical data in the target system.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
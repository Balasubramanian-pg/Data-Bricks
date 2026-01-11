# 017 Raw Data Landing Strategies

Canonical documentation for 017 Raw Data Landing Strategies. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of Raw Data Landing Strategies is to establish a reliable, scalable, and immutable entry point for data into a centralized data ecosystem. This topic addresses the critical need to decouple source systems from downstream consumption, ensuring that data is preserved in its original state for auditability, reprocessing, and lineage. By standardizing how data "lands," organizations can mitigate the risks of data loss, schema misalignment, and ingestion bottlenecks.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   Architectural patterns for data ingestion into a landing zone.
*   Data preservation principles (immutability and fidelity).
*   Organizational structures for storage (partitioning and naming conventions).
*   Metadata requirements for raw ingestion.

**Out of scope:**
*   Specific vendor implementations (e.g., AWS S3, Azure Data Lake Storage, Google Cloud Storage).
*   Downstream transformation logic (Silver/Gold layer processing).
*   Specific ETL/ELT tool configurations.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| Landing Zone | A transient or permanent storage area where data is first written after extraction from a source. |
| Raw Data | Data in its original format as provided by the source system, without any modifications, cleansing, or enrichment. |
| Immutability | The principle that once data is written to the landing zone, it is never modified or overwritten. |
| Data Fidelity | The degree to which the landed data accurately represents the state of the source system at the time of extraction. |
| Partitioning | The physical or logical grouping of data based on specific attributes (usually time) to optimize discovery and retrieval. |
| Schema-on-Read | A data handling strategy where the schema is applied only when the data is extracted from storage, rather than when it is loaded. |

## Core Concepts

### 1. The Principle of Immutability
Raw data must be treated as a read-only historical record. If a source system provides incorrect data, the landing strategy dictates that the incorrect data is stored, and the correction is stored as a subsequent entry. This ensures a complete audit trail and the ability to "replay" data processing from any point in time.

### 2. Source Fidelity
Data should be landed in the format most native to the source or the transport mechanism (e.g., JSON, Avro, Parquet, or CSV). No business logic, renaming of columns, or data type casting should occur during the landing phase.

### 3. Decoupling
The landing zone acts as a buffer. It protects the source system from the performance impacts of complex analytical queries and protects the downstream data platform from source system availability issues.

### 4. Metadata Enrichment
While the data itself remains raw, it should be accompanied by technical metadata, such as:
*   Source system identifier.
*   Ingestion timestamp.
*   Original filename or stream ID.
*   Batch/Job ID.

## Standard Model
The standard model for Raw Data Landing follows a hierarchical structure designed for scalability and discoverability.

### The Hierarchical Pathing Model
Data should be organized using a consistent URI or folder structure:
`[Zone]/[Source_System]/[Entity_Name]/[Ingestion_Year]/[Ingestion_Month]/[Ingestion_Day]/[Batch_ID]/`

*   **Zone:** The top-level container (e.g., "raw" or "bronze").
*   **Source System:** The originating application or database.
*   **Entity Name:** The specific object (e.g., "orders", "users").
*   **Temporal Partitioning:** Year, Month, and Day of ingestion (not necessarily the business date) to facilitate efficient data lifecycle management.

## Common Patterns

### 1. Full Snapshot Landing
The entire state of a source entity is extracted and landed as a complete file. This is common for small datasets or systems that do not support incremental tracking.

### 2. Incremental Delta Landing
Only records that have changed or been created since the last ingestion are landed. This requires a reliable "high-water mark" (e.g., an `updated_at` timestamp).

### 3. Change Data Capture (CDC)
Captures row-level changes (inserts, updates, deletes) from source database logs. These are landed as a stream of events, preserving the exact sequence of changes.

### 4. Event Streaming
Real-time data landing where individual events are captured and written to the landing zone in small batches or near-real-time files.

## Anti-Patterns

*   **In-Place Updates:** Overwriting existing files in the landing zone when new data arrives. This destroys historical context and lineage.
*   **Pre-Landing Transformation:** Applying business logic (e.g., calculating "Total Price") before the data hits the landing zone. If the logic is found to be flawed later, the original data is lost.
*   **Monolithic Files:** Landing massive, multi-gigabyte files that cannot be parallelized for downstream processing.
*   **Lack of Partitioning:** Storing all files for an entity in a single flat directory, leading to performance degradation as the file count grows (the "O(N) Directory Listing" problem).

## Edge Cases

*   **Late-Arriving Data:** Data that arrives significantly after the event occurred. The landing strategy must decide whether to partition by the *event time* (business logic) or *ingestion time* (system logic). Standard practice is to partition by ingestion time for the raw layer.
*   **Schema Drift:** When a source system adds, removes, or changes a column without notice. Raw landing strategies should use flexible formats (like JSON or Avro) to ingest the data regardless of schema changes.
*   **Large Binary Objects (BLOBs):** Handling images or PDFs. These are typically landed in a specialized object store with metadata pointers in the standard landing path.

## Related Topics
*   **018 Data Partitioning Standards:** Deep dive into physical storage optimization.
*   **022 Data Lineage and Traceability:** How raw landing supports the audit trail.
*   **045 Schema Evolution Frameworks:** Managing changes in data structure over time.
*   **009 Data Lake Architecture:** The broader context of the landing zone within a data lake.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
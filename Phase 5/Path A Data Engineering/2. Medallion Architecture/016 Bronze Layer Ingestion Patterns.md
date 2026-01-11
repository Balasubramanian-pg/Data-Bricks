# 016 Bronze Layer Ingestion Patterns

Canonical documentation for 016 Bronze Layer Ingestion Patterns. This document defines concepts, terminology, and standard usage.

## Purpose
The Bronze Layer (also known as the Raw Layer or Landing Zone) serves as the initial entry point for data into a structured data architecture. The purpose of Bronze Layer Ingestion Patterns is to provide a standardized approach for capturing data from disparate source systems while maintaining the highest possible fidelity to the original state. 

This layer addresses the need for data durability, auditability, and the decoupling of source system extraction from downstream business logic. By establishing formal ingestion patterns, organizations ensure that data is captured in a way that supports re-processability and historical analysis without placing undue burden on source operational systems.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Strategies for data extraction and landing.
* Metadata enrichment requirements for raw data.
* Handling of structured, semi-structured, and unstructured data at the point of entry.
* Consistency and idempotency in data movement.

**Out of scope:**
* Specific vendor implementations (e.g., Databricks Autoloader, AWS Glue, Azure Data Factory).
* Silver or Gold layer transformation logic (business logic).
* Physical storage hardware specifications.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Bronze Layer** | The first logical layer in a data architecture where data is stored in its rawest form, often with minimal transformation. |
| **Idempotency** | The property of an ingestion process where multiple executions with the same input produce the same output without duplicating data. |
| **Change Data Capture (CDC)** | A set of software design patterns used to determine and track the data that has changed so that action can be taken using the changed data. |
| **Schema Evolution** | The ability of the ingestion process to adapt to changes in the source data structure over time without failing. |
| **Immutable Storage** | A data storage strategy where data, once written to the Bronze layer, is never modified or deleted, only appended. |
| **Watermarking** | A threshold used to track the progress of data ingestion, typically based on a timestamp or a monotonically increasing identifier. |

## Core Concepts
The Bronze Layer is governed by several fundamental principles:

1.  **Raw Fidelity:** Data should be stored as close to the source format as possible. Any transformation at this stage should be limited to technical metadata enrichment (e.g., ingestion timestamps, source file names).
2.  **Append-Only Logic:** To maintain a historical record and ensure auditability, the Bronze layer typically follows an append-only pattern. Updates to source records are captured as new entries rather than overwriting existing ones.
3.  **Source Decoupling:** The ingestion process must minimize impact on the source system. Once data is in the Bronze layer, all subsequent processing occurs independently of the source.
4.  **Schema-on-Read Support:** While the Bronze layer may enforce some basic structure, it should be flexible enough to store data whose schema may change frequently.

## Standard Model
The standard model for Bronze Layer Ingestion involves a three-step logical flow:

1.  **Extraction:** Data is pulled or pushed from the source system.
2.  **Enrichment:** Technical metadata is appended to the record. Standard metadata includes:
    *   `_ingested_at`: The UTC timestamp of ingestion.
    *   `_source_system`: The identifier of the origin.
    *   `_file_name` or `_offset`: The specific pointer to the source grain.
3.  **Persistence:** Data is written to a durable, scalable storage medium, often partitioned by ingestion date (`yyyy/mm/dd`) to optimize discovery and lifecycle management.

## Common Patterns

### 1. Full Snapshot Ingestion
The entire state of a source entity is extracted and landed in the Bronze layer. This is typically used for small reference tables or systems that do not support incremental tracking. Each snapshot is stored in a new partition.

### 2. Incremental (Delta) Ingestion
Only records that have changed or been created since the last ingestion cycle are captured. This relies on a "high-water mark" (e.g., `updated_at` column) to identify new data.

### 3. Change Data Capture (CDC)
Captures row-level changes (INSERT, UPDATE, DELETE) from source database logs. This pattern provides a high-fidelity stream of events, allowing the Bronze layer to act as a historical ledger of every state change.

### 4. Event-Driven Streaming
Data is pushed to the Bronze layer in real-time or near-real-time via message queues or event hubs. This pattern is used for high-velocity data like telemetry or user clickstreams.

### 5. File-Based Landing
External systems drop files (CSV, JSON, Parquet) into a monitored location. The ingestion engine detects new files and moves them into the managed Bronze structure.

## Anti-Patterns

*   **Over-Transformation:** Applying business logic, aggregations, or complex filtering during the Bronze ingestion. This prevents the ability to "replay" the raw data if business requirements change.
*   **In-Place Updates:** Overwriting existing data in the Bronze layer. This destroys the audit trail and prevents historical point-in-time analysis.
*   **Manual Schema Management:** Hard-coding schemas that cause ingestion pipelines to fail when a source system adds a new column.
*   **Lack of Metadata:** Failing to record when and where data came from, making it impossible to debug data quality issues or perform lineage analysis.

## Edge Cases

*   **Late-Arriving Data:** Data that arrives at the Bronze layer significantly after the event occurred. Ingestion patterns must decide whether to partition by the *event time* or the *ingestion time*. (Standard practice is to partition by ingestion time for the Bronze layer to ensure write-order consistency).
*   **Schema Breaking Changes:** When a source system renames or deletes a column. The ingestion pattern should be "fail-safe" or "drift-tolerant," often by capturing the entire payload as a JSON blob if the schema validation fails.
*   **Large Binary Objects (BLOBs):** Ingesting very large files (e.g., images, PDFs). The pattern usually involves storing the file in an object store and recording the URI/path in the Bronze table rather than embedding the binary data.
*   **GDPR/Right to Erasure:** While Bronze is typically immutable, legal requirements for data deletion may necessitate "rewriting" historical raw data to purge specific identifiers, complicating the append-only model.

## Related Topics
*   **015 Data Lakehouse Architecture:** The broader framework containing the Bronze layer.
*   **017 Silver Layer Refinement Patterns:** The subsequent stage where Bronze data is cleaned.
*   **042 Data Lineage and Metadata Standards:** Standards for tracking data movement.
*   **088 Schema Evolution Strategies:** Detailed methods for handling changing data structures.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
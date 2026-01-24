# 018 Schema Evolution in Silver

Canonical documentation for 018 Schema Evolution in Silver. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of **018 Schema Evolution in Silver** is to provide a structured framework for managing changes to data structures within the intermediate (Silver) layer of a multi-stage data architecture. As source systems (Bronze) evolve and downstream requirements (Gold) shift, the Silver layer must balance the need for stability with the necessity of reflecting new data attributes. 

This standard addresses the problem of "schema drift" and ensures that data pipelines remain resilient, historical data remains accessible, and downstream consumers are not disrupted by upstream structural changes.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Mechanisms for handling additive, reductive, and transformative schema changes in conformed data layers.
* Strategies for maintaining data integrity during structural transitions.
* Governance principles for versioning and metadata tracking within the Silver tier.
* Compatibility standards (Forward and Backward).

**Out of scope:**
* Specific vendor implementations (e.g., Delta Lake, Iceberg, or BigQuery-specific syntax).
* Schema evolution in the Bronze (Raw) layer, where schema-on-read or schemaless patterns often prevail.
* Business logic changes that do not alter the physical or logical structure of the data.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Silver Layer** | The conformed, cleansed, and enriched data tier where data is structured for cross-functional use. |
| **Schema Evolution** | The process of modifying a data structure (adding, dropping, or changing columns) while preserving existing data and pipeline functionality. |
| **Schema Enforcement** | A mechanism that rejects data that does not conform to a predefined structure. |
| **Breaking Change** | A structural modification that prevents existing downstream queries or processes from executing successfully. |
| **Non-Breaking Change** | A structural modification (usually additive) that allows existing processes to continue without intervention. |
| **Type Widening** | The process of changing a data type to a more inclusive one (e.g., Integer to Long) to prevent data loss. |
| **Backfill** | The process of populating new schema elements for historical records. |

## Core Concepts

### 1. The Stability-Agility Paradox
The Silver layer acts as the "Source of Truth" for the enterprise. It requires more stability than the Bronze layer but must be more flexible than the Gold layer. Schema evolution in Silver is the practice of managing this tension by allowing the schema to grow without breaking the "contract" with downstream users.

### 2. Compatibility Levels
*   **Backward Compatibility:** New code can read old data.
*   **Forward Compatibility:** Old code can read new data (e.g., ignoring new columns).
*   **Full Compatibility:** Both forward and backward compatibility are maintained.

### 3. Schema Drift Detection
The ability of the system to identify when the incoming data structure from the Bronze layer no longer matches the established Silver schema. Evolution is the *response* to detected drift.

## Standard Model

The standard model for 018 Schema Evolution in Silver follows a **Controlled Additive Approach**:

1.  **Detection:** The ingestion engine compares the incoming batch/stream schema against the existing Silver table metadata.
2.  **Validation:** The system determines if the change is "Safe" (Additive) or "Unsafe" (Destructive).
3.  **Metadata Update:** For safe changes, the table metadata is updated to include new fields. Existing records are treated as having `NULL` values for new fields unless a default is specified.
4.  **Downstream Notification:** Metadata changes are broadcast to a catalog or notification service to alert downstream consumers of the new attributes.
5.  **Immutable History:** Physical data files are not rewritten for additive changes; the evolution is handled at the metadata layer to ensure performance.

## Common Patterns

### Additive Evolution
The most common pattern. New columns are appended to the end of the schema. This is non-breaking for most SQL-based downstream consumers.

### Type Widening
Automatically promoting a data type to a larger footprint (e.g., `FLOAT` to `DOUBLE`) when the incoming data exceeds the precision of the current schema.

### Virtual Renaming
Using a semantic layer or view to rename columns while keeping the underlying physical schema stable. This allows for "evolution" of naming conventions without breaking physical data links.

### Schema Versioning
Maintaining a version identifier in the metadata. When a breaking change is required, a new version of the Silver table is initialized, and the old version is deprecated over a transition period.

## Anti-Patterns

*   **In-Place Destructive Changes:** Dropping or renaming columns in a production Silver table without a deprecation strategy.
*   **Implicit Type Narrowing:** Allowing the system to truncate data (e.g., `STRING` to `INT`) to fit a schema, leading to silent data loss.
*   **Manual DDL Intervention:** Relying on manual `ALTER TABLE` statements for every upstream change, which creates a bottleneck and risks human error.
*   **Schema-on-Read in Silver:** Treating the Silver layer as schemaless. This shifts the burden of data cleansing to every downstream consumer, defeating the purpose of a conformed layer.

## Edge Cases

*   **Nullability Constraints:** Changing a column from `NULL` to `NOT NULL` is a breaking change if historical data contains nulls. This requires a backfill or a default value strategy.
*   **Nested Structure Evolution:** Adding fields to a deeply nested `STRUCT` or `JSON` column. Some evolution engines handle top-level changes well but struggle with nested depth.
*   **Primary Key Alteration:** If the Silver layer enforces uniqueness on a specific key, changing the composition of that key constitutes a fundamental structural shift that usually requires a table recreation.
*   **Collations and Encoding:** Changing the character encoding or collation of a string column can lead to unexpected sorting and join behavior in downstream Gold layers.

## Related Topics
*   **012 Medallion Architecture Standards:** The overarching framework for Bronze, Silver, and Gold layers.
*   **045 Data Quality Enforcement:** How schema validation ties into broader data quality checks.
*   **089 Metadata Cataloging:** The storage and retrieval of schema versions.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
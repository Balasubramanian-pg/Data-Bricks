# 065 Star Schema Design on Delta

Canonical documentation for 065 Star Schema Design on Delta. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of Star Schema Design on Delta is to apply proven dimensional modeling techniques within a modern lakehouse architecture. This approach addresses the need for high-performance analytical querying, data consistency, and business-friendly data structures while leveraging the distributed storage and ACID capabilities of the Delta protocol. It bridges the gap between traditional data warehousing rigor and the scalability of data lakes.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   Architectural principles of dimensional modeling (Star Schema) as applied to Delta tables.
*   Optimization strategies inherent to the Delta format (e.g., data skipping, clustering).
*   Handling of historical state and change tracking within a distributed storage environment.
*   Structural relationships between Fact and Dimension entities in a lakehouse context.

**Out of scope:**
*   Specific vendor-proprietary SQL syntax or orchestration tools.
*   Physical hardware configurations or cloud-specific storage tiering.
*   Data ingestion (ELT/ETL) logic outside of the schema design itself.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Fact Table** | A central table in a star schema containing quantitative metrics (measures) and foreign keys to dimension tables. |
| **Dimension Table** | A table containing descriptive attributes (context) that qualify the data in fact tables. |
| **Delta Table** | A table backed by the Delta Lake protocol, providing ACID transactions, scalable metadata handling, and time travel. |
| **Surrogate Key** | A unique identifier generated for a dimension record, independent of natural keys, used to handle changes over time. |
| **Natural Key** | An identifier assigned by a source system (e.g., a Product ID or Serial Number). |
| **Z-Ordering / Clustering** | A technique to co-locate related information in the same set of files to improve data skipping performance. |
| **Schema Enforcement** | A mechanism that ensures the table's structure conforms to a predefined schema during write operations. |

## Core Concepts

### 1. ACID-Compliant Dimensionality
Unlike traditional data lakes, Delta allows for atomic updates and deletes. This enables the Star Schema to maintain referential integrity and perform "upserts" (merge operations) into dimensions and facts without rewriting the entire dataset.

### 2. Decoupling of Logical and Physical Design
In a Delta-based Star Schema, the logical design remains a classic star (Facts and Dimensions), but the physical design focuses on file layout. Performance is derived from metadata-driven data skipping rather than physical disk proximity or traditional indexing.

### 3. Schema Evolution
Delta allows for the evolution of the Star Schema (e.g., adding new descriptive attributes to a dimension) without requiring a full table rebuild, supporting agile business requirements.

## Standard Model

The standard model for 065 Star Schema Design on Delta consists of:

1.  **The Fact Table:**
    *   Contains foreign keys to all related dimensions.
    *   Contains degenerate dimensions (e.g., invoice numbers) where appropriate.
    *   Optimized via partitioning on a low-cardinality temporal column (e.g., `EventDate`) and clustering on high-cardinality join keys.

2.  **Dimension Tables:**
    *   Contain descriptive text attributes.
    *   Utilize Surrogate Keys (often implemented as Identity columns or hashes) to maintain historical context.
    *   Are typically smaller than fact tables and are often broadcast-joined in memory during query execution.

3.  **The Relationship Layer:**
    *   Relationships are logical. While Delta does not enforce primary/foreign key constraints at the storage level, the schema design must ensure data integrity through the transformation layer.

## Common Patterns

### Slowly Changing Dimensions (SCD)
*   **SCD Type 1:** Overwriting old values with new values. Used when history tracking is not required.
*   **SCD Type 2:** Creating new records for changes, using effective/expiry dates and "current" flags. Delta’s `MERGE` command is the standard pattern for implementing this.

### Late-Arriving Facts
A pattern where fact data arrives before the corresponding dimension members are processed. The standard approach is to use a "placeholder" or "unknown" dimension member to maintain referential integrity until the dimension is updated.

### The "Gold" Layer Star
In a multi-hop (Medallion) architecture, the Star Schema typically resides in the "Gold" or "Curated" layer, optimized for consumption by BI tools and end-users.

## Anti-Patterns

*   **Over-Partitioning:** Partitioning on high-cardinality columns (e.g., `TransactionID`), which leads to the "small file problem" and degrades Delta's metadata performance.
*   **Massive Denormalization (The "One Big Table"):** While sometimes useful for specific ML use cases, it sacrifices the flexibility, reusability, and storage efficiency of a Star Schema.
*   **Ignoring File Compaction:** Failing to run `OPTIMIZE` or similar maintenance commands, leading to fragmented data and slow read performance.
*   **Hard-Coding Surrogate Keys:** Relying on source system IDs as primary keys when the source system may reuse IDs or lack uniqueness across different systems.

## Edge Cases

*   **High-Frequency Updates to Large Dimensions:** When a dimension table grows to billions of rows and requires frequent updates, the overhead of the Delta log and file rewriting can become a bottleneck. In these cases, "Mini-dimensions" or "Junk dimensions" should be considered.
*   **Multi-Valued Dimensions:** Scenarios where a fact record relates to multiple members of a dimension (e.g., multiple authors for one book). This requires a "Bridge Table," which complicates the star schema but preserves grain.
*   **Time Travel for Auditing:** Using Delta's `VERSION AS OF` to reconstruct the state of a Star Schema at a specific point in the past for regulatory reporting.

## Related Topics
*   **010 Medallion Architecture:** The organizational framework (Bronze/Silver/Gold) where Star Schemas are deployed.
*   **042 Data Skipping and Z-Ordering:** The underlying optimization techniques for Delta tables.
*   **088 Identity Columns and Sequences:** Mechanisms for generating surrogate keys in a distributed environment.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
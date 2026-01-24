# 019 Data Cleansing in Silver Layer

Canonical documentation for 019 Data Cleansing in Silver Layer. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of Data Cleansing in the Silver Layer is to transform raw, ingested data into a refined, consistent, and reliable state. In a multi-tier data architecture (such as the Medallion Architecture), the Silver Layer serves as the "Source of Truth" for enterprise-wide data. 

This process addresses the inherent noise, inconsistency, and corruption present in source systems. By applying systematic cleansing, organizations ensure that downstream analytical models, business intelligence reports, and machine learning features operate on high-fidelity data, reducing the risk of "garbage in, garbage out" scenarios.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   **Data Validation:** Checking for structural and semantic correctness.
*   **Standardization:** Aligning disparate formats into a unified schema.
*   **Deduplication:** Identifying and resolving redundant records.
*   **Integrity Enforcement:** Ensuring relationships between entities are maintained.
*   **Error Handling:** Defining protocols for data that fails cleansing criteria.

**Out of scope:**
*   **Specific vendor implementations:** (e.g., specific Spark configurations, dbt macros, or Informatica workflows).
*   **Business Logic Aggregation:** Complex summarizations or domain-specific KPIs (typically reserved for the Gold Layer).
*   **Physical Storage Optimization:** File format tuning or partitioning strategies.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Silver Layer** | The intermediate refinement tier where data is cleansed, normalized, and integrated. |
| **Schema Enforcement** | The process of ensuring incoming data strictly adheres to a predefined structure and data types. |
| **Deduplication** | The identification and removal of duplicate entries based on a defined business key or set of attributes. |
| **Imputation** | The process of replacing missing or null values with substituted values based on statistical or logical rules. |
| **Normalization** | The reorganization of data to minimize redundancy and ensure logical dependencies. |
| **Referential Integrity** | A property ensuring that relationships between tables remain consistent (e.g., foreign keys point to valid primary keys). |
| **Quarantine** | A storage area or state for records that fail validation and require manual intervention or programmatic correction. |

## Core Concepts
The Silver Layer cleansing process is governed by the **Six Dimensions of Data Quality**:
1.  **Accuracy:** Does the data correctly represent the real-world entity?
2.  **Completeness:** Are all required fields populated?
3.  **Consistency:** Is the data uniform across different systems and timeframes?
4.  **Timeliness:** Is the data available when needed and up to date?
5.  **Validity:** Does the data follow the required format, type, and range?
6.  **Uniqueness:** Are there duplicate records representing the same entity?

Cleansing in this layer is **idempotent**; re-running the cleansing logic on the same raw data should always yield the same refined result.

## Standard Model
The standard model for Silver Layer cleansing follows a linear pipeline of refinement:

1.  **Structural Validation:** Verify that the data matches the expected schema (column names, data types, nullability).
2.  **Standardization:** 
    *   Convert all timestamps to a standard (e.g., UTC).
    *   Normalize strings (e.g., trimming whitespace, casing).
    *   Standardize units of measure and currency.
3.  **Integrity Checks:** Validate lookups and foreign key relationships against other Silver Layer entities.
4.  **Deduplication:** Apply "Golden Record" logic to resolve multiple entries for the same entity, often using the most recent timestamp or a prioritized source system.
5.  **Transformation/Enrichment:** Perform simple calculations (e.g., age from birthdate) or join with reference data (e.g., mapping ZIP codes to cities).
6.  **Persistence:** Write the refined data to the Silver Layer, typically using a format that supports ACID transactions.

## Common Patterns
*   **The Quarantine Pattern:** Records that fail critical validation (e.g., missing a primary key) are diverted to a "quarantine" table rather than being dropped. This allows for observability and reprocessing.
*   **The Upsert Pattern:** Using "Merge" logic to update existing records in the Silver Layer if they have changed in the source, or insert them if they are new.
*   **Soft Deletes:** Instead of removing records, a `is_deleted` flag or `end_date` is used to maintain historical context while cleansing the "active" view of the data.
*   **Audit Metadata:** Adding columns such as `_source_file`, `_processed_timestamp`, and `_cleansing_rule_version` to every record for traceability.

## Anti-Patterns
*   **Hard-Coding Values:** Embedding specific data corrections (e.g., `IF id = 123 THEN name = 'Correct Name'`) directly in the transformation code.
*   **Cleansing in the Gold Layer:** Postponing data quality fixes until the final reporting layer, which leads to inconsistent logic across different reports.
*   **Silent Dropping:** Deleting records that fail validation without logging or alerting, leading to "missing data" mysteries.
*   **Over-Normalization:** Applying 3rd Normal Form (3NF) so strictly that performance degrades and downstream consumption becomes overly complex.
*   **Modifying Bronze:** Changing the raw ingested data. The Bronze layer must remain an immutable record of the source.

## Edge Cases
*   **Late-Arriving Data:** Data that arrives after a window has closed or out of chronological order. The cleansing logic must be able to handle updates to historical records without corrupting current state.
*   **Schema Evolution:** When a source system adds or changes a column. The Silver Layer must have a strategy (e.g., schema merging or versioning) to handle these changes without breaking the pipeline.
*   **Multi-Source Conflict:** When two different source systems provide conflicting information for the same entity (e.g., two different addresses for the same customer). This requires a "survivorship" rule (e.g., System A is more authoritative than System B).
*   **Empty Strings vs. Nulls:** Inconsistent representation of "no data" across different source systems requires a unified approach in the Silver Layer.

## Related Topics
*   **018 Data Ingestion (Bronze Layer):** The prerequisite step providing raw data.
*   **020 Data Modeling in Gold Layer:** The subsequent step where cleansed data is aggregated for business use.
*   **Data Governance:** The framework of rules and standards that cleansing logic must implement.
*   **Master Data Management (MDM):** Advanced deduplication and "Golden Record" management.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
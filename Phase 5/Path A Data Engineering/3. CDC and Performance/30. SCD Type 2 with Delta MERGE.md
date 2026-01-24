# 030 SCD Type 2 with Delta MERGE

Canonical documentation for 030 SCD Type 2 with Delta MERGE. This document defines concepts, terminology, and standard usage.

## Purpose
The 030 SCD Type 2 with Delta MERGE pattern addresses the requirement for maintaining a full history of dimensional changes within a data lake or data warehouse environment. It specifically solves the problem of efficiently reconciling a set of incoming changes (the "Delta") with an existing historical table (the "Target") using an atomic `MERGE` operation. 

This pattern ensures that when an attribute of a member changes, the previous record is preserved (marked as historical) and a new record is created (marked as current), allowing for point-in-time analysis and historical accuracy in reporting.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Logic for identifying new, changed, and unchanged records.
* The mechanics of the "close-out" process for expiring old records.
* The structural requirements for Target tables to support Type 2 history.
* The use of the `MERGE` statement as the primary engine for state reconciliation.

**Out of scope:**
* Specific SQL syntax for individual vendors (e.g., Databricks, Snowflake, BigQuery).
* Performance tuning specific to hardware or cloud-provider configurations.
* SCD Type 1, Type 3, or Type 6 implementations (except where they intersect with Type 2).

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Natural Key (NK)** | The unique identifier from the source system (e.g., CustomerID) used to correlate records across time. |
| **Surrogate Key (SK)** | A unique, system-generated identifier for every row in the target table, ensuring each version of a Natural Key has a unique ID. |
| **Delta Set** | The collection of new or updated records arriving from the source system to be processed. |
| **Effective Date** | The timestamp or date indicating when a specific version of a record became the "truth." |
| **Expiry Date** | The timestamp or date indicating when a record was superseded by a newer version. |
| **Current Flag** | A boolean or indicator (e.g., 'Y'/'N' or 1/0) used to quickly identify the most recent version of a record. |
| **Tombstoning** | The process of marking a record as deleted or expired without physically removing it from the storage layer. |

## Core Concepts
The fundamental idea of the 030 SCD Type 2 pattern is **versioning through immutability**. Instead of updating a record in place, the system treats the existing record as a historical artifact and the incoming record as the new state.

1.  **State Detection:** The system compares the incoming Delta Set against the Target based on the Natural Key and a hash of the descriptive attributes.
2.  **The "Split" Logic:** A single incoming record that represents a change to an existing Natural Key triggers two distinct actions:
    *   An **Update** to the existing "Current" record in the Target to set its Expiry Date and flip its Current Flag to false.
    *   An **Insert** of the new record into the Target with a new Effective Date and the Current Flag set to true.
3.  **Atomicity:** The `MERGE` operation must ensure that both the expiration of the old record and the insertion of the new record occur within a single transaction to maintain referential and temporal integrity.

## Standard Model
The standard model for an SCD Type 2 Target table includes the following mandatory columns:

*   **Surrogate Key:** Primary Key of the table.
*   **Natural Key:** The business key from the source.
*   **Attributes:** The data fields being tracked for changes.
*   **Effective Timestamp:** The start of the record's validity.
*   **End Timestamp:** The end of the record's validity (often defaulted to a "High Date" like `9999-12-31` for current records).
*   **Current Indicator:** A flag to denote the active record.
*   **Hash Key (Optional):** A deterministic hash of all tracked attributes to simplify change detection.

### The MERGE Logic Flow
1.  **Match Phase:** Match Delta records to Target records on Natural Key where `Current_Indicator = True`.
2.  **Change Detection:** If keys match but attribute hashes differ, the record is marked for "Update and Insert."
3.  **New Record Detection:** If the Natural Key does not exist in the Target, the record is marked for "Insert."
4.  **Execution:**
    *   `WHEN MATCHED AND (changes detected)` -> Update Target (expire old record).
    *   `WHEN NOT MATCHED` -> Insert Target (new record).
    *   *Note:* Many implementations require a "double-merge" or a subquery approach to handle the simultaneous insertion of the new version of an updated record.

## Common Patterns
*   **The Subquery Union Pattern:** Since a standard SQL `MERGE` cannot update a row and insert a new row for the same match in one clause, a common pattern involves unioning the Delta Set with a "null-key" set to force an insert for the new versions of updated records.
*   **High-Date Expiry:** Using a constant far-future date (e.g., `9999-12-31 23:59:59`) to represent the Expiry Date of the current record, which simplifies range-based queries.
*   **Audit Metadata:** Including `Created_By_Process_ID` and `Updated_By_Process_ID` to track the lineage of every version.

## Anti-Patterns
*   **Overwriting History:** Updating descriptive attributes on a historical (expired) record, which destroys the audit trail.
*   **Natural Key as Primary Key:** Using the source system's ID as the unique identifier in the Target table, which prevents storing multiple versions of the same entity.
*   **Missing the Current Flag:** Relying solely on date logic to find the "current" record, which can lead to performance degradation on large datasets compared to a boolean flag.
*   **Processing Out-of-Order Data without Validation:** Blindly applying a Delta Set that contains an older timestamp than the current record in the Target.

## Edge Cases
*   **Intra-Batch Changes:** When the Delta Set contains multiple changes for the same Natural Key. The implementation must pre-process the Delta to ensure only the latest state is used for the "Insert" while maintaining the sequence of history.
*   **Late-Arriving Data:** When a record arrives that is chronologically older than the current record in the Target. This requires "inserting into the middle" of the timeline and re-aligning the Effective/Expiry dates of surrounding records.
*   **Deletes in Source:** If a Natural Key disappears from the source, the pattern must decide whether to ignore it or "Tombstone" the record in the Target by expiring the current version.

## Related Topics
*   **010 Data Ingestion Patterns:** The preceding stage where Delta Sets are captured.
*   **040 Hash-Based Change Detection:** Detailed methodology for using MD5/SHA2 hashes to identify changes.
*   **Temporal Tables:** System-versioned tables that provide built-in SCD Type 2 functionality in some RDBMS.
*   **Idempotency in Data Pipelines:** Ensuring that re-running the MERGE with the same Delta Set does not create duplicate history.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
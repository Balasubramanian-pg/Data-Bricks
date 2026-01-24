# 029 SCD Type 1 with Delta MERGE

Canonical documentation for 029 SCD Type 1 with Delta MERGE. This document defines concepts, terminology, and standard usage.

## Purpose
The 029 SCD Type 1 with Delta MERGE pattern addresses the requirement for maintaining "current state" datasets within a data lakehouse or relational environment. In data warehousing, Slowly Changing Dimension (SCD) Type 1 is the method of updating existing records and inserting new ones without preserving historical versions of the data. 

The use of the `MERGE` operation (often referred to as an "UPSERT") provides an atomic, efficient mechanism to synchronize a target dimension table with a source dataset. This pattern solves the problem of data duplication and the complexity of manual "delete-and-reload" strategies by providing a declarative way to handle record evolution.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* The logical execution of the `MERGE` statement for Type 1 updates.
* Handling of Natural Keys and attribute synchronization.
* Requirements for atomicity and idempotency in data synchronization.
* Structural requirements for target tables to support overwrites.

**Out of scope:**
* Specific syntax for individual database vendors (e.g., Databricks, Snowflake, BigQuery).
* SCD Type 2 (historical versioning), Type 3 (partial history), or Type 4 (history tables).
* Physical storage optimization (e.g., Z-Ordering, Liquid Clustering).

## Definitions
| Term | Definition |
|------|------------|
| **SCD Type 1** | A data modeling technique where new data overwrites existing data in a dimension table. No history is maintained. |
| **MERGE** | A DML statement that performs functional "upserts" by joining a source and target and executing conditional actions. |
| **Natural Key** | A unique identifier from the source system (e.g., CustomerID) used to match records between source and target. |
| **Surrogate Key** | A unique identifier generated within the data warehouse, often used as a primary key, though not strictly required for Type 1 logic. |
| **Idempotency** | The property where an operation can be applied multiple times without changing the result beyond the initial application. |
| **Delta** | In this context, refers to the set of changes (increments) being applied to the target table. |

## Core Concepts
### 1. Overwrite Logic
The fundamental principle of SCD Type 1 is that the target table always reflects the most recent state of the source system. When a record in the source matches a record in the target via a Natural Key, all non-key attributes in the target are updated to match the source.

### 2. Atomicity
The `MERGE` operation must be atomic. The update of existing records and the insertion of new records must succeed or fail as a single unit of work. This ensures the target table is never left in a partially updated state.

### 3. Idempotency
A well-implemented 029 pattern is idempotent. If the same source data is processed multiple times against the same target, the state of the target table remains unchanged after the first successful execution.

### 4. Schema Evolution
While SCD Type 1 focuses on data values, the pattern must account for how new columns in the source are integrated into the target (e.g., adding a new attribute to the dimension).

## Standard Model
The standard model for 029 SCD Type 1 utilizes a two-way conditional logic based on a join between the **Source (S)** and the **Target (T)** on a **Natural Key (NK)**.

1.  **Match Condition (`T.NK = S.NK`)**:
    *   **Action**: `UPDATE SET *`
    *   **Behavior**: All non-key columns in the target are updated with values from the source.
2.  **Not Matched Condition (`T.NK != S.NK`)**:
    *   **Action**: `INSERT *`
    *   **Behavior**: New records from the source are appended to the target.

### Logical Flow
1.  Identify the source change set (the "Delta").
2.  Validate the uniqueness of the Natural Key within the source.
3.  Execute the `MERGE` operation.
4.  Commit the transaction.

## Common Patterns
### Change Data Capture (CDC) Integration
When the source is a CDC stream, the 029 pattern often includes a pre-processing step to "deduplicate" the source. If the source contains multiple updates for the same Natural Key, only the latest record (based on a sequence or timestamp) is used in the `MERGE`.

### Audit Metadata
Even though Type 1 does not track history, it is standard practice to include metadata columns:
*   `sys_updated_at`: The timestamp of the last time the record was modified.
*   `sys_created_at`: The timestamp when the record was first inserted.
*   `sys_source_system`: The origin of the data.

### Partial Updates
In some scenarios, the `MERGE` is configured to only update specific columns while leaving others (like `sys_created_at`) untouched.

## Anti-Patterns
*   **Delete-and-Insert**: Deleting records in the target and re-inserting them. This causes unnecessary churn in storage layers and breaks atomicity if the process fails mid-way.
*   **Ignoring Nulls**: Overwriting populated target fields with `NULL` values from the source unintentionally (unless the `NULL` represents a valid "current state").
*   **Non-Unique Source Keys**: Attempting to `MERGE` a source that contains multiple rows for the same Natural Key. This will cause the `MERGE` operation to fail or produce non-deterministic results.
*   **Full Table Scans**: Failing to provide join keys or predicates that allow the engine to prune data, leading to performance degradation as the dimension grows.

## Edge Cases
*   **Late-Arriving Data**: If a record arrives with an older timestamp than what is currently in the target, the implementation must decide whether to overwrite (strict Type 1) or ignore (version-checked Type 1).
*   **Hard Deletes**: SCD Type 1 typically handles Inserts and Updates. If a record is deleted in the source, the `MERGE` pattern requires an additional `WHEN NOT MATCHED BY SOURCE THEN DELETE` clause or a "Soft Delete" flag update to reflect the removal.
*   **Natural Key Changes**: If a Natural Key changes in the source system, the `MERGE` logic will treat it as a new record (Insert) and the old record will remain in the target (Orphan). This requires a manual reconciliation or a specific "Key Change" handling logic.

## Related Topics
*   **SCD Type 2**: For use cases requiring historical tracking.
*   **Idempotent Pipeline Design**: General principles for building resilient data flows.
*   **ACID Transactions in Data Lakes**: The underlying technology enabling `MERGE` operations on file-based storage.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
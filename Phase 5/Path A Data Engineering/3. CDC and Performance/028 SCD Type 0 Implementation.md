# 028 SCD Type 0 Implementation

Canonical documentation for 028 SCD Type 0 Implementation. This document defines concepts, terminology, and standard usage.

## Purpose
The 028 SCD Type 0 Implementation defines the standard for "Fixed" or "Passive" dimensions within a data warehousing environment. The primary purpose of this pattern is to preserve the original state of data at the moment it was first recorded. It addresses the requirement for immutability in data modeling, ensuring that once a record is committed to the dimension, its attributes are never updated or overwritten, regardless of changes in the source system.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Definition of immutability within dimensional modeling.
* Criteria for selecting attributes suitable for Type 0.
* Structural requirements for Type 0 dimension tables.
* Handling of late-arriving data and initial loads.

**Out of scope:**
* Specific SQL syntax or ETL tool configurations.
* Performance tuning for specific database engines.
* Implementation of SCD Types 1, 2, 3, 4, or 6 (except for comparison).

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| SCD Type 0 | A dimensional modeling technique where attribute values are permanent and never change once loaded. |
| Immutability | The property of data that prevents it from being modified after its initial creation. |
| Natural Key | An identifier produced by the source system that uniquely identifies a business entity. |
| Surrogate Key | A unique identifier generated within the data warehouse to serve as the primary key, shielding the warehouse from source system changes. |
| Passive Dimension | Another term for SCD Type 0, indicating the dimension does not react to changes in the source. |
| Original Grain | The specific state of an entity at the exact timestamp of its first ingestion into the analytical system. |

## Core Concepts
The fundamental principle of SCD Type 0 is **Persistence of Origin**. 

1. **Static Nature:** Unlike other SCD types that track history (Type 2) or reflect current state (Type 1), Type 0 reflects the "as-born" state. 
2. **Write-Once Logic:** The implementation logic must check for the existence of a record based on its Natural Key. If the record exists, the incoming data is ignored. If it does not exist, the record is inserted.
3. **Source Independence:** Once the data is captured, the dimension becomes independent of the source system's lifecycle for that specific record.

## Standard Model
The standard model for an SCD Type 0 dimension consists of the following components:

*   **Surrogate Key:** A unique integer or UUID used for joining with fact tables.
*   **Natural Key:** The business key used to identify the entity in the source system.
*   **Immutable Attributes:** Data fields that are defined as unchanging (e.g., `Date_of_Birth`, `Original_Project_Start_Date`, `Enrollment_ID`).
*   **Metadata - Ingestion Timestamp:** A field indicating when the record was first loaded into the warehouse.
*   **Metadata - Source System ID:** An identifier for the origin of the record.

**Model Visualization:**
```text
[ Surrogate_Key (PK) ]
[ Natural_Key        ]
[ Attribute_1        ] -- Fixed
[ Attribute_n        ] -- Fixed
[ Ingestion_TS       ] -- Metadata
```

## Common Patterns
*   **Reference Data:** Used for static codes or geographic identifiers that are standardized and not subject to change within the context of the analytical model.
*   **Fact-Based Dimensions:** Used when a dimension is essentially a snapshot of a transaction's context (e.g., the specific "Ship-to Address" at the moment an order was placed, which must remain linked to that order regardless of the customer's current address).
*   **Date/Time Dimensions:** The most common implementation of Type 0, where the attributes of a specific date (e.g., "2023-01-01 is a Sunday") never change.

## Anti-Patterns
*   **The "Correction" Update:** Manually updating a Type 0 record to fix a source system error. If data requires correction, it usually implies the attribute should have been Type 1 or that a new record version is required.
*   **Volatile Attributes:** Including fields like `Current_Status` or `Last_Login_Date` in a Type 0 dimension. These fields inherently contradict the "Fixed" definition.
*   **Over-reliance on Natural Keys:** Using the source system's primary key as the warehouse primary key without a surrogate, which risks collisions if the source system reuses keys.

## Edge Cases
*   **Late-Arriving Data:** If a fact record arrives before the dimension record, a "Placeholder" or "Inferred Member" may be created. In Type 0, when the actual dimension data arrives, the placeholder is updated *once* to the true values and then locked.
*   **Source System Deletions:** If a record is deleted in the source, the Type 0 record remains in the warehouse. The implementation must decide whether to flag the record as "Deleted in Source" (which technically modifies the record) or maintain strict immutability.
*   **Natural Key Collisions:** In scenarios where multiple source systems provide the same Natural Key for different entities, the implementation must use a "System Source" prefix to maintain the integrity of the Type 0 logic.

## Related Topics
*   **029 SCD Type 1 Implementation:** Overwriting history to maintain current state.
*   **030 SCD Type 2 Implementation:** Tracking full historical changes via versioning.
*   **Dimensional Modeling Standards:** General principles for star schema design.
*   **Idempotency in ETL:** Ensuring that re-running a load does not duplicate or alter Type 0 records.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
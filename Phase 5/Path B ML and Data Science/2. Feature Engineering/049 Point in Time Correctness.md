# 049 Point in Time Correctness

Canonical documentation for 049 Point in Time Correctness. This document defines concepts, terminology, and standard usage.

## Purpose
Point in Time Correctness (PITC) addresses the fundamental challenge of maintaining data integrity and reproducibility across a temporal dimension. In complex systems, the state of an entity is not merely its current value, but a progression of values over time. PITC ensures that a system can accurately reconstruct the state of any data element as it existed at any specific moment in the past, accounting for both the timing of real-world events and the timing of the system's awareness of those events.

This topic exists to solve the "Information Paradox": the discrepancy between what was true in reality at a certain time and what the system recorded as true at that same time. Without PITC, historical reporting, regulatory auditing, and retroactive processing become unreliable or impossible.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Temporal Dimensions:** Definition and management of Valid Time and Transaction Time.
* **State Reconstruction:** The logic required to view data as it appeared at a specific coordinate in time.
* **Immutability Principles:** The role of non-destructive updates in maintaining historical integrity.
* **Consistency Models:** Ensuring that related entities are viewed at the same temporal snapshot.

**Out of scope:**
* **Specific vendor implementations:** (e.g., SQL Server Temporal Tables, Datomic, or Oracle Flashback).
* **Physical Storage Optimization:** Specific compression algorithms or indexing structures used to store temporal data.
* **Clock Synchronization Protocols:** Low-level hardware protocols like NTP or PTP (though their effects are acknowledged).

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Valid Time** | The period during which a fact is true in the real world (also known as Business Time). |
| **Transaction Time** | The period during which a fact was stored in the database and available for query (also known as System Time). |
| **Bitemporal** | A data model that tracks both Valid Time and Transaction Time simultaneously. |
| **State Reconstruction** | The process of calculating the value of an entity at a specific point in time by aggregating historical logs or snapshots. |
| **Point-in-Time Query** | A request for data that specifies a temporal coordinate (e.g., "As of 2023-01-01 12:00:00"). |
| **Late-Arriving Data** | Information received by the system after the Valid Time to which it refers has already passed. |
| **Retroactive Correction** | An update to a record's Valid Time history that occurs after the original Transaction Time. |

## Core Concepts

### The Two-Axis Temporal Model
PITC relies on the distinction between two independent axes:
1.  **The Reality Axis (Valid Time):** When did the event happen? (e.g., a contract was signed on Monday).
2.  **The Knowledge Axis (Transaction Time):** When did the system learn about it? (e.g., the data entry clerk typed it in on Wednesday).

Correctness is achieved when a system can answer: *"On Thursday, what did we think the state of the contract was as of Monday?"*

### Immutability and Append-Only Logic
To achieve PITC, a system must treat data as immutable. Instead of updating a record in place (which destroys the previous state), the system appends a new version of the record with updated temporal metadata. This creates a "ledger" of state changes.

### Determinism
A PITC-compliant system must be deterministic. Given the same input and the same "As-Of" timestamp, the system must return the exact same result, regardless of when the query is executed.

## Standard Model

The standard model for Point in Time Correctness is the **Bitemporal Conceptual Model**. In this model, every record (or "fact") is associated with two time intervals:

1.  **Valid Time Interval:** `[Valid_From, Valid_To)`
2.  **Transaction Time Interval:** `[Tx_From, Tx_To)`

### The State Matrix
When querying, the system applies a filter on both intervals:
*   To see the **Current State**: Query where `Valid_To` and `Tx_To` are "Infinity" (or the maximum possible value).
*   To see the **Historical Audit**: Query where `Tx_From <= [Target_Time]` and `Tx_To > [Target_Time]`.
*   To see the **Retroactive View**: Query where `Valid_From <= [Target_Time]` and `Valid_To > [Target_Time]`.

## Common Patterns

### Event Sourcing
Storing the state as a sequence of immutable events. To find the state at a "Point in Time," the system replays all events from the beginning of time up to the target timestamp.

### Snapshotting
Periodically saving the full state of a system at a specific timestamp. This optimizes PITC by allowing the system to start reconstruction from the nearest snapshot rather than the beginning of time.

### Effective Dating
A pattern used in HR and Financial systems where records have an "Effective Date." This is a simplified version of Valid Time, often used without a corresponding Transaction Time axis.

## Anti-Patterns

### Destructive Updates (CRUD)
The most common failure of PITC. Using `UPDATE` or `DELETE` statements that overwrite existing data makes it impossible to reconstruct previous states.

### System-Time Dependency
Using the application server's local clock for business logic. This leads to inconsistencies in distributed systems due to clock skew and prevents the system from handling late-arriving data correctly.

### Soft Deletes with Single Timestamps
Using a `deleted_at` flag without a corresponding history table. While this preserves the fact that a record existed, it does not preserve the *state* of the record at the time it was active.

### Log-Only Auditing
Relying on text-based logs for PITC. While logs provide an audit trail, they are often not structured for efficient state reconstruction or querying.

## Edge Cases

### Retroactive Deletions
When a fact is discovered to have never been true. In a PITC system, you do not delete the record; you update the Transaction Time to indicate that, as of now, we know the record was invalid during that previous Valid Time.

### Future-Dated Transactions
Transactions that are recorded now but only become "Valid" in the future (e.g., a scheduled price change). The system must be able to distinguish between "What is true now" and "What will be true tomorrow."

### Clock Skew in Distributed Systems
In systems where multiple nodes record data, slight variations in system clocks can lead to "out of order" Transaction Times. PITC models must often use logical clocks (e.g., Lamport timestamps) or centralized sequencers to maintain a strict Transaction Time order.

### Leap Seconds and Timezone Shifts
Changes in global time standards or local daylight savings. Canonical PITC systems should store all timestamps in UTC or a monotonic time format to avoid ambiguity.

## Related Topics
* **012 Event Sourcing:** A method of achieving PITC through event logs.
* **088 Bitemporal Modeling:** The specific data architecture for two-axis time tracking.
* **024 Distributed Consensus:** Ensuring all nodes agree on the order of events.
* **005 Data Lineage:** Tracking the origin and evolution of data.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
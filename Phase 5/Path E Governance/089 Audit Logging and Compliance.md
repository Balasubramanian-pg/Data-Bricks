# 089 Audit Logging and Compliance

Canonical documentation for 089 Audit Logging and Compliance. This document defines concepts, terminology, and standard usage.

## Purpose
Audit Logging and Compliance exists to provide an immutable, chronological record of activities within a system. Its primary purpose is to ensure accountability, facilitate forensic reconstruction of events, and demonstrate adherence to regulatory, legal, and internal policy requirements. Unlike operational logging, which focuses on system health and debugging, audit logging is centered on "who did what, when, and to what effect."

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Data Integrity:** Mechanisms to ensure logs are not altered or deleted.
* **Event Schema:** The fundamental attributes required for a compliant audit entry.
* **Lifecycle Management:** Retention, archival, and disposal of audit data.
* **Accountability:** Mapping actions to specific identities (human or machine).

**Out of scope:**
* **Application Performance Monitoring (APM):** Metrics related to latency or throughput.
* **Debug Logging:** Verbose output used for software development and troubleshooting.
* **Specific Vendor Implementations:** Configuration steps for specific SIEM or cloud logging providers.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Audit Trail** | A chronological record of system activities that provides documentary evidence of the sequence of activities. |
| **Non-repudiation** | The assurance that a party to a transaction or event cannot later deny their participation. |
| **Immutability** | The property of data that prevents it from being modified or deleted after it has been written. |
| **PII/PHI** | Personally Identifiable Information or Protected Health Information; sensitive data that often requires special handling in logs. |
| **Retention Policy** | A defined set of rules governing how long audit data must be kept and when it should be destroyed. |
| **SIEM** | Security Information and Event Management; a functional category for aggregating and analyzing audit data. |
| **Tamper-evidence** | Techniques (such as hashing or digital signatures) that make unauthorized modifications to logs detectable. |

## Core Concepts

### The Five W’s of Auditing
Every audit event must answer five fundamental questions to be considered compliant:
1.  **Who:** The unique identifier of the actor (user, service account, or system process).
2.  **What:** The specific action performed (e.g., Create, Read, Update, Delete, Login).
3.  **When:** A precise, synchronized timestamp of the event.
4.  **Where:** The source IP, device ID, or service context where the action originated.
5.  **Why:** The outcome or result of the action (e.g., Success, Failure, Reason for denial).

### Immutability and Integrity
Audit logs serve as legal and forensic evidence. Therefore, the storage mechanism must guarantee that once a log is written, it cannot be altered. This is often achieved through Write-Once-Read-Many (WORM) storage, cryptographic hashing of log chains, or off-site replication to a restricted security zone.

### Separation of Duties
The individuals who perform actions within a system (e.g., System Administrators) should not have the authority or technical ability to modify or delete the audit logs that record their own actions.

## Standard Model

The standard model for Audit Logging follows a linear pipeline:

1.  **Event Generation:** The application or system detects a "significant event" (defined by policy) and captures the relevant metadata.
2.  **Normalization:** The event is formatted into a structured schema (e.g., JSON, CEF) to ensure consistency across different sources.
3.  **Transport:** The event is securely transmitted from the source to a centralized repository. This must be done via encrypted channels.
4.  **Ingestion & Indexing:** The central system receives the log, verifies its origin, and indexes it for searchability.
5.  **Storage & Retention:** The log is stored in a tamper-resistant environment for the duration of the retention period.
6.  **Review & Alerting:** Automated tools or human auditors analyze the logs for anomalies or compliance violations.

## Common Patterns

### Centralized Logging
Aggregating logs from all distributed components into a single, hardened repository. This prevents "log silos" and allows for cross-system correlation of events.

### Canonical Schema
Adopting a unified data structure for all audit events across an organization. This ensures that an "Update" event in a database looks structurally similar to an "Update" event in a file system, simplifying analysis.

### Real-time Streaming
Streaming audit events to a secondary location as they occur. This minimizes the "window of opportunity" for an attacker to delete local logs after compromising a system.

## Anti-Patterns

*   **Logging Sensitive Data:** Including passwords, API keys, or PII (Personally Identifiable Information) in audit logs. This turns the audit trail into a high-value target for attackers.
*   **Local-Only Storage:** Storing logs only on the device where the event occurred. If the device is compromised or suffers hardware failure, the audit trail is lost.
*   **Excessive Noise:** Logging every minor system operation (e.g., every packet in a high-traffic network). This creates "alert fatigue" and makes it impossible to find critical events.
*   **Lack of Time Synchronization:** Failing to use NTP (Network Time Protocol) across all systems. Inconsistent timestamps make it impossible to reconstruct the sequence of events during a forensic investigation.

## Edge Cases

*   **Clock Drift:** When a system's internal clock deviates significantly, the audit trail's chronological integrity is compromised. Documentation must specify how the system handles "out-of-order" events.
*   **Log Exhaustion:** If the logging destination is full or unreachable, the system must decide whether to "Fail Closed" (stop processing transactions to ensure no unaudited actions occur) or "Fail Open" (continue transactions but lose the audit trail).
*   **Administrative Bypass:** High-level "root" or "break-glass" accounts that may have the technical ability to disable logging services before performing actions.
*   **Long-term Cryptographic Decay:** If logs are digitally signed, the cryptographic algorithms used may become obsolete over a 10- or 20-year retention period, requiring "re-signing" or "timestamping" strategies.

## Related Topics
*   **042 Identity and Access Management (IAM):** Provides the "Who" in the audit trail.
*   **115 Data Retention and Disposal:** Defines the lifecycle of the stored logs.
*   **021 Incident Response:** Uses audit logs as the primary data source for investigations.
*   **077 Encryption and Key Management:** Ensures the security of logs at rest and in transit.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
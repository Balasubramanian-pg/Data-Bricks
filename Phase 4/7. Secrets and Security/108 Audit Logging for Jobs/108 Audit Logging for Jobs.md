# 108 Audit Logging for Jobs

Canonical documentation for 108 Audit Logging for Jobs. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of 108 Audit Logging for Jobs is to provide a definitive, immutable record of the lifecycle, execution context, and impact of automated or semi-automated background processes. Unlike standard application logging—which focuses on developer-centric debugging—audit logging for jobs is designed for governance, compliance, and security forensics. It addresses the "accountability gap" that occurs when system-level tasks execute without direct, real-time human supervision.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Lifecycle Tracking:** Recording the initiation, transition, and termination of jobs.
* **Identity Attribution:** Identifying the principal (human or machine) responsible for a job's execution.
* **Integrity and Non-repudiation:** Ensuring logs cannot be altered or deleted without detection.
* **State Transitions:** Capturing the "before" and "after" states of critical system parameters affected by a job.

**Out of scope:**
* **Application Performance Monitoring (APM):** Metrics such as CPU usage or memory pressure.
* **Debug Logging:** Verbose stack traces or internal variable states not relevant to auditing.
* **Job Scheduling Logic:** The mechanics of how jobs are queued or prioritized.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Job** | A discrete unit of work executed asynchronously or in the background, often triggered by a schedule, event, or manual invocation. |
| **Principal** | The entity (user, service account, or system process) that authorized or triggered the job. |
| **Audit Event** | A single, immutable record representing a significant change or milestone in the job's lifecycle. |
| **Non-repudiation** | The assurance that an entity cannot deny the validity of an action they performed. |
| **Immutability** | The property of a log entry that prevents it from being modified or deleted once written. |
| **Contextual Metadata** | Data surrounding the job execution, such as IP addresses, source systems, and versioning information. |

## Core Concepts

### The Five W’s of Job Auditing
To meet the 108 standard, every audit log entry must answer:
1.  **Who:** The Principal responsible for the job.
2.  **What:** The specific action or task performed.
3.  **When:** Precise timestamps for initiation, execution, and completion.
4.  **Where:** The source system and the target environment (e.g., Production vs. Staging).
5.  **Why:** The trigger or rationale (e.g., "Scheduled Maintenance," "User Request ID #123").

### Separation of Concerns
Audit logs must be decoupled from the job's operational data. While a job may fail due to a database error, the *audit log* must record the failure independently of the database's own internal logs to ensure a cross-referenced truth.

### Immutability and Chain of Custody
Audit logs for jobs must be stored in a "Write Once, Read Many" (WORM) environment. Any modification to a job's history must be recorded as a *new* audit event (a correction or reversal) rather than an edit to the original record.

## Standard Model

The standard model for 108 Audit Logging follows a structured schema that ensures consistency across different job types.

1.  **Submission Phase:** Records the intent. Captures the Principal, the requested parameters, and the timestamp of the request.
2.  **Authorization Phase:** Records the validation. Confirms that the Principal has the necessary permissions to execute the job.
3.  **Execution Phase:** Records the start of work. Captures the environment state, the version of the code being run, and the unique Job ID.
4.  **Outcome Phase:** Records the result. Captures the final status (Success, Failure, Aborted) and a summary of the impact (e.g., "1,000 records updated").

## Common Patterns

### The Correlation ID Pattern
Every job is assigned a unique Correlation ID at the moment of submission. This ID is passed through all downstream processes and included in every audit event, allowing disparate log entries to be reconstructed into a single chronological narrative.

### The "Before/After" Snapshot
For jobs that modify sensitive data, the audit log captures a cryptographic hash or a summarized snapshot of the data state before the job started and after it concluded.

### Centralized Aggregation
Jobs executing across distributed nodes or containers must ship their audit events to a centralized, hardened logging service to prevent local log tampering if a node is compromised.

## Anti-Patterns

*   **Logging Sensitive Data (PII/Secrets):** Including passwords, API keys, or personally identifiable information in the audit log.
*   **Log-Level Mixing:** Using the same stream for `DEBUG` messages and `AUDIT` events, leading to "signal noise" and potential loss of critical audit data during log rotation.
*   **Local-Only Storage:** Storing audit logs only on the ephemeral file system where the job executed.
*   **Silent Failures:** A job failing to log its own start or end, leaving a "ghost" execution in the system.
*   **Mutable Audit Records:** Allowing administrators to "clean up" or delete job history to save space or hide errors.

## Edge Cases

*   **Long-Running Jobs:** Jobs that span days or weeks require "heartbeat" audit events to prove the job is still authorized and active, rather than stalled.
*   **Recursive/Parent-Child Jobs:** When a job spawns sub-jobs, the audit trail must maintain a clear lineage (Parent ID) to ensure the entire tree of execution is traceable to the original Principal.
*   **System-Triggered Cascades:** When a job is triggered by another system's output, the "Who" must capture the upstream system's identity and the specific event that caused the trigger.
*   **Clock Skew:** In distributed systems, audit logs may arrive out of order. The standard model relies on atomic sequence numbers in addition to timestamps to maintain the true order of events.

## Related Topics

*   **Identity and Access Management (IAM):** Defines how Principals are authenticated before jobs are logged.
*   **Data Retention Policies:** Defines how long 108 Audit Logs must be preserved for legal and compliance reasons.
*   **Observability Frameworks:** While distinct, these provide the infrastructure for transporting audit data.
*   **Non-Repudiation Protocols:** Cryptographic methods used to sign audit logs.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
# 077 Production Drift Detection

Canonical documentation for 077 Production Drift Detection. This document defines concepts, terminology, and standard usage.

## Purpose
Production Drift Detection is a governance and operational discipline designed to identify discrepancies between the **Desired State** of a system (as defined in version-controlled configurations or models) and the **Actual State** of the live production environment. 

The primary purpose of this topic is to ensure environmental integrity, security compliance, and operational predictability. Drift occurs when manual interventions, automated scripts, or external factors modify production resources outside of established Change Management (CM) workflows. Without robust detection, systems become "snowflakes"—unique, undocumented, and difficult to reproduce or recover—leading to increased Mean Time to Recovery (MTTR) and heightened security risks.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Infrastructure Drift:** Discrepancies in hardware, virtualized resources, or cloud services.
* **Configuration Drift:** Deviations in software settings, environment variables, and OS-level parameters.
* **Security Drift:** Unauthorized changes to IAM roles, firewall rules, or encryption settings.
* **Data/Model Drift:** (In ML contexts) Deviations in statistical properties of input data compared to training data.
* **Reconciliation Logic:** The theoretical framework for comparing states.

**Out of scope:**
* Specific vendor implementations (e.g., Terraform Plan, AWS Config, Kubernetes Controllers).
* Performance monitoring (unless performance degradation is a symptom of drift).
* Initial provisioning workflows.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| Desired State | The authoritative specification of how a system should be configured, typically stored in Version Control Systems (VCS). |
| Actual State | The real-time configuration and status of resources currently running in the production environment. |
| Drift | The delta or divergence between the Desired State and the Actual State. |
| Reconciliation | The process of resolving drift, either by updating the Actual State to match the Desired State or updating the Desired State to reflect intentional changes. |
| State Store | A persistent record of the last known successful configuration of the environment. |
| Idempotency | The property where an operation can be applied multiple times without changing the result beyond the initial application. |

## Core Concepts

### The State Duality
Every managed system exists in two dimensions: the **Declared Dimension** (code) and the **Observed Dimension** (reality). Drift detection is the continuous act of verifying that these two dimensions are synchronized.

### Convergence vs. Divergence
* **Convergence:** The tendency of a system to return to its Desired State through automated or manual intervention.
* **Divergence:** The natural tendency of systems to decay or change over time due to entropy, emergency patches, or unmanaged updates.

### Detection Latency
The time interval between the occurrence of drift and its identification. High-criticality systems require low-latency detection (near real-time), while non-critical systems may rely on periodic polling.

## Standard Model

The standard model for Production Drift Detection follows a four-stage cyclical process:

1.  **Observation:** The system queries the production environment to extract the current metadata and configuration of all managed assets.
2.  **Comparison:** The observed metadata is compared against the "Source of Truth" (Desired State). This requires a deep-diffing algorithm capable of ignoring "noise" (e.g., timestamps, volatile metadata).
3.  **Classification:** The detected drift is categorized by severity (Critical, Warning, Informational) and type (Security, Functional, Cost).
4.  **Notification/Action:** The system generates an alert or triggers an automated reconciliation workflow to bring the Actual State back into alignment.

## Common Patterns

### Scheduled Polling
A centralized engine periodically scans the production environment at set intervals (e.g., every 60 minutes) to compare the current state against the configuration repository.

### Event-Driven Detection
The environment emits signals (e.g., CloudTrail logs, Webhooks) whenever a change occurs. The detection system intercepts these events and immediately validates them against the Desired State.

### GitOps / Continuous Reconciliation
An agent runs inside the environment that constantly watches the Desired State (in Git) and the Actual State. If they differ, the agent automatically applies the Desired State to the environment without human intervention.

### Read-Only Production
A restrictive pattern where all manual access to production is revoked. Changes can only be made via automated pipelines, effectively eliminating the possibility of manual drift.

## Anti-Patterns

*   **Silent Remediation:** Automatically fixing drift without logging the occurrence. This hides underlying issues in the deployment pipeline or unauthorized access patterns.
*   **Partial State Tracking:** Only monitoring a subset of attributes (e.g., monitoring CPU count but ignoring firewall rules), leading to a false sense of security.
*   **Manual "Hotfixing":** Making changes directly in production to solve an outage without updating the Desired State code, ensuring the drift will return upon the next automated deployment.
*   **Alert Fatigue:** Triggering high-priority alerts for non-functional drift (e.g., tag changes or metadata updates), causing operators to ignore genuine threats.

## Edge Cases

*   **Transient Drift:** Temporary discrepancies that occur during a deployment window where some resources have updated and others have not.
*   **External Dependencies:** Drift caused by a third-party provider changing an underlying platform default that is not explicitly managed in the Desired State.
*   **Circular Drift:** A scenario where an automated process (like an auto-scaler) changes the Actual State, and the Drift Detection tool attempts to revert it, creating a loop of conflicting changes.
*   **Orphaned Resources:** Resources that exist in the Actual State but have no representation in the Desired State. Standard diffing may miss these if the tool only looks for properties of *known* objects.

## Related Topics

*   **012 Infrastructure as Code (IaC):** The foundational method for defining Desired State.
*   **045 Immutable Infrastructure:** A strategy to prevent drift by replacing rather than modifying resources.
*   **089 Continuous Compliance:** The application of drift detection specifically for regulatory and security standards.
*   **102 Observability:** The broader discipline of monitoring system health and performance.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
# 054 Drift Detection and Monitoring

Canonical documentation for 054 Drift Detection and Monitoring. This document defines concepts, terminology, and standard usage.

## Purpose
Drift Detection and Monitoring exists to ensure the integrity and alignment between a system's **Desired State** (as defined in documentation, code, or policy) and its **Actual State** (the live environment). 

In complex distributed systems, "drift" occurs when manual changes, automated updates, or environmental decay cause the live environment to diverge from its authoritative specification. This topic addresses the requirement for continuous visibility into these discrepancies to prevent security vulnerabilities, operational instability, and "configuration creep."

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Mechanisms for identifying divergence between defined intent and live reality.
* Classification of drift types (Configuration, Data, and Model drift).
* The lifecycle of detection, notification, and reconciliation.
* Theoretical frameworks for maintaining state integrity.

**Out of scope:**
* Specific vendor implementations (e.g., Terraform Plan, AWS Config, Kubernetes Controllers).
* General performance monitoring (unless performance degradation is a symptom of state drift).
* Manual auditing processes that do not involve automated detection.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Actual State** | The real-time configuration and status of a system or resource in a live environment. |
| **Desired State** | The authoritative specification of how a system should be configured, typically stored in Version Control Systems (VCS). |
| **Drift** | The delta or discrepancy between the Desired State and the Actual State. |
| **Reconciliation** | The process of bringing the Actual State back into alignment with the Desired State. |
| **Source of Truth** | The repository or system designated as the master record for the Desired State. |
| **False Positive Drift** | A detected change that is expected or transient and does not represent a violation of intent. |
| **Configuration Creep** | The gradual accumulation of undocumented changes over time, leading to unique, non-reproducible environments. |

## Core Concepts

### 1. The State Duality
Every managed system exists in two planes: the **Conceptual Plane** (where intent is defined) and the **Physical Plane** (where the system operates). Drift is the measurement of the distance between these two planes.

### 2. Detection Triggers
Drift detection is initiated through two primary methods:
*   **Polling (Scheduled):** Periodic comparisons between the Source of Truth and the environment.
*   **Event-Driven:** Real-time notifications triggered by API calls or system changes that signal a potential deviation.

### 3. Directionality of Drift
*   **Managed Drift:** Changes made to the environment that are not reflected in the code.
*   **Definition Drift:** Changes made to the code/policy that have not yet been applied to the environment.

### 4. Convergence vs. Divergence
A healthy system trends toward **Convergence** (Actual State moving toward Desired State). Drift represents **Divergence**, which increases the entropy and unpredictability of the system.

## Standard Model

The standard model for Drift Detection and Monitoring follows a continuous loop:

1.  **Observation:** The monitoring system queries the live environment to capture the current configuration metadata.
2.  **Comparison:** The system fetches the Desired State from the Source of Truth and performs a diffing operation against the observed metadata.
3.  **Analysis:** The detected differences are filtered to exclude known transient states or "ignored" attributes (e.g., timestamps, auto-generated IDs).
4.  **Notification/Alerting:** If the delta exceeds a defined threshold or involves critical parameters, an alert is dispatched to stakeholders.
5.  **Remediation:** The system enters a reconciliation phase, either through manual intervention or automated "self-healing" to restore the Desired State.

## Common Patterns

### Continuous Reconciliation (GitOps)
A pattern where an agent constantly monitors the Source of Truth and the environment, automatically overwriting any manual changes in the environment to ensure the code remains the absolute authority.

### Read-Only Audit
A passive pattern used in highly regulated environments where drift is detected and logged for compliance purposes, but no automated action is taken to change the environment.

### Shadowing / Dry-Run
Running the comparison logic without the intent to reconcile, often used during deployment pipelines to see what *would* change if the new configuration were applied.

## Anti-Patterns

*   **Manual Hotfixing:** Making "emergency" changes directly to the live environment without updating the Desired State definition.
*   **Ignoring Non-Critical Drift:** Allowing "small" discrepancies to persist, which eventually leads to "Configuration Creep" and makes future automation unreliable.
*   **Broad-Spectrum Ignoring:** Configuring detection tools to ignore too many attributes to reduce "noise," which may mask legitimate unauthorized changes.
*   **Remediation Without Root Cause:** Automatically fixing drift without investigating why it occurred, potentially hiding malicious activity or flawed automated processes.

## Edge Cases

*   **Transient State:** Resources that change state rapidly (e.g., a scaling group adding/removing nodes) may appear as drift during the window between the event and the state update.
*   **Circular Dependencies:** When two resources depend on each other's live attributes, causing a perpetual state of perceived drift as they update sequentially.
*   **Provider-Side Defaults:** When a cloud provider assigns a default value to an unmanaged attribute, the detection tool may flag this as drift even though it was never explicitly defined.
*   **Clock Skew:** In distributed systems, discrepancies in system time can lead to false positives in drift detection for time-sensitive configurations or certificates.

## Related Topics

*   **Infrastructure as Code (IaC):** The primary method for defining Desired State.
*   **Observability:** The broader discipline of monitoring system health and performance.
*   **Change Management:** The organizational process for approving and documenting shifts in Desired State.
*   **Immutable Infrastructure:** A strategy that reduces drift by replacing components rather than modifying them.
*   **Policy as Code:** Using code to define the boundaries of "allowable" state, which acts as a secondary layer of drift detection.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
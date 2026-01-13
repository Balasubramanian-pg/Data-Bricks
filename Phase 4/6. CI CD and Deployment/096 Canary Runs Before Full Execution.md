# 096 Canary Runs Before Full Execution

Canonical documentation for 096 Canary Runs Before Full Execution. This document defines concepts, terminology, and standard usage.

## Purpose
The 096 Canary Runs Before Full Execution pattern exists to mitigate the risk of large-scale system failure by validating a subset of an operation in the target environment before committing to total execution. This pattern addresses the "blast radius" problem, where a configuration error, code defect, or environmental incompatibility could cause a catastrophic failure if applied to the entire fleet or dataset simultaneously.

By executing a representative sample—the "canary"—the system can observe real-world behavior and telemetry to ensure the integrity of the process before the remaining workload is authorized to proceed.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* The temporal sequencing of validation before full-scale commitment.
* Logic for gating mechanisms based on canary outcomes.
* Strategies for representative sampling within execution environments.
* Error handling and automated rollback/abort triggers.

**Out of scope:**
* Specific vendor implementations (e.g., AWS CodeDeploy, Jenkins Pipelines, Kubernetes Operators).
* General unit testing or integration testing performed outside the production-equivalent environment.
* Traffic-shifting canary deployments (focused on user routing rather than execution logic).

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Canary Run** | A preliminary, limited execution of a task or deployment used to verify stability. |
| **Full Execution** | The complete application of a change or task across the entire intended scope. |
| **Gating** | The decision-making logic that determines if the Full Execution should proceed based on Canary Run results. |
| **Blast Radius** | The potential impact zone of a failure; the scope of resources affected by an execution. |
| **Short-Circuiting** | The immediate termination of the execution sequence upon a Canary Run failure. |
| **Representative Sample** | A subset of the environment or data that accurately reflects the diversity of the whole. |

## Core Concepts
### 1. Temporal Decoupling
The Canary Run must be temporally separated from the Full Execution. This delay allows for the collection of telemetry, logs, and performance metrics that indicate the health of the operation.

### 2. Environmental Parity
For a Canary Run to be valid, it must occur in an environment that is functionally identical to the environment of the Full Execution. Validating in a "staging" environment is a test; validating on a subset of "production" is a Canary Run.

### 3. Automated Observability
The pattern relies on automated monitoring. If the Canary Run requires manual inspection to determine success, the pattern is considered "Manual Gating," which is a sub-type of the standard model but less resilient to scale.

### 4. Idempotency and State Management
Canary Runs must be designed to either be idempotent (safe to run again during Full Execution) or be able to clean up their state to prevent side effects from contaminating the subsequent Full Execution.

## Standard Model
The standard model for 096 Canary Runs follows a linear progression:

1.  **Initiation:** The system receives a request for a high-impact operation.
2.  **Selection:** A representative subset (the Canary) is identified. This may be a single server, a specific data partition, or a single region.
3.  **Canary Execution:** The operation is performed only on the selected subset.
4.  **Observation Period:** The system pauses for a defined duration to monitor for regressions, errors, or performance degradation.
5.  **Evaluation:** The Gating logic compares observed metrics against pre-defined success criteria (SLIs/SLOs).
6.  **Decision:**
    *   **Pass:** The system proceeds to Full Execution.
    *   **Fail:** The system triggers an Abort/Rollback and alerts stakeholders.

## Common Patterns
### Sequential Batching
The Full Execution is broken into increasingly larger batches (e.g., 1%, 10%, 50%, 100%). The first batch (1%) serves as the Canary Run.

### Synthetic Transaction Canary
Before executing a data-heavy job, the system runs a "synthetic" version of the job using a small, non-critical data set to ensure the logic and connections are valid.

### Parallel Shadowing
The Canary Run occurs in parallel with the existing stable version, but its output is discarded or sent to a "shadow" location. The Full Execution only occurs if the shadow output matches expected parameters.

## Anti-Patterns
*   **The "Golden" Canary:** Using a specific, "clean" subset of the environment for every canary run. This fails to catch issues that might exist in more "weathered" parts of the system.
*   **Ignoring the "Quiet" Failure:** Assuming a Canary Run is successful because it didn't crash, while ignoring the fact that it produced no output or timed out.
*   **Zero-Delay Gating:** Proceeding to Full Execution immediately after the Canary Run starts without allowing time for latent errors to manifest.
*   **Manual Override Dependency:** Designing a system where the Canary Run happens, but the Full Execution always waits for a human to click "Proceed," creating a bottleneck.

## Edge Cases
*   **Intermittent/Flaky Failures:** A Canary Run might fail due to a transient network blip unrelated to the execution logic. Standard models should include a "retry-once" logic or a "minimum failure threshold."
*   **Stateful Dependencies:** If a Canary Run modifies a database schema or a global state, the Full Execution must be able to handle the "partially applied" state without conflict.
*   **Long-Tail Latency:** Some defects only manifest after several hours of execution (e.g., memory leaks). A short Canary Run observation period will fail to detect these.
*   **Small Sample Bias:** In highly heterogeneous environments, a single canary might not encounter the specific configuration that causes the failure.

## Related Topics
*   **042 Automated Rollback Procedures:** The mechanism triggered when a Canary Run fails.
*   **115 Observability and Telemetry:** The data source used for Gating decisions.
*   **088 Blue-Green Deployment:** A related but distinct deployment strategy focusing on environment switching.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
# 071 External Orchestration Overview

Canonical documentation for 071 External Orchestration Overview. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of External Orchestration is to provide a centralized control plane for coordinating complex workflows, resource lifecycles, and data movements across disparate, heterogeneous systems. It addresses the problem of "siloed automation," where individual platforms manage their own internal state but lack the awareness or capability to synchronize with external dependencies.

By decoupling the coordination logic from the execution environment, External Orchestration ensures that multi-step processes spanning multiple domains (e.g., cloud infrastructure, on-premises legacy systems, and third-party SaaS) are executed in a predictable, observable, and resilient manner.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   **Cross-Domain Coordination:** Managing sequences that cross organizational or technical boundaries.
*   **State Management:** Maintaining the "source of truth" for long-running asynchronous processes.
*   **Lifecycle Management:** The creation, modification, and decommissioning of resources via external triggers.
*   **Error Handling and Recovery:** Standardized approaches to retries, rollbacks, and compensation logic.

**Out of scope:**
*   **Internal Scheduling:** Localized task scheduling within a single application or container orchestrator (e.g., Kube-scheduler).
*   **Specific Vendor Implementations:** Detailed configurations for specific tools like Airflow, Temporal, or Terraform Cloud.
*   **Network Layer Routing:** Low-level packet orchestration or SDN (Software Defined Networking) specifics.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Orchestrator** | The central engine responsible for interpreting workflow definitions and commanding external agents or APIs. |
| **Target System** | Any external platform, service, or infrastructure component that receives instructions from the Orchestrator. |
| **Workflow** | A defined sequence of tasks, transitions, and conditions that represent a business or technical process. |
| **Idempotency** | The property of a task where multiple identical requests have the same effect as a single request, preventing unintended side effects during retries. |
| **State Store** | A persistent repository used by the Orchestrator to track the progress and history of workflows. |
| **Reconciliation Loop** | A continuous process where the Orchestrator compares the "desired state" with the "actual state" and takes action to align them. |

## Core Concepts
### Separation of Concerns
External Orchestration separates the *logic* of the process (the "What" and "When") from the *implementation* of the task (the "How"). This allows the Orchestrator to remain lightweight and agnostic of the underlying technology stack of the Target Systems.

### Asynchronous Execution
Most external orchestration involves long-running tasks. The Orchestrator does not block resources while waiting for a task to complete; instead, it relies on polling, webhooks, or event-driven signals to advance the workflow state.

### Centralized Observability
By routing all cross-system commands through a single orchestrator, organizations gain a unified audit trail and monitoring interface, eliminating the need to aggregate logs from dozens of disconnected automation scripts.

## Standard Model
The standard model for External Orchestration follows a **Controller-Agent** or **Controller-API** architecture:

1.  **Definition Phase:** A workflow is defined using a domain-specific language (DSL), code, or a visual model.
2.  **Trigger Phase:** An external event (API call, time-based schedule, or system signal) initiates the workflow.
3.  **Execution Phase:** The Orchestrator evaluates the workflow logic and dispatches commands to Target Systems.
4.  **Feedback Phase:** Target Systems report success, failure, or progress back to the Orchestrator.
5.  **Completion Phase:** The Orchestrator updates the final state and triggers any necessary downstream notifications or cleanup.

## Common Patterns
*   **The Saga Pattern:** Used for managing distributed transactions. If one step fails, the Orchestrator executes "compensating transactions" to undo the effects of previous successful steps.
*   **Event-Driven Triggering:** The Orchestrator remains dormant until a specific event (e.g., a file upload or a database change) occurs in a Target System.
*   **Gatekeeper/Approval Pattern:** The workflow pauses at a specific state and waits for an external human or automated approval before proceeding to high-risk tasks.
*   **Fan-out/Fan-in:** The Orchestrator triggers multiple tasks in parallel across different systems and waits for all to complete before aggregating the results.

## Anti-Patterns
*   **The "God" Orchestrator:** Attempting to manage every minor internal detail of a Target System, leading to high coupling and brittle workflows.
*   **Synchronous Chaining:** Designing the Orchestrator to wait on open connections for long-running tasks, which leads to timeouts and resource exhaustion.
*   **Hard-Coded Dependencies:** Embedding specific IP addresses or credentials directly into the workflow logic rather than using a service registry or secret manager.
*   **Lack of Idempotency:** Designing tasks that create duplicate resources if the Orchestrator retries a failed connection.

## Edge Cases
*   **Zombie Resources:** A Target System successfully creates a resource, but the network fails before the Orchestrator receives the confirmation. The Orchestrator assumes failure, while the resource exists and consumes costs.
*   **Split-Brain Orchestration:** Two different Orchestrators attempting to manage the same Target System simultaneously, leading to conflicting states.
*   **Version Mismatch:** A workflow definition is updated while an instance of that workflow is still running, requiring a strategy for "in-flight" migration or legacy support.
*   **Throttling and Rate Limiting:** The Orchestrator dispatches commands faster than the Target System's API can process them, requiring back-off logic.

## Related Topics
*   **Infrastructure as Code (IaC):** Often the "payload" delivered by an external orchestrator.
*   **Observability and Telemetry:** Critical for monitoring the health of orchestrated workflows.
*   **Service Mesh:** Handles internal service-to-service orchestration, whereas External Orchestration handles cross-platform logic.
*   **State Machines:** The mathematical foundation upon which most robust orchestrators are built.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
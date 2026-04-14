# 079 Managing DAG Dependencies

Canonical documentation for 079 Managing DAG Dependencies. This document defines concepts, terminology, and standard usage.

## Purpose
The management of Directed Acyclic Graph (DAG) dependencies addresses the requirement to coordinate execution across independent units of work. In complex data ecosystems, a single monolithic DAG is often impractical due to scalability, organizational boundaries, or varying execution frequencies. 

This topic establishes the framework for "Inter-DAG" dependencies, ensuring that workflows execute in the correct sequence, respect data lineage, and maintain system integrity without creating tightly coupled or circular relationships.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Mechanisms for establishing execution order between independent DAGs.
* Strategies for state-based and data-aware triggering.
* Logical boundaries for decoupling workflows.
* Governance of cross-DAG communication.

**Out of scope:**
* Intra-DAG task dependencies (internal task-to-task routing).
* Specific vendor-specific syntax (e.g., Airflow Operators, Dagster Assets, Prefect Deployments).
* Infrastructure-level resource scheduling (e.g., Kubernetes pod priority).

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Upstream DAG** | A workflow that must reach a specific state (usually completion) before another workflow can proceed. |
| **Downstream DAG** | A workflow whose execution is contingent upon the state or output of one or more predecessor workflows. |
| **Cross-DAG Dependency** | A logical link between two separate DAGs where the execution of one influences the other. |
| **Trigger** | An active mechanism where an Upstream DAG explicitly initiates a Downstream DAG. |
| **Sensor / Probe** | A passive mechanism where a Downstream DAG waits for a specific state or signal from an Upstream DAG. |
| **Data-Aware Scheduling** | A dependency model based on the availability of data artifacts rather than the completion of a specific process. |
| **Circular Dependency** | A logical error where two or more DAGs depend on each other, creating an infinite loop or deadlock. |

## Core Concepts

### 1. Decoupling and Modularity
Managing dependencies allows for the decomposition of "Mega-DAGs" into smaller, manageable units. This improves maintainability, allows for independent scaling, and enables different teams to own different segments of a pipeline.

### 2. State vs. Data Dependencies
*   **State-based:** The dependency is satisfied when the Upstream DAG reaches a terminal state (e.g., `Success`).
*   **Data-based:** The dependency is satisfied when a specific dataset or artifact is updated, regardless of which process updated it.

### 3. Temporal Alignment
Dependencies must account for the "Data Interval" or "Execution Date." A Downstream DAG running for "2023-10-01" must typically wait for the Upstream DAG corresponding to the same logical period.

## Standard Model
The standard model for managing DAG dependencies follows a **Producer-Consumer** architecture. 

1.  **The Producer (Upstream):** Executes its logic and, upon completion, emits a signal. This signal can be a state change in a metadata database, a message in a queue, or the creation of a file.
2.  **The Consumer (Downstream):** Monitors for the signal. Once the criteria are met, the Consumer enters the execution phase.
3.  **The Orchestrator:** Acts as the registry that maps these relationships, ensuring that the lineage is visible and that failures in the Upstream DAG prevent the Downstream DAG from processing stale or non-existent data.

## Common Patterns

### Active Triggering (Push)
The Upstream DAG contains a final step that calls the API of the Downstream DAG to start it. 
*   *Use case:* When the Downstream DAG should only run if the Upstream DAG succeeds.

### Passive Sensing (Pull)
The Downstream DAG starts on a schedule but begins with a "Sensor" task that polls the status of the Upstream DAG.
*   *Use case:* When the Downstream DAG has its own independent schedule but requires a safety check.

### Data-Aware / Event-Driven
The orchestrator monitors a data catalog or file system. When a specific URI is updated, any DAGs subscribed to that URI are automatically queued.
*   *Use case:* Complex environments where many different processes might update a shared dataset.

### External Signal / Handshake
Using an intermediate state store (like a key-value store or a status table) to record completion. The Downstream DAG queries this store.
*   *Use case:* Cross-platform dependencies (e.g., a legacy system triggering a modern cloud DAG).

## Anti-Patterns

*   **The Mega-DAG:** Putting hundreds of unrelated tasks into one DAG to avoid managing inter-DAG dependencies, leading to slow UI performance and "all-or-nothing" failure states.
*   **Time-Based Guessing:** Scheduling a Downstream DAG to start two hours after an Upstream DAG "usually" finishes, rather than using an explicit dependency.
*   **Circular Logic:** Creating a chain where DAG A triggers B, and B triggers A, leading to infinite execution loops.
*   **Tight Coupling:** Hard-coding specific Upstream task IDs into Downstream logic, making it impossible to refactor one without breaking the other.
*   **Deep Nesting:** Creating a chain of triggers (A -> B -> C -> D -> E) that makes it difficult to trace the root cause of a delay.

## Edge Cases

*   **Granularity Mismatch:** An Upstream DAG runs hourly, but the Downstream DAG runs daily. The Downstream DAG must decide whether to wait for all 24 hourly runs or just the final one.
*   **Backfills:** When re-running historical data, the dependency mechanism must ensure that triggering an Upstream backfill correctly propagates to the Downstream DAGs for the same historical period.
*   **Partial Success:** An Upstream DAG succeeds in its primary objective but fails in a non-critical cleanup task. The dependency logic must define if "Success" requires a total or partial terminal state.
*   **Cross-Environment Dependencies:** A DAG in a Production environment depending on a DAG in a Staging environment (generally discouraged but occasionally necessary during migrations).

## Related Topics
*   **012 Data Lineage:** Tracking the flow of data through these dependencies.
*   **045 Idempotency:** Ensuring that re-running triggered DAGs does not result in duplicate data.
*   **088 Error Handling and Retries:** How failures in the dependency chain are managed.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
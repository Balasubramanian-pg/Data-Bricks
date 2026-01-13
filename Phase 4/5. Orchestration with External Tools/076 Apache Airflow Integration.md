# 076 Apache Airflow Integration

Canonical documentation for 076 Apache Airflow Integration. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of Apache Airflow Integration is to facilitate the orchestration of complex computational workflows by bridging the gap between the Airflow scheduler and external systems. Airflow is designed as a "configuration as code" platform that does not perform heavy data processing itself; instead, it acts as a control plane. Integration allows Airflow to command, monitor, and exchange metadata with third-party services, databases, and compute engines, ensuring that disparate tasks are executed in the correct sequence and under defined conditions.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   The architectural framework for connecting Airflow to external environments.
*   The lifecycle of communication between the Airflow worker and remote APIs/services.
*   Standardized methods for credential management and connection abstraction.
*   The theoretical boundaries of task delegation.

**Out of scope:**
*   Specific vendor-specific provider packages (e.g., specific AWS, Google Cloud, or Azure operator code).
*   Installation and configuration of the Airflow webserver or scheduler.
*   General Python programming tutorials.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Operator** | A template for a single task in a workflow, defining the logic for interacting with an external system. |
| **Hook** | An abstraction layer that handles the low-level communication and authentication with an external platform. |
| **Sensor** | A specialized operator that pauses execution until a specific external criteria or state is met. |
| **Connection** | A set of parameters (URI, login, password, port) stored in the Airflow metadata database used by Hooks to authenticate. |
| **XCom (Cross-Communication)** | A mechanism allowing tasks to exchange small amounts of metadata or state information. |
| **Provider** | A package containing all relevant operators, hooks, and sensors for a specific external service. |
| **Idempotency** | The property where an integration task can be executed multiple times with the same input and produce the same result without side effects. |

## Core Concepts

### The Delegation Model
Airflow integration operates on a delegation model. The Airflow worker initiates a request to an external system (the "executor of work") and then monitors that work until completion. The integration layer is responsible for translating Airflow's internal state into a protocol the external system understands.

### Abstraction Layers
Integration is built on three layers of abstraction:
1.  **The Connection Layer:** Manages "where" and "how" to connect (credentials).
2.  **The Hook Layer:** Manages the "protocol" (API calls, SQL queries).
3.  **The Operator Layer:** Manages the "what" (the specific business logic or task).

### State Synchronization
A critical component of integration is ensuring the Airflow task state (Running, Success, Failed, Up for Retry) accurately reflects the state of the external process. This requires robust error handling and status polling within the integration logic.

## Standard Model

The standard model for Airflow integration follows the **Hook-Operator Pattern**:

1.  **Authentication:** The Operator retrieves a Connection object by its unique ID.
2.  **Instantiation:** The Operator instantiates the appropriate Hook, passing the Connection details.
3.  **Execution:** The Operator calls a method on the Hook to trigger an external action.
4.  **Monitoring:** The Hook maintains the connection or polls the external API until a terminal state is reached.
5.  **Reporting:** The Operator interprets the Hook's return value to mark the Airflow task as successful or failed.

## Common Patterns

### The Deferrable Pattern (Triggers)
For long-running integrations, the standard model of holding a worker slot open is inefficient. The Deferrable pattern allows an integration to suspend itself, releasing the worker slot and handing off the monitoring to a high-efficiency "Triggerer" process until the external event occurs.

### The Push-Pull Pattern
Integrations often use XComs to pass identifiers (like a Job ID) from an "Initiator" operator to a "Monitor" or "Downstream" operator. This decouples the start of a process from the consumption of its results.

### The Listener Pattern
Using specialized sensors or asynchronous triggers to wait for external data arrival (e.g., a file appearing in an S3 bucket or a message arriving in a queue) before proceeding with the DAG.

## Anti-Patterns

*   **Heavy Processing in Hooks:** Performing data transformation or heavy computation within the Hook or Operator rather than delegating it to the external compute engine.
*   **Hardcoded Credentials:** Embedding API keys or passwords directly in the DAG file or Operator arguments instead of using the Airflow Connection URI system.
*   **Lack of Idempotency:** Designing integrations that create duplicate records or side effects if a task is retried after a partial failure.
*   **Synchronous Polling for Long Tasks:** Occupying a worker slot for hours while waiting for a remote process to finish, leading to resource exhaustion (Worker Starvation).
*   **Over-reliance on XComs for Large Data:** Passing actual data payloads (e.g., large DataFrames) through XComs instead of passing references (e.g., S3 paths).

## Edge Cases

*   **Zombie Tasks:** When an external system completes a task, but the Airflow worker dies before it can record the success, leaving the integration in an ambiguous state.
*   **API Rate Limiting:** When an integration triggers too many concurrent requests to an external service, causing the service to throttle or block the Airflow environment.
*   **Partial Success/Atomic Failures:** Scenarios where an external system performs 50% of a task and then fails. The integration must define whether to roll back or allow a "resume" on retry.
*   **Connection Masking:** Ensuring that sensitive integration metadata is not leaked into Airflow logs during execution.

## Related Topics
*   **012 Distributed Task Scheduling:** The underlying mechanism that triggers these integrations.
*   **045 Secret Management:** How credentials for integrations are secured.
*   **089 Observability and Lineage:** How data movement via integrations is tracked across the enterprise.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
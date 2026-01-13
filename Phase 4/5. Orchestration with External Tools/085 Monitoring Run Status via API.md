# 085 Monitoring Run Status via API

Canonical documentation for 085 Monitoring Run Status via API. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of monitoring run status via API is to provide programmatic observability into the lifecycle of an asynchronous execution unit (a "run"). In distributed systems, long-running processes—such as data pipelines, CI/CD jobs, or batch computations—operate independently of the request-response cycle. 

This topic addresses the need for external systems to programmatically determine the current state, progress, and eventual outcome of these processes to facilitate automation, orchestration, and error recovery.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   **State Lifecycle:** The theoretical progression of a run from initiation to termination.
*   **Retrieval Mechanisms:** The methods by which status information is transferred from the executor to the monitor.
*   **Data Schema:** The standard metadata required to describe a run's status.
*   **Consistency Models:** How status reporting handles latency and distributed state.

**Out of scope:**
*   Specific vendor implementations (e.g., GitHub Actions API, AWS Step Functions API).
*   The internal logic of the process being executed.
*   User Interface (UI) design for monitoring dashboards.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Run** | A single, discrete execution instance of a defined process or task. |
| **Status** | The specific point within a lifecycle that a run occupies at a given timestamp. |
| **Terminal State** | A final status from which no further state transitions are possible (e.g., Succeeded, Failed). |
| **Polling** | A mechanism where the client repeatedly requests the status from the API at defined intervals. |
| **Webhook** | A mechanism where the API provider pushes status changes to a client-defined endpoint. |
| **Idempotency Key** | A unique identifier used to ensure that status requests or transitions are processed exactly once. |
| **Heartbeat** | A periodic signal sent by the running process to indicate it is still active and has not stalled. |

## Core Concepts

### The State Machine
Every run is governed by a finite state machine. Monitoring via API is essentially the act of querying the current node of that state machine. A robust API must ensure that state transitions are unidirectional toward a terminal state and that "illegal" transitions (e.g., moving from 'Failed' back to 'Running') are prevented or explicitly handled as new runs.

### Asynchronicity and Decoupling
Monitoring APIs decouple the *execution* of a task from the *observation* of that task. This allows the invoking system to remain stateless or perform other duties while the run progresses independently.

### Metadata and Context
A status is rarely useful in isolation. Comprehensive monitoring requires context, including:
*   **Temporal Data:** Start time, end time, and duration.
*   **Resource Attribution:** Which agent or cluster is executing the run.
*   **Progress Indicators:** Quantitative measures (e.g., percentage complete, items processed).

## Standard Model

The standard model for monitoring run status involves a transition through three primary phases:

1.  **Pending Phase:** The run has been initialized or queued but has not yet begun execution.
    *   *States:* `QUEUED`, `PENDING`, `SCHEDULED`.
2.  **Active Phase:** The run is currently consuming resources and performing logic.
    *   *States:* `RUNNING`, `IN_PROGRESS`, `CANCELLING`.
3.  **Terminal Phase:** The run has reached a conclusion.
    *   *States:* `SUCCEEDED`, `FAILED`, `CANCELLED`, `TIMED_OUT`.

### The Status Object
A standard API response for a run status should include:
*   `id`: Unique identifier for the run.
*   `status`: The current state string.
*   `created_at`: ISO 8601 timestamp of initiation.
*   `updated_at`: ISO 8601 timestamp of the last state change.
*   `errors`: An array of error objects (if applicable).
*   `output_ref`: A pointer or URI to the results of the run.

## Common Patterns

### 1. Short Polling
The client requests the status at a fixed interval (e.g., every 30 seconds). This is simple to implement but can lead to high overhead and latency in status reporting.

### 2. Long Polling
The client requests the status, and the server holds the request open until a state change occurs or a timeout is reached. This reduces the number of requests and provides near-real-time updates.

### 3. Webhook Subscriptions
The client registers a callback URL. The API provider pushes a payload to this URL whenever the run enters a new state. This is the most efficient pattern for the client but requires the client to host a reachable endpoint.

### 4. Status-to-Artifact Linkage
The API provides a status that includes a "next steps" or "result" link only once the terminal state is reached. This prevents clients from attempting to consume incomplete data.

## Anti-Patterns

*   **Tight Polling (The "Thundering Herd"):** Polling an API at sub-second intervals without backoff logic, which can degrade the performance of the monitoring service.
*   **Status Ambiguity:** Using the same status (e.g., `ERROR`) for both retriable infrastructure failures and non-retriable logic failures.
*   **Lack of Idempotency:** Designing a monitoring API where the act of checking the status changes the state of the run.
*   **Ignoring Heartbeats:** Assuming a run is "Running" simply because it hasn't reported "Failed" yet, without verifying active heartbeats.

## Edge Cases

*   **The "Zombie" Run:** A run that is marked as `RUNNING` in the database but the underlying process has crashed without updating the status. This requires a timeout or heartbeat mechanism to resolve.
*   **Race Conditions in Webhooks:** Receiving a "Succeeded" webhook before a "Running" webhook due to network jitter. Systems must be designed to handle out-of-order delivery.
*   **Partial Success:** A multi-step run where some steps succeed and others fail. The API must decide whether to report this as `FAILED` or a specialized `PARTIAL_SUCCESS` state.
*   **API Rate Limiting:** When the monitoring system is throttled by the API provider, leading to a "blind spot" where the status of a run is unknown despite the run potentially finishing.

## Related Topics
*   **042 Asynchronous Request-Reply Pattern:** The architectural pattern that necessitates status monitoring.
*   **112 Webhook Security and Verification:** Best practices for securing the push-model of status updates.
*   **015 Distributed Tracing:** For monitoring the internal steps of a run across multiple services.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
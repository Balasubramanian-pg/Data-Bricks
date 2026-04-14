# 084 REST API Job Triggering

Canonical documentation for 084 REST API Job Triggering. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of REST API Job Triggering is to provide a standardized interface for initiating, managing, and monitoring asynchronous processes over HTTP. This mechanism addresses the fundamental limitation of the request-response cycle when dealing with long-running tasks that exceed the practical duration of a single synchronous connection. By decoupling the initiation of a task from its execution, systems can maintain high availability, improve resource utilization, and provide predictable user experiences.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* The lifecycle of a job triggered via a RESTful interface.
* Communication patterns between the client (triggerer) and the server (executor).
* Standardized status reporting and resource identification.
* Idempotency and reliability in job initiation.

**Out of scope:**
* Specific programming language implementations or frameworks.
* Internal scheduling algorithms or worker queue architectures.
* Vendor-specific API syntaxes (e.g., specific cloud provider CLI commands).

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Job** | A discrete unit of work intended for asynchronous execution. |
| **Trigger** | The specific API request that initiates the execution of a job. |
| **Job Resource** | A unique URI representing an instance of a triggered job, used for tracking and management. |
| **Idempotency Key** | A unique identifier provided by the client to ensure that a job is not triggered multiple times for the same intent. |
| **Callback (Webhook)** | A mechanism where the executor notifies the client of job completion or state changes via an outbound HTTP request. |
| **Polling** | A pattern where the client periodically requests the status of a Job Resource to determine its state. |
| **State** | The current condition of a job (e.g., Pending, Running, Succeeded, Failed). |

## Core Concepts

### Asynchronous Decoupling
The primary concept of REST API Job Triggering is the separation of the **Trigger Phase** from the **Execution Phase**. The API response acknowledges the receipt of the request rather than the completion of the work.

### Resource-Oriented Tracking
Every triggered job should be treated as a first-class resource. Upon a successful trigger, the server provides a handle (usually a URL) that allows the client to interact with that specific instance of the job throughout its lifecycle.

### State Determinism
A job must progress through a defined state machine. Transitions should be unidirectional where possible (e.g., a job cannot move from "Succeeded" back to "Running") to ensure consistency in client-side logic.

## Standard Model

The standard model for REST API Job Triggering follows the **Asynchronous Request-Reply Pattern**:

1.  **Initiation:** The client sends a `POST` request to a collection endpoint (e.g., `/v1/jobs`).
2.  **Acknowledgment:** The server validates the request and returns an `HTTP 202 Accepted` status code.
3.  **Resource Location:** The response includes a `Location` header pointing to the Job Resource (e.g., `/v1/jobs/{job_id}`) and/or the job ID in the response body.
4.  **Monitoring:** The client monitors the job via Polling or awaits a Callback.
5.  **Finalization:** Once the job reaches a terminal state (Succeeded/Failed), the Job Resource provides the final output or error details.

## Common Patterns

### The Polling Pattern
The client issues `GET` requests to the Job Resource URI at defined intervals. The server responds with the current state and, if complete, the results or a link to the results.

### The Webhook/Callback Pattern
The client includes a `callback_url` in the initial trigger payload. Upon reaching a terminal state, the server sends a `POST` request to that URL with the job's final status.

### The Fire-and-Forget Pattern
Used when the client requires no knowledge of the job's outcome. The server acknowledges the trigger, and no further communication occurs regarding that specific job instance.

## Anti-Patterns

*   **Synchronous Execution of Long Tasks:** Holding an HTTP connection open for minutes while a task completes. This leads to timeouts, resource exhaustion, and poor scalability.
*   **Using GET for Triggering:** Using a `GET` request to start a job violates the idempotent and safe nature of the HTTP GET method.
*   **Lack of Idempotency:** Failing to support idempotency keys, which can result in duplicate jobs (and potential data corruption) if a client retries a request due to a network flicker.
*   **Opaque Status Codes:** Returning `200 OK` when a job has been queued but not yet started, without providing a way to track the eventual outcome.

## Edge Cases

*   **Zombie Jobs:** Jobs that are marked as "Running" in the database but the underlying worker process has crashed. The API must have a mechanism (like heartbeats or timeouts) to transition these to a "Failed" or "Unknown" state.
*   **Trigger Timeout:** If the connection drops *after* the server starts the job but *before* the client receives the `202 Accepted` response. This is where Idempotency Keys are critical for safe retries.
*   **Result Expiration:** Job Resources and their associated output data are rarely permanent. Documentation must define the "Retention Policy" (e.g., "Job results are deleted after 24 hours").
*   **Partial Success:** In multi-step jobs, a job may finish with a "Warning" or "Partial" state. The standard model must define whether this constitutes a success or failure at the API level.

## Related Topics
*   **012 API Idempotency:** Standards for ensuring repeated requests yield the same result.
*   **045 Webhook Security:** Best practices for securing callback endpoints (signing, secret validation).
*   **102 Rate Limiting:** Managing the volume of job triggers to prevent system saturation.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
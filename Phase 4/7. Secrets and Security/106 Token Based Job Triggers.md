# 106 Token Based Job Triggers

Canonical documentation for 106 Token Based Job Triggers. This document defines concepts, terminology, and standard usage.

## Purpose
The 106 Token Based Job Trigger framework exists to provide a secure, decoupled, and stateless mechanism for initiating automated workflows or computational tasks. In distributed systems, it is often necessary for an external entity (the "Emitter") to signal a processing system (the "Executor") to begin a specific task without requiring a persistent connection or shared user session. 

This topic addresses the problem of secure cross-boundary orchestration, ensuring that job execution is restricted to authorized requests while maintaining the flexibility required for high-scale automation.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   **Token Lifecycle:** Generation, validation, and revocation of trigger-specific identifiers.
*   **Trigger Mechanics:** The logic governing how a validated token translates into an active job state.
*   **Security Constraints:** Authorization boundaries and scope limitation for individual tokens.
*   **Payload Handling:** The relationship between a trigger token and the input data it may carry.

**Out of scope:**
*   **Specific Vendor Implementations:** Proprietary syntax for CI/CD platforms or specific cloud provider CLI tools.
*   **Network Layer Protocols:** Detailed specifications of transport protocols (e.g., HTTPS, MQTT) used to deliver the token.
*   **Job Logic:** The internal code or business logic executed once the job has been successfully triggered.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Trigger Token** | A unique, cryptographically secure string or object that serves as the credential and signal for job initiation. |
| **Emitter** | The system, service, or user that generates or presents the token to initiate a job. |
| **Executor** | The environment or service responsible for validating the token and managing the resulting job lifecycle. |
| **Scope (Claims)** | The specific permissions or parameters bound to a token, defining which jobs it can trigger and under what conditions. |
| **TTL (Time-to-Live)** | The temporal validity period of a token, after which it is rejected by the Executor. |
| **Idempotency Key** | An optional value within or alongside a token used to prevent the same job from being triggered multiple times by the same request. |

## Core Concepts

### Stateless Initiation
The 106 pattern relies on the principle of statelessness. The Executor does not need to maintain a record of the Emitter’s session. All information required to authorize the job execution is either contained within the token (e.g., JWT) or retrievable via a reference lookup keyed to the token.

### Decoupling of Identity
Unlike standard user authentication, a Token Based Job Trigger often represents a "Service Identity" or a "Task Identity." This allows for the rotation of credentials and the auditing of specific automation flows without compromising individual user accounts.

### Validation vs. Authorization
*   **Validation** confirms the token is well-formed, signed by a trusted source, and not expired.
*   **Authorization** confirms that the specific token presented has the "106-Trigger" permission for the specific job requested.

## Standard Model

The standard model for a 106 Token Based Job Trigger follows a four-stage pipeline:

1.  **Issuance:** An authorized authority generates a token with specific metadata (Job ID, Parameters, Expiration).
2.  **Presentation:** The Emitter sends the token to the Executor's trigger endpoint.
3.  **Verification:** The Executor performs a cryptographic check and validates the scope/claims.
4.  **Activation:** The Executor places the job into a queue or execution state and returns a receipt (often a Job Run ID) to the Emitter.

## Common Patterns

### The "One-Shot" Pattern
A token is generated for a single execution. Once the Executor validates the token, it is immediately invalidated or marked as used. This is ideal for sensitive deployments or high-security administrative tasks.

### The "Webhook" Pattern
A long-lived token is assigned to an external service. The service presents this token repeatedly to trigger recurring jobs (e.g., a build triggered by a code commit).

### The "Scoped Parameter" Pattern
The token contains encrypted or signed parameters that the Executor must use for the job. This prevents the Emitter from tampering with job configurations at the time of the trigger.

## Anti-Patterns

*   **Infinite TTL:** Issuing tokens without expiration dates, which increases the window of opportunity for replay attacks if a token is intercepted.
*   **Token as Secret Storage:** Placing sensitive, unencrypted data (like database passwords) directly inside a transparent token (like a base64 encoded string).
*   **Over-Privileged Tokens:** Creating a single token that can trigger any job in the system rather than restricting it to specific Job IDs.
*   **Lack of Audit Logging:** Failing to log the presentation of a token, making it impossible to trace which system initiated a specific job.

## Edge Cases

*   **Token Revocation during Execution:** If a token is revoked while the job it triggered is still running, the standard model dictates the job should continue to completion unless the job logic specifically polls for token validity.
*   **Clock Skew:** In distributed systems, slight differences in system time between the Emitter and Executor can cause premature token expiration. Implementation should allow for a small "grace period" (typically 60 seconds).
*   **Race Conditions:** If two identical trigger requests arrive simultaneously, the Executor must use an Idempotency Key to ensure only one job instance is created.

## Related Topics

*   **Job Queue Management:** How jobs are handled after the trigger is accepted.
*   **Role-Based Access Control (RBAC):** The underlying security framework that often governs token issuance.
*   **Webhook Security:** Specific implementations of token-based triggers over HTTP.
*   **Audit Logging and Observability:** Tracking the lifecycle of triggered events.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
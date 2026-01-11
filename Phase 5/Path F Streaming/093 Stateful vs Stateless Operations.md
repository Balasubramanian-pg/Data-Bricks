# 093 Stateful vs Stateless Operations

Canonical documentation for 093 Stateful vs Stateless Operations. This document defines concepts, terminology, and standard usage.

## Purpose
The distinction between stateful and stateless operations addresses the fundamental problem of how systems manage information across time and multiple interactions. This topic exists to provide a framework for designing scalable, reliable, and maintainable architectures by defining how context is preserved, where data resides, and how components interact.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Core architectural definitions of state and context.
* Theoretical boundaries between stateful and stateless processing.
* Impact on scalability, fault tolerance, and complexity.
* Relationship between operations and data persistence.

**Out of scope:**
* Specific vendor implementations (e.g., AWS Lambda vs. EC2).
* Programming language-specific syntax for state management.
* Database-specific storage engines.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **State** | The stored information or "memory" of a system at a specific point in time, representing the cumulative result of previous operations. |
| **Context** | The specific subset of state required to complete a single operation or transaction. |
| **Statelessness** | A property of an operation where the outcome depends solely on the input provided in the request, without reliance on previous interactions. |
| **Statefulness** | A property of an operation where the outcome depends on both the input provided and the current internal state of the system. |
| **Idempotency** | The property of an operation where multiple identical requests have the same effect as a single request, often a critical goal in stateless design. |
| **Session** | A temporary stateful connection or series of interactions between two entities. |

## Core Concepts

### The Nature of State
State is the "condition" of a system. In computing, state is what allows a system to know "who is logged in," "what is in the shopping cart," or "what was the last processed sequence number." 

### Stateless Operations
In a stateless operation, the server or processing unit does not retain any information about the client or the history of the interaction after the operation completes. 
* **Self-Containment:** Every request must contain all the information necessary for the server to understand and process it.
* **Isolation:** Each request is treated as an independent transaction.

### Stateful Operations
In a stateful operation, the system maintains a record of previous interactions. The behavior of the current operation is influenced by what happened before.
* **Continuity:** The system tracks the "flow" of a process.
* **Dependency:** The success or logic of an operation may depend on the existence of a specific prior state.

## Standard Model

The standard model for modern distributed systems favors **Stateless Logic** supported by **Stateful Storage**.

1.  **Stateless Processing Tier:** Application servers or functions should ideally be stateless. This allows them to be scaled horizontally (adding more instances) because any instance can handle any incoming request.
2.  **Stateful Storage Tier:** State is offloaded to specialized external systems (databases, caches, or distributed file systems). 
3.  **Context Transfer:** In stateless architectures, the "state" required for a transaction is often passed back and forth between the client and server (e.g., via tokens or cookies) or retrieved from the storage tier using a unique identifier provided in the request.

## Common Patterns

### Stateless Patterns
*   **Token-Based Authentication:** Instead of the server remembering a user is logged in, the client sends a cryptographically signed token (like a JWT) with every request.
*   **Functional Programming:** Functions that return the same output for the same input without modifying any external variables.
*   **REST (Representational State Transfer):** An architectural style where the communication is constrained to be stateless.

### Stateful Patterns
*   **Sticky Sessions:** A load-balancing technique where a client is routed to the same physical server for the duration of a session to utilize local memory.
*   **TCP Connections:** At the networking layer, TCP is stateful as it tracks sequence numbers and acknowledgments to ensure reliable delivery.
*   **Sagas/Workflows:** Long-running processes that must track the progress of multiple distributed steps over time.

## Anti-Patterns

*   **Local State in Scalable Tiers:** Storing session data in the local memory of a web server that is part of an auto-scaling group. If the server terminates or the load balancer redirects the user, the state is lost.
*   **The "Fat" Token:** Including too much state in a stateless token, leading to high network overhead and latency.
*   **Hidden Statefulness:** Designing an API that appears stateless but relies on global variables or side effects that are not explicitly managed, leading to non-deterministic behavior.
*   **Stateful Retries:** Retrying a stateful operation without considering the side effects of the previous failed attempt (violating idempotency).

## Edge Cases

*   **Pseudo-Statelessness (Caching):** A system may be architecturally stateless, but performance optimizations (like local caching) introduce temporary state. If the cache is inconsistent, the system may behave as if it were stateful and broken.
*   **Distributed State Consistency:** In highly distributed systems, "state" may not be uniform across all nodes at the same time (Eventual Consistency). An operation might be stateful but yield different results depending on which node processes it.
*   **Termination Signals:** In "stateless" serverless environments, the brief period between the end of an execution and the reclamation of resources can sometimes be exploited to persist state (e.g., keeping a database connection open), blurring the line between stateless and stateful.

## Related Topics
*   **012 Idempotency in API Design**
*   **045 Distributed Systems Consistency Models**
*   **102 RESTful Architecture Standards**
*   **115 Database Normalization and Persistence**

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
# 083 Multi System Workflows

Canonical documentation for 083 Multi System Workflows. This document defines concepts, terminology, and standard usage.

## Purpose
The 083 Multi System Workflows framework addresses the complexities inherent in executing business logic that spans multiple independent computational environments, data stores, or administrative domains. In modern distributed architectures, a single business process rarely resides within a single monolithic application. 

This topic exists to provide a standardized approach to managing state, ensuring data integrity, and handling failures across heterogeneous systems that do not share a common database or memory space. It addresses the "Distributed Transaction" problem where traditional ACID (Atomicity, Consistency, Isolation, Durability) properties cannot be guaranteed across network boundaries.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **State Management:** How the "source of truth" for a workflow is maintained across systems.
* **Communication Paradigms:** The theoretical methods of interaction (Synchronous vs. Asynchronous).
* **Error Recovery:** Strategies for handling partial failures in a multi-step process.
* **Consistency Models:** Definitions of eventual consistency and compensation logic.

**Out of scope:**
* **Specific vendor implementations:** (e.g., AWS Step Functions, Azure Logic Apps, Temporal, or Camunda).
* **Network-level protocols:** (e.g., specific TCP/IP or HTTP/3 configurations).
* **Programming language syntax:** Specific SDK implementations.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Orchestration** | A centralized approach where a single "controller" directs the flow of the workflow and manages the state of all participants. |
| **Choreography** | A decentralized approach where each system responds to events and decides its own next steps without a central controller. |
| **Idempotency** | The property of an operation where multiple identical requests have the same effect as a single request. |
| **Compensation** | An action taken to undo the effects of a previously committed step when a subsequent step in the workflow fails. |
| **Saga Pattern** | A failure management pattern that coordinates a series of local transactions to ensure eventual consistency. |
| **State Store** | A persistent repository used to track the current progress and data context of a running workflow. |
| **Side Effect** | A change in state or an action (like sending an email) that occurs outside the immediate scope of the workflow's data return. |

## Core Concepts

### Distributed State
In a multi-system workflow, state is fragmented. No single system has a complete view of the entire process at any given millisecond. The core challenge is ensuring that the "Workflow State" remains coherent even when individual "System States" are in flux.

### The Fallacy of Reliability
Multi-system workflows must be designed with the assumption that the network is unreliable, latency is non-zero, and remote systems will eventually fail. Reliability is achieved through software patterns (retries, timeouts, and circuit breakers) rather than hardware guarantees.

### Eventual Consistency
Unlike local transactions, multi-system workflows typically operate on the principle of eventual consistency. There will be periods where System A has updated its records, but System B has not yet received the instruction to do so. The workflow is considered "complete" only when all participating systems reach a terminal state.

## Standard Model

The standard model for 083 Multi System Workflows follows a **State-Machine-as-a-Service** architecture.

1.  **Trigger:** An external or internal event initiates the workflow.
2.  **Context Injection:** The workflow gathers necessary metadata (IDs, Auth tokens, Correlation IDs).
3.  **Step Execution:** The workflow executes a discrete unit of work.
4.  **Transition Logic:** Based on the output of the step, the workflow determines the next state.
5.  **Persistence:** The current state and context are saved to a durable store before moving to the next step.
6.  **Termination:** The workflow reaches a "Success" or "Failure" terminal state and notifies interested parties.

## Common Patterns

### 1. The Saga Pattern (Command-based)
A sequence of local transactions. Each local transaction updates the database and triggers the next local transaction. If a local transaction fails because it violates a business rule, the saga executes a series of compensating transactions that undo the changes that were made by the preceding local transactions.

### 2. Routing Slip
A pattern where the workflow definition (the "slip") is attached to the message itself. As each system processes the message, it looks at the slip to determine where to send the message next.

### 3. Observer/Pub-Sub
Systems emit events (e.g., "OrderCreated"). Other systems subscribe to these events and perform their own logic. This is the primary pattern for Choreography.

## Anti-Patterns

*   **The Distributed Monolith:** Designing systems that are so tightly coupled that they must all be online and functional for any single part of the workflow to succeed.
*   **Lack of Idempotency:** Failing to ensure that retrying a failed step results in duplicate records or side effects (e.g., charging a credit card twice).
*   **Synchronous Chaining:** System A waits for System B, which waits for System C. This leads to resource exhaustion and cascading failures.
*   **The "Ghost" State:** Updating a remote system but failing to record that update in the workflow's state store, leading to a mismatch between reality and the workflow's perception.

## Edge Cases

*   **Zombie Workflows:** Workflows that are neither failed nor progressing, often due to a lost callback or an unhandled timeout.
*   **Clock Drift:** When systems rely on timestamps for ordering, but the internal clocks of the different systems are not perfectly synchronized.
*   **Race Conditions in Choreography:** Two systems responding to the same event and attempting to move the workflow into conflicting states simultaneously.
*   **Poison Pill Messages:** A specific input that causes every system that attempts to process it to crash or error out, leading to infinite retry loops.

## Related Topics
*   **042 Distributed Systems Theory**
*   **105 Event-Driven Architecture**
*   **012 API Design and Contract Management**
*   **099 Observability and Distributed Tracing**

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
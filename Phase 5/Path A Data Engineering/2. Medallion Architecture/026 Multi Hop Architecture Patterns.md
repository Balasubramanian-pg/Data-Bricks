# 026 Multi Hop Architecture Patterns

Canonical documentation for 026 Multi Hop Architecture Patterns. This document defines concepts, terminology, and standard usage.

## Purpose
Multi-hop architecture patterns address the complexities inherent in distributed systems where a request, data packet, or execution context must pass through one or more intermediate nodes before reaching its final destination. This topic exists to provide a framework for managing the trade-offs between decoupling, security, and performance.

In modern distributed environments, direct point-to-point communication is often insufficient for requirements involving cross-cutting concerns such as protocol translation, security enforcement, load balancing, and observability. Multi-hop patterns provide the structural logic for how these intermediate stages are organized and how state and identity are preserved across the chain.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   Architectural topologies for sequential and branching request flows.
*   Mechanisms for context and identity propagation across intermediate nodes.
*   Strategies for error handling and observability in multi-stage paths.
*   Theoretical limits of latency and reliability in chained systems.

**Out of scope:**
*   Specific vendor implementations (e.g., specific service mesh products or cloud-native API gateways).
*   Low-level physical networking protocols (e.g., BGP, OSPF) unless used as an analogy for application-layer routing.
*   Hardware-specific optimization techniques.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Hop** | A single segment of a communication path between two nodes (source to intermediate, intermediate to intermediate, or intermediate to destination). |
| **Node** | Any processing entity (proxy, gateway, service, or broker) that receives, processes, and forwards a request. |
| **Ingress Node** | The first point of entry into a multi-hop system. |
| **Egress Node** | The final point of exit or the terminal destination in a multi-hop chain. |
| **Context Propagation** | The process of passing metadata (trace IDs, security tokens, tenant info) through every hop in the architecture. |
| **Backpressure** | A signal sent upstream from a downstream hop to slow down the flow of data when the downstream node is overwhelmed. |
| **Fan-out/Fan-in** | A pattern where one hop triggers multiple parallel hops (fan-out) which are later aggregated (fan-in). |

## Core Concepts

### 1. The Principle of Intermediate Processing
Every hop in a multi-hop architecture must justify its existence by providing one of the following:
*   **Transformation:** Converting data formats or protocols.
*   **Validation:** Ensuring the request meets security or schema requirements.
*   **Routing:** Determining the next optimal hop based on logic or state.
*   **Resilience:** Providing caching, retries, or circuit breaking.

### 2. Context Preservation
As a request moves through multiple hops, the original intent and identity must be preserved. This is typically achieved through an envelope or header mechanism that travels alongside the payload, ensuring that the final destination can verify the origin and the path taken.

### 3. Latency Accumulation
Each hop introduces a "tax" in the form of processing time, serialization/deserialization overhead, and network transit. Multi-hop architectures must account for the cumulative latency (the sum of all hop latencies) rather than just the individual node performance.

## Standard Model
The standard model for Multi-Hop Architecture is the **Linear Chain of Responsibility**, supplemented by **Sidecar** or **Gateway** patterns.

1.  **Origin:** The client initiates the request.
2.  **Edge/Gateway Hop:** Handles authentication, rate limiting, and initial routing.
3.  **Intermediate/Service Hop(s):** Business logic processing, data enrichment, or transformation.
4.  **Terminal Node:** The final destination (e.g., a database or a third-party API).

In this model, each node is only required to know about the "Next Hop," effectively decoupling the Origin from the Terminal Node.

## Common Patterns

### 1. The Gateway-Aggregator Pattern
A single ingress hop receives a request and initiates multiple downstream hops to various services, aggregates their responses, and returns a unified result to the client. This reduces the number of hops the client must manage.

### 2. The Sidecar Proxy Pattern
Communication logic is moved out of the application and into a separate, adjacent hop (the sidecar). All inbound and outbound traffic passes through this hop, standardizing observability and security without modifying the application code.

### 3. Message-Bus/Asynchronous Multi-Hop
Requests are placed on a message broker. Multiple "worker" hops consume, process, and re-publish the message to subsequent queues. This pattern prioritizes durability and decoupling over real-time latency.

### 4. Onion Routing (Security-Centric)
Each hop only knows the identity of the previous and next hop. The data is wrapped in multiple layers of encryption, with each hop "peeling" one layer to reveal the next destination, ensuring anonymity and path integrity.

## Anti-Patterns

### 1. The "Telephone Game" (Data Degradation)
Occurs when intermediate hops modify the primary payload in ways that lose original intent or precision, leading to the terminal node receiving "corrupted" or incomplete business logic.

### 2. Circular Dependencies
A configuration error where Hop A routes to Hop B, which eventually routes back to Hop A. This creates infinite loops that consume system resources until a Time-to-Live (TTL) or timeout is reached.

### 3. Deep Nesting (The "Spaghetti" Chain)
Creating an excessive number of hops (e.g., 10+) for a single request. This leads to "latency death" and makes debugging nearly impossible without sophisticated distributed tracing.

### 4. Hidden Hops
Introducing intermediate nodes (like transparent proxies or load balancers) that are not accounted for in the architectural diagram or timeout budget, leading to "ghost" failures.

## Edge Cases

### 1. Partial Failure in Fan-out
In a multi-hop fan-out scenario, if 4 out of 5 hops succeed, the architecture must define whether the entire request fails or if a partial result is acceptable.

### 2. Clock Skew
In time-sensitive multi-hop patterns (e.g., expiring tokens or time-stamped logs), variations in system clocks across different nodes can lead to premature expiration or incorrect sequencing of events.

### 3. Protocol Mismatch
When a middle hop attempts to upgrade or downgrade a protocol (e.g., HTTP/2 to HTTP/1.1) and loses specific metadata or streaming capabilities required by the terminal node.

## Related Topics
*   **012 Distributed Tracing:** The primary method for observing multi-hop flows.
*   **045 Service Mesh Topology:** A specific implementation of multi-hop sidecar patterns.
*   **089 Zero Trust Architecture:** Principles often enforced at every hop in a chain.
*   **102 Circuit Breaker Patterns:** Essential for preventing cascading failures in multi-hop systems.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
# 090 Data Contracts Between Teams

Canonical documentation for 090 Data Contracts Between Teams. This document defines concepts, terminology, and standard usage.

## Purpose
Data Contracts exist to resolve the inherent friction between data producers (who generate data as a byproduct of business processes) and data consumers (who derive value from that data). In distributed systems, changes to upstream schemas or logic often cause silent, downstream failures. 

This topic addresses the "brittle pipeline" problem by establishing a formal, bidirectional agreement that decouples the internal implementation of a service from its external data interface. It shifts the responsibility of data quality upstream, ensuring that data is treated as a first-class product rather than an exhaust.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative. It focuses on the architectural and organizational requirements of data contracts rather than specific serialization formats or orchestration tools.

## Scope
**In scope:**
* The structural, semantic, and operational components of a data contract.
* The lifecycle of a contract (negotiation, enforcement, and evolution).
* Roles and responsibilities of producers and consumers.
* Governance frameworks for cross-team data sharing.

**Out of scope:**
* Specific vendor implementations (e.g., dbt, Great Expectations, Confluent Schema Registry).
* Physical storage layer optimizations.
* General API design (except where it intersects with data delivery).

## Definitions
| Term | Definition |
|------|------------|
| **Data Contract** | A formal agreement between a producer and a consumer that defines the structure, semantics, and quality of data being exchanged. |
| **Producer** | The team or service responsible for generating, maintaining, and delivering the data defined in the contract. |
| **Consumer** | The team or service that relies on the data for downstream applications, analytics, or machine learning. |
| **Schema** | The technical definition of data structure, including field names, types, and constraints. |
| **Service Level Objective (SLO)** | Specific, measurable goals regarding the reliability and performance of the data delivery (e.g., freshness, uptime). |
| **Semantic Versioning** | A versioning scheme (Major.Minor.Patch) used to communicate the nature of changes to the contract. |
| **Breaking Change** | Any modification to the contract that requires the consumer to update their implementation to avoid failure. |

## Core Concepts

### 1. Data as a Product
Data contracts treat data as a product with a defined interface. The producer is accountable for the "uptime" and quality of the data, just as a software team is accountable for an API.

### 2. Decoupling
The contract acts as a buffer. Producers are free to change their internal database schemas or business logic as long as the output defined in the contract remains consistent.

### 3. Shift-Left Quality
Validation occurs at the point of creation. By enforcing the contract during the CI/CD process or at the ingestion layer, errors are caught before they propagate to downstream warehouses or lakes.

### 4. Explicit Ownership
Every data element in a contract must have a clear owner. If a field fails a quality check, the contract identifies exactly which team is responsible for the remediation.

## Standard Model
The standard model for a Data Contract consists of four primary layers:

1.  **Structural Layer (The Schema):** Defines the fields, data types (Integer, String, Boolean), and nesting. It ensures the data can be parsed.
2.  **Semantic Layer (The Meaning):** Defines what the data represents. For example, does `price` include tax? Is `timestamp` in UTC or local time?
3.  **Quality Layer (The Constraints):** Defines the expected state of the data, such as nullability, uniqueness, range constraints (e.g., `age > 0`), and referential integrity.
4.  **Operational Layer (The Metadata):** Defines the "terms of service," including data freshness (latency), frequency of updates, and the deprecation policy.

## Common Patterns

### The Registry Pattern
Contracts are stored in a central, version-controlled repository (a Registry). Both producers and consumers reference this registry to generate code, validate data, and discover available datasets.

### The Gatekeeper Pattern
The contract is enforced during the CI/CD pipeline. If a producer attempts to deploy a change that violates the contract (e.g., dropping a column), the build fails, preventing the breaking change from reaching production.

### The Consumer-Driven Contract (CDC)
Consumers define their specific requirements, and the producer agrees to meet them. This ensures the producer does not provide unnecessary data and understands exactly how their data is being used.

## Anti-Patterns

### The "Black Box" Contract
Defining a contract without semantic descriptions. While the data may be technically valid (e.g., a string is a string), the consumer has no understanding of the business logic behind the values.

### The God Contract
Attempting to create a single contract that covers every possible data point a team produces. This leads to bloated, unmanageable interfaces. Contracts should be granular and purpose-built.

### Post-Ingestion Validation
Checking for contract compliance only after the data has landed in the data warehouse. This allows "garbage" data to enter the system, requiring expensive and complex cleanup.

### Manual Enforcement
Relying on "handshake agreements" or Wiki documentation that is not tied to the actual code or data pipeline. These contracts invariably drift from reality.

## Edge Cases

### Late-Arriving Data
In streaming architectures, data may arrive out of order or significantly after the event occurred. The contract must specify how latency SLOs account for these events and whether the producer is responsible for backfilling.

### Schema Evolution (Non-Breaking)
Adding an optional field is generally considered non-breaking. However, in some strict consumer environments (e.g., certain static-type languages or legacy systems), even an addition can cause failures. The contract must define the "Strictness" of the consumer.

### PII and Security Redaction
A contract may define a field as "Required," but security policies may require it to be masked for certain consumers. The contract must reconcile the technical requirement for the field with the legal requirement for privacy.

## Related Topics
* **042 Data Governance:** The overarching framework for data management.
* **075 Schema Registry:** The technical implementation for storing and versioning schemas.
* **110 Observability and Monitoring:** The mechanisms used to alert when contract SLOs are missed.
* **120 Data Lineage:** Tracking the flow of data to understand the impact of contract changes.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
# 076 Unit Tests for Models

Canonical documentation for 076 Unit Tests for Models. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of Unit Tests for Models is to verify the integrity, business logic, and state transitions of data structures in isolation. Models represent the "Source of Truth" for an application's domain; therefore, ensuring their internal consistency is critical for preventing downstream data corruption. This topic addresses the need for deterministic verification of domain invariants, validations, and computed properties without the overhead of external dependencies.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   Verification of domain logic and business rules encapsulated within a model.
*   Validation of data integrity constraints (e.g., required fields, range checks).
*   Testing of computed or derived attributes.
*   State transition logic and lifecycle hooks (logic-only).
*   Mocking and stubbing of model dependencies.

**Out of scope:**
*   Integration testing with databases or persistence layers.
*   End-to-end (E2E) testing of application flows.
*   Performance benchmarking of data retrieval.
*   Verification of third-party framework internals (e.g., testing if the ORM itself works).

## Definitions
| Term | Definition |
|------|------------|
| **Model** | A representation of a domain entity or data structure containing state and behavior. |
| **Invariant** | A condition that must always remain true for a model to be considered in a valid state. |
| **Validation** | The process of checking if the model's data conforms to defined rules before it is processed or persisted. |
| **Computed Property** | A model attribute derived from other attributes rather than being stored directly. |
| **Factory/Builder** | A pattern used to generate valid model instances for testing purposes. |
| **Isolation** | The practice of testing a model without interacting with the database, network, or file system. |

## Core Concepts
### 1. Isolation from Persistence
Unit tests for models must be decoupled from the database. A model test should be able to run in-memory. If a test requires a database connection to pass, it is classified as an integration test, not a unit test.

### 2. Domain Invariants
Models often house the "rules of the world" for an application. Unit tests must ensure that these rules are enforced. For example, if an `Order` model cannot have a negative `TotalAmount`, the unit test must verify that the model rejects such a state.

### 3. State Determinism
Given a specific input or initial state, a model's methods must produce the same output or state transition every time. Tests should focus on the transition from `State A` to `State B` based on a specific action.

## Standard Model
The standard approach for testing models follows the **Arrange-Act-Assert (AAA)** pattern:

1.  **Arrange:** Initialize the model with a known set of attributes. Use a Factory or Builder to ensure the model starts in a valid state.
2.  **Act:** Execute the method or property being tested (e.g., calling `calculate_tax()` or updating a `status` field).
3.  **Assert:** Verify that the model's state matches the expected outcome or that the correct value was returned.

### Validation Testing
Tests should cover both "Happy Path" (valid data) and "Sad Path" (invalid data) scenarios. For every validation rule, there should be at least one test case ensuring the rule triggers correctly.

## Common Patterns
*   **The Factory Pattern:** Using a centralized utility to create model instances with sensible defaults. This prevents test breakage when new required fields are added to a model.
*   **Boundary Testing:** Testing the extreme ends of input ranges (e.g., testing a value of 0, 1, and -1 for a "positive integer" constraint).
*   **Method-Level Isolation:** Testing individual methods of a model in isolation from other methods to pinpoint failures.
*   **Mocking Collaborators:** If a model depends on a service (e.g., a currency converter), that service should be mocked to ensure the test remains a "unit" test.

## Anti-Patterns
*   **Testing the Framework:** Writing tests to verify that a framework's built-in features (like basic field assignments) work. Trust the framework; test your custom logic.
*   **Database Dependency:** Allowing model tests to hit a real or even an in-memory SQL database. This slows down the test suite and introduces environmental fragility.
*   **Fragile Assertions:** Asserting against the entire state of a model when only one attribute was changed. This makes tests brittle to unrelated changes.
*   **Logic in Factories:** Putting complex business logic inside test factories, which can hide bugs or create "mystery guests" in tests.

## Edge Cases
*   **Null/Nil Handling:** How the model behaves when optional or mandatory fields are missing.
*   **Type Coercion:** Ensuring the model handles unexpected data types gracefully (if the language is dynamically typed).
*   **Temporal Logic:** Testing models that rely on the current time (e.g., an `is_expired` property). These require "freezing" or mocking the system clock to remain deterministic.
*   **Concurrency State:** While difficult in unit tests, testing how a model handles rapid, sequential state changes can reveal logic flaws in state machines.

## Related Topics
*   **021 Integration Testing:** Testing how models interact with the database.
*   **044 Mocking and Stubbing:** Techniques for isolating models from external services.
*   **089 Domain-Driven Design (DDD):** Architectural patterns that define how models should be structured.
*   **102 Data Validation Strategies:** Higher-level strategies for ensuring data quality across an application.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
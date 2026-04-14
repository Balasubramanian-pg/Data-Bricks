# 098 Unit Tests for Transformations

Canonical documentation for 098 Unit Tests for Transformations. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of Unit Tests for Transformations is to verify the correctness of discrete data manipulation logic in isolation. In data-centric systems, transformations represent the "business logic" layer where raw data is converted into meaningful information. 

This topic addresses the risks associated with data corruption, incorrect business rule application, and regression during pipeline evolution. By isolating the transformation logic from infrastructure (such as databases, file systems, or network protocols), these tests ensure that the mathematical and logical operations performed on data structures are accurate, predictable, and resilient to change.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   Validation of logic that maps input schemas to output schemas.
*   Verification of data type conversions, aggregations, and filtering logic.
*   Testing of stateless and stateful transformation functions.
*   Methodologies for mocking input data structures.
*   Assertion strategies for structured and semi-structured data.

**Out of scope:**
*   Integration testing (verifying the connection between systems).
*   Performance or load testing of data pipelines.
*   End-to-end (E2E) validation of data lineage.
*   Validation of physical storage layers or hardware-specific optimizations.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Transformation** | A discrete function or operation that accepts a set of input data and produces a set of output data based on predefined rules. |
| **Unit Under Test (UUT)** | The specific transformation function or module being isolated for verification. |
| **Mock Data** | Synthetically constructed data structures used to simulate input for a transformation without requiring external dependencies. |
| **Determinism** | The property of a transformation where the same input always produces the identical output. |
| **Schema Contract** | The formal definition of fields, types, and constraints that the transformation expects as input and guarantees as output. |
| **Side Effect** | Any change to the state of the system or external environment caused by a transformation (ideally minimized in unit-testable transformations). |

## Core Concepts
### 1. Isolation of Logic
The fundamental principle of transformation unit testing is the separation of "how to process" from "how to move" data. A testable transformation should be a pure function or a modular component that accepts a data structure (e.g., a collection, data frame, or record) and returns a new structure.

### 2. Statelessness and Determinism
For a transformation to be effectively unit tested, it should ideally be stateless. If the transformation relies on external state (like a lookup table), that state must be injected or mocked during the test to ensure the test remains deterministic and repeatable.

### 3. Schema-First Validation
Transformations are bound by their input and output schemas. Unit tests must verify not only the values within the data but also that the structure (column names, nesting, data types) adheres to the defined Schema Contract.

## Standard Model
The standard model for testing transformations follows the **Arrange-Act-Assert (AAA)** pattern adapted for data:

1.  **Arrange:** Define a minimal set of input records (Mock Data) that represent the scenario being tested. Define the expected output structure.
2.  **Act:** Pass the Mock Data into the Transformation function.
3.  **Assert:** Compare the actual output against the expected output. This involves:
    *   **Value Assertion:** Checking if calculations (e.g., sums, string concatenations) are correct.
    *   **Schema Assertion:** Checking if the resulting fields and types match the specification.
    *   **Count Assertion:** Ensuring the number of records returned matches the logic (e.g., after a filter or join).

## Common Patterns
*   **The Happy Path:** Testing the transformation with well-formed, standard data to ensure basic logic works as intended.
*   **Boundary Testing:** Testing the minimum and maximum allowable values for numerical or temporal fields.
*   **Null/Empty Handling:** Providing inputs with missing values or empty collections to ensure the transformation handles "No Data" scenarios gracefully without crashing.
*   **Equivalence Partitioning:** Grouping inputs into sets that should be processed identically and testing one representative from each set.
*   **Error Propagation:** Ensuring that invalid data triggers the expected error handling or "dead-letter" logic defined within the transformation.

## Anti-Patterns
*   **Testing with Production Data:** Using actual production data samples in unit tests. This introduces privacy risks (PII), makes tests brittle to data changes, and often results in "heavy" tests that are slow to run.
*   **Infrastructure Coupling:** Including database connections, API calls, or file system paths within the unit test. This transforms a unit test into an integration test.
*   **Testing the Framework:** Writing tests that verify the underlying engine (e.g., verifying that a sorting algorithm works) rather than the specific business logic applied *using* that engine.
*   **Over-Assertion:** Asserting on every single field in a wide dataset when only two fields were transformed. This creates high maintenance overhead when unrelated schema changes occur.

## Edge Cases
*   **Schema Evolution:** How the transformation behaves when the input contains extra fields not defined in the original contract (e.g., "ignore" vs. "pass-through").
*   **Character Encoding:** Handling of non-standard characters, emojis, or different UTF encodings during string transformations.
*   **Timezone Ambiguity:** Transformations involving timestamps where the offset is not explicitly provided or where Daylight Savings Time transitions occur.
*   **Precision Loss:** Scenarios where floating-point arithmetic or type casting (e.g., Decimal to Float) results in unexpected rounding or loss of significant digits.
*   **Large Object Handling:** Transformations involving extremely large strings or nested arrays that may exceed memory limits of the testing environment.

## Related Topics
*   **097 Data Quality Frameworks:** The broader context of ensuring data health.
*   **102 Integration Testing for Data Pipelines:** Testing the movement of data between systems.
*   **045 Schema Registry and Management:** Defining the contracts used by transformations.
*   **Mocking Strategies for Distributed Systems:** Advanced techniques for simulating large-scale data environments.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
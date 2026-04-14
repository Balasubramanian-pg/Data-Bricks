# 100 Schema Validation Tests

Canonical documentation for 100 Schema Validation Tests. This document defines concepts, terminology, and standard usage.

## Purpose
The "100 Schema Validation Tests" framework exists to provide an exhaustive, systematic methodology for verifying the integrity, constraints, and structural requirements of data against a defined schema. In modern distributed systems, schemas serve as the formal contract between producers and consumers. Without a rigorous testing battery, subtle regressions in data structure can lead to downstream system failures, data corruption, and security vulnerabilities.

This topic addresses the problem of "shallow validation," where only basic types are checked, leaving systems exposed to edge cases, boundary violations, and logical inconsistencies.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative. It applies equally to JSON Schema, XML/XSD, SQL DDL, Protobuf, Avro, and other formal modeling languages.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Structural Integrity:** Verification of hierarchy, nesting, and required fields.
* **Type Safety:** Validation of primitive and complex data types.
* **Constraint Enforcement:** Testing of ranges, lengths, and patterns.
* **Logical Consistency:** Cross-field dependencies and conditional requirements.
* **Boundary Analysis:** Testing the limits of defined constraints.

**Out of scope:**
* **Specific vendor implementations:** (e.g., specific configurations for Ajv, Joi, or Pydantic).
* **Network Protocol Testing:** (e.g., HTTP status codes, though the payload validation is in scope).
* **Performance Benchmarking:** The speed of the validator is secondary to the correctness of the test suite.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Schema** | A formal description of the structure, types, and constraints of a data object. |
| **Payload** | The specific instance of data being validated against the schema. |
| **Positive Test** | A test case designed to confirm that valid data is accepted by the schema. |
| **Negative Test** | A test case designed to confirm that invalid data is rejected by the schema. |
| **Cardinality** | The numerical relationship between elements (e.g., minItems, maxItems). |
| **Invariant** | A condition that must remain true for every valid instance of the schema. |
| **Discriminator** | A field used to determine which sub-schema or polymorphic type to apply. |

## Core Concepts
The 100 Schema Validation Tests are built upon three fundamental pillars:

1.  **Exhaustive Boundary Analysis:** Testing not just the "happy path," but the exact limits of every constraint (e.g., if a maximum is 100, testing 99, 100, and 101).
2.  **Negative Space Mapping:** Explicitly defining what the schema *should not* allow, which is often more critical for security than defining what it should allow.
3.  **Contractual Invariance:** Ensuring that the schema acts as a stable interface, where any change to the validation logic is treated as a breaking change to the system contract.

## Standard Model
The "100 Tests" are categorized into five primary domains, each containing 20 specialized test vectors to reach the comprehensive total.

### 1. Structural & Presence Tests (1-20)
Focuses on the "shape" of the data.
*   Missing required fields.
*   Presence of unexpected/additional properties (when `additionalProperties` is false).
*   Deeply nested object structures.
*   Empty payloads (null, empty object, empty array).

### 2. Primitive Type & Format Tests (21-40)
Focuses on the underlying data representation.
*   Type mismatch (e.g., string provided where integer is expected).
*   Format validation (ISO-8601 dates, UUIDs, Email, IPv4/v6).
*   Nullability (testing fields that allow null vs. fields that are merely optional).

### 3. Numeric & Length Constraints (41-60)
Focuses on the quantitative boundaries.
*   Minimum/Maximum values (inclusive and exclusive).
*   String length (min/max characters).
*   Array length (min/max items).
*   MultipleOf constraints (e.g., ensuring a value is a multiple of 0.01 for currency).

### 4. Content & Pattern Matching (61-80)
Focuses on the qualitative data within the fields.
*   Regular Expression (Regex) compliance.
*   Enumeration (Enum) membership.
*   String encoding (UTF-8, ASCII, Base64).
*   Whitespace handling (leading/trailing/internal).

### 5. Logical & Relational Tests (81-100)
Focuses on the inter-dependencies within the schema.
*   Conditional logic (`if-then-else` structures).
*   Dependency validation (if Field A exists, Field B must also exist).
*   Polymorphism/Union types (`oneOf`, `anyOf`, `allOf`).
*   Uniqueness constraints within collections.

## Common Patterns
*   **The "Golden Payload" Pattern:** Maintaining a single, perfectly valid instance of the schema that passes all 100 tests as a baseline.
*   **The "Mutation" Pattern:** Generating negative tests by taking the Golden Payload and programmatically mutating a single field to violate a specific constraint.
*   **Fail-Fast vs. Comprehensive Error Reporting:** A standard approach where the validator returns all 100 potential errors at once rather than stopping at the first failure.

## Anti-Patterns
*   **The God Schema:** Creating a single, massive schema that attempts to validate every possible state of an object across its entire lifecycle, leading to overly complex conditional logic.
*   **Type-Only Validation:** Relying solely on primitive types (string, number) without defining formats or ranges, which allows "legal but nonsensical" data.
*   **Regex Over-Reliance:** Using complex regular expressions for logic that should be handled by structural constraints or dedicated format types.
*   **Silent Failures:** Implementing validation that logs errors but does not reject the payload, effectively rendering the schema a suggestion rather than a contract.

## Edge Cases
*   **Floating Point Precision:** Testing values like `0.1 + 0.2` against a schema requiring `0.3`.
*   **Zero-Width Characters:** Strings that appear empty but contain non-printing Unicode characters.
*   **Large Integers:** Testing numbers that exceed the 64-bit integer limit in systems that default to IEEE 754 floats.
*   **Circular References:** Schemas that define recursive structures (e.g., a folder containing folders) and testing the depth limits to prevent stack overflows.
*   **Timezone Ambiguity:** Validating dates that lack offset information in a system expecting UTC.

## Related Topics
*   **Contract Testing:** Using schema validation as the basis for Consumer-Driven Contracts (CDC).
*   **Data Governance:** The organizational policy governing how schemas are versioned and retired.
*   **Fuzz Testing:** Automated generation of random data to discover unhandled edge cases in schema definitions.
*   **Semantic Versioning (SemVer):** Applying versioning logic to schemas to manage breaking changes in validation rules.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
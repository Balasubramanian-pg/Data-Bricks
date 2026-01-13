# 074 Passing Parameters in ADF

Canonical documentation for 074 Passing Parameters in ADF. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of parameterization within an Application Development Framework (ADF) is to decouple the execution logic from the data configuration. By passing parameters, an orchestration engine can achieve reusability, maintainability, and environmental portability. This mechanism addresses the problem of "hard-coding" values, allowing a single logic definition (e.g., a pipeline or activity) to operate across diverse datasets, connection strings, and execution contexts without manual intervention.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Mechanisms for data transfer between hierarchical execution layers.
* The lifecycle of a parameter from definition to consumption.
* Theoretical boundaries of parameter scope and visibility.
* Evaluation of expressions and dynamic content.

**Out of scope:**
* Specific vendor-specific expression languages (e.g., specific T-SQL, JSON, or Python syntax).
* UI-specific navigation steps for particular cloud providers.
* Performance tuning for specific database engines.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Parameter** | An external input defined at the start of a component's lifecycle that remains constant throughout that specific execution. |
| **Variable** | An internal state holder that can be modified during the execution of a process. |
| **Binding** | The process of mapping an output from one component to the input (parameter) of another. |
| **Expression** | A formulaic string evaluated at runtime to produce a value for a parameter. |
| **Scope** | The boundary within which a parameter or variable is accessible and valid. |
| **Late Binding** | The practice of resolving parameter values at the last possible moment (runtime) rather than at design time. |

## Core Concepts

### 1. Hierarchical Inheritance
Parameters typically flow from the highest level of abstraction down to the most granular. In a standard orchestration model, this follows the path:
`Trigger/Caller -> Parent Container -> Child Activity -> Linked Resource.`

### 2. Immutability of Parameters
Unlike variables, parameters are generally considered immutable once an execution instance has started. To change a value during execution, the value must be transitioned into a variable state.

### 3. Dynamic Evaluation
Parameters are rarely static strings. They are often the result of expressions that concatenate system variables (like execution time), metadata, and upstream outputs.

## Standard Model
The standard model for passing parameters relies on a **Contract-Based Interface**. 

1.  **Definition:** The receiving component defines a "formal parameter" (a placeholder with a specific data type).
2.  **Assignment:** The calling component provides an "actual parameter" (the value).
3.  **Resolution:** The orchestration engine resolves the value at the moment the component is instantiated.

This model ensures that the internal logic of a component is "black-boxed"—it does not need to know where the value came from, only that the value conforms to the expected type and schema.

## Common Patterns

### Cascading Parameters
Passing a value through multiple layers of nested containers. A global parameter (e.g., `EnvironmentID`) is passed from the main pipeline to a sub-pipeline, and finally to a specific data-copy activity.

### Metadata-Driven Execution
Using a parameter to pass a JSON object or a database record that contains multiple configuration settings. This reduces the number of individual parameters and allows for schema-flexible configurations.

### Secure String Passing
A pattern where sensitive parameters (credentials, keys) are passed using specialized "Secret" types, ensuring the values are masked in logs and encrypted at rest.

## Anti-Patterns

*   **Hard-coded Overrides:** Defining parameters but then overriding them with static values inside the component logic, defeating the purpose of parameterization.
*   **Deep Coupling:** Designing a child component that expects a parameter specifically named after a parent component's internal variable, breaking modularity.
*   **The "God Parameter":** Passing a single, massive string or object containing unrelated configurations, making debugging and validation difficult.
*   **Circular Referencing:** Attempting to pass a parameter to a component that is responsible for defining that same parameter.

## Edge Cases

*   **Null vs. Empty String:** Orchestration engines often treat `null` and `""` differently. Failure to handle null inputs in a parameter-driven expression can lead to runtime failures.
*   **Type Coercion:** When a numeric parameter is passed into a string-based expression, the engine may perform implicit casting, which can lead to precision loss or formatting errors (e.g., dates).
*   **Recursive Depth:** In systems allowing recursive calls, passing parameters through too many layers of recursion can lead to stack overflow or memory exhaustion in the orchestration engine.
*   **Special Character Escaping:** Parameters containing delimiters (commas, semicolons) or quotes can break the parsing of the expression language if not properly escaped.

## Related Topics
*   **075 Variable Management:** The counterpart to parameters, focusing on mutable state.
*   **102 Metadata-Driven Orchestration:** Advanced usage of parameters to drive entire system behaviors.
*   **201 Expression Language Syntax:** The mechanics of writing the formulas that populate parameters.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
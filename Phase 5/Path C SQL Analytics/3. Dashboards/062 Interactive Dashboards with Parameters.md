# 062 Interactive Dashboards with Parameters

Canonical documentation for 062 Interactive Dashboards with Parameters. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of interactive dashboards with parameters is to transition data visualization from a static reporting format to a dynamic exploration environment. This topic addresses the limitation of "one-size-fits-all" views by allowing end-users to inject variables into the data processing pipeline. By decoupling the visualization logic from hard-coded values, parameters enable multi-dimensional analysis, "what-if" scenario modeling, and personalized data consumption without requiring modifications to the underlying codebase or schema.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Variable Injection:** The mechanism by which user-defined values influence data queries or calculations.
* **State Management:** How the dashboard maintains and propagates parameter values across multiple visual components.
* **User Interface Controls:** The abstraction of input methods (sliders, dropdowns, text fields) that interface with parameters.
* **Execution Logic:** The lifecycle of a parameterized request from input to re-rendering.

**Out of scope:**
* **Specific Vendor Implementations:** Proprietary syntax for tools like Tableau, Power BI, or Looker.
* **Database Optimization:** Specific indexing strategies for SQL or NoSQL backends, though the concept of query performance is acknowledged.
* **Visual Design Theory:** General UI/UX principles regarding color theory or layout, unless directly related to parameter placement.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Parameter** | A developer-defined variable that acts as a placeholder for a value provided by the end-user at runtime. |
| **Control** | The UI element (e.g., toggle, date picker) that allows a user to interact with and modify a parameter. |
| **Binding** | The logical connection between a parameter and a specific data query, calculation, or visual property. |
| **Default Value** | The initial state of a parameter used to render the dashboard before user intervention. |
| **Scope** | The boundary within which a parameter value is active (e.g., a single chart, a page, or the entire dashboard). |
| **Dynamic Calculation** | A formula or measure that re-evaluates its output based on the current state of one or more parameters. |

## Core Concepts

### 1. The Parameter Lifecycle
The lifecycle begins with the **Definition Phase**, where the developer specifies the data type (string, integer, date) and allowable values. It moves to the **Interaction Phase**, where the user modifies the control. Finally, the **Execution Phase** triggers a refresh of the bound components using the new value.

### 2. Parameter vs. Filter
While often used interchangeably, they are distinct:
* **Filters** narrow down an existing dataset (e.g., "Show only 'Region = North'").
* **Parameters** are variables that can be used *outside* of simple filtering, such as changing a growth rate in a projection or switching the currency of all displayed values.

### 3. Reactive State
Interactive dashboards rely on a reactive model. When a parameter changes, the system must identify all dependent objects (queries, labels, calculations) and update them in the correct order to ensure data consistency.

## Standard Model
The standard model for parameterized dashboards follows a three-tier architecture:

1.  **Input Layer (The Control):** Captures user intent. It must validate that the input matches the parameter's expected data type.
2.  **Logic Layer (The Binding):** The parameter value is passed into a transformation engine. This might be a SQL `WHERE` or `CASE` clause, a functional programming script, or a spreadsheet-style formula.
3.  **Presentation Layer (The Visualization):** The visual components subscribe to the logic layer. When the logic layer updates, the presentation layer re-renders to reflect the new state.

## Common Patterns

### Global Date Range
A single date-picker control that updates every visualization on the dashboard simultaneously, ensuring a synchronized temporal view of the data.

### "What-If" Analysis
Using sliders to adjust variables—such as interest rates, churn rates, or price points—to see the projected impact on key performance indicators (KPIs) in real-time.

### Dynamic Dimension/Measure Switching
Allowing the user to choose which metric (e.g., Revenue vs. Volume) or which dimension (e.g., Product Category vs. Geography) is displayed on the axes of a chart.

### Top-N Selection
A parameter that allows users to define the granularity of a list (e.g., "Show me the Top 5, 10, or 20 customers").

## Anti-Patterns

### The Paradox of Choice (Over-Parameterization)
Including too many parameters can overwhelm the user, leading to "analysis paralysis" and increasing the likelihood of the user selecting a combination of values that produces nonsensical results.

### Unbound Parameters
Defining parameters that are visible to the user but are not logically tied to any data updates, leading to a "broken" user experience.

### Lack of Input Validation
Allowing users to input values that the underlying data source cannot handle (e.g., entering text into a numeric parameter), which can trigger unhandled system errors.

### Performance Blindness
Binding parameters directly to heavy, unoptimized queries without considering the latency of the re-render. This results in a "laggy" experience where the dashboard becomes unresponsive after every click.

## Edge Cases

### Null and Empty States
How the dashboard behaves when a parameter is cleared or when a selected parameter value returns no data. A robust system must provide a "No data found" state rather than a blank screen or error.

### Circular Dependencies
A scenario where Parameter A influences the available values of Parameter B, which in turn influences Parameter A. This must be architecturally prevented to avoid infinite loops.

### URL Parameter Injection
When parameters are passed via URL strings, security risks (such as injection attacks) and state-sharing issues (long URLs being truncated) must be managed.

### Multi-Value Conflict
When a parameter allows multiple selections (e.g., selecting three specific cities), the underlying logic must be capable of handling arrays or sets rather than single scalar values.

## Related Topics
* **024 Data Visualization Best Practices:** Fundamental rules for clear data communication.
* **089 Query Optimization:** Techniques for ensuring fast response times in data-heavy environments.
* **112 User Session Management:** How parameter states are saved or cleared when a user logs out.
* **145 Reactive Programming:** The underlying programming paradigm for state-based UI updates.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
# 023 Reusable Gold Models

Canonical documentation for 023 Reusable Gold Models. This document defines concepts, terminology, and standard usage.

## Purpose
The 023 Reusable Gold Models standard addresses the challenges of data fragmentation, redundant computation, and inconsistent business logic across an organization. In a multi-layered data architecture, the "Gold" layer represents the final stage of data transformation where data is curated for end-user consumption. 

The purpose of this standard is to ensure that these models are designed for maximum reusability across diverse business units, preventing the proliferation of "siloed" logic and ensuring a single version of truth. By centralizing complex business rules into reusable assets, organizations reduce technical debt and improve the reliability of downstream analytics and machine learning applications.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Architectural principles for the final consumption layer.
* Standards for business logic encapsulation and modularity.
* Theoretical frameworks for dimensional and flattened data structures.
* Governance requirements for "Gold" status designation.

**Out of scope:**
* Specific vendor implementations (e.g., Databricks, Snowflake, BigQuery).
* Ingestion patterns (Bronze) or intermediate cleaning patterns (Silver).
* Specific programming languages or transformation frameworks.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| Gold Model | A high-quality, business-level dataset that has undergone final transformation, aggregation, and validation. |
| Reusability | The characteristic of a data model that allows it to serve multiple downstream use cases without modification to its core logic. |
| Semantic Layer | A representation of data that translates complex structures into business-friendly terms, often built upon Gold Models. |
| Grain | The level of detail or the definition of what a single row represents in a model. |
| Conformed Dimension | A dimension that has the same meaning and content when referred to by different fact tables. |
| Idempotency | The property where an operation can be applied multiple times without changing the result beyond the initial application. |

## Core Concepts

### Business Logic Encapsulation
Reusable Gold Models must encapsulate all complex business calculations (e.g., "Active Customer" definitions, "Net Revenue" formulas). Downstream consumers should not need to re-apply filters or logic to arrive at standard business metrics.

### High Data Quality and Trust
A model is only "Gold" if it meets strict data quality thresholds. This includes completeness, accuracy, and timeliness. Reusability depends on trust; if a model is perceived as unreliable, users will revert to creating their own siloed logic.

### Schema Stability
Gold models serve as a contract between data engineering and data consumers. Changes to the schema must be managed through strict versioning to prevent breaking downstream reports or applications.

### Decoupling from Source
Gold models should be agnostic of the source system's structure. They are organized around business entities (e.g., Customer, Product, Sale) rather than source table structures.

## Standard Model

The standard for 023 Reusable Gold Models follows a hybrid approach depending on the consumption pattern:

1.  **Dimensional Modeling (Star Schema):** The preferred model for BI and exploratory analysis. It consists of central Fact tables (quantitative data) linked to multiple Dimension tables (descriptive attributes). This promotes reusability through *Conformed Dimensions*.
2.  **One Big Table (OBT):** The preferred model for high-performance machine learning and specific reporting tools. This involves denormalizing facts and dimensions into a single wide table to eliminate join overhead at query time.
3.  **Aggregated Summaries:** Pre-calculated tables at specific grains (e.g., Monthly Sales by Region) to support high-concurrency dashboards.

## Common Patterns

### The "Hub and Spoke" Consumption
A single, authoritative Gold Model (the Hub) serves as the source for multiple specialized downstream views or extracts (the Spokes). The Hub contains the core logic, while Spokes handle specific formatting or filtering for niche use cases.

### Point-in-Time Snapshots
Capturing the state of a Gold Model at specific intervals (e.g., Month-End) to allow for historical trend analysis and "as-was" reporting.

### Slowly Changing Dimensions (SCD)
Implementing Type 2 SCDs within Gold Models to track historical changes in descriptive attributes, ensuring that reusability extends to historical accuracy.

## Anti-Patterns

*   **Logic Leakage:** Allowing business logic to reside in the BI tool (e.g., Power BI measures or Tableau calculations) rather than in the Gold Model. This makes the logic non-reusable for other tools.
*   **The "Everything Table":** Creating a single table with hundreds of columns and no clear grain, leading to performance degradation and user confusion.
*   **Hard-Coded Filters:** Embedding specific, temporary business exclusions (e.g., `WHERE region != 'Test'`) directly into the Gold Model instead of using metadata-driven flags.
*   **Circular Dependencies:** Creating Gold Models that depend on other Gold Models in a way that creates a loop, complicating orchestration and lineage.

## Edge Cases

*   **Late-Arriving Data:** Handling data that arrives after a Gold Model snapshot has been generated. The standard requires a "re-statement" policy or an "inference" pattern to maintain consistency.
*   **Multi-Tenant Isolation:** In scenarios where data must be isolated by client or region, Gold Models may use "Row-Level Security" (RLS) or "Dynamic Data Masking" to maintain a single reusable structure while enforcing data privacy.
*   **Schema Evolution:** When a business entity changes fundamentally (e.g., a company merger), the Gold Model must support "Bridge Tables" or versioned schemas to allow for both old and new world views.

## Related Topics
*   **021 Silver Layer Normalization:** The prerequisite stage for Gold Model creation.
*   **045 Data Governance Framework:** The policy governing who can certify a model as "Gold."
*   **088 Semantic Layer Integration:** How Gold Models interface with BI semantic layers.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
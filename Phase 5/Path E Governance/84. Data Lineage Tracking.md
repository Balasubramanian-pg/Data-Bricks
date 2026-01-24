# 084 Data Lineage Tracking

Canonical documentation for 084 Data Lineage Tracking. This document defines concepts, terminology, and standard usage.

## Purpose
Data Lineage Tracking exists to provide a transparent, auditable, and reproducible record of data’s lifecycle. In complex data ecosystems, data undergoes numerous transformations, aggregations, and migrations. Without lineage, organizations suffer from "black box" processing, where the origin, quality, and reliability of information are unknown.

This topic addresses the problem space of data governance, impact analysis, and root cause determination. It ensures that stakeholders can verify the integrity of data, comply with regulatory requirements (such as GDPR or BCBS 239), and understand the downstream consequences of upstream changes.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Metadata Persistence:** The recording of data origins and transformations.
* **Directionality:** Tracking data flow both forward (impact analysis) and backward (provenance).
* **Granularity Levels:** Defining lineage at the system, dataset, and attribute levels.
* **Temporal Tracking:** Capturing how lineage evolves over time (versioning).

**Out of scope:**
* **Data Orchestration:** The actual execution of data pipelines (though lineage often integrates with it).
* **Data Quality Validation:** The act of checking if data is "good," though lineage helps identify where "bad" data originated.
* **Specific vendor implementations:** Proprietary features of specific catalogs or ETL tools.

## Definitions
| Term | Definition |
|------|------------|
| **Lineage** | The complete path of data from its point of origin to its final consumption point, including all transformations. |
| **Provenance** | A subset of lineage focusing specifically on the origin and ownership of a data entity. |
| **Upstream** | Entities or processes that occur earlier in the data lifecycle relative to the current point. |
| **Downstream** | Entities or processes that occur later in the data lifecycle, consuming the output of the current point. |
| **Transformation** | Any logic, calculation, or movement applied to data that alters its state, format, or location. |
| **Node** | A discrete entity in a lineage graph (e.g., a table, a file, or a report). |
| **Edge** | The relationship or process connecting two nodes, representing the flow of data. |
| **Granularity** | The level of detail at which lineage is captured (e.g., Table-level vs. Column-level). |

## Core Concepts

### 1. Directional Analysis
Lineage tracking supports two primary analytical directions:
*   **Backward Lineage (Provenance):** Starting from a data point and looking "left" to find its source. Used for debugging and verifying data integrity.
*   **Forward Lineage (Impact Analysis):** Starting from a source and looking "right" to see which reports or applications will be affected if the source changes.

### 2. Metadata Layers
Lineage is essentially "metadata about movement." It typically operates across three layers:
*   **Business Lineage:** High-level view showing how data flows between business domains (e.g., "Sales Data" to "Financial Reporting").
*   **Logical Lineage:** Shows the relationships between logical entities (e.g., "Customer Table" to "Invoice View").
*   **Physical Lineage:** The most granular view, showing specific database instances, file paths, and actual code executions.

### 3. Granularity
The utility of lineage is determined by its resolution:
*   **System Level:** Data moving between applications.
*   **Dataset/Table Level:** Data moving between tables or files.
*   **Attribute/Column Level:** The most precise level, showing how a specific field (e.g., `Total_Price`) is calculated from other fields.

## Standard Model
The standard model for Data Lineage Tracking is represented as a **Directed Acyclic Graph (DAG)**.

1.  **Entities (Nodes):** Represent data objects (Sources, Sinks, Intermediary tables).
2.  **Processes (Edges):** Represent the transformations or movements.
3.  **Attributes:** Metadata attached to nodes and edges (e.g., timestamps, user IDs, transformation logic, schema versions).

In this model, every data transformation must be recorded as a transition from `State A` to `State B` via `Process P`. To maintain a "Source of Truth," the model must be immutable; rather than overwriting lineage, new versions are appended to reflect changes in the pipeline logic.

## Common Patterns

### Passive Collection (Log-Based)
Lineage is extracted by parsing system logs, SQL query histories, or job execution metadata. This is non-intrusive but relies on the quality of the logs.

### Active Collection (Instrumentation)
The data pipeline itself is programmed to "report" its lineage to a central repository during execution. This is highly accurate but requires modifying the processing code.

### Design-Time Lineage
Lineage is derived from the code or visual mapping in an ETL tool before the data even moves. This shows the *intended* flow.

### Run-Time Lineage
Lineage is captured during actual execution, reflecting the *actual* flow, including dynamic variables or conditional logic that design-time lineage might miss.

## Anti-Patterns

*   **Manual Documentation:** Relying on spreadsheets or static diagrams to track lineage. These become obsolete the moment the first change is made to the production environment.
*   **Siloed Lineage:** Tracking lineage within a single tool (e.g., only inside the ETL tool) while ignoring the upstream source systems and downstream BI tools.
*   **Over-Granularity (The "Noise" Trap):** Capturing every single temporary table or transient state in a complex pipeline, making the lineage graph unreadable and difficult to maintain.
*   **Ignoring Transformation Logic:** Tracking that "Table A moved to Table B" without recording *how* it was transformed (e.g., the SQL or code used).

## Edge Cases

*   **Black Box Transformations:** When data enters a third-party API or a proprietary compiled binary where the internal logic is hidden. Lineage must mark these as "opaque" nodes.
*   **Circular Dependencies:** While the standard model is a DAG, poorly designed systems may have circular references (e.g., Table A updates Table B, which then updates Table A). Lineage must detect and flag these as risks.
*   **Data Masking and PII:** Lineage must track the flow of sensitive data even when it is masked or anonymized, ensuring that the "masked" version is still linked to its "sensitive" origin for compliance purposes.
*   **Late-Arriving Data:** When data from the past is injected into a current pipeline, lineage must account for the temporal discrepancy to avoid misleading audit trails.

## Related Topics
*   **012 Data Governance:** The overarching framework that lineage supports.
*   **045 Metadata Management:** The technical discipline of managing the data that lineage relies upon.
*   **092 Impact Analysis:** The process of using lineage to predict the consequences of changes.
*   **101 Regulatory Compliance:** The legal drivers (GDPR, HIPAA) for maintaining lineage.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
# 027 Medallion Best Practices

Canonical documentation for 027 Medallion Best Practices. This document defines concepts, terminology, and standard usage.

## Purpose
The 027 Medallion Best Practices framework exists to provide a structured, multi-layered approach to data architecture. It addresses the challenges of data quality, lineage, and reliability in complex data ecosystems. By segregating data into distinct processing stages, organizations can ensure that raw data is preserved, intermediate transformations are standardized, and final outputs are optimized for business consumption. This framework mitigates the risks of data corruption, loss of historical context, and inconsistent reporting.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   Architectural layering (Bronze, Silver, Gold)
*   Data quality enforcement and validation strategies
*   Schema evolution and versioning principles
*   Idempotency and reproducibility in data pipelines
*   Metadata management across layers

**Out of scope:**
*   Specific vendor implementations (e.g., Databricks, Snowflake, BigQuery)
*   Specific programming language syntax (e.g., Python, SQL, Scala)
*   Hardware or infrastructure provisioning

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Bronze Layer** | The raw ingestion layer where data is stored in its native format, acting as a historical archive. |
| **Silver Layer** | The refined layer where data is cleansed, normalized, and integrated into a consistent schema. |
| **Gold Layer** | The consumption layer where data is aggregated and optimized for specific business use cases or analytics. |
| **Idempotency** | The property of a process where multiple executions with the same input produce the same output without side effects. |
| **Schema Drift** | The phenomenon where source systems change their data structure (adding, removing, or modifying columns) without prior notice. |
| **Data Lineage** | The chronological record of data's movement and transformation from source to destination. |
| **Quality Gate** | A programmatic check that prevents data from moving to the next layer if it fails predefined validation rules. |

## Core Concepts
The 027 Medallion framework is built upon four fundamental pillars:

1.  **Immutability of Raw Data:** Once data enters the Bronze layer, it should never be modified. This ensures a "Source of Truth" for re-processing and auditing.
2.  **Incremental Refinement:** Data quality should improve monotonically as it moves through the layers. Each layer adds value through cleaning, enrichment, or aggregation.
3.  **Separation of Concerns:** Each layer has a specific responsibility. Bronze handles ingestion; Silver handles standardization; Gold handles business logic.
4.  **Auditability:** Every transformation must be traceable. The system must be able to explain how a specific value in the Gold layer was derived from the Bronze layer.

## Standard Model
The standard 027 Medallion model follows a linear progression of data maturity:

### 1. Bronze (Raw/Ingestion)
*   **Input:** Source systems (APIs, DB Logs, IoT, Files).
*   **State:** Raw, unstructured, or semi-structured.
*   **Retention:** Permanent or long-term (subject to compliance).
*   **Logic:** Minimal. Includes metadata such as `ingestion_timestamp` and `source_file_name`.

### 2. Silver (Cleansed/Normalized)
*   **Input:** Bronze layer.
*   **State:** Structured, cleaned, and joined.
*   **Logic:** Data type casting, null handling, deduplication, and initial schema enforcement. This layer often represents a "Digital Twin" of the business entities (e.g., Customers, Orders).

### 3. Gold (Curated/Business)
*   **Input:** Silver layer.
*   **State:** Highly aggregated, denormalized, or formatted for specific tools.
*   **Logic:** Complex business calculations, KPIs, and read-optimized structures (e.g., Star Schema or Flat Tables).

## Common Patterns
*   **The Replay Pattern:** The ability to delete the Silver and Gold layers and re-generate them entirely from the Bronze layer in the event of logic errors.
*   **SCD Type 2 in Silver:** Implementing Slowly Changing Dimensions in the Silver layer to track historical changes of entities over time.
*   **Write-Audit-Publish (WAP):** Writing data to a temporary location, performing quality checks, and only then merging it into the production Silver/Gold tables.
*   **Flattening:** Converting nested JSON/XML structures from Bronze into relational columns in Silver.

## Anti-Patterns
*   **Skipping Layers:** Moving data directly from Bronze to Gold. This bypasses standardization and creates "silos" of business logic that are difficult to maintain.
*   **Business Logic in Bronze:** Applying filters or aggregations during the ingestion phase. This loses raw data context and prevents future re-processing.
*   **Circular Dependencies:** A Silver table depending on a Gold table, or a Silver table depending on another Silver table in a way that creates a loop.
*   **Hard Deletes:** Deleting records in Silver or Gold without a trace. Use "Soft Deletes" (is_deleted flags) to maintain lineage.

## Edge Cases
*   **Late-Arriving Data:** When data for a previous time period arrives after the Gold layer has already been aggregated. Best practice requires a "Look-back" window or a reprocessing trigger.
*   **Schema Evolution (Breaking Changes):** When a source system removes a column. The 027 standard dictates that the pipeline should fail gracefully or quarantine the data rather than corrupting the Silver layer.
*   **PII/GDPR Compliance:** Handling "Right to be Forgotten" requests. While Bronze is immutable, compliance may require specific "masking" views or targeted deletions in the raw layer, which contradicts the standard immutability principle but is legally required.

## Related Topics
*   **Data Governance:** The overarching policy framework that defines who can access which layer.
*   **Data Quality Frameworks:** Specific tools used to implement Quality Gates.
*   **CI/CD for Data:** The automation of code deployment for the transformations between layers.
*   **Data Mesh:** An alternative organizational approach where Medallion layers are managed by domain-specific teams.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
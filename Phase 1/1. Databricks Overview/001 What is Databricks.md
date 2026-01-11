# 001 What is Databricks

Canonical documentation for 001 What is Databricks. This document defines concepts, terminology, and standard usage.

## Purpose
Databricks exists to resolve the architectural complexity and operational friction inherent in traditional fragmented data stacks. Historically, organizations were forced to maintain separate environments for data warehousing (structured data for BI) and data lakes (unstructured data for ML). This separation created data silos, inconsistent governance, and significant latency in moving data between systems.

Databricks addresses this by providing a unified platform that consolidates data engineering, data science, machine learning, and business analytics into a single collaborative environment. It is the primary realization of the **Data Lakehouse** architecture.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative, focusing on the architectural role of Databricks rather than specific cloud provider nuances.

## Scope
**In scope:**
* The Data Lakehouse architectural paradigm.
* Core functional layers (Storage, Compute, Governance, Workspace).
* Open-source foundations (Apache Spark, Delta Lake, MLflow).
* Collaborative lifecycle management for data and AI.

**Out of scope:**
* Cloud-specific pricing models (AWS, Azure, GCP).
* Step-by-step UI tutorials or "How-to" guides.
* Specific API syntax or library documentation.

## Definitions
| Term | Definition |
|------|------------|
| **Data Lakehouse** | An architectural pattern that combines the cost-effective, flexible storage of a data lake with the performance, ACID transactions, and schema enforcement of a data warehouse. |
| **Delta Lake** | An open-source storage layer that brings reliability to data lakes by providing ACID transactions, scalable metadata handling, and unified streaming/batch data processing. |
| **Unity Catalog** | A unified governance solution for all data and AI assets in the Lakehouse, providing centralized access control, auditing, and lineage. |
| **Photon** | A high-performance, vectorized query engine written in C++ designed to accelerate SQL workloads and data processing. |
| **Workspace** | The collaborative software-as-a-service (SaaS) environment where users interact with notebooks, experiments, and SQL editors. |
| **Medallion Architecture** | A data design pattern used to organize data into layers (Bronze, Silver, Gold) based on the quality and structure of the data. |

## Core Concepts

### 1. Unified Analytics
Databricks eliminates the "wall" between data engineers and data scientists. By providing a shared environment that supports multiple languages (SQL, Python, R, Scala), it allows different personas to work on the same underlying data without moving or copying it.

### 2. Open Standards
The platform is built upon open-source technologies. This prevents vendor lock-in and ensures that the data stored in the Lakehouse remains accessible via open formats (Parquet/Delta) even outside of the Databricks environment.

### 3. Decoupled Storage and Compute
Databricks follows a cloud-native architecture where compute resources are ephemeral and scale independently of the persistent storage. This allows for massive horizontal scaling and cost optimization based on specific workload requirements.

### 4. Integrated Governance
Governance is not an afterthought but a core layer. Through Unity Catalog, the platform provides a single pane of glass for managing permissions across files, tables, machine learning models, and dashboards.

## Standard Model
The standard model for Databricks implementation is the **Lakehouse Architecture** utilizing the **Medallion Design Pattern**:

1.  **Ingestion (Bronze):** Raw data is ingested from source systems (APIs, Databases, IoT) into the Lakehouse in its original format.
2.  **Refinement (Silver):** Data is cleaned, filtered, and joined to create a "source of truth" for downstream analysis.
3.  **Aggregation (Gold):** Data is transformed into business-level aggregates and structured for high-performance consumption by BI tools and ML models.

## Common Patterns
*   **ETL/ELT Pipelines:** Using Delta Live Tables (DLT) to automate the movement and transformation of data.
*   **Streaming/Batch Unification:** Treating real-time streams and historical batch data as the same table structure.
*   **MLOps:** Using MLflow to track experiments, package code into reproducible runs, and manage the model lifecycle from development to production.
*   **SQL Analytics:** Using Serverless SQL warehouses to provide data analysts with a familiar environment for querying the data lake with warehouse-like performance.

## Anti-Patterns
*   **Data Siloing:** Creating separate workspaces or storage accounts for different teams that prevent cross-functional collaboration and unified governance.
*   **Small File Problem:** Frequently writing very small files to the data lake, which degrades metadata performance (mitigated by using `OPTIMIZE` and `Auto-Optimize`).
*   **Over-Provisioning:** Leaving compute clusters running when idle or using oversized instances for small workloads.
*   **Treating Lakehouse as a Traditional RDBMS:** Attempting to perform high-frequency, single-row updates/deletes (OLTP) rather than focusing on analytical (OLAP) workloads.

## Edge Cases
*   **Extremely Low Latency Requirements:** While Databricks is optimized for high-throughput and near-real-time streaming, applications requiring sub-millisecond response times (e.g., high-frequency trading) may require a specialized OLTP database.
*   **Air-Gapped Environments:** Databricks is a cloud-native SaaS platform; implementing it in strictly offline, on-premises, or air-gapped environments requires significant architectural workarounds and may not support all features.
*   **Small Data Overhead:** For datasets that fit entirely in memory on a single machine (e.g., < 100MB), the distributed overhead of Spark may result in slower performance than a local Python script.

## Related Topics
*   **002 Apache Spark Fundamentals:** The distributed processing engine powering Databricks.
*   **003 Delta Lake Protocol:** The storage layer providing reliability.
*   **004 Unity Catalog and Governance:** The framework for data security and lineage.
*   **005 Machine Learning on Databricks:** Utilizing MLflow and Feature Stores.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2024-05-22 | Initial AI-generated canonical documentation |
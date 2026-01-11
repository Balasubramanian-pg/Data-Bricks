# 001 What is Databricks

Canonical documentation for 001 What is Databricks. This document defines concepts, terminology, and standard usage.

## Purpose
Databricks exists to resolve the historical friction between data engineering, data science, and business intelligence. Traditionally, organizations maintained separate environments: **Data Lakes** (for massive scale, unstructured data, and machine learning) and **Data Warehouses** (for structured data, performance, and SQL-based reporting). 

This topic addresses the "Data Silo" problem by defining a unified architectural paradigm—the **Data Lakehouse**. Databricks provides a single, collaborative platform that combines the performance and governance of a data warehouse with the flexibility and cost-effectiveness of a data lake.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative, focusing on the architectural principles of the Databricks platform rather than specific cloud provider integrations.

## Scope
**In scope:**
* The Data Lakehouse architectural paradigm.
* Core components of the unified platform (Processing, Storage, Governance, and Orchestration).
* Theoretical boundaries of collaborative data environments.
* Open-source foundations (Spark, Delta Lake, MLflow).

**Out of scope:**
* Step-by-step UI tutorials or "How-to" guides.
* Specific pricing models or cloud-provider-specific networking (AWS/Azure/GCP).
* Third-party library documentation.

## Definitions
| Term | Definition |
|------|------------|
| **Lakehouse** | An architectural pattern that implements data warehousing features (ACID transactions, schema enforcement) directly on top of low-cost cloud object storage. |
| **Delta Lake** | An open-source storage layer that brings reliability to data lakes by providing ACID transactions, scalable metadata handling, and unified streaming/batch processing. |
| **Apache Spark** | A multi-language engine for executing data engineering, data science, and machine learning on single-node machines or clusters. |
| **Unity Catalog** | A unified governance solution for all data and AI assets in the Lakehouse, providing centralized access control and lineage. |
| **Workspace** | An interactive environment that enables data professionals to collaborate on notebooks, experiments, and jobs. |
| **Photon** | A high-performance, vectorized query engine designed to accelerate SQL workloads within the Lakehouse. |

## Core Concepts

### 1. The Unified Data Lakehouse
The Lakehouse is the foundational concept of Databricks. It eliminates the need to move data between a data lake and a downstream data warehouse. By using open formats (like Parquet) and a transactional layer (Delta Lake), the platform allows BI tools and ML frameworks to query the same source of truth simultaneously.

### 2. Decoupled Storage and Compute
Databricks adheres to the principle of separating storage from compute. Data resides in the customer's cloud object storage, while compute resources (clusters or SQL warehouses) are spun up on demand to process that data. This allows for independent scaling and cost optimization.

### 3. Open Standards and Interoperability
The platform is built on open-source technologies. This prevents vendor lock-in, as the underlying data formats (Delta/Parquet) and processing logic (Spark/SQL) are portable and accessible by other systems.

### 4. Collaborative Intelligence
Databricks integrates notebooks, automated orchestration (Workflows), and model tracking (MLflow) into a single interface. This ensures that data engineers, analysts, and data scientists work on the same version of the data with shared logic.

## Standard Model

The standard model for Databricks implementation is the **Medallion Architecture**. This is a data design pattern used to logically organize data as it flows through the Lakehouse:

1.  **Bronze (Raw):** The landing zone for raw data in its native format. No transformations are applied; it serves as a historical record.
2.  **Silver (Validated/Filtered):** Data is cleaned, normalized, and joined. This layer provides a "Single Source of Truth" for ad-hoc analysis and data science.
3.  **Gold (Enriched/Aggregated):** Data is modeled for consumption. It is structured into business-level aggregates or star schemas optimized for BI reporting and executive dashboards.

## Common Patterns

*   **ETL/ELT Pipelines:** Using Spark or Delta Live Tables (DLT) to ingest and transform data from various sources into the Medallion layers.
*   **Streaming and Batch Unification:** Using the same code and logic to process real-time data streams and historical batch data.
*   **Machine Learning Lifecycle:** Using MLflow to track experiments, package code into reproducible runs, and manage model versioning and deployment.
*   **SQL Analytics:** Using SQL Warehouses to provide analysts with a familiar environment for querying the Lakehouse with low latency.

## Anti-Patterns

*   **The "Data Swamp":** Ingesting data into the Lakehouse without implementing schema enforcement or the Medallion architecture, leading to unsearchable and low-quality data.
*   **Over-Provisioning:** Maintaining "Always-On" compute clusters for workloads that are intermittent or could be handled by Serverless compute.
*   **Proprietary Silos:** Moving data out of the Lakehouse into a proprietary data warehouse for BI, which re-introduces data duplication and synchronization issues.
*   **Ignoring Governance:** Managing permissions at the storage bucket level rather than using a centralized governance layer (Unity Catalog), leading to security gaps.

## Edge Cases

*   **Small Data Workloads:** While Databricks scales to petabytes, using it for very small datasets (e.g., a few megabytes) may introduce unnecessary overhead compared to a simple SQL database.
*   **Highly Regulated Air-Gapped Environments:** While Databricks supports various compliance standards, implementations in environments with no internet egress require specialized "Private Link" configurations and manual management.
*   **Real-time Point Lookups:** While the Lakehouse is optimized for analytical queries (OLAP), it is not a replacement for a low-latency NoSQL database (OLTP) for millisecond-level point lookups in a web application.

## Related Topics
*   **002 Delta Lake Fundamentals:** Deep dive into the storage layer.
*   **003 Apache Spark Architecture:** Understanding the distributed processing engine.
*   **004 Unity Catalog and Governance:** Standards for data security and lineage.
*   **005 Medallion Architecture Design:** Detailed implementation of data layers.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
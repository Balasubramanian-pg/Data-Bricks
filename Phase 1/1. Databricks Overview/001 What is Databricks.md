# 001 What is Databricks

Canonical documentation for 001 What is Databricks. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of this topic is to define the Databricks platform as a unified data analytics architecture. It addresses the historical fragmentation between data lakes (used for unstructured data and machine learning) and data warehouses (used for structured data and business intelligence). Databricks resolves this dichotomy by providing a single platform that unifies data engineering, data science, machine learning, and analytics.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative. While Databricks is a specific commercial product, this document focuses on its architectural identity, core components, and the "Lakehouse" paradigm it pioneered.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   The Data Lakehouse architecture.
*   The separation of compute and storage.
*   The relationship between the Control Plane and Data Plane.
*   Core open-source technologies managed by the platform (Spark, Delta Lake, MLflow).
*   Unified governance models.

**Out of scope:**
*   Specific cloud provider implementation details (AWS, Azure, GCP specific configurations).
*   Pricing models (DBUs) and billing specifics.
*   Step-by-step installation or workspace provisioning guides.
*   Comparisons with competitor platforms (e.g., Snowflake, BigQuery) except where necessary to define architectural differences.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Databricks** | A cloud-based unified data analytics platform built on top of Apache Spark, designed to facilitate data engineering, data science, and analytics within a single environment. |
| **Data Lakehouse** | An architecture that combines the flexibility, cost-efficiency, and scale of data lakes with the data management and ACID transaction capabilities of data warehouses. |
| **Apache Spark** | The underlying unified analytics engine for large-scale data processing, providing the distributed compute power for Databricks clusters. |
| **Delta Lake** | An open-source storage layer that brings reliability (ACID transactions) to data lakes. It is the default storage format in Databricks. |
| **Control Plane** | The backend management layer managed by Databricks, containing the web application, notebook management, job scheduling, and cluster management services. |
| **Data Plane** | The environment where data is actually processed. This usually resides within the customer's cloud account (VPC/VNet), ensuring data locality and security. |
| **Unity Catalog** | A unified governance solution for data, analytics, and AI assets, providing a single metastore across multiple workspaces. |
| **Photon** | A proprietary, vectorized query engine written in C++ designed to accelerate SQL queries and Spark workloads on Databricks. |
| **Medallion Architecture** | A data design pattern used within Databricks to organize data quality levels (Bronze/Raw, Silver/Cleaned, Gold/Aggregated). |

## Core Concepts

### 1. The Lakehouse Paradigm
Databricks is the primary architect of the "Lakehouse" concept. Historically, organizations maintained separate systems:
*   **Data Lakes:** For raw, unstructured data and ML (low cost, low governance).
*   **Data Warehouses:** For structured, BI-ready data (high cost, high governance).

Databricks merges these by implementing a transactional storage layer (Delta Lake) on top of low-cost object storage (S3, ADLS, GCS). This allows structured SQL queries and unstructured ML workloads to operate on the same data without duplication.

### 2. Decoupled Compute and Storage
Databricks strictly separates compute resources from storage resources.
*   **Storage:** Data persists in the customer's cloud object storage (e.g., AWS S3). It is durable and independent of the platform.
*   **Compute:** Clusters (virtual machines) are ephemeral. They spin up to process data and terminate when idle.
This decoupling allows for independent scaling; an organization can store petabytes of data cheaply while only paying for compute when queries are running.

### 3. Unified Governance
Through Unity Catalog, Databricks provides a centralized governance layer. Unlike traditional setups where permissions are managed separately for files (in the lake) and tables (in the warehouse), Unity Catalog enforces ACLs (Access Control Lists), lineage, and auditing across all assets, including data tables, ML models, and files.

### 4. Polyglot Development
The platform supports multiple languages natively within the same workflow. A single notebook can contain cells written in Python (PySpark), SQL, Scala, and R, allowing different personas (Data Engineers, Data Scientists, SQL Analysts) to collaborate on the same underlying data objects.

## Standard Model

The standard architectural model for Databricks deployment is the **Hybrid Cloud Architecture** (also known as the Split Plane Architecture).

### The Control Plane (Managed Service)
*   Hosted in the Databricks cloud account.
*   Handles authentication, workspace UI, job scheduling, and cluster management.
*   Stores notebook code and configuration metadata.
*   **Crucially:** Customer data does *not* reside here.

### The Data Plane (Customer Environment)
*   Hosted in the customer's cloud account.
*   Contains the compute resources (Spark Clusters/VMs).
*   Connects directly to the customer's object storage.
*   Data is processed here, encrypted, and returned to the user; it does not persist in the Control Plane.

### Serverless Compute Model
An evolution of the standard model where the compute resources are also managed by Databricks (Serverless SQL, Serverless Jobs) to abstract infrastructure management from the user, though strict network isolation boundaries remain.

## Common Patterns

### 1. The Medallion Architecture (Multi-hop)
The standard data engineering pattern in Databricks:
*   **Bronze (Raw):** Ingestion of data in its native format. Append-only.
*   **Silver (Refined):** Filtered, cleaned, and augmented data. Conformed schema.
*   **Gold (Business Level):** Aggregated data ready for analytics and reporting. Star schemas.

### 2. Streaming ETL
Using Spark Structured Streaming to process data in near real-time. Databricks treats batch and streaming data symmetrically; a static table can be treated as a stream, and a stream can be materialized as a table (Delta Table).

### 3. ML Lifecycle Management (MLflow)
Databricks is used as an end-to-end ML platform.
1.  **Experiment Tracking:** Logging parameters and metrics during model training.
2.  **Model Registry:** Versioning models and managing stage transitions (Staging -> Production).
3.  **Inference:** Deploying models for batch or real-time serving.

## Anti-Patterns

### 1. Treating Databricks as a Traditional RDBMS
While Databricks supports SQL, treating it exactly like a single-node Postgres or SQL Server instance leads to performance issues.
*   *Anti-pattern:* Running millions of single-row `INSERT` or `UPDATE` statements.
*   *Correction:* Use bulk appends or merge operations optimized for columnar storage.

### 2. The "Small File" Problem
Ingesting data as thousands of tiny files (KB size) degrades performance due to metadata overhead.
*   *Correction:* Use "Optimize" and "Auto-optimize" features to compact small files into larger files (typically 128MB - 1GB) suitable for distributed reading.

### 3. Driver Node Overload
Collecting massive datasets back to the driver node (e.g., `pandas_df = spark_df.toPandas()`) defeats the purpose of distributed computing and causes Out Of Memory (OOM) errors.
*   *Correction:* Perform aggregations and heavy lifting on the worker nodes using PySpark/SQL before collecting results.

## Edge Cases

### 1. Low-Latency OLTP
Databricks is an OLAP (Online Analytical Processing) system. It is not designed for OLTP (Online Transaction Processing) applications requiring sub-millisecond latency for single-row lookups (e.g., powering a banking app's backend).
*   *Solution:* Sync Gold tables to a dedicated NoSQL or RDBMS for operational serving.

### 2. Complex Dependency Management
In environments with strict air-gapped security or complex library conflicts, managing cluster libraries can become difficult.
*   *Solution:* Use Container Services (Docker images) to encapsulate the runtime environment rather than installing libraries via scripts at cluster launch.

## Related Topics
*   **002 Apache Spark Architecture:** Deep dive into the compute engine.
*   **003 Delta Lake Protocol:** Technical specification of the storage layer.
*   **004 Unity Catalog:** Governance and lineage implementation.
*   **005 MLflow:** Machine learning lifecycle management.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
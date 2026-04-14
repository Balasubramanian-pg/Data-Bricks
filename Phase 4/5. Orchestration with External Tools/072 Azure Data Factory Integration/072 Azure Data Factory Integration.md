# 072 Azure Data Factory Integration

Canonical documentation for 072 Azure Data Factory Integration. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of 072 Azure Data Factory (ADF) Integration is to provide a centralized, cloud-native orchestration layer for complex data movement and transformation across disparate environments. It addresses the problem of data silos by enabling seamless connectivity between on-premises, multi-cloud, and SaaS data sources. This integration framework allows for the decoupling of data ingestion from data processing, providing a scalable environment for ETL (Extract, Transform, Load) and ELT (Extract, Load, Transform) workflows.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative, focusing on the architectural principles of the integration service rather than specific UI walkthroughs.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Orchestration Logic:** The definition and execution of data workflows.
* **Connectivity Abstraction:** The methodology for defining connections to external systems.
* **Compute Management:** The theoretical application of various compute resources to execute data tasks.
* **Metadata Management:** The handling of data structures and schemas during integration.

**Out of scope:**
* **Specific Vendor API Documentation:** Detailed specifications for third-party endpoints (e.g., Salesforce API versions).
* **Network Infrastructure Setup:** Physical configuration of firewalls or ExpressRoute circuits.
* **Downstream Data Consumption:** How data is used by BI tools or ML models after the integration process is complete.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Linked Service** | A configuration object that defines the connection information (connection string, authentication) to an external resource. |
| **Dataset** | A named view of data that simply points or references the data to be used in activities as inputs and outputs. |
| **Activity** | A single processing step within a pipeline (e.g., Copy, Lookup, Web Activity). |
| **Pipeline** | A logical grouping of activities that perform a unit of work. |
| **Integration Runtime (IR)** | The compute infrastructure used by Azure Data Factory to provide data integration capabilities across different network environments. |
| **Trigger** | A unit of processing that determines when a pipeline execution needs to be kicked off (e.g., Schedule, Tumbling Window, Event). |
| **Control Flow** | The orchestration of pipeline activities, including sequencing, branching, and looping. |

## Core Concepts

### 1. Separation of Concerns
The integration architecture enforces a strict separation between the **Connection** (Linked Service), the **Data Structure** (Dataset), and the **Logic** (Activity). This allows for the reuse of connection logic across multiple datasets and the reuse of datasets across multiple activities.

### 2. Compute Abstraction
Integration tasks are decoupled from the underlying hardware. The Integration Runtime (IR) acts as the bridge between the activity and the compute resource. This allows the integration to occur in different network security zones (Public Cloud vs. Private Network) without changing the pipeline logic.

### 3. Metadata-Driven Orchestration
The framework supports dynamic integration where parameters and variables drive the execution. Instead of hardcoding table names or file paths, the integration utilizes metadata to determine the source and destination at runtime.

## Standard Model
The standard model for 072 Azure Data Factory Integration follows a hierarchical execution pattern:

1.  **Trigger Layer:** Initiates the process based on time or external events.
2.  **Pipeline Layer:** Defines the workflow, variables, and parameters.
3.  **Activity Layer:** Executes specific tasks (Data Movement or Transformation).
4.  **Integration Runtime Layer:** Provides the necessary CPU, memory, and network bandwidth.
5.  **Linked Service Layer:** Authenticates and establishes the secure tunnel to the data source/sink.

## Common Patterns

### Copy Activity Pattern
The most fundamental pattern used for high-scale data ingestion. It involves a source dataset, a sink dataset, and a mapping configuration. It supports performance tuning through parallel copies and staged uploads.

### Metadata-Driven Ingestion
A pattern where a single pipeline is designed to handle hundreds of tables. The pipeline queries a control table (metadata store) to retrieve a list of objects to be moved, then iterates through them using a "ForEach" loop.

### Modern Data Warehouse (MDW) Pattern
Integration that moves raw data into a "Bronze" or "Landing" zone, triggers a transformation (via Databricks, Snowflake, or SQL), and moves the refined data into "Silver" and "Gold" zones for consumption.

## Anti-Patterns

*   **Hardcoding Credentials:** Storing passwords or keys directly within Linked Services instead of referencing a secure Key Vault.
*   **Monolithic Pipelines:** Creating a single pipeline with dozens of activities, making it difficult to debug, maintain, and restart upon failure.
*   **Over-provisioning Compute:** Using high-core count Integration Runtimes for small, infrequent data movements, leading to unnecessary costs.
*   **Ignoring Error Handling:** Failing to implement "On Failure" paths or retry policies, resulting in silent data gaps.

## Edge Cases

*   **Schema Drift:** When the source system changes its column structure without notice. The integration must be configured to handle "Allow Schema Drift" to prevent pipeline failure.
*   **Large Binary Objects (BLOBs):** Moving multi-terabyte files requires specific chunking strategies and may exceed the timeout limits of standard integration runtimes.
*   **On-Premises Connectivity via Proxy:** In highly restrictive corporate environments, the Integration Runtime must navigate multiple layers of proxy servers, requiring specific environment variable configurations.
*   **Polyglot Persistence:** Scenarios where data must be synchronized across different types of databases (e.g., NoSQL to Relational) requiring complex type mapping.

## Related Topics
*   **073 Data Lake Storage:** The primary destination for raw data integration.
*   **084 Key Vault Management:** Essential for securing integration credentials.
*   **102 CI/CD for Data Integration:** The lifecycle management of ADF entities via Git integration.
*   **115 Compute Orchestration:** Deep dive into external compute engines like Spark and SQL.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
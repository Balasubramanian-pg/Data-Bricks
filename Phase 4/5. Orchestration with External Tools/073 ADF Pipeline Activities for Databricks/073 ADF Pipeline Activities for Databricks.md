# 073 ADF Pipeline Activities for Databricks

Canonical documentation for 073 ADF Pipeline Activities for Databricks. This document defines concepts, terminology, and standard usage.

## Purpose
The integration between orchestration engines (Azure Data Factory) and distributed compute environments (Databricks) addresses the need for scalable, managed data processing within a broader workflow. This topic exists to define how an orchestration layer triggers, monitors, and passes metadata to specialized compute tasks, ensuring that data engineering pipelines can leverage high-performance clusters without sacrificing centralized control and monitoring.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative regarding the architectural relationship between the orchestrator and the compute provider.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* The lifecycle of Databricks activities within an ADF pipeline (Notebook, Jar, and Python activities).
* Authentication and connectivity models between the orchestrator and the compute cluster.
* Parameterization and metadata exchange mechanisms.
* Compute resource management strategies (Job Clusters vs. Existing Clusters).

**Out of scope:**
* Internal Spark optimization or code-level performance tuning within Databricks.
* Configuration of Databricks SQL Warehouses (unless used via generic activity).
* Third-party library development for Spark.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Linked Service** | The configuration entity that defines the connection string, authentication method, and workspace URL for the Databricks environment. |
| **Job Cluster** | An ephemeral compute resource created by the orchestrator for the duration of a specific activity and terminated immediately upon completion. |
| **Interactive Cluster** | A persistent compute resource (often shared) that remains active across multiple activity executions. |
| **Base Parameters** | A set of key-value pairs passed from the pipeline activity to the Databricks execution environment as arguments. |
| **DBFS** | Databricks File System; a virtual file system used to store scripts, jars, and logs accessible by the compute nodes. |
| **Notebook Activity** | A specific pipeline task that executes a Databricks Notebook (.ipynb or .py source). |
| **Jar Activity** | A pipeline task that executes a compiled Java or Scala library containing a Main method. |
| **Python Activity** | A pipeline task that executes a standalone Python script (.py) stored in DBFS or a supported cloud storage. |

## Core Concepts
### Compute Abstraction
The primary concept is the separation of **Orchestration Logic** (ADF) from **Execution Logic** (Databricks). The pipeline defines *when* and *under what conditions* a task runs, while the activity defines *what* runs and *on what hardware*.

### Authentication Handshake
The orchestrator must prove its identity to the Databricks workspace. This is typically achieved through:
1.  **Access Tokens:** Static, long-lived tokens (less secure).
2.  **Managed Identities:** Azure-native identity management (recommended).
3.  **Service Principals:** OAuth-based authentication for automated systems.

### State Management
The orchestrator monitors the Databricks Job API. The activity state (Queued, Running, Success, Failed) is mapped from the Databricks job status back to the pipeline's monitoring interface.

## Standard Model
The recommended model for 073 ADF Pipeline Activities involves:
1.  **Dynamic Linked Services:** Using parameters within the Linked Service to point to different workspaces (Dev/Test/Prod) based on the environment.
2.  **Ephemeral Compute:** Defaulting to **Job Clusters** for production workloads to ensure cost-efficiency and environment isolation.
3.  **Externalized Configuration:** Passing file paths, date ranges, and environment flags via **Base Parameters** rather than hardcoding them within the Databricks code.

## Common Patterns
*   **The Chained Notebook Pattern:** Using the output of one Notebook activity (via `dbutils.notebook.exit()`) to inform the conditional logic of the next activity in the pipeline.
*   **The Library Pre-loading Pattern:** Defining required libraries (Maven, PyPI, or DBFS paths) within the activity settings to ensure the environment is prepared before code execution.
*   **The Fan-out/Parallel Pattern:** Using a "ForEach" activity in the pipeline to trigger multiple Databricks activities simultaneously across different data partitions.

## Anti-Patterns
*   **Production Use of Interactive Clusters:** Using persistent clusters for scheduled jobs, leading to higher costs and potential resource contention/state pollution.
*   **Monolithic Notebooks:** Creating a single, massive notebook for an entire ETL process rather than modularizing tasks into discrete activities.
*   **Hardcoded Credentials:** Storing secrets or connection strings within the Databricks notebook instead of using Databricks Secrets or Azure Key Vault integrated with ADF.
*   **Ignoring Exit Values:** Failing to provide a meaningful exit message from the Databricks task, making pipeline debugging difficult.

## Edge Cases
*   **Cluster Startup Timeouts:** If a Job Cluster takes longer than the internal timeout to provision (often due to cloud provider capacity), the activity may fail before the code even runs.
*   **Concurrency Limits:** Databricks workspaces have limits on the number of concurrent jobs. A pipeline with a high-degree of parallelism may trigger "429 Too Many Requests" errors.
*   **Library Version Conflicts:** When an activity specifies a library version that conflicts with the Databricks Runtime (DBR) pre-installed libraries.
*   **Token Expiration:** Pipelines failing suddenly because the underlying Personal Access Token (PAT) used in the Linked Service has expired.

## Related Topics
*   **012 Linked Services and Integration Runtimes:** The underlying connectivity layer.
*   **045 Pipeline Control Flow:** Managing the logic (If/Else, ForEach) surrounding Databricks activities.
*   **088 Databricks Secret Scopes:** How Databricks handles sensitive data during activity execution.
*   **102 Managed Identities for Azure Resources:** The preferred authentication method for this integration.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
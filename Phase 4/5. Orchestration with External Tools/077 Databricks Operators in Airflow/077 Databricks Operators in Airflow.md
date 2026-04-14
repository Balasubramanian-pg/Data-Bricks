# 077 Databricks Operators in Airflow

Canonical documentation for 077 Databricks Operators in Airflow. This document defines concepts, terminology, and standard usage.

## Purpose
The integration between Apache Airflow and Databricks exists to bridge the gap between workflow orchestration and high-performance distributed computing. While Airflow excels at managing task dependencies, scheduling, and state monitoring, it is not designed for heavy data processing. Databricks provides the computational infrastructure (Spark, Photon, SQL Warehouses) required for data engineering, machine learning, and analytics.

The 077 Databricks Operators provide a standardized interface for Airflow to trigger, monitor, and manage workloads within a Databricks workspace, ensuring that data pipelines are scalable, observable, and decoupled from the orchestration layer.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative, focusing on the architectural relationship between the orchestrator and the compute provider.

## Scope
**In scope:**
* Lifecycle management of Databricks jobs via Airflow.
* Authentication and connectivity abstractions.
* Execution models (Submit vs. Run Now).
* Parameter passing and state synchronization.
* Asynchronous and Deferrable execution patterns.

**Out of scope:**
* Internal Spark optimization or tuning.
* Databricks-side cluster configuration details (e.g., instance types).
* Third-party plugins not included in the standard provider packages.

## Definitions
| Term | Definition |
|------|------------|
| **Connection ID** | A unique identifier in Airflow that stores the Databricks workspace URL and authentication credentials (PAT or OAuth). |
| **Submit Run** | An execution model where the entire job definition (cluster spec, task, libraries) is sent at runtime. |
| **Run Now** | An execution model that triggers a pre-existing Job ID defined within the Databricks workspace. |
| **Repair Run** | The process of re-running only the failed tasks within a multi-task Databricks job. |
| **Notebook Task** | A workload type that executes a Databricks Notebook. |
| **SQL Operator** | A specialized operator for executing queries against Databricks SQL Warehouses. |
| **Polling** | The mechanism by which Airflow periodically queries the Databricks API to check the status of a running task. |
| **Deferrable Operator** | An operator that releases its worker slot while waiting for a Databricks job to complete, utilizing the Airflow Triggerer. |

## Core Concepts

### The Provider Architecture
The Databricks-Airflow integration relies on the `DatabricksHook`, which abstracts the Databricks REST API. Operators use this hook to communicate with the workspace. The relationship is fundamentally asynchronous: Airflow sends an instruction, receives a `run_id`, and then monitors that `run_id` until completion.

### Execution Paradigms
1.  **Ephemeral Workloads (Submit):** Airflow defines the infrastructure requirements. The cluster is created, the task is executed, and the cluster is terminated. This ensures isolation and cost-efficiency.
2.  **Persistent Workloads (Run Now):** Airflow acts as a remote trigger for a job already configured in the Databricks UI. This is preferred for complex multi-task jobs managed via Databricks Workflows.

### Authentication
Standardized access is managed through Airflow Connections. While Personal Access Tokens (PAT) are common, enterprise-grade implementations utilize OAuth or Service Principal credentials to ensure security and auditability.

## Standard Model
The recommended model for 077 Databricks Operators follows the "Orchestrator-Worker" separation:

1.  **Minimal Logic in Airflow:** Airflow should only contain the logic for *when* and *in what order* to run tasks. The *how* (the business logic) resides in Databricks (Notebooks, JARs, or Python files).
2.  **Connection Management:** Use a centralized `databricks_default` connection.
3.  **Idempotency:** Tasks should be designed so that re-running a `run_id` or triggering a new run with the same parameters does not result in duplicate or corrupted data.
4.  **State Mapping:** Airflow maps Databricks life cycle states (PENDING, RUNNING, TERMINATING) and result states (SUCCESS, FAILED, SKIPPED) to Airflow task states.

## Common Patterns

### The Deferrable Pattern
For long-running Spark jobs, using the `deferrable=True` parameter is the standard. This prevents "worker starvation" by allowing the Airflow worker to hand off the monitoring task to the Airflow Triggerer, which can handle thousands of concurrent watches with minimal resources.

### Parameter Injection
Passing dynamic values (like execution dates or S3 paths) from Airflow to Databricks is handled via `notebook_params` or `spark_submit_params`. This allows the same Databricks notebook to be reused across different DAG runs.

### SQL Warehouse Integration
Using the `DatabricksSqlOperator` to execute transformation logic (dbt-like patterns) directly on SQL Warehouses, leveraging compute optimized for SQL rather than general-purpose Spark clusters.

## Anti-Patterns

*   **Heavy Lifting in Airflow:** Performing data transformations within an Airflow PythonOperator before passing the result to Databricks.
*   **Hardcoding Cluster Specs:** Defining specific instance types and node counts inside DAG files, which makes infrastructure updates difficult.
*   **Synchronous Polling on Busy Workers:** Running hundreds of Databricks tasks without the deferrable flag, leading to exhausted worker slots.
*   **Overlapping Runs:** Failing to set `max_active_runs=1` or appropriate concurrency limits when the underlying Databricks job or data target cannot handle parallel execution.

## Edge Cases

*   **Token Expiration:** If a PAT expires mid-run, the operator may fail to poll for status, leading to an "Upstream Failed" or "Unknown" state in Airflow, even if the Databricks job succeeds.
*   **API Rate Limiting:** High-frequency polling or triggering hundreds of jobs simultaneously can trigger Databricks API 429 errors. Implementing exponential backoff in the hook is the standard mitigation.
*   **Cluster Start-up Timeouts:** If a Databricks cluster takes longer to provision than the Airflow timeout settings, the task may be marked as failed while the cluster is still spinning up.
*   **Zombies:** If an Airflow scheduler dies, a job might continue running in Databricks with no orchestrator to track it.

## Related Topics
*   **012 Airflow Connections:** The underlying mechanism for storing Databricks credentials.
*   **045 Deferrable Operators:** The architectural pattern used for resource-efficient waiting.
*   **Databricks Workflows:** The native Databricks orchestration tool which can be triggered by Airflow.
*   **XComs:** Used to pass the `run_id` or return values from Databricks back to the Airflow ecosystem.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
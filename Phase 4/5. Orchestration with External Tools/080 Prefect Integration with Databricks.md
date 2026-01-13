# 080 Prefect Integration with Databricks

Canonical documentation for 080 Prefect Integration with Databricks. This document defines concepts, terminology, and standard usage.

## Purpose
The integration between Prefect and Databricks exists to bridge the gap between high-level workflow orchestration and distributed big-data processing. While Prefect manages the state, scheduling, and dependency logic of a workflow, Databricks provides the scalable compute environment necessary for data-intensive operations. This integration addresses the need for a unified control plane that can trigger, monitor, and manage the lifecycle of remote compute resources without coupling the business logic to the underlying infrastructure.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative, focusing on the architectural relationship rather than specific library versions.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* The architectural relationship between the Orchestrator (Prefect) and the Compute Provider (Databricks).
* Lifecycle management of remote jobs and clusters.
* Authentication and security patterns for cross-platform communication.
* State synchronization and error propagation between the two systems.

**Out of scope:**
* Internal Databricks Spark optimization techniques.
* Specific versions of the `prefect-databricks` collection or library.
* General Prefect deployment patterns (e.g., Kubernetes vs. Serverless) unrelated to Databricks.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Orchestrator** | The system (Prefect) responsible for defining the order of operations, managing state, and handling retries. |
| **Compute Provider** | The system (Databricks) responsible for executing the heavy-duty data processing tasks. |
| **Job** | A discrete unit of work defined within Databricks, often consisting of one or more tasks (Notebooks, JARs, or Python scripts). |
| **Ephemeral Cluster** | A Databricks cluster created specifically for the duration of a single job run and terminated immediately upon completion. |
| **All-Purpose Cluster** | A persistent Databricks cluster used for interactive analysis or multiple sequential job runs. |
| **Personal Access Token (PAT)** | A credential used by the Orchestrator to authenticate with the Databricks REST API. |
| **State Mapping** | The process of translating Databricks run statuses (e.g., PENDING, RUNNING, TERMINATED) into Prefect states (e.g., Running, Completed, Failed). |

## Core Concepts

### Separation of Concerns
The fundamental principle of this integration is the separation of the **Control Plane** from the **Data Plane**. Prefect acts as the Control Plane, maintaining the "source of truth" for the workflow's progress. Databricks acts as the Data Plane, where the actual data transformation occurs.

### Remote Execution Lifecycle
The integration follows a standard lifecycle:
1.  **Submission:** The Orchestrator sends a request to the Databricks API to trigger a job or create a cluster.
2.  **Monitoring:** The Orchestrator polls the Databricks API or waits for a webhook to determine the status of the remote task.
3.  **Log Aggregation:** The Orchestrator retrieves or links to the remote execution logs for centralized visibility.
4.  **Finalization:** The Orchestrator records the final state and triggers downstream dependencies or cleanup actions.

### Authentication Scoping
Security must be handled via least-privilege principles. The Orchestrator requires credentials (PATs or Service Principal tokens) that are scoped specifically to the workspaces and clusters necessary for the workflow.

## Standard Model
The standard model for Prefect-Databricks integration is the **Remote Job Trigger Pattern**. 

In this model, the Prefect flow does not execute data logic locally. Instead, it encapsulates a "Databricks Task" which:
1.  Connects to the Databricks REST API.
2.  Submits a configuration payload (defining the cluster requirements and the code to run).
3.  Monitors the `run_id` provided by Databricks.
4.  Blocks (asynchronously or synchronously) until the Databricks job reaches a terminal state.
5.  Returns the result or raises an exception based on the Databricks exit code.

## Common Patterns

### 1. Ephemeral Job Clusters
For production workloads, the standard pattern is to spin up a new cluster for every job run. This ensures environment isolation and cost-efficiency, as the cluster only exists while the task is active.

### 2. Persistent Cluster Task Submission
For low-latency requirements where cluster start-up time (3–5 minutes) is unacceptable, the Orchestrator submits tasks to a pre-existing, "always-on" All-Purpose Cluster.

### 3. Notebook-as-a-Task
Executing Databricks Notebooks directly from a Prefect flow. This allows data scientists to develop in a UI-driven environment while engineers manage the operational orchestration.

### 4. SQL Warehouse Execution
Using Prefect to trigger queries on Databricks SQL Warehouses, separating the compute used for ETL from the compute used for analytical querying.

## Anti-Patterns

*   **Monolithic Jobs:** Triggering a single, massive Databricks job that contains 50 internal steps. This bypasses Prefect's granular observability and retry logic.
*   **Hardcoded Credentials:** Embedding Databricks PATs directly in the flow code rather than using secure Secret storage or Blocks.
*   **Infinite Polling:** Configuring the Orchestrator to poll the Databricks API too frequently (e.g., every second), which can lead to API rate limiting.
*   **Ignoring Spark Failures:** Failing to propagate a Spark "Success with Errors" state back to Prefect, leading to "silent failures" in the pipeline.

## Edge Cases

*   **Cluster Start-up Timeouts:** If a Databricks cluster takes longer than the cloud provider's limit to provision, the Orchestrator must handle the timeout gracefully and decide whether to retry.
*   **Token Expiration:** Long-running flows may fail if the authentication token expires mid-execution. Refreshing tokens or using Service Principals is required for high-availability.
*   **Network Partitioning:** If the connection between Prefect and Databricks is severed after a job is submitted, the Orchestrator must be able to "re-attach" to the existing `run_id` upon reconnection to avoid duplicate execution.
*   **Partial Success in Multi-Task Jobs:** When a Databricks Job contains multiple internal tasks, the Orchestrator must decide if the entire Prefect task is "Failed" if only one sub-task fails.

## Related Topics
*   **010 Workflow Orchestration Concepts:** General principles of flow and task management.
*   **042 Infrastructure as Code (IaC):** For provisioning the Databricks workspaces and clusters.
*   **095 Secret Management:** Best practices for handling API keys and tokens.
*   **Distributed Compute Patterns:** Theoretical foundations of Spark and cluster computing.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
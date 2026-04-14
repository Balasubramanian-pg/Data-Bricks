# 081 Dagster Integration with Databricks

Canonical documentation for 081 Dagster Integration with Databricks. This document defines concepts, terminology, and standard usage.

## Purpose
The integration between Dagster and Databricks exists to bridge the gap between high-level data orchestration and distributed heavy-compute environments. While Databricks provides the infrastructure for processing massive datasets via Apache Spark, Dagster provides the control plane for modeling data dependencies, managing metadata, and ensuring observability across the entire data platform. This integration addresses the need for a unified lineage and operational view when compute is offloaded to external, high-performance clusters.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative, focusing on the architectural relationship rather than specific API versions.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Orchestration of Databricks workloads (Jobs, Notebooks, and Scripts) from Dagster.
* Metadata synchronization between the compute layer and the orchestration layer.
* Resource management and authentication patterns for cross-platform communication.
* Asset-based modeling of Databricks-resident data.

**Out of scope:**
* Internal Databricks administration (workspace configuration, VPC peering).
* Spark optimization techniques or Spark-specific code debugging.
* Third-party library management within Databricks clusters.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Databricks Job** | A unit of execution within Databricks that can run notebooks, JARs, or Python scripts. |
| **Dagster Resource** | A configuration object in Dagster that defines the connection and authentication to a Databricks workspace. |
| **External Execution** | A pattern where the orchestrator triggers logic on a remote compute provider rather than executing it locally. |
| **Pipes (Dagster)** | A protocol used to stream metadata, logs, and events from an external process (like a Databricks cluster) back to the Dagster orchestrator in real-time. |
| **Ephemeral Cluster** | A Databricks cluster created specifically for a single job run and terminated immediately upon completion. |
| **All-Purpose Cluster** | A persistent Databricks cluster used for interactive analysis or multiple scheduled jobs. |

## Core Concepts

### 1. Orchestration vs. Execution
The fundamental concept is the separation of the **Control Plane** (Dagster) from the **Data Plane** (Databricks). Dagster decides *when* and *why* a task runs based on upstream dependencies, while Databricks handles the *how* of distributed data processing.

### 2. Metadata Propagation
Integration is not merely triggering a remote process; it involves the bidirectional flow of information. This includes:
* **Input Parameters:** Passing Dagster partition keys or configuration to Databricks.
* **Execution Status:** Monitoring the lifecycle of the Databricks job.
* **Output Metadata:** Capturing Spark-specific metrics (rows processed, table paths) and surfacing them in the Dagster UI.

### 3. Asset-Centric Lineage
In this integration, Databricks operations are typically modeled as **Software-Defined Assets**. This allows the orchestrator to track the state of tables in the Delta Lake or Unity Catalog as part of a global data lineage, regardless of where the compute occurred.

## Standard Model
The standard model for integration follows the **Remote Trigger and Monitor** pattern:

1.  **Initialization:** Dagster initiates a request to the Databricks Jobs API using a configured Resource.
2.  **Context Injection:** Dagster provides execution context (e.g., partition info, run IDs) to the Databricks environment.
3.  **Remote Execution:** Databricks executes the specified workload (Notebook, Python file, or Wheel).
4.  **Observation:** Dagster polls the API or uses a callback mechanism (like Dagster Pipes) to receive real-time updates.
5.  **Materialization:** Upon completion, Dagster records the asset materialization, including metadata generated during the Spark session.

## Common Patterns

### Ephemeral Job Clusters
The most common pattern for production workloads. Dagster defines the cluster configuration (node type, worker count) as part of the job metadata. This ensures cost-efficiency and environment isolation.

### The "Pipes" Pattern
Using the Dagster Pipes protocol to allow Python code running inside Databricks to send structured metadata back to Dagster. This avoids "black box" execution where the orchestrator only knows if a job succeeded or failed.

### Notebook-as-an-Op
Treating a Databricks Notebook as a single step in a larger Dagster graph. This is common in data science workflows where exploratory code is transitioned into production.

## Anti-Patterns

*   **Nested Scheduling:** Using Databricks' internal "Workflows" scheduler to trigger Dagster, or vice-versa in a circular fashion. This fragments the source of truth for scheduling.
*   **Hardcoded Credentials:** Embedding Databricks tokens directly in Dagster code rather than using environment variables or secret management systems.
*   **Over-polling:** Setting excessively high polling frequencies on the Databricks API, which can lead to rate-limiting and performance degradation of the control plane.
*   **Fat Notebooks:** Placing complex orchestration logic (retries, branching) inside a Databricks Notebook instead of letting Dagster handle the logic.

## Edge Cases

*   **Cluster Start-up Latency:** Databricks ephemeral clusters can take several minutes to initialize. Dagster timeouts must be configured to account for this "cold start" period.
*   **Partial Failures in Multi-task Jobs:** If a Databricks Job contains multiple tasks, Dagster must be configured to interpret which specific task failure constitutes an asset failure.
*   **Token Expiration:** Long-running Dagster instances may encounter expired Databricks PATs (Personal Access Tokens) if a robust secret rotation or OIDC mechanism is not in place.
*   **Network Isolation:** When Dagster is hosted in a private network and Databricks is in another, the integration must account for proxy settings or VPC peering to reach the Databricks API.

## Related Topics
*   **024 Software-Defined Assets:** The primary method for modeling data in Dagster.
*   **045 External Execution Protocol (Pipes):** The underlying mechanism for cross-platform metadata exchange.
*   **012 Resource Management:** How Dagster handles connections to external services.
*   **Delta Lake & Unity Catalog:** The storage and governance layers typically targeted by Databricks compute.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
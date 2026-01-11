# 041 MLflow Experiment Tracking

Canonical documentation for 041 MLflow Experiment Tracking. This document defines concepts, terminology, and standard usage.

## Purpose
MLflow Experiment Tracking exists to solve the problem of non-determinism and lack of traceability in machine learning development. In traditional software engineering, version control (Git) manages code state; however, in machine learning, the outcome is a product of code, data, environment, and hyperparameters. 

This topic addresses the need for a systematic record of every computational execution (a "Run") to ensure reproducibility, facilitate comparative analysis between model iterations, and provide a centralized audit trail for model governance.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative. While it references the MLflow framework, it focuses on the architectural and conceptual standards of the Experiment Tracking component.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Metadata Persistence:** The logging of parameters, metrics, and tags.
* **Artifact Management:** The association of binary outputs (models, plots, data snapshots) with specific runs.
* **Hierarchical Organization:** The relationship between Experiments and Runs.
* **Tracking Infrastructure:** The conceptual roles of the Tracking Server, Backend Store, and Artifact Store.

**Out of scope:**
* **Model Deployment:** The serving or orchestration of models (covered in Model Serving).
* **Workflow Orchestration:** The scheduling of pipelines (e.g., Airflow or Kubeflow integration).
* **Vendor-Specific UI:** Specific features of managed platforms (e.g., Databricks, Azure ML, or AWS SageMaker implementations).

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Experiment** | The primary unit of organization; a named collection of related runs usually focused on a specific task or hypothesis. |
| **Run** | A single execution of data science code. It is the atomic unit of tracking where data is recorded. |
| **Parameter** | A key-value pair representing a constant input to a run (e.g., `learning_rate=0.01`). Parameters are immutable once logged. |
| **Metric** | A numeric value representing a quantitative measure of a run's performance (e.g., `accuracy`). Metrics can be logged multiple times to track changes over time (steps). |
| **Artifact** | An output file in any format (e.g., a `.pkl` model file, a `.png` plot, or a `.csv` dataset) associated with a specific run. |
| **Tag** | A user-defined metadata label used for filtering and categorizing runs (e.g., `env=production`). |
| **Tracking Server** | The API and UI layer that facilitates the communication between the client code and the storage backends. |

## Core Concepts

### The Tracking Hierarchy
MLflow Experiment Tracking operates on a two-tier hierarchy:
1.  **Experiment:** Acts as a namespace. All comparative analysis is typically performed within the boundaries of an experiment.
2.  **Run:** The execution instance. Every run belongs to exactly one experiment.

### Storage Architecture
The tracking system is composed of two distinct storage components:
*   **Backend Store:** Persists "thin" metadata (Experiments, Runs, Parameters, Metrics, Tags). This is typically a relational database (SQL).
*   **Artifact Store:** Persists "thick" binary data (Models, Images, Data). This is typically a block storage system (S3, Blob Storage, or a shared filesystem).

### The Tracking Client
The client is the interface (Python, R, Java, or REST API) used to instrument code. It communicates with the Tracking Server to record data. The client is responsible for capturing the execution context, such as the source code version (Git hash) and the user who initiated the run.

## Standard Model

The standard model for MLflow Experiment Tracking follows a **Centralized Tracking Pattern**:

1.  **Initialization:** The client specifies a `Tracking URI` pointing to a remote server.
2.  **Context Creation:** An experiment is identified or created. A run is started within that experiment context.
3.  **Instrumentation:** During execution, the code explicitly (or via autologging) pushes parameters and metrics to the Tracking Server.
4.  **Artifact Finalization:** Upon completion, the code uploads resulting files to the Artifact Store via the Tracking Server's proxy or direct-to-store access.
5.  **State Resolution:** The run is marked as `FINISHED`, `FAILED`, or `KILLED`, providing a definitive status for downstream consumers.

## Common Patterns

### Nested Runs
Used for hyperparameter optimization or multi-step pipelines. A "Parent Run" represents the overall execution (e.g., a Grid Search), while "Child Runs" represent individual iterations. This maintains a clean UI and logical grouping.

### Autologging
The use of framework-specific integrations (e.g., Scikit-Learn, PyTorch, XGBoost) to automatically capture parameters and metrics without manual instrumentation. This reduces code boilerplate and ensures consistency.

### Environment Capture
Logging the `conda.yaml` or `requirements.txt` as an artifact. This ensures that the exact software environment used to generate a result can be reconstructed.

## Anti-Patterns

*   **The Monolithic Experiment:** Logging all unrelated work into the "Default" experiment (ID 0). This prevents effective filtering and comparison.
*   **Logging Large Datasets as Artifacts:** Using the Artifact Store as a primary Data Lake. Artifacts should be models or summaries; large datasets should be referenced via URI or hash rather than re-uploaded.
*   **High-Cardinality Metrics as Parameters:** Logging values that change every iteration (like "loss") as parameters. Parameters are for configuration; metrics are for measurements.
*   **Ignoring Run Status:** Failing to properly close runs or handle exceptions, leading to "Active" runs that have actually crashed.

## Edge Cases

*   **Disconnected/Offline Logging:** When a tracking server is unavailable, the client may log to a local filesystem. Reconciling these local runs with a central server requires manual synchronization or custom scripts.
*   **Concurrent Writes:** Multiple processes logging to the same Run ID simultaneously. While MLflow handles this at the API level, it can lead to interleaved metrics that are difficult to interpret.
*   **Large Metric History:** Logging metrics at an extremely high frequency (e.g., every millisecond) can degrade the performance of the Backend Store and the UI. Metrics should be sampled or aggregated if frequency is excessive.

## Related Topics
*   **042 MLflow Model Registry:** The lifecycle management system for transitioning models from experiments to production.
*   **043 MLflow Projects:** The standard format for packaging reusable data science code.
*   **044 MLflow Models:** The standard format for packaging machine learning models for use in diverse downstream tools.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
# 072 Model Lineage Tracking

Canonical documentation for 072 Model Lineage Tracking. This document defines concepts, terminology, and standard usage.

## Purpose
Model Lineage Tracking exists to provide a verifiable, end-to-end record of a machine learning model’s lifecycle. In complex AI systems, a model is not a static file but the result of a specific combination of data, code, environment configurations, and hyperparameters. 

The primary problem space addressed is the "black box" nature of model development. Without lineage, organizations cannot reliably reproduce results, audit for bias, comply with regulatory requirements, or perform root-cause analysis when a model fails in production. Lineage tracking ensures transparency and accountability by mapping the relationships between all entities involved in the creation and evolution of a model.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Metadata Persistence:** Recording the specific versions of inputs (data, code, parameters).
* **Relationship Mapping:** Defining the directional dependencies between artifacts (e.g., which dataset produced which model).
* **Lifecycle Traceability:** Tracking a model from experimentation through training, validation, and deployment.
* **Auditability:** Providing a historical record for compliance and governance.

**Out of scope:**
* **Specific vendor implementations:** (e.g., MLflow, Kubeflow Metadata, Weights & Biases).
* **Data Lineage (General):** While model lineage depends on data lineage, the broader tracking of non-ML data pipelines is considered a separate domain.
* **Model Performance Monitoring:** Real-time telemetry of model inference (though lineage provides the context for such monitoring).

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Artifact** | A discrete, immutable file or object produced during the ML lifecycle (e.g., a dataset, a trained model weight file, a plot). |
| **Provenance** | The chronology of ownership, custody, or location of an artifact; the "origin story" of a specific model. |
| **Metadata** | Structured information describing the characteristics of an artifact (e.g., training duration, hyperparameter values, hardware used). |
| **Upstream** | Entities or processes that occur earlier in the pipeline and influence the current state (e.g., the raw dataset is upstream of the model). |
| **Downstream** | Entities or processes that consume the output of the current state (e.g., a web service is downstream of the model). |
| **Immutable Record** | A lineage entry that, once written, cannot be altered, ensuring the integrity of the audit trail. |

## Core Concepts

### 1. The Lineage Graph
Model lineage is fundamentally represented as a Directed Acyclic Graph (DAG). Nodes represent artifacts (data, code, models) or actions (training, transformation), and edges represent the flow of information. This graph structure allows for "recursive lookups"—finding every component that contributed to a specific model version.

### 2. Reproducibility vs. Traceability
*   **Reproducibility** is the ability to recreate the exact same model using the recorded lineage.
*   **Traceability** is the ability to follow the path of a model to understand *why* it reached its current state, even if the exact environment cannot be perfectly replicated.

### 3. Versioning Anchors
Lineage relies on "anchors"—unique identifiers for every input. This includes:
*   **Code:** Git commit SHAs.
*   **Data:** Content-based hashes or snapshot IDs.
*   **Environment:** Container image digests (e.g., Docker SHA) or specific library manifests.

## Standard Model

The standard model for 072 Model Lineage Tracking follows a three-tier architecture:

1.  **The Execution Layer:** Where the actual computation happens (training jobs, preprocessing). This layer emits events.
2.  **The Metadata Store:** A centralized database that captures the "who, what, when, and how." It stores the relationships between artifacts.
3.  **The Artifact Repository:** A storage system (e.g., S3, Blob Storage) that holds the actual physical files (the "blobs") referenced by the Metadata Store.

### The Lineage Tuple
A standard lineage record should ideally contain the following tuple:
`{Model_ID, Parent_Artifacts[], Transformation_Logic, Environment_Context, Output_Metrics, Timestamp}`

## Common Patterns

### Event-Based Tracking
In this pattern, the training pipeline emits "events" at every stage (e.g., `DataIngested`, `TrainingStarted`, `ModelExported`). A listener service captures these events and constructs the lineage graph asynchronously.

### Wrapper-Based Tracking
Developers use a standardized library or SDK that wraps training functions. These wrappers automatically extract hyperparameters, git state, and library versions, sending them to the metadata store without manual intervention.

### Post-Hoc Reconstruction
In legacy systems where active tracking is absent, lineage is reconstructed by scanning logs and file timestamps. This is considered a "last resort" pattern as it is prone to gaps and inaccuracies.

## Anti-Patterns

*   **Manual Logging:** Relying on data scientists to manually record parameters or data versions in a spreadsheet or wiki. This is error-prone and unscalable.
*   **Mutable Tags:** Using mutable identifiers like `latest` or `production` in lineage records. If `latest` changes, the lineage record becomes decoupled from the actual artifact used.
*   **Siloed Lineage:** Tracking code in one system, data in another, and models in a third without a unifying key. This breaks the "chain of custody."
*   **Logging Only Successes:** Failing to track failed experiments. Lineage should include the "path not taken" to provide full context for model evolution.

## Edge Cases

*   **Transfer Learning:** When Model B is initialized using the weights of Model A. The lineage must capture the link to Model A as an upstream dependency, even if Model A was trained in a different environment or by a different team.
*   **Online/Continuous Learning:** In systems where models are updated incrementally with new data packets. Lineage must handle high-frequency updates and potentially aggregate lineage records to avoid "metadata bloat."
*   **Federated Learning:** Tracking lineage when the training data never leaves the edge device. In this case, lineage tracks the "global model" updates and the "aggregation logic" rather than the raw edge data.
*   **Human-in-the-loop (HITL):** When a human manually adjusts weights or labels during the process. The human intervention must be recorded as a "transformation" node in the lineage graph.

## Related Topics
*   **070 Model Registry:** The central catalog where tracked models are stored and managed.
*   **071 Data Provenance:** The specialized tracking of data origins and transformations.
*   **085 AI Governance:** The framework of rules and practices that lineage tracking supports.
*   **042 Reproducible Environments:** The use of containers and lockfiles to ensure the "Environment Context" of lineage.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
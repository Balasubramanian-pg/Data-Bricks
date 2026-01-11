# 043 Model Versioning and Rollback

Canonical documentation for 043 Model Versioning and Rollback. This document defines concepts, terminology, and standard usage.

## Purpose
Model Versioning and Rollback addresses the inherent volatility and non-deterministic nature of machine learning and computational models. Unlike traditional software versioning, a model's behavior is defined by the intersection of code, data, and hyperparameters. 

The purpose of this topic is to establish a framework for ensuring reproducibility, auditability, and operational stability. It provides a mechanism to track the evolution of a model, validate its performance against historical benchmarks, and provide a safety net (rollback) to restore a known-good state when a newly deployed model underperforms or fails in a production environment.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
**In scope:**
* **Artifact Management:** The identification and storage of model binaries, weights, and metadata.
* **Lineage Tracking:** The relationship between training data, code versions, and the resulting model artifact.
* **Lifecycle States:** The progression of a model version through various stages (e.g., Development, Staging, Production).
* **Rollback Mechanisms:** The logic and strategies for reverting to a previous model state.
* **Immutability Principles:** The requirement that a versioned artifact remains unchanged once published.

**Out of scope:**
* **Specific Vendor Implementations:** Detailed guides for tools like MLflow, DVC, or AWS SageMaker.
* **Data Engineering:** The specific processes of ETL (Extract, Transform, Load), except where data versioning intersects with model lineage.
* **Model Architecture:** The internal design of neural networks or statistical algorithms.

## Definitions
| Term | Definition |
|------|------------|
| **Model Artifact** | The serialized file or collection of files (weights, parameters, configuration) that constitute a trained model. |
| **Model Registry** | A centralized repository for managing model artifacts, metadata, and version history. |
| **Lineage** | The comprehensive record of the inputs (data, code, hyperparameters) that produced a specific model version. |
| **Immutable Version** | A model version that, once assigned a unique identifier, cannot be modified. Any change requires a new version identifier. |
| **Rollback** | The operational process of redirecting traffic or execution from a current model version to a previously validated version. |
| **Promotion** | The transition of a model version from one lifecycle stage to a higher-priority stage (e.g., Staging to Production). |
| **Checkpoint** | An intermediate saved state of a model during the training process, often used for recovery or fine-tuning. |

## Core Concepts

### 1. The Trinity of Model Versioning
A model version is not defined by code alone. A complete version must encapsulate:
*   **Code:** The specific algorithm implementation and preprocessing logic.
*   **Data:** A reference to the specific dataset version used for training and validation.
*   **Configuration:** The hyperparameters, environment dependencies, and hardware constraints.

### 2. Immutability
Once a model version is registered, it must be immutable. If a model requires retraining—even with the same parameters—it must be issued a new version identifier. This prevents "silent failures" where the behavior of a production system changes without a corresponding change in the version record.

### 3. Model Lineage (Provenance)
Lineage provides the "audit trail" for a model. It allows an organization to answer exactly how a model was created, what data it saw, and who authorized its deployment. This is critical for regulatory compliance and debugging "model drift."

### 4. State Management
Models exist in various operational states. A robust versioning system tracks whether a version is:
*   **Candidate:** Undergoing evaluation.
*   **Active:** Currently serving production traffic.
*   **Deprecated:** Still available but scheduled for removal.
*   **Archived:** Stored for historical or legal reasons but not available for deployment.

## Standard Model

The standard model for Versioning and Rollback follows a **Centralized Registry Pattern**:

1.  **Registration:** Upon completion of training, the model artifact and its metadata are pushed to a Model Registry.
2.  **Versioning Scheme:** Versions are assigned using either Semantic Versioning (Major.Minor.Patch) or Sequential Integer Versioning (v1, v2, v3).
3.  **Validation:** The version undergoes automated testing (e.g., accuracy thresholds, latency checks).
4.  **Tagging/Aliasing:** Instead of hardcoding version numbers in production, "aliases" (e.g., `@prod`, `@latest`) are used.
5.  **Deployment:** The infrastructure pulls the artifact associated with the `@prod` alias.
6.  **Rollback:** If an issue is detected, the `@prod` alias is re-pointed to the previous stable version identifier (e.g., from v12 to v11).

## Common Patterns

### Semantic Versioning for Models
*   **Major:** Breaking changes in input/output schema or fundamental algorithm shifts.
*   **Minor:** Significant improvements in accuracy or retraining with new data.
*   **Patch:** Minor optimizations or retraining with the same data distribution.

### Blue/Green Deployment
Two identical environments exist. The "Green" environment hosts the new model version. Once validated, traffic is cut over from "Blue" (old) to "Green" (new). Rollback is an instantaneous cut-back to "Blue."

### Canary Releases
A small percentage of traffic is routed to the new model version. If performance metrics remain stable, the percentage is increased. Rollback involves setting the canary traffic weight to zero.

## Anti-Patterns

*   **Overwriting the "Latest" File:** Saving a new model over an old file (e.g., `model_final.pkl`) destroys lineage and makes rollback impossible.
*   **Code-Only Versioning:** Relying on Git hashes to version models. This ignores the fact that the same code can produce different models if the training data changes.
*   **Manual Rollbacks:** Manually moving files or renaming artifacts during an incident. This is error-prone and lacks an audit trail.
*   **Environment Mismatch:** Versioning the model but failing to version the environment (e.g., Python library versions). The model may fail to load or behave differently.

## Edge Cases

*   **Breaking Schema Changes:** If a new model version requires a different input format than the previous version, a simple rollback may fail if the upstream application has already updated its request format. This requires "Coordinated Rollbacks."
*   **Hardware-Dependent Artifacts:** A model versioned for a GPU-accelerated environment may not be compatible with a CPU-only fallback environment.
*   **Large-Scale Distributed Models:** In systems with thousands of edge devices, rolling back a model version may be constrained by bandwidth or intermittent connectivity, leading to "Version Fragmentation."
*   **Online Learning:** Models that update continuously in production present a challenge for discrete versioning. In these cases, "State Snapshots" are used as version markers.

## Related Topics
*   **012 Data Lineage and Versioning:** The foundation for model reproducibility.
*   **088 Model Monitoring and Observability:** The trigger mechanism for rollbacks.
*   **104 CI/CD for Machine Learning (MLOps):** The automation framework for versioning.
*   **025 Model Governance:** The policy layer governing who can promote or roll back versions.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
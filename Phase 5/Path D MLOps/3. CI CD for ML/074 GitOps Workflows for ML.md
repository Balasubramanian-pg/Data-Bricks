# 074 GitOps Workflows for ML

Canonical documentation for 074 GitOps Workflows for ML. This document defines concepts, terminology, and standard usage.

## Purpose
GitOps Workflows for ML (Machine Learning) address the inherent friction between the experimental nature of data science and the rigorous requirements of production software operations. Traditional ML deployments often suffer from "manual handoffs," lack of reproducibility, and configuration drift. 

By applying GitOps principles—where the entire system state is described declaratively and stored in version control—organizations can achieve automated, auditable, and repeatable ML lifecycles. This topic establishes the framework for managing model training, evaluation, and deployment through a single source of truth.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
**In scope:**
*   Declarative definitions of ML infrastructure and model environments.
*   The reconciliation loop between desired state (Git) and actual state (Production).
*   Version control strategies for ML metadata, code, and configuration.
*   Promotion logic for moving models across environments (Dev, Staging, Prod).

**Out of scope:**
*   Specific vendor implementations (e.g., ArgoCD, Flux, Kubeflow).
*   Data engineering pipelines (ETL/ELT) unless directly triggering a GitOps reconciliation.
*   Exploratory Data Analysis (EDA) methodologies.

## Definitions
| Term | Definition |
|------|------------|
| **GitOps** | An operational framework that takes DevOps best practices used for application development—such as version control, collaboration, compliance, and CI/CD—and applies them to infrastructure automation. |
| **Declarative ML State** | A complete description of the ML system (model version, container image, resource requirements, environment variables) stored in a structured format (e.g., YAML, JSON). |
| **Reconciliation Loop** | A continuous process that compares the state defined in Git with the live environment and applies changes to align them. |
| **Hydrated Manifest** | The final, fully-rendered configuration file generated from templates, containing specific model versions and environment-specific parameters. |
| **Model Registry** | A centralized repository for managing model metadata and artifacts, acting as the bridge between training pipelines and GitOps deployment workflows. |
| **Source of Truth** | The Git repository containing the desired state of the ML system. |

## Core Concepts
### 1. Declarative Infrastructure and Models
Every component of the ML stack—from the inference server version to the specific model weights URI—must be defined declaratively. This ensures that the environment can be recreated from scratch using only the contents of the repository.

### 2. Versioned Artifacts
In ML GitOps, "versioning" extends beyond code. It includes:
*   **Code:** The inference logic or training scripts.
*   **Environment:** The container image and hardware requirements (e.g., GPU drivers).
*   **Model:** A unique identifier (hash) pointing to the model weights in a storage layer.

### 3. Automated Reconciliation
The system must automatically detect discrepancies between the Git repository and the running ML service. If a developer updates the model version in Git, the reconciliation agent must trigger a rolling update in the production environment without manual intervention.

### 4. Immutable Releases
Once a model and its configuration are committed to the main branch, they are considered immutable. Changes require a new commit, ensuring a clear audit trail and the ability to roll back by reverting a commit.

## Standard Model
The standard model for GitOps in ML follows a dual-loop architecture:

1.  **The Inner Loop (Experimental/CI):**
    *   Data scientists push code/parameters.
    *   CI system triggers training.
    *   Model artifacts are stored in a Model Registry.
    *   A "Candidate" manifest is generated.

2.  **The Outer Loop (Operational/CD):**
    *   The Candidate manifest is committed to a dedicated "Environment" Git repository.
    *   The GitOps controller detects the change.
    *   The controller pulls the specified model from the registry and deploys it to the target cluster.
    *   The controller monitors for drift and ensures the live state matches the Git state.

## Common Patterns
### Pull-based Deployment
An agent residing within the production cluster periodically polls the Git repository for changes. This is more secure as it does not require the CI system to have direct access to the production cluster.

### Environment-per-Branch/Folder
Configurations for `development`, `staging`, and `production` are separated into distinct branches or directories. Promotion involves merging a Pull Request (PR) from one branch/folder to another, allowing for peer review of model deployments.

### Shadow/Challenger Deployment
The GitOps controller deploys a new model version alongside the current production model. The "Challenger" receives a copy of live traffic for evaluation, but its results are not returned to the end-user until it is promoted via a Git update.

## Anti-Patterns
*   **Storing Binaries in Git:** Committing large model weights (`.bin`, `.pth`, `.onnx`) directly to Git. This bloats the repository and degrades performance. Use pointers/URIs instead.
*   **Manual Hotfixes:** Changing the model version or environment variables directly in the production cluster (e.g., via CLI) without updating Git. This leads to "drift" and will be overwritten by the reconciliation loop.
*   **Ambiguous Versioning:** Using the `latest` tag for container images or model versions. This breaks reproducibility and makes rollbacks impossible.
*   **Coupled Pipelines:** Forcing the training pipeline to directly update the production cluster. This bypasses the declarative state and audit benefits of GitOps.

## Edge Cases
### Large Model Weights (LLMs)
When models exceed several gigabytes, the reconciliation loop may time out during the "pull" phase. Standard GitOps workflows must be augmented with specialized artifact streaming or pre-caching mechanisms.

### Hardware Heterogeneity
Deploying ML models often requires specific hardware (e.g., NVIDIA A100 vs. T4). The GitOps manifest must include node affinity and toleration rules to ensure the model is scheduled on compatible hardware, or the reconciliation will fail.

### Online Learning
In systems where models update continuously based on streaming data, the "Source of Truth" in Git may lag behind the actual model state. In these cases, GitOps should manage the *infrastructure and training logic*, while the model state is managed by a specialized state store.

## Related Topics
*   **042 Model Versioning:** The methodology for identifying and storing model artifacts.
*   **089 Infrastructure as Code (IaC):** The underlying layer upon which GitOps for ML is built.
*   **102 Continuous Integration for ML:** The process of validating code and data before it reaches the GitOps loop.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
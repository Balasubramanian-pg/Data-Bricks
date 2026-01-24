# 042 Model Registry Staging and Prod

Canonical documentation for 042 Model Registry Staging and Prod. This document defines concepts, terminology, and standard usage.

## Purpose
The Model Registry Staging and Production framework exists to provide a governed, repeatable, and safe lifecycle for machine learning models. In the transition from experimental development to operational deployment, there is a critical need to separate models that are undergoing validation from those that are actively serving business-critical traffic. 

This topic addresses the problem of "model drift" in deployment processes, the lack of auditability in model promotion, and the risk of deploying unverified artifacts into live environments. By establishing formal stages, organizations can ensure that every model version meets specific quality, performance, and compliance benchmarks before reaching end-users.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
**In scope:**
* **Lifecycle Management:** The logical progression of model versions through discrete states.
* **Promotion Logic:** The criteria and mechanisms for moving models between environments.
* **Governance and Metadata:** The tracking of lineage, validation results, and approvals.
* **Environment Parity:** The conceptual alignment between registry stages and physical infrastructure.

**Out of scope:**
* **Specific vendor implementations:** (e.g., MLflow, SageMaker Model Registry, Vertex AI).
* **Model Training Algorithms:** The internal logic of how a model is built.
* **Infrastructure Provisioning:** The specific DevOps tools used to spin up servers or clusters.

## Definitions
| Term | Definition |
|------|------------|
| **Model Registry** | A centralized repository for managing model metadata, versions, and artifacts. |
| **Model Version** | An immutable snapshot of a model artifact, including its weights, architecture, and dependencies. |
| **Staging** | A pre-production state where models are validated against production-like data and integration tests. |
| **Production** | The state representing the model version currently authorized to serve live traffic or generate business output. |
| **Promotion** | The formal process of transitioning a model version from one stage to a higher-priority stage. |
| **Archived** | A terminal state for model versions that are no longer in use but are retained for historical or compliance reasons. |
| **Validation Gate** | A set of automated or manual checks that must be passed to allow promotion. |

## Core Concepts

### 1. State Immutability
Once a model version is registered, its underlying artifacts (weights, code, environment definitions) must be immutable. The "Staging" or "Production" designation is a metadata tag or pointer that moves between these immutable versions, rather than the version itself being modified.

### 2. Logical vs. Physical Separation
While the Model Registry manages the *logical* state (e.g., "Version 4 is now Production"), the deployment infrastructure must reflect this *physically*. The registry acts as the "Source of Truth" that deployment pipelines query to determine which artifact to pull.

### 3. Promotion Gates
Promotion is not merely a status change; it is the result of a successful validation gate. 
* **Dev to Staging:** Requires successful training completion, basic unit tests, and signature verification.
* **Staging to Production:** Requires performance benchmarking, integration testing, bias/fairness audits, and often a manual stakeholder approval.

### 4. Lineage and Traceability
Every model in Production must be traceable back to its Staging validation results, its training dataset, and the specific code commit that generated it.

## Standard Model

The standard model for Registry management follows a linear progression with feedback loops:

1.  **Registration:** A new model version is registered and assigned a "None" or "Experimental" stage.
2.  **Staging Promotion:** The model is promoted to "Staging." This triggers automated integration tests in an environment that mirrors production.
3.  **Validation:** The model undergoes "Shadow" or "Canary" testing where it processes live data but its outputs are not yet used for primary business decisions.
4.  **Production Promotion:** Upon meeting KPIs, the "Production" tag is moved from the old version to the new version.
5.  **Demotion/Archival:** The previous production version is moved to "Archived" or kept as a "Rollback" candidate.

## Common Patterns

### The "Blue/Green" Registry Pattern
Two versions are maintained in the registry: one tagged `Current_Prod` and one tagged `Next_Prod`. Traffic is shifted at the infrastructure level, and once stable, `Next_Prod` is promoted to `Current_Prod`.

### The "Shadow" Deployment Pattern
A model version in the `Staging` stage receives a mirror of production traffic. The registry tracks the performance of the `Staging` model against the `Production` model in real-time to justify promotion.

### Automated Promotion (Continuous Deployment)
In mature MLOps environments, if a model version passes all automated validation gates (e.g., accuracy > 0.95, latency < 100ms), the registry automatically updates the `Production` tag without human intervention.

## Anti-Patterns

*   **The "Latest" Tag Trap:** Relying on a generic `latest` tag for production deployments. This lacks version control and makes rollbacks nearly impossible.
*   **Manual Artifact Movement:** Manually copying model files from a "Staging bucket" to a "Prod bucket" instead of using the Registry's logical promotion features.
*   **Skipping Staging:** Promoting a model directly from training/experimental phase to Production because "the metrics look good."
*   **Ghost Models:** Running models in production that are not registered in the Model Registry, leading to a loss of lineage and auditability.

## Edge Cases

### Hotfixes
In the event of a production failure, a "Hotfix" path may bypass some non-critical staging gates. However, the model must still be registered and eventually back-filled through the standard validation process to maintain the audit trail.

### Multi-Region Consistency
In global deployments, the Model Registry must ensure that the "Production" tag is synchronized across all regions simultaneously to prevent inconsistent user experiences (e.g., Region A running V1 while Region B runs V2).

### Model Decay and Auto-Rollback
If a model in the `Production` stage begins to perform poorly (detected via monitoring), the registry should support an automated "Rollback" where the `Production` tag is immediately reverted to the last known-good `Archived` version.

## Related Topics
* **041 Model Versioning and Lineage:** The foundational requirement for registry stages.
* **043 Model Monitoring and Observability:** The feedback loop that informs promotion and demotion.
* **050 CI/CD for Machine Learning:** The automation framework that executes the transitions defined in the registry.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
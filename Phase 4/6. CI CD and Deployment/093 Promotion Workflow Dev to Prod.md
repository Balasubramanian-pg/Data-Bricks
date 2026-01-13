# 093 Promotion Workflow Dev to Prod

Canonical documentation for 093 Promotion Workflow Dev to Prod. This document defines concepts, terminology, and standard usage.

## Purpose
The 093 Promotion Workflow Dev to Prod exists to provide a structured, repeatable, and verifiable framework for transitioning software artifacts and configurations from a development state to a live production state. Its primary objective is to mitigate risk by ensuring that only validated, tested, and approved changes reach the end-user environment. This workflow addresses the problem of "environment drift," unverified changes, and the lack of traceability in the software delivery lifecycle.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* The logical progression of artifacts through discrete environments.
* Validation requirements and quality gates.
* The principle of immutability in promotion.
* Governance and traceability standards.

**Out of scope:**
* Specific vendor implementations (e.g., Jenkins, GitHub Actions, GitLab CI/CD).
* Specific cloud provider services (e.g., AWS CodePipeline, Azure DevOps).
* Detailed scripting or coding languages used to execute the workflow.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| Artifact | A discrete, versioned package or container representing the state of the application at a specific point in time. |
| Promotion | The act of advancing an artifact from a lower-order environment (e.g., Dev) to a higher-order environment (e.g., Prod). |
| Quality Gate | A set of predefined criteria (automated or manual) that must be satisfied before an artifact can be promoted. |
| Environment Parity | The practice of keeping development, staging, and production environments as similar as possible to ensure consistent behavior. |
| Immutability | The principle that an artifact, once built, is never modified; changes require the creation of a new artifact. |
| Drift | The divergence in configuration or state between environments over time. |

## Core Concepts

### 1. Build Once, Deploy Many
The cornerstone of the 093 workflow is that the artifact generated in the initial phase (Dev) is the exact same binary or package that eventually reaches Production. Configuration is injected at runtime, but the code remains untouched.

### 2. Environment Hierarchy
Environments are organized by increasing levels of stability and criticality:
*   **Development (Dev):** High churn, experimental, used for initial integration.
*   **Testing/QA:** Focused on functional and regression testing.
*   **Staging/Pre-Prod:** A mirror of Production used for final validation and performance testing.
*   **Production (Prod):** The live environment serving end-users.

### 3. Progressive Trust
As an artifact moves through the workflow, it gains "trust." Each successful gate completion increases the confidence level in the artifact's stability and security.

### 4. Traceability and Auditability
Every promotion must be recorded, identifying what was promoted, who (or what system) authorized it, and the results of the validation gates.

## Standard Model
The standard model for the 093 Promotion Workflow follows a linear progression with mandatory gates:

1.  **Commit/Build:** Code is committed, and a versioned artifact is created.
2.  **Dev Deployment:** Artifact is deployed to the Dev environment for initial smoke testing.
3.  **Gate 1 (Automated):** Unit tests, linting, and security scans must pass.
4.  **QA Promotion:** Artifact is promoted to QA for functional and integration testing.
5.  **Gate 2 (Manual/Automated):** User Acceptance Testing (UAT) and regression tests must pass.
6.  **Staging Promotion:** Artifact is promoted to Staging for final environment-specific checks.
7.  **Gate 3 (Approval):** Final stakeholder or automated "Go/No-Go" decision.
8.  **Production Promotion:** Artifact is deployed to the live environment.

## Common Patterns

### Linear Promotion
The most common pattern where an artifact moves sequentially through every environment in the hierarchy.

### Parallel Testing
Promoting an artifact to multiple testing environments simultaneously (e.g., Performance and Security environments) to reduce lead time, provided both must pass before reaching Staging.

### Blue/Green Deployment
A promotion pattern where the new version (Green) is deployed alongside the old version (Blue) in Production. Traffic is switched only after the Green environment is validated.

### Canary Release
A pattern where the artifact is promoted to a small subset of the Production environment to monitor behavior before a full-scale rollout.

## Anti-Patterns

*   **Manual Patching:** Modifying code or configuration directly in Production without going through the Dev-to-Prod workflow.
*   **Skipping Environments:** Promoting directly from Dev to Prod to save time, bypassing necessary validation gates.
*   **Environment-Specific Builds:** Recompiling or rebuilding the artifact for each environment, which introduces the risk of subtle differences between what was tested and what was deployed.
*   **Snowflake Environments:** Allowing environments to drift so significantly that a success in Staging no longer guarantees success in Production.

## Edge Cases

### Hotfixes (Emergency Promotion)
In the event of a critical production failure, a "Fast Track" workflow may be utilized. This still requires a build in Dev and a move through the hierarchy, but with an abbreviated or prioritized gate process. It must never bypass the creation of a new artifact.

### Data Schema Migrations
Promotions involving database changes require synchronized state management. The workflow must account for "forward-only" or "backward-compatible" schema changes to prevent breaking the application during the promotion window.

### Rollbacks vs. Roll-forwards
When a promotion fails in Production, the workflow must define whether to revert to the previous known-good artifact (Rollback) or quickly promote a new fixed artifact (Roll-forward).

## Related Topics
*   **042 Configuration Management:** How environment-specific variables are handled.
*   **115 Artifact Versioning:** Standards for naming and tracking artifacts.
*   **088 Continuous Integration:** The precursor to the promotion workflow.
*   **102 Identity and Access Management (IAM):** Controlling who has the authority to trigger promotions.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
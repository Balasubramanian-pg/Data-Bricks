# 092 Git Branches as Environments

Canonical documentation for 092 Git Branches as Environments. This document defines concepts, terminology, and standard usage.

## Purpose
The "Git Branches as Environments" pattern addresses the requirement to synchronize the state of software source code with specific deployment targets (environments). It provides a mechanism for teams to manage the progression of code through various stages of the software development lifecycle (SDLC)—such as development, testing, staging, and production—by utilizing the native branching and merging capabilities of Git.

This approach aims to provide visibility into what code is currently deployed where, using the Git repository as the "source of truth" for environment state.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative. While it describes workflows often used in CI/CD pipelines, it does not mandate specific tooling.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* The conceptual mapping of Git branches to logical deployment environments.
* Promotion strategies and merge workflows between environment branches.
* The relationship between version control state and infrastructure state.
* Theoretical boundaries of branch-based environment management.

**Out of scope:**
* Specific CI/CD vendor configurations (e.g., GitHub Actions YAML, GitLab CI/CD syntax).
* Infrastructure as Code (IaC) implementation details, except where they intersect with branching logic.
* Container orchestration specifics.

## Definitions
| Term | Definition |
|------|------------|
| **Environment** | A logical collection of hardware, software, and configurations where a specific version of an application is executed (e.g., Production, UAT). |
| **Promotion** | The process of moving a specific code state from a lower-order environment (e.g., Staging) to a higher-order environment (e.g., Production). |
| **Environment Branch** | A long-lived Git branch that represents the desired state of a specific environment. |
| **Drift** | The divergence in code or configuration between two environment branches that should otherwise be synchronized. |
| **Back-porting** | The act of merging changes from a higher-order environment branch (often a hotfix) back into lower-order branches to maintain parity. |
| **Upstream/Downstream** | The directional flow of code; typically, "Downstream" refers to environments closer to production. |

## Core Concepts

### 1. Branch-to-Environment Mapping
In this model, a 1:1 relationship is established between a persistent Git branch and a deployment target. A push or merge to a specific branch serves as the authoritative signal to update the corresponding environment to that branch's latest commit.

### 2. Promotion via Merging
Code moves through the SDLC by merging one environment branch into the next. This creates a linear or semi-linear history of how features progressed from development to production.

### 3. Environment Parity
The goal of using branches as environments is to ensure that the code running in "Staging" is identical to the code that will eventually run in "Production," with the only differences being externalized configurations.

### 4. Configuration Decoupling
To maintain the integrity of the "build once, deploy anywhere" principle, environment-specific configurations (database strings, API keys, resource limits) must be decoupled from the branch itself. The branch should contain the logic; the environment provides the context.

## Standard Model
The standard model follows a hierarchical progression. Code is integrated into the lowest-level branch and promoted upward.

1.  **Development Branch (`dev`/`develop`):** The integration point for new features. Represents the "bleeding edge" environment.
2.  **Staging Branch (`staging`/`qa`):** Represents the environment used for final testing or User Acceptance Testing (UAT).
3.  **Production Branch (`main`/`master`/`prod`):** Represents the current live state of the application.

**The Flow:**
`Feature Branch` → `Development` → `Staging` → `Production`

## Common Patterns

### The Promotion Flow
A strictly linear path where `Branch A` is merged into `Branch B` only after validation. This ensures that no code reaches production without first existing in (and being tested in) all preceding environments.

### The Release Branch Pattern
Instead of a permanent "Staging" branch, a temporary release branch (e.g., `release/v1.2.0`) is created from `develop`. This branch is deployed to a staging environment. Once validated, it is merged into `main` (Production) and `develop`.

### Environment-per-Feature (Ephemeral)
Short-lived branches that trigger the creation of temporary, isolated environments. Once the branch is merged or deleted, the environment is decommissioned.

## Anti-Patterns

### 1. Environment-Specific Code
Committing code changes that are only intended for one environment (e.g., hardcoding a production URL in the `main` branch) breaks the principle of parity and makes merges difficult.

### 2. Manual Hotfixing without Back-porting
Applying a fix directly to the `production` branch without merging it back into `develop` and `staging`. This leads to "regression" where the bug reappears in the next standard release.

### 3. Long-Lived Divergence
Allowing environment branches to diverge so significantly that merging becomes high-risk. This usually happens when "Staging" becomes a dumping ground for unfinished features that are not ready for "Production."

### 4. Rebuilding Artifacts per Branch
Compiling or building a new binary/image for each branch. Ideally, an artifact is built once (usually at the `develop` or `staging` stage) and the *same* artifact is promoted across environments.

## Edge Cases

### Hotfixes
When a critical bug is found in Production, a hotfix branch is created from the `production` branch. The challenge lies in the "Double Merge": the fix must be merged into `production` for immediate relief and into `develop` to prevent regression.

### Rollbacks
Reverting an environment to a previous state. In a branch-based model, this can be achieved via `git revert` or by pointing the environment to a previous commit hash. However, this can cause the branch state to conflict with the intended "forward-only" flow of the SDLC.

### Database Schema Migrations
Git branches manage code state effectively, but data state is persistent. A branch promotion that includes a destructive schema change cannot be easily "merged" or "reverted" in the same way as source code.

## Related Topics
* **012 Trunk-Based Development:** An alternative workflow that avoids long-lived environment branches in favor of short-lived feature flags.
* **045 Continuous Integration/Continuous Deployment (CI/CD):** The automation layer that executes the deployment when branches are updated.
* **078 GitOps:** A specialized implementation of branches-as-environments focusing on declarative infrastructure.
* **102 Twelve-Factor App Methodology:** Specifically the "Config" factor, which is critical for successful branch-based environments.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
# 091 Using Databricks Repos for Versioning

Canonical documentation for 091 Using Databricks Repos for Versioning. This document defines concepts, terminology, and standard usage.

## Purpose
The integration of version control systems (VCS) within a unified data analytics platform addresses the critical need for software engineering best practices in data science and data engineering workflows. 

Historically, data workspaces suffered from "siloed" development where code lived only within the platform's proprietary storage, leading to difficulties in collaboration, lack of audit trails, and manual, error-prone deployment processes. Databricks Repos (now frequently referred to as Git Folders) provides a bridge between the interactive workspace and remote Git providers. This enables professionalized development lifecycles, including code reviews, automated testing, and Continuous Integration/Continuous Deployment (CI/CD) pipelines.

> [!NOTE]
> This documentation is intended to be implementation-agnostic regarding the specific Git provider (e.g., GitHub, GitLab, Bitbucket, Azure DevOps) and authoritative regarding the architectural role of Repos within the Databricks ecosystem.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Integration logic between the Databricks Workspace and Remote Git Providers.
* Management of notebook and non-notebook files (Python, SQL, YAML, etc.) within a versioned context.
* The conceptual workflow of branching, committing, and merging within a data platform.
* Security and permission models for repository access.

**Out of scope:**
* Specific tutorials for third-party Git providers (e.g., "How to create a GitHub account").
* Deep-dive Git CLI instruction (e.g., advanced rebasing techniques).
* Legacy "Workspace Sync" tools that do not utilize the native Repos API.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Remote Provider** | An external Git-based service (e.g., GitHub, GitLab) that acts as the source of truth for the codebase. |
| **Databricks Repo** | A specialized folder within the Databricks workspace that is backed by a remote Git repository. |
| **Sparse Checkout** | A Git feature that allows only a subset of the repository's directories to be populated in the workspace to improve performance. |
| **Personal Access Token (PAT)** | A credential used to authenticate the Databricks workspace with the Remote Provider. |
| **Arbitrary Files** | Non-notebook files (e.g., .py, .txt, .json, .yaml) that are stored and versioned alongside notebooks within a Repo. |
| **Head Revision** | The most recent commit in the current branch of the repository. |

## Core Concepts

### 1. Bidirectional Synchronization
Databricks Repos facilitate a bidirectional flow of information. Changes made in the interactive workspace (notebooks or files) can be committed and pushed to the remote provider. Conversely, changes made by other contributors or automated processes on the remote provider can be pulled into the workspace.

### 2. The Workspace as a Git Client
The platform acts as a visual Git client. It manages the underlying Git state (the `.git` directory) within the platform's control plane, abstracting the complexity of Git commands into a user interface while maintaining the integrity of the Git tree.

### 3. Notebook Serialization
While notebooks appear as interactive documents in the UI, the Repo system serializes them into standard source formats (e.g., `.py`, `.sql`, `.scala`) or a specialized JSON format when pushed to the remote. This ensures that code reviews can be conducted using standard diffing tools on the remote provider.

### 4. Environment Isolation
Repos allow for the isolation of code across different stages of the software development lifecycle (SDLC). By mapping different workspace folders or different workspaces entirely to specific branches (e.g., `dev`, `staging`, `main`), organizations can enforce strict promotion paths.

## Standard Model
The standard model for using Repos involves a "Branch-per-Developer" or "Branch-per-Feature" workflow:

1.  **Clone:** A developer clones the remote repository into their private user folder within the Databricks workspace.
2.  **Branch:** The developer creates a new feature branch from the latest `main` or `develop` branch.
3.  **Develop:** The developer modifies notebooks and files. The platform tracks these changes in real-time.
4.  **Commit & Push:** The developer commits changes with a descriptive message and pushes them to the remote provider.
5.  **Pull Request (PR):** Outside of Databricks, a PR is opened on the remote provider for peer review.
6.  **Merge & Deploy:** Once the PR is merged, a CI/CD pipeline may trigger a "Pull" operation in a production-level Repo folder to update the live environment.

## Common Patterns

### The Production Service Principal Pattern
For automated environments (Production), Repos are often cloned using a Service Principal rather than an individual user's identity. This ensures that the production code remains accessible and updatable even if an individual developer leaves the organization.

### Multi-Repo Project Structure
Large projects are often broken into multiple repositories:
*   **Infrastructure Repo:** For Terraform/ARM templates.
*   **Data Pipeline Repo:** For ETL logic and DLT pipelines.
*   **Analysis Repo:** For exploratory data analysis (EDA) and reporting.

### Library Integration
Using Repos to manage custom Python wheels or utility modules. By placing a `utils.py` file in a Repo, other notebooks within the same Repo can import functions using standard Python import syntax, provided the Repo is added to the Python path.

## Anti-Patterns

*   **Committing Secrets:** Hardcoding API keys or credentials in notebooks and pushing them to the remote. (Standard practice: Use Secret Scopes).
*   **Monolithic Repositories:** Including massive data files (CSV, Parquet) or large binary artifacts in the Repo, which degrades performance and violates Git best practices.
*   **Direct Editing in Production:** Manually changing code in a "Production" Repo folder instead of following the PR and merge process.
*   **Ignoring Merge Conflicts:** Allowing the workspace state to diverge significantly from the remote, leading to complex, unresolvable conflicts.

## Edge Cases

*   **Detached HEAD State:** Occurs when the workspace is pointing to a specific commit rather than a branch. This prevents direct commits until a branch is checked out.
*   **Notebook Metadata Conflicts:** Frequent changes to notebook metadata (like cell execution counts) can cause "noisy" diffs. Modern platforms attempt to strip this metadata, but it can still appear in certain configurations.
*   **Large File Limits:** Git providers and the Databricks platform impose limits on individual file sizes (typically 10MB to 100MB). Files exceeding these limits will fail to sync.
*   **Renaming/Moving Files:** Renaming a notebook in the UI may be interpreted by Git as a "Delete" and "Add" operation, potentially losing historical context in the Git log if not handled correctly by the provider's diff engine.

## Related Topics
*   **042 CI/CD for Data Engineering:** The automation of testing and deployment using Repo APIs.
*   **015 Workspace Security:** Managing permissions (Can Read, Can Run, Can Edit, Can Manage) on Repo folders.
*   **108 Secret Management:** The proper way to handle credentials that should never enter version control.
*   **077 Delta Live Tables (DLT):** Using Repos as the source for declarative pipeline definitions.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
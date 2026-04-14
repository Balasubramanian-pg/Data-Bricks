# 088 Azure DevOps Pipelines

Canonical documentation for 088 Azure DevOps Pipelines. This document defines concepts, terminology, and standard usage.

## Purpose
Azure DevOps Pipelines exists to provide a unified automation framework for the continuous integration (CI) and continuous delivery (CD) of software. It addresses the problem of manual, error-prone software delivery processes by providing a scalable, programmable environment to build, test, and deploy code across diverse platforms and cloud environments. Its primary function is to orchestrate the lifecycle of an application from source code to production-ready state, ensuring consistency, traceability, and quality at every stage of the software development life cycle (SDLC).

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative regarding the architectural principles of Azure DevOps Pipelines.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Orchestration Logic:** The hierarchy of stages, jobs, and tasks.
* **Execution Environment:** The role of agents (hosted and private) and pools.
* **Configuration Paradigms:** Declarative (YAML) and visual (Classic) pipeline definitions.
* **Integration Points:** Service connections, environments, and artifact management.
* **Security & Governance:** Permissions, variable groups, and approval gates.

**Out of scope:**
* **Source Control Management:** While pipelines interact with repositories (Azure Repos, GitHub), the internal mechanics of Git are out of scope.
* **Third-party Tool Internals:** The internal logic of specific compilers (e.g., MSBuild, Maven) or deployment targets (e.g., Kubernetes, AWS) beyond their interface with the pipeline.
* **Pricing and Licensing:** Commercial aspects of the service.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Pipeline** | The top-level automation process that defines the CI/CD workflow for a project. |
| **Stage** | A logical boundary within a pipeline, often used to separate major phases like "Build," "Test," and "Production." |
| **Job** | A series of steps that run on a single agent. Jobs can run in parallel or sequentially. |
| **Step** | The smallest unit of execution, which can be a script or a pre-defined task. |
| **Agent** | A computing resource (virtual or physical) that executes the jobs defined in the pipeline. |
| **Artifact** | A collection of files or packages produced by a build process to be consumed by subsequent stages or external systems. |
| **Trigger** | An event (such as a code push, pull request, or schedule) that initiates the execution of a pipeline. |
| **Environment** | A collection of resources (VMs, Kubernetes clusters) targeted by a deployment, often associated with deployment history and approval gates. |
| **Variable Group** | A set of key-value pairs used to store configuration data and secrets across multiple pipelines. |
| **Service Connection** | An abstraction layer used to securely connect the pipeline to external services (e.g., Azure, Docker Hub, SonarQube). |

## Core Concepts

### Declarative vs. Imperative Configuration
Azure DevOps Pipelines supports two primary modes:
1.  **YAML (Declarative):** The modern standard where the pipeline is defined as code and stored in the version control system alongside the application code. This enables versioning, code reviews, and consistency.
2.  **Classic (Imperative):** A visual designer interface where pipelines are configured through a web UI. While accessible, it lacks the native versioning and portability of YAML.

### The Execution Hierarchy
The pipeline follows a strict hierarchy: **Pipeline > Stage > Job > Step**. 
*   **Stages** provide isolation and can require manual approvals.
*   **Jobs** define the execution context (the agent) and can be distributed across multiple agents for parallelism.
*   **Steps** are executed linearly within the job's context.

### Agent Orchestration
Pipelines rely on **Agent Pools**. Agents can be **Microsoft-hosted** (managed by the provider, ephemeral) or **Self-hosted** (managed by the user, persistent). The choice of agent impacts build speed, security, and access to internal network resources.

### Artifact Traceability
A core concept is the "Build Once, Deploy Many" principle. The pipeline produces an immutable artifact in an early stage, which is then promoted through various environments without being recompiled, ensuring that what was tested is exactly what is deployed.

## Standard Model
The standard model for an Azure DevOps Pipeline is the **Multi-Stage YAML Pipeline**. This model emphasizes:
1.  **Source Control Integration:** The `azure-pipelines.yml` file resides in the root of the repository.
2.  **Separation of Concerns:** Distinct stages for Build, Alpha/Test, and Production.
3.  **Environment-Based Deployment:** Using the `deployment` job type to target specific environments, enabling features like "Run Once," "Rolling," or "Canary" strategies.
4.  **Template Reuse:** Utilizing templates to standardize common tasks (e.g., security scanning) across multiple pipelines to reduce duplication.

## Common Patterns

### Fan-out/Fan-in
Executing multiple jobs in parallel (e.g., running unit tests on different operating systems) and then merging the results into a single downstream job (e.g., a deployment stage).

### Matrix Strategy
A pattern used to run the same job multiple times with different configurations (e.g., testing a Node.js application against versions 14, 16, and 18) without duplicating code.

### Infrastructure as Code (IaC) Integration
Pipelines that include stages to provision or update infrastructure (using Terraform, Bicep, or ARM) before deploying the application code.

### Gated Releases
The use of "Approvals and Checks" on environments to ensure that deployments to sensitive areas (like Production) require manual intervention or automated health checks.

## Anti-Patterns

*   **Monolithic Jobs:** Placing all tasks (Build, Test, Deploy) into a single job. This prevents parallelism and makes troubleshooting difficult.
*   **Hardcoded Secrets:** Placing passwords or API keys directly in the YAML file instead of using Variable Groups or Azure Key Vault integration.
*   **Script Overload:** Using massive, complex Bash or PowerShell scripts for logic that could be handled by native, maintained Pipeline Tasks.
*   **Lack of Artifact Versioning:** Deploying directly from a source branch without creating a versioned, immutable artifact.
*   **Ignoring Build Telemetry:** Failing to publish test results or code coverage, which limits the visibility of software quality.

## Edge Cases

*   **Self-Hosted Agents in Air-Gapped Networks:** Scenarios where agents have no outbound internet access and must rely on local mirrors for dependencies and artifacts.
*   **Long-Running Deployments:** Jobs that exceed the default timeout limits (e.g., complex database migrations), requiring specific timeout overrides and heartbeat monitoring.
*   **Flaky Test Management:** Handling tests that intermittently fail due to environmental factors rather than code bugs, often requiring "Retry" logic or "Test Slicing."
*   **Multi-Repo Triggers:** Complex scenarios where a pipeline must trigger based on changes in a repository other than the one containing the YAML definition.

## Related Topics
*   **087 Azure Repos:** The source control system that typically triggers the pipeline.
*   **089 Azure Artifacts:** The package management service often used to store pipeline outputs.
*   **090 Azure Environments:** The logical targets for deployment orchestration.
*   **Service Connections:** The security mechanism for cross-platform integration.
*   **YAML Schema:** The specific syntax and structure required for pipeline definitions.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
# 107 Unity Catalog Permissions on Pipelines

Canonical documentation for 107 Unity Catalog Permissions on Pipelines. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of 107 Unity Catalog Permissions on Pipelines is to establish a centralized governance framework for data orchestration entities. In modern data architectures, pipelines represent the bridge between raw data and consumable assets. By integrating pipeline permissions into a unified catalog, organizations can ensure that data transformation logic is subject to the same rigorous access controls, auditability, and security standards as the data itself. This prevents unauthorized modification of data flows and ensures that only authorized principals can execute or observe data processing logic.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* The theoretical framework for governing pipeline entities within a centralized metadata repository.
* The hierarchy of privileges required for pipeline lifecycle management (creation, execution, modification).
* The relationship between pipeline identities and data access permissions.
* Security principles governing the interaction between the orchestration layer and the storage layer.

**Out of scope:**
* Specific syntax for cloud-provider-specific CLI or API calls.
* Step-by-step UI tutorials for specific vendor platforms.
* General network-level security (e.g., VPC peering, firewalls) not managed by the catalog.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Pipeline** | A logical entity representing a series of data processing steps, including ingestion, transformation, and loading. |
| **Principal** | An identity (user, group, or service principal) that can be granted permissions on a pipeline. |
| **Privilege** | A specific action or set of actions that a principal is authorized to perform on a pipeline entity. |
| **Unified Catalog** | A centralized governance layer that manages metadata and access control for data and compute assets across an organization. |
| **Ownership** | A special state where a principal has full control over a pipeline, including the ability to grant permissions to others. |
| **Execution Context** | The security identity under which a pipeline runs, determining its access to underlying data sources and sinks. |

## Core Concepts

### 1. The Principle of Unified Governance
Pipelines are treated as first-class citizens within the catalog. This means they are not merely external scripts but are registered entities with unique identifiers, metadata, and associated Access Control Lists (ACLs).

### 2. Identity Decoupling
A fundamental concept is the separation between the **Owner** (who manages the pipeline definition) and the **Execution Identity** (the principal whose permissions are used to read/write data). In a secure model, the pipeline often executes as a service principal rather than the individual user who triggered it.

### 3. Permission Inheritance
Permissions may flow from higher-level containers (such as catalogs or schemas) down to the pipeline entity. However, pipelines also require specific functional permissions that differ from standard data object permissions (e.g., "Run" vs. "Read").

### 4. Lineage Integration
Permissions on pipelines are intrinsically linked to data lineage. To view the lineage produced by a pipeline, a principal typically requires specific visibility permissions on the pipeline entity itself.

## Standard Model

The standard model for pipeline permissions follows a hierarchical RBAC (Role-Based Access Control) structure:

*   **MANAGE (Owner):** Full control. Can modify the pipeline code, change settings, delete the pipeline, and grant/revoke permissions to other principals.
*   **CAN_RUN:** Permission to trigger a pipeline execution. This does not necessarily grant permission to view the underlying code or modify the configuration.
*   **CAN_VIEW:** Read-only access to the pipeline's metadata, execution history, and status. This is essential for monitoring and observability roles.
*   **CAN_EDIT:** Permission to modify the pipeline configuration and code, but typically excludes the ability to change ownership or delete the entity.

## Common Patterns

### The Service Principal Pattern
To ensure production stability, pipelines are owned and executed by a Service Principal rather than a human user. Human developers are granted `CAN_VIEW` or `CAN_RUN` permissions in staging environments, but only the Service Principal has `MANAGE` rights in production.

### Environment-Based Isolation
Permissions are segregated by environment (Dev, QA, Prod). A principal may have `MANAGE` rights in a Development catalog/schema but only `CAN_VIEW` rights in the Production equivalent.

### Group-Based Access Control
Permissions are rarely granted to individuals. Instead, they are mapped to functional groups (e.g., `Data_Engineers`, `Data_Analysts`, `Operations_Team`), simplifying the onboarding and offboarding process.

## Anti-Patterns

*   **Individual Ownership in Production:** Allowing a human user to own a production pipeline. If the user leaves the organization, the pipeline may become unmanageable or fail due to identity deactivation.
*   **Over-Privileged Execution Context:** Running a pipeline with "Super User" or "Account Admin" privileges. Pipelines should follow the "Least Privilege" principle, having access only to the specific tables and locations required for their task.
*   **Hardcoded Credentials:** Storing secrets or access keys within the pipeline definition rather than leveraging the catalog's integrated secret management and identity-based access.
*   **Manual Permission Drift:** Modifying permissions directly on the storage layer (e.g., S3/ADLS) instead of through the Unified Catalog, leading to a "split-brain" security state.

## Edge Cases

*   **Orphaned Pipelines:** When the owner of a pipeline is removed from the system without transferring ownership. The catalog must provide a mechanism for an administrator to reclaim and reassign these entities.
*   **Cross-Catalog Dependencies:** A pipeline residing in Catalog A that transforms data residing in Catalog B. This requires the pipeline's execution identity to have cross-catalog permissions, which must be explicitly governed.
*   **Dynamic Pipeline Generation:** Systems that programmatically create pipelines. These systems must be designed to automatically inject the correct ACLs at the moment of creation to avoid "permissionless" entities.
*   **Temporary Elevated Access:** Scenarios where a developer needs `CAN_EDIT` access to a production pipeline for emergency debugging. This should be handled via time-bound "Just-In-Time" (JIT) access requests rather than permanent permission changes.

## Related Topics
*   **101 Data Access Control:** The foundational model for table and file permissions.
*   **105 Identity Management:** Standards for users, groups, and service principals.
*   **202 Data Lineage and Observability:** How pipeline execution metadata is captured and displayed.
*   **308 Secret Management:** Securely handling credentials within automated workflows.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
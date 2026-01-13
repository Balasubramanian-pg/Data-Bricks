# 102 Databricks Secret Scopes

Canonical documentation for 102 Databricks Secret Scopes. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of Secret Scopes is to provide a secure, abstracted layer for managing sensitive information—such as API keys, database credentials, and encryption tokens—within a data processing environment. By decoupling credentials from source code, Secret Scopes enable secure collaboration, prevent accidental credential exposure in logs or version control, and facilitate the rotation of credentials without requiring modifications to application logic.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative regarding the architectural behavior of secret management within the Databricks ecosystem.

## Scope
**In scope:**
* The logical architecture of secret containers (scopes).
* Access Control List (ACL) mechanisms for secret management.
* The relationship between secret backends and the Databricks abstraction layer.
* Security properties such as redaction and non-exportability.

**Out of scope:**
* Specific cloud provider CLI commands (e.g., `databricks secrets create-scope`).
* Network-level security configurations (VNets, Private Links).
* Third-party password manager integrations not natively supported by the Secret Scope API.

## Definitions
| Term | Definition |
|------|------------|
| **Secret Scope** | A logical container for secrets, identified by a unique name within a workspace. |
| **Secret** | A key-value pair where the key is a unique identifier and the value is the sensitive data. |
| **Secret Backend** | The underlying storage mechanism for the secrets (e.g., Databricks-encrypted database or a cloud-native Key Vault). |
| **ACL (Access Control List)** | A set of permissions attached to a scope that defines which principals can read, write, or manage the secrets therein. |
| **Principal** | A user, service principal, or group to which permissions are granted. |
| **Redaction** | The automated process of intercepting and masking secret values in standard output and logs. |

## Core Concepts
### Abstraction Layer
Secret Scopes act as a pointer. Code references a secret by its scope name and key name (e.g., `dbutils.secrets.get(scope="production", key="db_password")`). This ensures that the code remains identical across development, staging, and production environments, while the underlying values differ based on the environment's scope configuration.

### Backend Types
There are two primary types of backends for Secret Scopes:
1.  **Databricks-backed:** Secrets are stored in a proprietary, encrypted database managed by the platform.
2.  **Cloud-native-backed:** The scope acts as a transparent proxy to a cloud provider's managed key vault (e.g., Azure Key Vault).

### Security Persistence
Secrets are designed to be "write-only" via the management interface and "read-only" via programmatic execution. Once a secret is stored, its value cannot be retrieved through the UI or CLI; it can only be injected into a runtime environment where the platform's redaction engine monitors for its presence in output buffers.

## Standard Model
The standard model for Secret Scope implementation follows the **Principle of Least Privilege (PoLP)**:

1.  **Isolation:** Scopes should be partitioned by business unit, environment (Dev/Prod), or application boundary.
2.  **Access Levels:**
    *   **READ:** Allows a principal to retrieve the secret value within a notebook or job.
    *   **WRITE:** Allows a principal to create or update secrets within the scope.
    *   **MANAGE:** Allows a principal to change ACLs and delete the scope.
3.  **Naming Convention:** Scopes should follow a predictable naming convention (e.g., `[project_name]_[environment]`) to prevent collisions and simplify automation.

## Common Patterns
### Environment Mirroring
Using identical scope names across different workspaces (e.g., a "Finance" scope in both Dev and Prod workspaces) allows code to be promoted through the CI/CD pipeline without configuration changes.

### Service Principal Delegation
Assigning `MANAGE` permissions to a Service Principal rather than individual users. This ensures that secret lifecycle management is handled by automated pipelines, reducing human exposure to sensitive material.

### Group-Based ACLs
Permissions are granted to directory groups (e.g., `data-engineers-group`) rather than individual users to simplify onboarding and offboarding.

## Anti-Patterns
*   **The "Global" Scope:** Creating a single scope for all secrets in an organization. This violates isolation principles and increases the blast radius of a credential leak.
*   **Hardcoding Scope Names in Libraries:** Hardcoding specific scope names inside shared library code makes the library less reusable across different projects.
*   **Manual Secret Entry in Production:** Manually typing secrets into a production scope via CLI, which is prone to human error and lacks an audit trail.
*   **Over-Privileged Users:** Granting `MANAGE` permissions to data scientists or analysts who only require `READ` access for job execution.

## Edge Cases
*   **Scope Name Collisions:** Scope names must be unique within a workspace. If a cloud-native vault is linked, the scope name in Databricks does not necessarily have to match the vault name, but it is recommended for clarity.
*   **Secret Size Limits:** Most backends impose a limit on the size of the secret value (typically 128 KB). Attempting to store large configuration files as secrets will result in failure.
*   **Redaction Limitations:** Redaction is based on exact string matching. If a secret is transformed (e.g., Base64 encoded or reversed) within a script, the platform will no longer recognize it, and it may be leaked in plain text to logs.
*   **Deleted Backends:** If a cloud-native Key Vault is deleted or its permissions are changed externally, the Databricks Secret Scope will remain but will throw execution-time errors when accessed.

## Related Topics
*   **Identity Federation:** How users and groups are synced to the workspace to be used in ACLs.
*   **Cluster Security Modes:** How different cluster types (High Concurrency vs. Standard) handle secret isolation between users.
*   **CI/CD Integration:** Automating the deployment of secret scopes using Infrastructure as Code (IaC).

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
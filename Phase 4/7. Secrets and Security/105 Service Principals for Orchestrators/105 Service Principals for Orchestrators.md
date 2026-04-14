# 105 Service Principals for Orchestrators

Canonical documentation for 105 Service Principals for Orchestrators. This document defines concepts, terminology, and standard usage.

## Purpose
The 105 Service Principals for Orchestrators framework addresses the requirement for non-human identities to interact with infrastructure, APIs, and protected resources within automated environments. In modern computing, orchestrators—systems that automate the deployment, management, and scaling of applications—require a mechanism to prove their identity and authority without relying on human credentials.

This topic exists to standardize how these machine identities are defined, scoped, and secured, ensuring that automated workflows can operate autonomously while maintaining a robust security posture and audit trail.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Identity Lifecycle:** The creation, rotation, and decommissioning of non-human identities.
* **Authentication Mechanisms:** The methods by which an orchestrator proves its identity to a resource provider.
* **Authorization Scoping:** The theoretical boundaries of "Least Privilege" as applied to automated systems.
* **Trust Relationships:** The establishment of trust between an Identity Provider (IdP) and an Orchestration Engine.

**Out of scope:**
* **Specific Vendor Implementations:** Detailed guides for Azure Service Principals, AWS IAM Roles, or Google Cloud Service Accounts.
* **Human Identity Management:** Standard User Access Management (UAM) or Single Sign-On (SSO) for physical persons.
* **Network-level Security:** Firewalls, VPCs, or SD-WAN configurations, except where they intersect with identity verification.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Service Principal** | A unique identity created for use with applications, hosted services, and automated tools to access specific resources. |
| **Orchestrator** | A centralized system (e.g., CI/CD platform, container orchestrator, or workflow engine) that manages the execution of automated tasks. |
| **Identity Provider (IdP)** | The authoritative system that creates, maintains, and manages identity information and provides authentication services. |
| **Secret/Credential** | A piece of information (e.g., certificate, key, or token) used by the Service Principal to authenticate its identity. |
| **Trust Policy** | A document or configuration that defines which entities are allowed to assume or use a specific Service Principal. |
| **Workload Identity** | A specific subset of Service Principals assigned to a granular unit of work (e.g., a single microservice or job). |

## Core Concepts

### Non-Human Identity (NHI)
Unlike human identities, Service Principals do not have passwords that change based on human memory or biometric factors. They rely on cryptographic keys, certificates, or short-lived tokens. The core concept is the decoupling of the action from a specific human user, allowing for 24/7 operations and consistency.

### The Principle of Least Privilege (PoLP)
Service Principals must be granted only the minimum permissions necessary to perform their designated task. Because these identities are used by code, any over-provisioning represents a significant security risk, as automated systems can execute commands at a scale and speed impossible for humans.

### Identity Federation
Modern orchestration often utilizes federation, where the orchestrator does not store long-lived secrets. Instead, it exchanges a trusted token from its own environment for a short-lived access token from the target resource's Identity Provider.

## Standard Model

The standard model for 105 Service Principals follows a four-stage architecture:

1.  **Registration:** The Service Principal is defined within the Identity Provider with a unique identifier (UUID/Client ID).
2.  **Attestation:** The Orchestrator presents a "proof of identity." In the standard model, this is ideally a short-lived, dynamically generated token rather than a static client secret.
3.  **Authorization:** The Resource Provider checks the Service Principal's identity against an Access Control List (ACL) or Role-Based Access Control (RBAC) policy.
4.  **Audit:** Every action performed by the Service Principal is logged with its unique identifier, providing a clear trail of "who" (the principal) did "what" (the action) and "when."

## Common Patterns

### The "Sidecar" Identity Pattern
In containerized environments, a sidecar process manages the retrieval and rotation of tokens for the Service Principal, abstracting the authentication logic away from the primary application code.

### Environment-Specific Principals
A pattern where distinct Service Principals are created for Development, Staging, and Production environments. This ensures that a compromise in a lower environment does not grant access to production data.

### Just-In-Time (JIT) Elevation
The Service Principal maintains a baseline of zero permissions and is granted temporary elevated access only during the execution of a specific orchestration window (e.g., during a deployment).

## Anti-Patterns

*   **Shared Principals:** Using a single Service Principal for multiple unrelated applications or orchestrators. This breaks the principle of isolation and makes auditing impossible.
*   **Hardcoded Secrets:** Embedding Service Principal credentials (Client Secrets/Keys) directly into source code or unencrypted configuration files.
*   **Perpetual Credentials:** Using secrets that never expire. This increases the window of opportunity for an attacker if the credential is leaked.
*   **Over-Privileged "Admin" Principals:** Assigning "Owner" or "Global Admin" roles to an orchestrator to "avoid permission issues" during development.

## Edge Cases

### Cross-Tenant/Cross-Organization Access
When an orchestrator in Organization A must manage resources in Organization B. This requires complex trust relationship configurations and careful monitoring of "Confused Deputy" vulnerabilities.

### Legacy System Integration
Interacting with systems that do not support modern identity protocols (OIDC/OAuth2). In these cases, the Service Principal must often be "downgraded" to use static API keys, requiring external secret management (e.g., a Vault).

### High-Churn Environments
In environments where thousands of short-lived ephemeral tasks are spawned per minute, the Identity Provider may face rate-limiting or "identity bloat." This requires a model of "Identity Reuse" or "Scoped Token Exchange" rather than unique principal registration for every task.

## Related Topics
*   **101 Identity and Access Management (IAM):** The foundational layer for all identity concepts.
*   **202 Secrets Management:** The technical implementation of storing and rotating the credentials used by Service Principals.
*   **305 Workload Identity Federation:** The advanced method of connecting disparate identity providers without static secrets.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
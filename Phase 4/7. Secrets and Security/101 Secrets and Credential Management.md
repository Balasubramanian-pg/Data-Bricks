# 101 Secrets and Credential Management

Canonical documentation for 101 Secrets and Credential Management. This document defines concepts, terminology, and standard usage.

## Purpose
Secrets and Credential Management addresses the critical need to protect sensitive information that grants access to protected resources, systems, and data. In modern distributed computing, applications and services must authenticate with one another, necessitating the exchange of non-public data. Without a formal management framework, these "secrets" are often stored insecurely (e.g., in source code, configuration files, or environment variables), leading to "secret sprawl" and increased risk of unauthorized access or systemic compromise.

This topic establishes the framework for the secure lifecycle management of these assets, ensuring that only authorized entities can access them, that their usage is audited, and that their exposure is minimized through temporal and procedural constraints.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Lifecycle Management:** The creation, storage, distribution, rotation, and revocation of secrets.
* **Access Control:** Mechanisms for defining who or what can access a secret.
* **Auditability:** The tracking of secret access and modification.
* **Identity-Based Access:** The transition from static credentials to identity-driven authorization.
* **Encryption at Rest and Transit:** The protection of secrets during storage and movement.

**Out of scope:**
* **Specific vendor implementations:** (e.g., HashiCorp Vault, AWS Secrets Manager, Azure Key Vault).
* **Cryptographic Algorithm Design:** The mathematical implementation of AES, RSA, etc.
* **General Network Security:** Firewalls, VPCs, and transport layer security (TLS) except where they directly facilitate secret delivery.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Secret** | Any sensitive data used to prove identity or authorize access, such as API keys, certificates, or passwords. |
| **Credential** | A specific type of secret used to verify the identity of a principal (user or machine). |
| **Secret Sprawl** | The uncontrolled distribution of secrets across various insecure locations like code, logs, and chat platforms. |
| **Rotation** | The process of retiring an old secret and replacing it with a new one to limit the window of opportunity for an attacker. |
| **Dynamic Secret** | A secret generated on-demand with a limited lifespan, often tied to a specific session or task. |
| **Lease/TTL** | The "Time To Live" or duration for which a secret is valid before it expires or requires renewal. |
| **Secret Zero** | The initial credential required by a machine to authenticate with a secret management system to retrieve further secrets. |
| **Principal** | An entity (human user, service, or device) that can be identified and assigned permissions. |

## Core Concepts

### The Secret Lifecycle
A secret must be managed through a defined lifecycle to maintain security integrity:
1.  **Creation:** Generation of the secret using high-entropy sources.
2.  **Storage:** Persistence in a secure, encrypted, and centralized repository.
3.  **Distribution:** Secure delivery to the authorized principal at runtime.
4.  **Rotation:** Periodic or event-driven replacement of the secret.
5.  **Revocation:** Immediate invalidation of a secret due to compromise or end-of-use.
6.  **Destruction:** Permanent removal of the secret and its metadata.

### Programmatic Access vs. Human Access
*   **Machine-to-Machine (M2M):** Secrets used by applications to talk to databases or APIs. These require automated delivery and rotation.
*   **Human-to-Machine:** Secrets used by developers or admins. These should ideally be replaced by short-lived, identity-based tokens rather than long-lived passwords.

### Least Privilege
The principle that a principal should only have access to the specific secrets required to perform its designated function, and only for the duration necessary.

## Standard Model
The standard model for Secrets Management centers on a **Centralized Secret Authority (CSA)**. 

1.  **Centralization:** All secrets are removed from application code and configuration files and stored in a hardened, dedicated system.
2.  **Identity-Based Retrieval:** Instead of using a password to get a password, the principal proves its identity (via platform-specific metadata, OIDC, or hardware-bound keys) to the CSA.
3.  **Encryption:** Secrets are encrypted at rest using a Master Key (often protected by a Hardware Security Module) and encrypted in transit via TLS.
4.  **Decoupling:** The application logic is decoupled from the secret value. The application requests a "key name," and the CSA provides the "value" at runtime.

## Common Patterns

### Just-in-Time (JIT) Provisioning
Secrets are created at the moment of request and expire immediately after use. This eliminates the risk of "stale" credentials sitting in a database.

### Sidecar/Init-Container Injection
In containerized environments, a secondary process retrieves secrets and places them in a shared memory volume or environment space for the primary application, preventing the application from needing to know how to interact with the CSA.

### Secret Wrapping
A method where a one-time-use token is used to "wrap" a secret. The recipient unwraps the token to get the secret; if the token has already been unwrapped, it signals a potential interception.

## Anti-Patterns

*   **Hardcoding:** Embedding secrets directly into source code or version control systems (VCS).
*   **Environment Variable Persistence:** Storing secrets in long-lived environment variables that can be dumped via process inspection or logs.
*   **Shared Credentials:** Using the same API key or password across multiple services or environments (e.g., Dev and Prod sharing a DB password).
*   **Lack of Rotation:** Allowing secrets to remain valid indefinitely, increasing the impact of a silent leak.
*   **In-House Crypto/Storage:** Attempting to build a custom secret storage solution using standard databases or flat files.

## Edge Cases

### The "Secret Zero" Problem
The paradox of how to securely provide the first secret to a machine so it can fetch other secrets. This is typically solved through "Trusted Execution Environments" or platform-specific identity documents (e.g., cloud instance identity documents).

### Air-Gapped Environments
Managing secrets in environments with no external connectivity requires localized CSA instances and manual synchronization of "root" secrets or hardware-based transport.

### Break-Glass Scenarios
Procedures for accessing secrets when the CSA itself is unavailable or when the primary authentication mechanism fails. This usually involves highly guarded, multi-party authorization (M-of-N control).

## Related Topics
*   **Identity and Access Management (IAM):** The framework for managing digital identities.
*   **Public Key Infrastructure (PKI):** The system for managing digital certificates.
*   **Zero Trust Architecture (ZTA):** A security model that requires strict identity verification for every person and device.
*   **Key Management Service (KMS):** Systems focused specifically on the lifecycle of cryptographic keys.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
# 104 Rotating Tokens and Credentials

Canonical documentation for 104 Rotating Tokens and Credentials. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of rotating tokens and credentials is to minimize the "blast radius" and window of opportunity associated with compromised secrets. In any distributed system, secrets (such as API keys, database passwords, and service tokens) are subject to potential exposure through logging, accidental disclosure, or unauthorized access. 

Rotation ensures that even if a credential is exfiltrated, its utility to an adversary is strictly time-bound. Furthermore, regular rotation enforces "cryptographic hygiene," ensuring that systems are capable of updating secrets without manual intervention or service disruption, thereby increasing the overall resilience of the security infrastructure.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Lifecycle management of authentication artifacts (passwords, keys, tokens).
* Strategies for seamless transition between secret versions.
* Theoretical frameworks for automated vs. manual rotation.
* Propagation and synchronization logic in distributed environments.

**Out of scope:**
* Specific vendor implementations (e.g., AWS Secrets Manager, HashiCorp Vault, Azure Key Vault).
* Cryptographic algorithm selection (e.g., RSA vs. Ed25519).
* Identity Provider (IdP) configuration specifics.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Credential** | A long-lived secret used to establish identity (e.g., a password or private key). |
| **Token** | A short-lived artifact, often derived from a credential, used to authorize specific actions (e.g., OAuth2 access token). |
| **Rotation** | The process of generating a new secret and invalidating the old one. |
| **Grace Period** | A window of time during which both the old and new secrets are considered valid to prevent service interruption. |
| **Entropy** | The randomness or unpredictability of a generated secret. |
| **TTL (Time-to-Live)** | The defined lifespan of a token or credential before it is considered expired. |
| **Secret Versioning** | The practice of maintaining multiple iterations of a secret (Current, Previous, Pending) to facilitate smooth transitions. |

## Core Concepts

### The Principle of Least Privilege (Time-Bound)
Rotation extends the Principle of Least Privilege by adding a temporal dimension. Access is granted not just to the "who" and "what," but for a specific "how long."

### Blast Radius Mitigation
If a static credential is compromised, the attacker maintains access indefinitely until the breach is discovered. With rotation, the access is automatically revoked at the end of the rotation cycle, regardless of whether the breach was detected.

### Automation and Orchestration
Manual rotation is prone to human error and often leads to "secret stale-ness." A robust rotation model relies on automated orchestration where the system generating the secret, the system storing the secret, and the system consuming the secret are synchronized.

## Standard Model

The standard model for credential rotation follows a four-stage lifecycle:

1.  **Generation:** A new secret is generated using a high-entropy random number generator.
2.  **Propagation:** The new secret is distributed to the consuming services and the target resource (e.g., the database or API).
3.  **Verification:** The system confirms that the new secret is functional and that all consumers have received it.
4.  **Invalidation:** The old secret is revoked or deleted.

### The N+1 Versioning Strategy
To ensure zero downtime, systems should support at least two versions of a secret simultaneously during the rotation window:
*   **Primary (New):** Used for all new requests.
*   **Secondary (Old):** Accepted for a limited time to allow for propagation lag.

## Common Patterns

### Scheduled Rotation
Secrets are rotated at a fixed interval (e.g., every 30, 60, or 90 days). This is the most common pattern for compliance requirements.

### Event-Driven Rotation
Rotation is triggered by a specific security event, such as a detected anomaly, an employee departure, or a suspected leak.

### Just-in-Time (JIT) Credentials
Rather than rotating a persistent secret, the system generates ephemeral credentials on-demand that expire immediately after a single use or a very short duration.

### Refresh Token Pattern
A long-lived "refresh token" is used to obtain short-lived "access tokens." When the refresh token is used, a new refresh token is issued (rotation), and the old one is invalidated.

## Anti-Patterns

*   **Hardcoding Secrets:** Embedding credentials directly in source code, making rotation impossible without a full deployment.
*   **Infinite TTL:** Assigning no expiration date to tokens or credentials.
*   **Manual Rotation via Email/Chat:** Distributing new secrets through insecure, non-auditable communication channels.
*   **Zero-Overlap Rotation:** Revoking the old secret at the exact microsecond the new one is created, which invariably leads to race conditions and service outages.
*   **Shared Credentials:** Using the same secret across multiple environments (e.g., Dev, Staging, Prod), which forces a global outage if one environment is compromised.

## Edge Cases

### Distributed System Propagation Delay
In globally distributed systems, a new secret may take several seconds or minutes to propagate to all nodes. If the old secret is invalidated too quickly, nodes that haven't received the update will fail.

### Clock Skew
If the system generating the token and the system validating the token have unsynchronized clocks, a newly rotated token may be rejected as "not yet valid" or "already expired."

### Legacy System Constraints
Some legacy systems do not support multiple concurrent passwords or keys. In these cases, rotation requires a scheduled maintenance window or a "proxy" layer that can handle the credential translation.

### High-Frequency Rotation
Rotating secrets too frequently (e.g., every few seconds) can lead to "thundering herd" problems where the overhead of rotation exceeds the utility of the service.

## Related Topics
*   **101 Identity and Access Management (IAM)**
*   **102 Secrets Management and Vaulting**
*   **105 Ephemeral Infrastructure Security**
*   **201 OAuth2 and OpenID Connect Frameworks**

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
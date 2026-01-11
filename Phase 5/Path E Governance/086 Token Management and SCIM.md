# 086 Token Management and SCIM

Canonical documentation for 086 Token Management and SCIM. This document defines concepts, terminology, and standard usage.

## Purpose
The integration of Token Management and the System for Cross-domain Identity Management (SCIM) addresses the critical need for automated identity lifecycle management and secure programmatic access. While SCIM provides a standardized protocol for provisioning and deprovisioning users and groups across heterogeneous systems, Token Management ensures that the communication between Identity Providers (IdPs) and Service Providers (SPs) remains secure, auditable, and revocable. This topic exists to bridge the gap between identity state (who a user is) and access state (what credentials they hold).

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   **Identity Lifecycle Automation:** The synchronization of user identities and their associated security metadata.
*   **Authentication of SCIM Endpoints:** The mechanisms used to secure the SCIM API itself.
*   **Token-Identity Binding:** The relationship between a provisioned identity and the tokens issued to or for that identity.
*   **Revocation Logic:** The propagation of identity deletion or suspension to the invalidation of active sessions and tokens.

**Out of scope:**
*   Specific vendor implementations (e.g., Okta, Azure AD, SailPoint).
*   Detailed cryptographic algorithms for token signing.
*   Network-level security (e.g., mTLS, IP whitelisting), except where it directly impacts token exchange.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| SCIM (System for Cross-domain Identity Management) | An HTTP-based protocol (RFC 7643 and 7644) designed to simplify user management in the cloud by defining a common schema and API. |
| Identity Provider (IdP) | The system of record that maintains user identities and authenticates them (the SCIM Client). |
| Service Provider (SP) | The application or service that receives identity data from the IdP and provides resources (the SCIM Server). |
| Bearer Token | A security token that grants access to the holder (the "bearer") without requiring further proof of identity. |
| Provisioning | The process of creating, updating, and managing user accounts and permissions in a target system. |
| Deprovisioning | The process of removing or disabling user accounts and access rights, triggered by the IdP. |
| Token Lifecycle | The stages of a token's existence: generation, distribution, validation, rotation, and revocation. |

## Core Concepts

### 1. The SCIM-Token Relationship
SCIM and Token Management operate on two different planes:
*   **The Control Plane (SCIM):** Manages the existence and attributes of the user.
*   **The Data Plane (Tokens):** Manages the active sessions and permissions used to access resources.
Effective management requires that changes in the Control Plane (e.g., a user being terminated) are immediately reflected in the Data Plane (e.g., all active tokens for that user are revoked).

### 2. Schema Extensibility
SCIM allows for custom schema extensions. In the context of token management, this is often used to store token metadata, such as public keys for user-generated tokens or references to issued API keys, directly within the SCIM User resource.

### 3. State Synchronization
The primary goal of SCIM is to ensure that the SP's internal representation of a user matches the IdP's. When a user's status is set to `active: false` via SCIM, the SP must have a mechanism to map this state change to the immediate invalidation of all tokens associated with that user.

## Standard Model

The standard model for 086 Token Management and SCIM follows a "Push" architecture:

1.  **Authorization of the SCIM Channel:** The IdP authenticates to the SP's SCIM endpoint using a long-lived Bearer Token or an OAuth 2.0 Client Credentials flow.
2.  **Identity Provisioning:** The IdP pushes user objects to the SP. The SP creates a local user record and maps a unique identifier (usually the `externalId` or `id`).
3.  **Token Issuance:** Once provisioned, the user (or a process acting on their behalf) requests tokens from the SP's authorization server.
4.  **Lifecycle Hook:** The SP monitors SCIM `PATCH` or `DELETE` requests. Upon receiving a deactivation command, the SP triggers a "Revocation Event" in its Token Management system.

## Common Patterns

### OAuth 2.0 Client Credentials for SCIM
The IdP acts as a confidential client. It requests an access token from the SP’s token endpoint using its own credentials, then uses that token to authorize SCIM requests. This ensures that the SCIM channel itself is subject to token expiration and rotation.

### Personal Access Token (PAT) Management via SCIM
In developer-centric platforms, SCIM is used to manage the lifecycle of PATs. When a user is deprovisioned via SCIM, the SP iterates through all PATs owned by that user ID and marks them as revoked in the database.

### Just-In-Time (JIT) Provisioning with SCIM Reconciliation
Users are created JIT upon their first login (via OIDC/SAML), but their ongoing lifecycle (updates/deletions) is managed via SCIM. This ensures that tokens issued during the JIT process are eventually governed by the SCIM deprovisioning logic.

## Anti-Patterns

*   **Orphaned Tokens:** Deleting a user via SCIM without revoking their active refresh tokens or API keys. This leaves a "backdoor" into the system.
*   **Hardcoded SCIM Tokens:** Using a single, non-expiring "Master Token" for SCIM operations that cannot be rotated without system downtime.
*   **Synchronous Token Validation on SCIM Requests:** Attempting to validate every token in the system during a bulk SCIM import, which can lead to catastrophic performance degradation (N+1 query problem).
*   **Ignoring `active: false`:** Treating a SCIM "disable" request as a mere attribute change rather than a security event requiring session termination.

## Edge Cases

*   **Soft-Delete vs. Hard-Delete:** SCIM supports both setting `active: false` (soft-delete) and the `DELETE` verb (hard-delete). Token management systems must decide if a soft-deleted user can have their tokens "paused" or if they must be permanently revoked.
*   **Attribute Mapping Collisions:** If a SCIM update changes a user's unique identifier (e.g., `userName` or `email`), the link between the user and their existing tokens may break, leading to "ghost tokens" that cannot be easily traced to the new identity.
*   **Bulk Operations:** SCIM `/Bulk` endpoints can update thousands of users at once. The underlying token management system must be able to handle mass revocation events without impacting API availability.
*   **Downstream Propagation:** In complex environments, an SP might also act as an IdP for other downstream services. A SCIM delete request must propagate through the entire chain to ensure total token invalidation.

## Related Topics
*   **042 OAuth 2.0 and OIDC Frameworks:** The underlying protocols for token issuance.
*   **115 Identity Lifecycle Management:** The broader strategy of managing identities from onboarding to offboarding.
*   **021 API Security Standards:** General best practices for securing programmatic interfaces.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
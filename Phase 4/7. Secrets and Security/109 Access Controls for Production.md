# 109 Access Controls for Production

Canonical documentation for 109 Access Controls for Production. This document defines concepts, terminology, and standard usage.

## Purpose
The 109 Access Controls for Production framework exists to mitigate the risk of unauthorized access, accidental configuration changes, and data breaches within live environments. It addresses the fundamental tension between operational velocity and system stability by establishing a rigorous boundary between development activities and production assets. This documentation ensures that access is governed by policy rather than convenience, providing a verifiable audit trail for all interactions with critical infrastructure.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Identity Governance:** Management of human and machine identities interacting with production.
* **Authorization Logic:** The mechanisms for granting, verifying, and revoking permissions.
* **Auditability:** Requirements for logging and monitoring access events.
* **Environment Isolation:** The logical and physical separation of production from non-production environments.

**Out of scope:**
* **Specific vendor implementations:** (e.g., specific configurations for AWS IAM, Azure AD, or Okta).
* **Physical Security:** Physical data center access (covered under 200-series controls).
* **Network Layer Security:** Firewalls and VPC peering (covered under 300-series controls), except where they function as an identity-aware proxy.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Principal** | Any entity (human user, service account, or automated process) that requests access to a production resource. |
| **Entitlement** | A specific privilege or set of permissions granted to a principal (e.g., Read, Write, Execute). |
| **Least Privilege** | The principle that a principal must be granted only the minimum entitlements necessary to perform a specific task for the shortest duration required. |
| **Just-in-Time (JIT)** | A method of granting elevated access only when needed and for a limited time, rather than maintaining permanent "standing" privileges. |
| **Break-glass** | An emergency access procedure used to bypass standard controls during a critical system failure or "out-of-band" incident. |
| **Attestation** | The periodic review and formal confirmation by a resource owner that a principal’s access remains necessary and appropriate. |
| **Separation of Duties (SoD)** | A control requiring that more than one person is needed to complete a critical task to prevent fraud and error. |

## Core Concepts

### 1. Identity-Centric Security
Access is tied to a verified identity rather than a network location. In the 109 framework, the "who" is more critical than the "where." Every action in production must be attributable to a unique principal.

### 2. Ephemeral Access
The 109 standard favors temporary access over permanent assignments. By defaulting to "no access," the attack surface is minimized. Access is granted for a specific window and automatically expires, reducing the risk of "privilege creep."

### 3. Policy-as-Code
Access controls should be defined in version-controlled files rather than manual console configurations. This ensures that changes to access levels are peer-reviewed, tested, and auditable in the same manner as application code.

### 4. Immutable Audit Trails
Every access request, grant, and action taken within the production environment must generate a non-repudiable log. These logs must be stored in a location isolated from the production environment itself to prevent tampering by a compromised administrative account.

## Standard Model

The 109 Standard Model follows a **Request-Validate-Execute-Revoke** lifecycle:

1.  **Request:** A principal requests access to a specific resource for a defined purpose.
2.  **Validate:** An automated system or authorized peer validates the request against existing policies (e.g., Is the user on call? Is there an open incident ticket?).
3.  **Execute:** The system grants the minimum necessary entitlements. The principal performs the task.
4.  **Revoke:** Upon task completion or time-to-live (TTL) expiration, the entitlements are automatically stripped.

## Common Patterns

### Role-Based Access Control (RBAC)
Grouping permissions into roles (e.g., "SRE-On-Call," "DB-ReadOnly") and assigning principals to those roles. This simplifies management but can lead to "role explosion" if not governed strictly.

### Attribute-Based Access Control (ABAC)
Granting access based on attributes of the principal, the resource, and the environment (e.g., "Allow access to Database X if Principal is in the 'Engineering' group AND the time is between 09:00-17:00").

### Proxy-Based Access
Principals do not connect directly to production resources. Instead, they connect through an intermediary (a "bastion" or "identity-aware proxy") that authenticates the user and tunnels the session, providing a centralized point for logging and session recording.

## Anti-Patterns

*   **Shared Accounts:** Using generic "admin" or "root" credentials that prevent individual accountability.
*   **Standing Privileges:** Allowing engineers to maintain high-level access (e.g., "Domain Admin") 24/7, regardless of whether they are performing active work.
*   **Manual Provisioning:** Relying on human administrators to manually add users to groups via a GUI, which is prone to error and lacks a clear audit trail.
*   **Production Access for Developers:** Granting developers direct write access to production to "fix things quickly" without going through a deployment pipeline or JIT process.

## Edge Cases

### Break-glass Scenarios
In a total system collapse where the identity provider or the JIT system is offline, a "break-glass" mechanism must exist. This usually involves a highly secured, physical or multi-party digital credential that triggers an immediate, high-priority alert to the entire security organization when used.

### Third-Party/Vendor Access
When external vendors require access for support, they must be treated as temporary principals. Their access must be scoped to the specific sub-system they support and must be monitored via live session recording.

### Automated Service Accounts
Non-human identities (CI/CD bots, monitoring agents) often require persistent access. These must be governed by secret rotation policies and restricted to specific IP ranges or cryptographic identities to prevent token theft from leading to environment-wide compromise.

## Related Topics
*   **102 Identity Management:** The foundational system for managing user lifecycles.
*   **110 Secrets Management:** The storage and distribution of API keys, passwords, and certificates.
*   **405 Incident Response:** The procedures triggered when unauthorized access is detected.
*   **602 Compliance and Auditing:** The framework for reviewing 109 logs for regulatory requirements.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
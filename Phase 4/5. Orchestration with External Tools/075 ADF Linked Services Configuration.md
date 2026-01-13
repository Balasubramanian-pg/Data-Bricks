# 075 ADF Linked Services Configuration

Canonical documentation for 075 ADF Linked Services Configuration. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of the 075 ADF Linked Services Configuration is to provide a secure, decoupled, and reusable mechanism for defining connection information between a data orchestration engine and external data sources or compute resources. 

By abstracting connection strings and authentication logic away from the data transformation activities, this configuration ensures that data pipelines remain portable across different environments (Development, Testing, Production) without requiring modifications to the underlying logic. It addresses the problem of hardcoded credentials, lack of centralized connection management, and the complexity of managing diverse protocol requirements in a heterogeneous data landscape.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative. While "ADF" often refers to Azure Data Factory, the principles outlined here apply to any Automated Data Framework or Application Data Fabric utilizing a linked-service architecture.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Connection Abstraction:** The theoretical framework for separating connection metadata from execution logic.
* **Authentication Frameworks:** Standard methods for securing access to external endpoints.
* **Environment Parameterization:** The logic of dynamic connection resolution.
* **Security Integration:** Standards for externalizing secrets and sensitive metadata.

**Out of scope:**
* **Specific Vendor Implementations:** Detailed "how-to" guides for specific databases (e.g., Snowflake, SAP, Oracle).
* **Network Infrastructure:** Physical routing, firewall rules, or DNS configuration (though the *reference* to these is in scope).
* **Data Transformation Logic:** The actual processing of data once a connection is established.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Linked Service** | A configuration object that defines the connection information (endpoint, port, protocol) and authentication credentials required to access an external resource. |
| **Integration Runtime (IR)** | The compute infrastructure used by the linked service to provide the bridge between the data orchestration engine and the target resource. |
| **Secret Management** | The practice of storing sensitive information (passwords, keys, tokens) in a dedicated, encrypted vault rather than within the linked service definition. |
| **Parameterization** | The use of variables within a linked service to allow connection details to be injected at runtime. |
| **Managed Identity** | A system-assigned or user-assigned identity that allows the orchestration engine to authenticate to resources without storing explicit credentials. |
| **Endpoint** | The specific network address or URL where the target service is hosted. |

## Core Concepts
The 075 ADF Linked Services Configuration is built upon three fundamental pillars:

1.  **Decoupling of Concerns:** The data pipeline (the "What") is entirely separate from the connection (the "Where" and "How"). This allows a single pipeline to run against different data sources simply by swapping the linked service.
2.  **Security by Reference:** Credentials should never be stored as plaintext within the configuration. Instead, the configuration should store a *reference* to a secret management system.
3.  **Compute Context:** Every linked service must be associated with an execution environment (Integration Runtime) that has the necessary network line-of-sight to the target resource.

## Standard Model
The standard model for a 075-compliant Linked Service configuration follows a hierarchical structure:

*   **Identity Layer:** Defines *who* is connecting (Managed Identity, Service Principal, or User Credential).
*   **Transport Layer:** Defines *how* the connection is made (Protocol, Port, Encryption requirements).
*   **Resource Layer:** Defines *where* the connection is going (Server Name, Database Name, Instance ID).
*   **Execution Layer:** Defines the compute resource (Integration Runtime) responsible for initiating the handshake.

In a mature implementation, the **Standard Model** mandates that all environment-specific values (e.g., `ServerURL`) are passed as parameters, while all sensitive values (e.g., `Password`) are retrieved from a secure vault.

## Common Patterns
*   **The Vault-Proxy Pattern:** The linked service contains no secrets; it contains a reference to a Secret Vault and the name of the secret to retrieve.
*   **Dynamic Endpoint Pattern:** Using parameters to construct a connection string at runtime, allowing a single linked service to connect to multiple databases on the same server.
*   **Identity Delegation:** Utilizing the system's own identity (Managed Identity) to access resources, eliminating the need for password rotation and management.
*   **Gateway-Based Connectivity:** Using a self-hosted or private integration runtime to bridge the gap between cloud orchestration and on-premises/private network resources.

## Anti-Patterns
*   **Hardcoded Credentials:** Storing plaintext passwords or API keys directly within the linked service JSON/configuration.
*   **Monolithic Linked Services:** Creating a single linked service with excessive permissions (e.g., "SuperUser") rather than following the Principle of Least Privilege.
*   **Environment Hardcoding:** Creating separate, static linked services for "Dev_DB" and "Prod_DB" instead of using a single parameterized linked service.
*   **Bypassing Encryption:** Disabling SSL/TLS verification to resolve certificate trust issues rather than properly configuring the trust store.

## Edge Cases
*   **Cross-Tenant Authentication:** When the orchestration engine and the target resource reside in different organizational tenants, requiring complex service principal negotiations.
*   **Transient Resources:** Connecting to resources that are spun up and down on demand (e.g., ephemeral clusters), where the endpoint address is not known until runtime.
*   **Legacy Protocol Support:** Handling resources that require deprecated or non-standard authentication methods (e.g., NTLM in a modern cloud environment).
*   **Multi-Factor Authentication (MFA):** Linked services generally cannot handle interactive MFA; service-to-service authentication must be used instead.

## Related Topics
*   **076 Data Dataset Configuration:** Defines the structure of the data within the resource identified by the Linked Service.
*   **082 Secret Management Standards:** Defines how credentials should be stored and rotated.
*   **104 Integration Runtime Architecture:** Defines the compute environments used by Linked Services.
*   **012 Network Security and Firewalls:** Defines the connectivity requirements for successful handshakes.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
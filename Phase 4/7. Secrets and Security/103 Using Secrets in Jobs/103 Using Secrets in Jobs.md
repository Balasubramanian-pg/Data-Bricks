# 103 Using Secrets in Jobs

Canonical documentation for 103 Using Secrets in Jobs. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of this topic is to define the secure management and consumption of sensitive information—known as "secrets"—within automated execution units (jobs). In modern automation and orchestration, jobs frequently require access to protected resources (databases, APIs, remote servers). This topic addresses the challenge of providing necessary access to these resources without compromising the security posture of the system, exposing credentials in source code, or leaking sensitive data through execution logs.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
**In scope:**
* The lifecycle of a secret during job execution (injection, consumption, and cleanup).
* Security principles governing secret access within automated environments.
* Methods of abstracting sensitive data from logic and configuration.
* Mechanisms for preventing accidental exposure (masking and redaction).

**Out of scope:**
* Specific vendor implementations (e.g., GitHub Actions Secrets, Kubernetes Secrets, HashiCorp Vault).
* The physical encryption algorithms used to store secrets at rest.
* General user identity and access management (IAM) outside the context of job execution.

## Definitions
| Term | Definition |
|------|------------|
| Secret | Any sensitive data point that requires restricted access, such as API keys, passwords, certificates, or SSH keys. |
| Job | A discrete, automated unit of work or process executed within a controlled environment. |
| Injection | The process of making a secret available to a job's runtime environment. |
| Masking | The automated process of identifying and redacting secret values from logs or standard output. |
| Least Privilege | The principle of providing a job with only the specific secrets required to perform its designated task, and no more. |
| Secret Provider | An external system or internal service responsible for storing, managing, and dispensing secrets. |
| Scope | The defined boundary (e.g., project, environment, or specific job) within which a secret is accessible. |

## Core Concepts
### Separation of Concerns
Secrets must be strictly decoupled from the application logic (code) and the execution instructions (configuration). This ensures that code can be shared or audited without exposing credentials and that secrets can be rotated without modifying the codebase.

### Ephemeral Availability
Secrets should ideally exist within the job's execution environment only for the duration of the job. Once the job completes, the environment should be destroyed or cleansed to ensure no residual sensitive data remains.

### Trust Boundaries
A job operates within a trust boundary. The mechanism that triggers the job and the environment that executes it must be verified. Secrets are only injected after the execution environment has been authenticated and authorized by the Secret Provider.

## Standard Model
The standard model for using secrets in jobs follows a four-phase lifecycle:

1.  **Declaration:** The job configuration specifies which secrets are required by referencing a unique identifier (key) rather than the value itself.
2.  **Authentication & Authorization:** Before execution, the job runner authenticates with the Secret Provider. The provider verifies that the specific job has the authority to access the requested keys.
3.  **Injection:** The Secret Provider retrieves the values and injects them into the job's runtime environment. This is typically done via environment variables or temporary files.
4.  **Consumption:** The application or script within the job reads the secret from the environment to perform its task.
5.  **Redaction:** Throughout the process, the job runner monitors the output (STDOUT/STDERR) and replaces any detected secret values with a placeholder (e.g., `***`) to prevent leakage into logs.

## Common Patterns
### Environment Variable Injection
The most common pattern where secrets are mapped to environment variables. This is widely supported across almost all operating systems and programming languages.

### Filesystem Mounting
Secrets are provided as temporary files in a specific directory (e.g., a RAM disk or a virtual volume). This is preferred for multi-line secrets like private keys or SSL certificates that may be difficult to parse as environment variables.

### Dynamic Secret Generation
Instead of retrieving a static, long-lived password, the job requests a "just-in-time" credential from the Secret Provider. This credential is created specifically for that job and expires automatically after a short duration.

## Anti-Patterns
*   **Hardcoding:** Including secret values directly in scripts or source code.
*   **Logging Secrets:** Explicitly printing secret values to the console for debugging purposes.
*   **Broad Scoping:** Granting a job access to an entire "vault" or "folder" of secrets when it only requires one or two.
*   **Storing Secrets in Version Control:** Committing encrypted or unencrypted secrets to a repository.
*   **Persistent Environment Storage:** Saving secrets in persistent configuration files or machine-level environment variables that survive beyond the job's lifecycle.

## Edge Cases
### Secret Rotation During Execution
In long-running jobs, a secret may be rotated (changed) by the provider while the job is still active. If the job does not have a mechanism to refresh the secret, it may fail mid-execution.

### Multi-Value Secrets
Some secrets are structured (e.g., JSON blobs). If the job runner only supports flat string masking, it may fail to redact individual components of the structured secret if they are parsed and printed separately.

### Nested Jobs
When a job triggers a sub-job or a child process, secrets may not be automatically inherited. Explicitly passing secrets to child processes can lead to accidental exposure if not handled via secure inheritance mechanisms.

## Related Topics
*   **101 Principles of Least Privilege:** The foundational security theory for secret access.
*   **105 Log Management and Redaction:** Deep dive into how output is scrubbed for sensitive data.
*   **202 Identity-Based Security in CI/CD:** How runners prove their identity to fetch secrets.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
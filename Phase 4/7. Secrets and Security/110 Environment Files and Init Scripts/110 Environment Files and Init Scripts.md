# 110 Environment Files and Init Scripts

Canonical documentation for 110 Environment Files and Init Scripts. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of environment files and initialization (init) scripts is to provide a standardized mechanism for configuring the execution context of processes, users, and systems. These components decouple configuration data and setup logic from the underlying application code or binary. 

Environment files serve as static or semi-static repositories of key-value pairs that define the state of a process's environment. Init scripts provide the procedural logic required to prepare a system or session for operation, ensuring that dependencies are met, paths are set, and the environment is consistent across different execution instances.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* The lifecycle and inheritance of environment variables.
* The distinction between static configuration files and executable initialization logic.
* Standardized sequencing of environment loading.
* Security considerations regarding the storage and injection of environment data.

**Out of scope:**
* Specific syntax for individual shells (e.g., Bash vs. Zsh vs. PowerShell).
* Vendor-specific service managers (e.g., Systemd, OpenRC, Launchd).
* Specific cloud provider secret management implementations.

## Definitions
| Term | Definition |
|------|------------|
| **Environment Variable** | A dynamic-named value that can affect the way running processes will behave on a computer. |
| **Environment File** | A text file containing key-value pairs used to populate the environment variables of a process. |
| **Init Script** | A script executed automatically during the startup of a system, service, or user session to perform setup tasks. |
| **Sourcing** | The act of executing a script within the current shell context rather than a subshell, allowing it to modify the current environment. |
| **Exporting** | The process of marking a variable to be inherited by child processes. |
| **Idempotency** | The property of a script where multiple executions result in the same state without unintended side effects. |
| **Scope** | The boundary within which an environment variable or configuration is visible and active. |

## Core Concepts

### 1. Environment Inheritance
Environment variables follow a hierarchical inheritance model. A child process inherits a copy of the environment of its parent process at the time of creation. Modifications made to the environment in the child process do not propagate back to the parent.

### 2. Static vs. Dynamic Configuration
*   **Environment Files (.env, etc.):** These are declarative. They represent the "what"—the data required for the process to run.
*   **Init Scripts (.sh, .profile, etc.):** These are imperative. They represent the "how"—the logic required to reach a desired state (e.g., checking if a directory exists before setting it as a path).

### 3. Execution Contexts
Environment and init scripts operate in different contexts:
*   **System-wide:** Affects all users and processes (e.g., boot-time scripts).
*   **User-specific:** Affects a specific user's session (e.g., login scripts).
*   **Process-specific:** Affects only a specific application or container instance.

## Standard Model
The standard model for environment initialization follows a specific order of operations to ensure stability and predictability:

1.  **System Defaults:** The kernel or init system loads global configurations.
2.  **System Initialization:** Global init scripts execute to set up hardware, networking, and global paths.
3.  **User Authentication:** The user session begins; the system identifies the user and their specific configuration needs.
4.  **User Environment Loading:** User-specific environment files and scripts are sourced.
5.  **Application Injection:** The specific application or shell is launched, often loading a local environment file (e.g., `.env`) to override broader settings.

## Common Patterns

### The Wrapper Pattern
A common pattern where a primary application is launched via a "wrapper" script. This script sources necessary environment files and performs pre-flight checks before executing the application binary.

### The `.env` Pattern
Used extensively in development and containerized environments, this pattern involves placing a file named `.env` in the root of a project. Tools or libraries read this file to populate the process environment at runtime.

### Conditional Sourcing
Init scripts often use conditional logic to source other files only if they exist. This allows for modular configuration where a base script remains unchanged while allowing for local overrides.

## Anti-Patterns

### Hardcoding Secrets in Init Scripts
Storing sensitive information (API keys, passwords) directly within version-controlled init scripts. Secrets should be injected from secure environment files or external secret managers.

### Side Effects in Environment Files
Including executable logic or commands that modify the filesystem within a file intended only for key-value pairs. This makes the environment unpredictable and difficult to audit.

### Circular Sourcing
Creating a loop where `Script A` sources `Script B`, which in turn sources `Script A`. This leads to stack overflow or infinite loops during session initialization.

### Over-reliance on Global Scope
Defining application-specific variables in system-wide init scripts. This creates "environment pollution" and can lead to conflicts between different applications.

## Edge Cases

### Non-Interactive Shells
Scripts or cron jobs often run in non-interactive shells that do not source standard user profile scripts. This frequently leads to "command not found" errors because the `PATH` variable was not initialized.

### Race Conditions in Boot Scripts
In parallel init systems, two scripts may attempt to modify or rely on the same resource simultaneously. Without proper dependency mapping, the environment may be initialized inconsistently.

### Character Encoding and Special Characters
Environment files containing spaces, quotes, or non-ASCII characters can be interpreted differently by various parsers, leading to truncated values or syntax errors.

## Related Topics
*   **Process Management:** How init systems manage the lifecycle of services.
*   **Secret Management:** Secure handling of sensitive environment data.
*   **Container Orchestration:** How environments are injected into isolated namespaces.
*   **Shell Execution Modes:** Differences between login, non-login, interactive, and non-interactive shells.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
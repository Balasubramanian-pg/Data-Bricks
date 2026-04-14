# 094 Versioned Deployments

Canonical documentation for 094 Versioned Deployments. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of Versioned Deployments is to provide a deterministic, reproducible, and reversible method for delivering software updates. In traditional "in-place" deployment models, software is updated by overwriting existing files, which often leads to configuration drift, unpredictable downtime, and difficult recovery paths. 

Versioned Deployments address these issues by treating each deployment as a unique, immutable instance of the system. This allows organizations to maintain multiple versions of a service simultaneously, enabling sophisticated traffic routing, instant rollbacks, and precise auditing of what code is running in any given environment at any time.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Lifecycle management of versioned software artifacts.
* Strategies for traffic transition between deployment versions.
* Principles of immutability in the deployment pipeline.
* Versioning schemes and their impact on deployment logic.

**Out of scope:**
* Specific cloud provider services (e.g., AWS Lambda Aliases, Kubernetes Deployments).
* Source code management (Git branching strategies), except where they directly influence the deployment identifier.
* Specific CI/CD tool syntax.

## Definitions
| Term | Definition |
|------|------------|
| **Artifact** | A compiled, packaged, and immutable unit of software (e.g., container image, binary, or ZIP archive) ready for deployment. |
| **Immutable Deployment** | A deployment pattern where components are replaced rather than modified. Once a version is deployed, it is never changed; updates require a new versioned deployment. |
| **Traffic Shifting** | The process of incrementally or instantly redirecting requests from an older version of a deployment to a newer one. |
| **Rollback** | The act of redirecting traffic back to a previously known-good versioned deployment following a failure in a newer version. |
| **Active Version** | The specific deployment version currently receiving the majority or entirety of production traffic. |
| **Semantic Versioning (SemVer)** | A versioning scheme (Major.Minor.Patch) used to communicate the nature of changes and compatibility between deployments. |

## Core Concepts

### Immutability
The cornerstone of versioned deployments is immutability. Once a deployment artifact is created and assigned a version, it must never be altered. Any change—whether a code fix or a configuration update—must result in a new version with a unique identifier. This eliminates "snowflake servers" and ensures that the environment is reproducible.

### Version Identification
Every deployment must be uniquely identifiable. Common identifiers include:
* **Semantic Versions:** (e.g., v1.2.4) indicating the scope of change.
* **Build Numbers:** (e.g., build-592) indicating sequential progression.
* **Git Hashes:** (e.g., commit-a1b2c3d) providing direct traceability to source code.

### Decoupling Deployment from Release
Versioned deployments separate the technical act of **deploying** (moving bits to a server) from the business act of **releasing** (making those bits available to users). A version can be deployed and verified in a production environment before a single user request is routed to it.

## Standard Model
The standard model for Versioned Deployments follows a linear progression through a registry or repository:

1.  **Artifact Creation:** A build process generates a versioned artifact.
2.  **Registration:** The artifact is stored in a central registry with its unique version tag.
3.  **Instantiation:** The specific version is instantiated in a target environment alongside (or in place of) existing versions.
4.  **Validation:** The new version undergoes health checks and smoke tests while receiving no (or minimal) traffic.
5.  **Promotion:** Traffic is routed to the new version based on a defined pattern (e.g., Blue/Green).
6.  **Decommissioning:** Older versions are retained for a "grace period" before being terminated to save resources.

## Common Patterns

### Blue/Green Deployment
Two identical environments exist. "Blue" is the current production version; "Green" is the new version. Once Green is verified, traffic is switched entirely from Blue to Green.

### Canary Releases
A small percentage of traffic is routed to the new version (the "canary"). If the canary performs well, the percentage is increased until the new version handles 100% of the traffic.

### Linear/Rolling Updates
New versions are introduced incrementally, replacing old versions one instance at a time. In a versioned context, this ensures that at any point, the system is composed of known, tagged versions rather than indeterminate states.

### Shadow Deployment
The new version receives a copy of production traffic (read-only) to test performance and correctness without affecting the end-user experience.

## Anti-Patterns

### The "Latest" Tag
Using a generic `latest` or `stable` tag for production deployments. This obscures which specific code is running and makes rollbacks nearly impossible without manual intervention.

### In-Place Patching
Logging into a running production instance to change a configuration file or hot-fix code. This breaks the chain of custody and ensures the next deployment will likely fail or regress.

### Manual Versioning
Relying on humans to manually increment version numbers during the deployment phase, which leads to collisions and "version gaps."

### Configuration Coupling
Hard-coding environment-specific configurations into the versioned artifact. Artifacts should be built once and configured at runtime via environment variables or secret stores.

## Edge Cases

### Database Schema Migrations
Versioned deployments are often stateless, but databases are stateful. A new version may require a schema change that is incompatible with the previous version. This requires "Expand and Contract" patterns (Parallel Changes) to ensure both the old and new deployment versions can operate simultaneously during the transition.

### Long-Lived Connections
Applications using WebSockets or long-running gRPC streams may have "sticky" sessions. Versioned deployments must account for "draining" periods where old versions remain active until all legacy connections are naturally closed.

### Circular Dependencies
When Version A of Service 1 requires Version B of Service 2, but Service 2 is not yet deployed. Versioned deployments require strict adherence to backward compatibility to prevent "lock-step" deployment requirements.

## Related Topics
* **042 Immutable Infrastructure:** The underlying philosophy of never modifying deployed assets.
* **115 Traffic Management:** The mechanics of routing requests between versions.
* **088 Observability and Telemetry:** Necessary for determining the health of a new version during promotion.
* **021 Continuous Integration:** The process that generates the versioned artifacts.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
# 095 Blue Green Deployment for Pipelines

Canonical documentation for 095 Blue Green Deployment for Pipelines. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of Blue Green Deployment within automated pipelines is to provide a release methodology that ensures high availability, minimizes service disruption, and enables near-instantaneous recovery from failed deployments. By maintaining two identical production-ready environments, organizations can decouple the act of deploying software from the act of releasing it to users. This strategy addresses the risks associated with "in-place" updates, such as configuration drift, service downtime during restarts, and complex rollback procedures.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Orchestration logic for environment switching.
* Traffic routing principles and state management.
* Validation requirements for environment promotion.
* Rollback mechanisms and lifecycle management of the "Idle" environment.

**Out of scope:**
* Specific cloud provider load balancer configurations.
* Container orchestration syntax (e.g., Kubernetes YAML).
* Specific CI/CD tool syntax (e.g., Jenkinsfiles, GitHub Actions).

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Blue Environment** | One of two identical production environments; traditionally represents the "Live" or "Active" environment currently serving traffic. |
| **Green Environment** | The secondary environment used for staging the new version of the application before traffic is cut over. |
| **Traffic Router** | The mechanism (Load Balancer, DNS, or Service Mesh) that directs incoming requests to either the Blue or Green environment. |
| **Cutover** | The atomic or gradual process of reconfiguring the Traffic Router to point from the old version to the new version. |
| **Rollback** | The process of redirecting traffic back to the previous stable environment in the event of a failure in the new release. |
| **Environment Parity** | The requirement that Blue and Green environments are identical in configuration, hardware/resource allocation, and network topology. |
| **Idle Environment** | The environment not currently serving production traffic, used for deployment and smoke testing. |

## Core Concepts
### 1. Environment Duplication
The strategy relies on the existence of two distinct but identical environments. At any given time, one is "Live" (serving production traffic) and the other is "Idle" (receiving the new deployment).

### 2. Immutable Infrastructure
Blue Green deployments are most effective when environments are treated as immutable. Rather than patching existing servers, the pipeline builds a fresh Green environment (or updates an existing Idle one) to match the desired state.

### 3. Traffic Redirection
The core of the pattern is the separation of **Deployment** (moving code to an environment) and **Release** (moving users to that code). The release is controlled by the Traffic Router.

### 4. State Persistence
Because the application logic is duplicated but the data (database) is usually shared or synchronized, managing state is the most critical technical challenge in Blue Green deployments.

## Standard Model
The standard lifecycle of a Blue Green Pipeline follows these sequential phases:

1.  **Preparation:** The pipeline identifies the current "Idle" environment (e.g., if Blue is Live, Green is Idle).
2.  **Deployment:** The new version of the application is deployed to the Idle environment.
3.  **Warm-up & Validation:** Automated smoke tests and health checks are performed against the Idle environment to ensure readiness.
4.  **Cutover:** The Traffic Router is updated. Production traffic is redirected from the Live environment to the newly updated environment.
5.  **Monitoring:** The new Live environment is monitored for a "soak period."
6.  **Finalization/Decommission:** If the release is successful, the old environment becomes the new Idle environment. If it fails, a Rollback is triggered by reverting the Traffic Router.

## Common Patterns
### All-at-Once Cutover
The Traffic Router redirects 100% of traffic from Blue to Green instantaneously. This is common with DNS weighted records or Load Balancer target group swaps.

### Incremental Shifting (Canary-style Blue Green)
Traffic is shifted in percentages (e.g., 10%, 25%, 100%). While similar to Canary deployments, it differs because the destination is a fully-formed parallel environment rather than a subset of nodes within the same cluster.

### Shared Database Pattern
Both Blue and Green environments point to the same production database. This requires the application to maintain backward and forward compatibility with the database schema.

## Anti-Patterns
*   **Configuration Drift:** Allowing the Blue and Green environments to diverge in OS patches, instance sizes, or environment variables.
*   **Manual Cutover:** Relying on human intervention to update routing tables, which introduces delay and potential for error during rollbacks.
*   **Shared Session State:** Storing user sessions locally on the Blue nodes, causing users to lose their sessions when the cutover to Green occurs.
*   **Breaking Database Changes:** Deploying a Green version that modifies the database schema in a way that the Blue version (still live) can no longer read or write.

## Edge Cases
*   **Long-Lived Connections:** Applications using WebSockets or long-polling may require "connection draining," where the Blue environment continues to serve existing connections until they terminate, while Green receives all new connections.
*   **Database Migrations:** If a deployment requires a destructive schema change, a standard Blue Green deployment cannot be performed without a maintenance window or a complex multi-step migration strategy (Expand/Contract pattern).
*   **Orphaned Resources:** Failure to properly clean up or "reset" the Idle environment after a successful cutover can lead to "ghost" deployments or resource exhaustion.

## Related Topics
*   **082 Canary Deployments:** A related strategy focusing on incremental exposure.
*   **104 Circuit Breaker Pattern:** Often used in conjunction with traffic routing to handle failures.
*   **112 Database Schema Evolution:** Strategies for managing data during environment transitions.
*   **Immutable Infrastructure:** The underlying philosophy supporting environment parity.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
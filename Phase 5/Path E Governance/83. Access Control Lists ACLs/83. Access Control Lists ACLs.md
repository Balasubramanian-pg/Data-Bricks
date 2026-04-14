# 083 Access Control Lists ACLs

Canonical documentation for 083 Access Control Lists ACLs. This document defines concepts, terminology, and standard usage.

## Purpose
Access Control Lists (ACLs) exist to provide a granular mechanism for mediating access between subjects and objects within a system. They address the problem of resource protection by defining explicit permissions that govern which entities are permitted or denied specific actions. ACLs serve as a fundamental component of security policy enforcement, ensuring that the principle of least privilege can be applied to data, network traffic, and system operations.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   The logical structure and evaluation mechanics of ACLs.
*   Theoretical frameworks for filesystem and network-layer ACLs.
*   Inheritance and propagation logic.
*   The relationship between subjects, objects, and permissions.

**Out of scope:**
*   Specific vendor syntax (e.g., Cisco IOS, Juniper Junos, AWS Security Groups).
*   Hardware-specific optimization (e.g., TCAM utilization).
*   Identity Provider (IdP) configuration.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Access Control Entry (ACE)** | An individual element or "row" within an ACL that defines a specific rule (e.g., Permit User A to Read File B). |
| **Subject** | The entity requesting access, such as a user, group, process, or IP address. |
| **Object** | The resource being accessed, such as a file, directory, network port, or service. |
| **Action** | The operation being performed (e.g., Read, Write, Execute, Delete, Permit, Deny). |
| **Implicit Deny** | A security principle where access is automatically refused if no explicit permit rule is found in the ACL. |
| **Mask** | A bitmask or wildcard used to define a range of subjects or objects within a single ACE. |
| **Propagation** | The process by which ACL settings are applied from a parent container to its child objects. |

## Core Concepts

### The Access Control Matrix
The ACL is a functional decomposition of the Access Control Matrix. While a matrix maps every subject against every object, an ACL attaches the list of authorized subjects and their permissions directly to the object itself.

### Evaluation Logic
ACLs are typically evaluated as ordered lists. When a subject attempts an action on an object, the system traverses the ACEs in a specific sequence (usually top-to-bottom) until a match is found. Once a match is identified, the corresponding action (Permit or Deny) is taken, and further evaluation usually ceases.

### Granularity
ACLs provide finer control than simple owner/group/others models (like standard POSIX permissions). They allow for the specification of multiple individual users and multiple groups with varying levels of access on a single resource.

## Standard Model

### The Sequential Evaluation Model
The standard model for ACL processing follows these steps:
1.  **Request Receipt:** A subject initiates an operation on an object.
2.  **Sequential Comparison:** The system compares the request attributes against the first ACE.
3.  **Match Determination:** 
    *   If the attributes match the ACE, the specified action is enforced.
    *   If the attributes do not match, the system moves to the next ACE.
4.  **Final Disposition:** If the end of the list is reached without a match, the **Implicit Deny** rule is applied, and access is rejected.

### Permission Types
Standard ACL models recognize three primary permission categories:
*   **Read:** Ability to view data or list directory contents.
*   **Write:** Ability to modify data, create new files, or delete existing ones.
*   **Execute:** Ability to run a file as a program or traverse a directory.

## Common Patterns

### Whitelisting (Default Deny)
The most secure pattern where the ACL contains only "Permit" entries for known-good subjects, relying on the Implicit Deny at the end of the list to block all other traffic or access.

### Blacklisting (Default Permit)
A pattern where the ACL contains "Deny" entries for known-bad subjects, with a final "Permit All" entry. This is generally discouraged in high-security environments.

### Role-Based ACLs
Instead of assigning permissions to individual users, ACEs reference group identifiers or roles. This simplifies administration by decoupling identity management from resource protection.

### Reflexive (Stateful) ACLs
In network contexts, these ACLs dynamically create temporary entries to allow returning traffic for an outbound session that was initiated from within a protected network.

## Anti-Patterns

### Shadowing
Occurs when a broad ACE is placed above a more specific ACE, rendering the specific rule unreachable. For example, permitting "All Users" at the top of a list makes a "Deny User A" rule further down the list ineffective.

### Over-Provisioning
Granting broad permissions (e.g., "Full Control" or "Any/Any") to simplify troubleshooting, which violates the Principle of Least Privilege.

### Orphaned ACEs
Maintaining entries for subjects (users or groups) that no longer exist in the system, leading to "security debt" and potential exploitation if an identifier is reused.

### Excessive Length
Creating ACLs with thousands of entries. This increases computational overhead for evaluation and makes the policy human-unreadable, leading to configuration errors.

## Edge Cases

### Conflicting Permissions
In systems that allow both "Permit" and "Deny" ACEs for the same subject, a conflict resolution strategy must be defined. Most systems prioritize "Deny" entries (Deny-overrides), but some follow the "First Match" rule regardless of the action type.

### Inheritance Blocking
A scenario where a child object is configured to ignore the ACLs of its parent. This can create "dark data" or "hidden ports" that do not conform to the global security policy.

### Empty ACLs
The behavior of an empty ACL varies by implementation. In some systems, an empty ACL denies everything (Implicit Deny); in others, it may grant full access (Default Open). Canonical security requires an empty ACL to result in a Deny.

## Related Topics
*   **042 Role-Based Access Control (RBAC):** A higher-level abstraction for managing permissions.
*   **015 Attribute-Based Access Control (ABAC):** A dynamic access model based on environmental and subject attributes.
*   **102 Network Firewalls:** Implementations of ACLs at the network layer.
*   **009 Filesystem Permissions:** Implementation of ACLs in storage systems.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
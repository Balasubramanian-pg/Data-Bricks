# Creating and Switching Branches

Canonical documentation for Creating and Switching Branches. This document defines concepts, terminology, and standard usage.

## Purpose

Describe why this topic exists and what problem space it addresses. This section should be descriptive, not instructional.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

The purpose of creating and switching branches is to manage different versions of codebases, collaborate with team members, and isolate changes during the development process. Branching allows developers to work on new features, bug fixes, or experiments without affecting the main codebase, thereby reducing the risk of introducing errors or breaking existing functionality. This topic addresses the need for a structured approach to branching, ensuring that developers can efficiently create, manage, and switch between branches.

## Scope

Clarify what is in scope and out of scope for this topic.

**In scope:**
* Branch creation and management
* Branch switching and navigation
* Branch merging and resolution

**Out of scope:**
* Tool-specific implementations (e.g., Git, SVN, Mercurial)
* Vendor-specific behavior (e.g., GitHub, Bitbucket, Azure DevOps)
* Code review and testing processes

## Definitions

Provide precise definitions for key terms used throughout the documentation.

| Term | Definition |
|------|------------|
| Branch | A separate line of development in a version control system, allowing for independent changes and experimentation. |
| Master Branch | The primary branch in a version control system, typically representing the production-ready state of the codebase. |
| Feature Branch | A branch created to develop a new feature or functionality, often merged into the master branch upon completion. |
| Hotfix Branch | A branch created to quickly address a critical issue or bug, often merged into the master branch as soon as possible. |

> [!TIP]
> Definitions should be stable over time; avoid contextual language.

## Core Concepts

Explain the fundamental ideas that make up the topic.

### Branch Creation
Branch creation involves creating a new, independent line of development from an existing branch. This allows developers to work on new features, bug fixes, or experiments without affecting the main codebase.

### Branch Switching
Branch switching involves changing the current branch to a different one, allowing developers to switch between different lines of development. This is often necessary to work on different features, collaborate with team members, or resolve conflicts.

### Branch Merging
Branch merging involves integrating changes from one branch into another, often from a feature branch into the master branch. This process can be manual or automated, depending on the version control system and team workflows.

## Standard Model

Describe the generally accepted or recommended model for this topic.

> [!IMPORTANT]
> Deviations from the standard model should be explicitly documented and justified.

The standard model for creating and switching branches involves the following steps:

1. Create a new branch from the master branch or an existing feature branch.
2. Make changes and commit them to the new branch.
3. Switch to the new branch to work on the changes.
4. Merge the changes into the master branch or another feature branch.
5. Resolve any conflicts that arise during the merge process.
6. Delete the feature branch after merging, if necessary.

## Common Patterns

Document recurring patterns or approaches associated with this topic.

* Feature branch workflow: Create a new branch for each feature, work on the feature, and merge it into the master branch upon completion.
* Hotfix workflow: Create a new branch for each hotfix, work on the hotfix, and merge it into the master branch as soon as possible.
* Release branching: Create a new branch for each release, allowing for stabilization and testing before merging into the master branch.

## Anti-Patterns

Describe common mistakes or discouraged practices.

> [!WARNING]
> These anti-patterns often lead to maintenance or scalability issues.

* Working directly on the master branch, which can lead to unstable code and conflicts.
* Not merging changes regularly, which can lead to large, complex merges and conflicts.
* Not deleting feature branches after merging, which can lead to branch clutter and confusion.

## Edge Cases

Explain unusual, ambiguous, or boundary scenarios related to the topic.

> [!CAUTION]
> Edge cases are frequently overlooked and may cause incorrect assumptions.

* Conflicts between branches: When changes from one branch conflict with changes from another branch, manual resolution is often necessary.
* Branching in a distributed team: When team members are distributed across different locations, branching and merging can become more complex, requiring additional communication and coordination.
* Branching with multiple repositories: When working with multiple repositories, branching and merging can become more complex, requiring additional planning and coordination.

## Related Topics

Link to adjacent or dependent topics.

* Version Control Systems
* Code Review and Testing
* Continuous Integration and Continuous Deployment (CI/CD)

## References

List authoritative external references, specifications, or papers.

* Git Documentation: [https://git-scm.com/docs](https://git-scm.com/docs)
* SVN Documentation: [https://svnbook.red-bean.com/](https://svnbook.red-bean.com/)
* Mercurial Documentation: [https://www.mercurial-scm.org/wiki/](https://www.mercurial-scm.org/wiki/)

## Change Log

Document notable changes to this topic over time.

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial documentation |
| 1.1 | 2026-01-15 | Added section on edge cases |
| 1.2 | 2026-01-20 | Updated section on common patterns |
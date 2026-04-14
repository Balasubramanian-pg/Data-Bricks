# Failure Only vs Every Run Notifications

Canonical documentation for Failure Only vs Every Run Notifications. This document defines concepts, terminology, and standard usage.

## Purpose

Describe why this topic exists and what problem space it addresses. This section should be descriptive, not instructional.

The purpose of Failure Only vs Every Run Notifications is to provide a clear understanding of the different notification strategies that can be employed in various systems, applications, and workflows. This topic exists to address the problem space of notification management, where the goal is to balance the need for timely and relevant information with the risk of overwhelming users with unnecessary notifications. By understanding the differences between Failure Only and Every Run Notifications, developers, administrators, and users can make informed decisions about how to design and configure notification systems to meet their specific needs.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope

Clarify what is in scope and out of scope for this topic.

**In scope:**
* Notification strategies (Failure Only, Every Run)
* Notification triggers (events, errors, completions)
* Notification delivery mechanisms (email, messaging, logging)

**Out of scope:**
* Tool-specific implementations (e.g., Jenkins, GitLab, Slack)
* Vendor-specific behavior (e.g., Microsoft, Amazon, Google)
* Custom notification development

## Definitions

Provide precise definitions for key terms used throughout the documentation.

| Term | Definition |
|------|------------|
| Failure Only Notification | A notification that is sent only when a process, task, or job fails or encounters an error. |
| Every Run Notification | A notification that is sent after every execution of a process, task, or job, regardless of the outcome. |
| Notification Trigger | An event or condition that initiates the sending of a notification. |
| Notification Delivery Mechanism | The method or channel used to deliver notifications to users or systems. |

> [!TIP]
> Definitions should be stable over time; avoid contextual language.

## Core Concepts

Explain the fundamental ideas that make up the topic.

### Failure Only Notification
Failure Only Notification is a strategy where notifications are sent only when a process, task, or job fails or encounters an error. This approach is useful for minimizing noise and ensuring that users are only alerted to critical issues that require attention.

### Every Run Notification
Every Run Notification is a strategy where notifications are sent after every execution of a process, task, or job, regardless of the outcome. This approach is useful for providing a record of all activity, allowing users to track progress and identify trends.

## Standard Model

Describe the generally accepted or recommended model for this topic.

The standard model for Failure Only vs Every Run Notifications involves configuring notification systems to use a combination of both strategies. For example, a system might send Failure Only Notifications for critical tasks, while sending Every Run Notifications for less critical tasks or for tasks that require auditing or compliance.

> [!IMPORTANT]
> Deviations from the standard model should be explicitly documented and justified.

## Common Patterns

Document recurring patterns or approaches associated with this topic.

* Using Failure Only Notifications for critical tasks, such as payment processing or data backups.
* Using Every Run Notifications for less critical tasks, such as daily reports or system checks.
* Configuring notifications to be sent to different recipients or channels based on the outcome of a task or process.

## Anti-Patterns

Describe common mistakes or discouraged practices.

> [!WARNING]
> These anti-patterns often lead to maintenance or scalability issues.

* Sending Every Run Notifications for all tasks, resulting in notification fatigue and decreased user engagement.
* Using Failure Only Notifications for all tasks, resulting in missed opportunities for auditing and compliance.
* Failing to configure notification systems to handle errors or exceptions, resulting in lost or undelivered notifications.

## Edge Cases

Explain unusual, ambiguous, or boundary scenarios related to the topic.

> [!CAUTION]
> Edge cases are frequently overlooked and may cause incorrect assumptions.

* Handling notifications for tasks that have multiple possible outcomes, such as a task that can succeed, fail, or be cancelled.
* Handling notifications for tasks that have complex or conditional logic, such as a task that only sends notifications if certain conditions are met.
* Handling notifications for tasks that are executed in parallel or concurrently, such as a task that is executed multiple times simultaneously.

## Related Topics

Link to adjacent or dependent topics.

* Notification Systems
* Alerting and Monitoring
* Workflow Automation

## References

List authoritative external references, specifications, or papers.

* ITIL (Information Technology Infrastructure Library) - Notification Management
* ISO 20000 - Notification and Communication
* NIST Special Publication 800-53 - Notification and Response

## Change Log

Document notable changes to this topic over time.

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial documentation |
| 1.1 | 2026-02-01 | Added section on Edge Cases |
| 1.2 | 2026-03-01 | Updated section on Common Patterns |
# Cron Expressions for Scheduling

Canonical documentation for Cron Expressions for Scheduling. This document defines concepts, terminology, and standard usage.

## Purpose

Cron expressions are a fundamental component of scheduling systems, allowing users to define the timing and frequency of tasks, jobs, or events. The primary problem space addressed by cron expressions is the need for a standardized, flexible, and expressive way to specify time-based triggers for automated processes. This documentation exists to provide a comprehensive and authoritative guide to cron expressions, ensuring consistency and clarity across various implementations and use cases.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope

**In scope:**
* Cron expression syntax and semantics
* Standard cron expression formats
* Best practices for designing and using cron expressions

**Out of scope:**
* Tool-specific implementations (e.g., Quartz, cron daemon)
* Vendor-specific behavior or extensions
* Scheduling algorithms or engine internals

## Definitions

| Term | Definition |
|------|------------|
| Cron Expression | A string that specifies the timing and frequency of a scheduled task or event, using a standardized syntax. |
| Field | A component of a cron expression that represents a unit of time (e.g., minute, hour, day). |
| Wildcard | A special character (`*`) used to match any value in a field. |
| Literal | A specific value used in a field (e.g., `15` for the minute field). |

> [!TIP]
> Definitions should be stable over time; avoid contextual language.

## Core Concepts

### Concept One: Cron Expression Structure
A cron expression consists of five fields: minute, hour, day of month, month, and day of week. Each field is separated by a space, and the fields are ordered in a specific way to ensure consistency and readability.

### Concept Two: Field Values and Ranges
Field values can be specified using literals, wildcards, or ranges. Ranges are defined using a dash (`-`) and allow for specifying a contiguous set of values (e.g., `1-5` for the day of week field).

## Standard Model

The standard model for cron expressions is based on the following syntax:
```
minute hour day month day_of_week
```
Each field has a specific set of allowed values, and the fields are combined to form a unique cron expression. The standard model is widely adopted and supported by most scheduling systems.

> [!IMPORTANT]
> Deviations from the standard model should be explicitly documented and justified.

## Common Patterns

* Pattern A: Using wildcards to specify any value in a field (e.g., `* * * * *` to run a task every minute).
* Pattern B: Specifying a range of values in a field (e.g., `1-5 * * * *` to run a task every hour on weekdays).

## Anti-Patterns

> [!WARNING]
> These anti-patterns often lead to maintenance or scalability issues.

* Anti-pattern A: Using ambiguous or unclear cron expressions (e.g., `1-2 * * * *` without specifying the intended behavior).
* Anti-pattern B: Overusing wildcards or ranges, leading to unintended matches or overlaps.

## Edge Cases

> [!CAUTION]
> Edge cases are frequently overlooked and may cause incorrect assumptions.

* Edge case: Specifying a cron expression with a field value that is outside the allowed range (e.g., `60 * * * *` for the minute field).
* Edge case: Using a cron expression with conflicting field values (e.g., `1 * * * 1` and `* 1 * * *`).

## Related Topics

* Related Topic A: Scheduling algorithms and engine internals
* Related Topic B: Task or job management systems

## References

* [RFC 5545: Internet Calendaring and Scheduling Core Object Specification](https://tools.ietf.org/html/rfc5545)
* [Cron Expression Syntax (Wikipedia)](https://en.wikipedia.org/wiki/Cron#Syntax)

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial documentation |
| 1.1 | 2026-02-20 | Added section on edge cases and updated references |
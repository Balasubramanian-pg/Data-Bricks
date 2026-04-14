# 003 Databricks and Apache Spark Relationship

Canonical documentation for 003 Databricks and Apache Spark Relationship. This document defines concepts, terminology, and standard usage.

## Purpose
The relationship between Databricks and Apache Spark represents a foundational paradigm in modern data engineering: the distinction between an open-source core engine and a managed commercial platform. This topic exists to clarify how these two entities interact, the governance of the technology, and the architectural boundaries between the open-source framework and the proprietary optimizations built upon it. Understanding this relationship is critical for making informed decisions regarding portability, performance, and vendor lock-in.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* The historical and governance relationship between the Apache Software Foundation (ASF) and Databricks.
* The architectural distinction between the Spark core and the Databricks Runtime (DBR).
* The concept of API compatibility and "upstream" contributions.
* The evolution from a processing engine to a unified data platform.

**Out of scope:**
* Specific cloud provider integrations (e.g., Azure Databricks vs. Databricks on AWS).
* Step-by-step installation guides for Spark or Databricks.
* Pricing models or commercial licensing details.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| Apache Spark | An open-source, multi-language engine for executing data engineering, data science, and machine learning on single-node machines or clusters. |
| Databricks | A unified data analytics platform founded by the original creators of Apache Spark, providing a managed environment for Spark-based workloads. |
| Databricks Runtime (DBR) | A proprietary set of software components that include Apache Spark but add optimized libraries, security layers, and performance enhancements. |
| Photon | A vectorized execution engine developed by Databricks, written in C++, designed to accelerate Spark SQL and DataFrame workloads while maintaining Spark API compatibility. |
| Upstream | The process of contributing code or features from a private or commercial entity back to the original open-source project (the Apache Spark repository). |
| OSS (Open Source Software) | Software with source code that anyone can inspect, modify, and enhance; in this context, referring to the Apache-licensed version of Spark. |

## Core Concepts

### 1. The Engine vs. The Platform
Apache Spark is a **compute engine**. It provides the libraries for distributed data processing (SQL, Streaming, MLlib, GraphX). Databricks is a **platform** that wraps this engine with an integrated development environment (IDE), cluster management, security (RBAC), and automated scaling.

### 2. Governance and Stewardship
While Databricks employs many of the original creators and current maintainers of Apache Spark, the project itself is governed by the **Apache Software Foundation (ASF)**. This ensures that the Spark project remains independent of any single commercial entity's interests, even though Databricks is the primary contributor to the codebase.

### 3. The "Unified" Philosophy
Both Spark and Databricks adhere to the philosophy of "unification"—the idea that data processing (batch), real-time ingestion (streaming), and analytical modeling (machine learning) should occur within a single framework rather than disparate systems.

## Standard Model
The standard model of the Databricks-Spark relationship is defined by **API Parity with Performance Divergence**.

1.  **API Parity:** Databricks maintains strict compatibility with the Apache Spark APIs (Python, Scala, SQL, R). Code written for OSS Spark should, in theory, run on Databricks without modification.
2.  **Performance Divergence:** While the APIs are the same, the underlying execution layer in Databricks (via the Databricks Runtime and Photon) is optimized for cloud environments. This creates a "Super-set" model where Databricks offers the full functionality of Spark plus proprietary performance and usability enhancements.
3.  **The Lakehouse Architecture:** The relationship has evolved to support the "Lakehouse" model, where Spark serves as the processing engine for data stored in open formats (like Delta Lake) on cloud object storage, managed by the Databricks control plane.

## Common Patterns

### The "Develop Local, Scale Cloud" Pattern
Developers often use OSS Apache Spark for local development and unit testing (using small datasets or containers) and then deploy the same code to a Databricks environment for production-scale processing.

### The Upstream Contribution Cycle
New features often debut in the Databricks platform to gather telemetry and user feedback. Once stabilized, many of these core features (such as Adaptive Query Execution or certain SQL enhancements) are contributed "upstream" to the Apache Spark project to ensure the health of the ecosystem.

### Decoupled Storage and Compute
In the Databricks-Spark model, data is rarely stored "in" Spark or "in" Databricks. Instead, Spark acts as a transient compute layer that interacts with decoupled storage (S3, ADLS, GCS), with Databricks providing the orchestration.

## Anti-Patterns

### Proprietary Dependency Over-reliance
Using Databricks-specific utility libraries (e.g., `dbutils`) within core data processing logic. This breaks the portability of the Spark code, making it difficult to run on standard Apache Spark distributions.
*   *Correction:* Keep business logic in pure Spark/Python/Scala and use platform utilities only for environment-level orchestration.

### Version Mismatching
Assuming that "Spark 3.x" on Databricks is identical to "Spark 3.x" in the OSS distribution. Databricks often backports fixes or includes "pre-release" features from the Spark master branch into their runtime.
*   *Correction:* Always verify the specific Databricks Runtime (DBR) release notes to understand which Spark patches are included.

### Treating Databricks as a Database
Using Spark/Databricks for low-latency, point-lookup queries typical of an RDBMS.
*   *Correction:* Use Spark for high-throughput analytical processing and export results to a serving layer for point-lookups.

## Edge Cases

### The "Photon" Execution Layer
Photon is a significant edge case because it replaces the Spark JVM-based execution with a C++ engine. While it respects Spark APIs, the internal execution plan and error handling may differ from standard Spark, occasionally leading to different behaviors in complex floating-point calculations or memory management.

### Custom Spark Distributions
Organizations running "Vanilla" Spark on Kubernetes or EMR may find that certain optimizations (like Z-Order indexing or certain Auto-loader features) are unavailable because they reside in the Databricks proprietary layer, not the Spark core.

## Related Topics
*   **001 Distributed Computing Fundamentals:** The underlying theory of Spark's architecture.
*   **004 Delta Lake Protocol:** The storage layer often used in conjunction with Spark and Databricks.
*   **010 Data Lakehouse Architecture:** The broader architectural pattern enabled by this relationship.
*   **015 Vectorized Execution:** The technical concept behind the Photon engine.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
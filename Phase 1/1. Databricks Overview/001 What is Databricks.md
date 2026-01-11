# What is Databricks

Canonical documentation for What is Databricks. This document defines concepts, terminology, and standard usage.

## Purpose

Databricks is a unified analytics platform that enables data engineers, data scientists, and data analysts to collaborate and work together on big data analytics. This topic exists to provide a comprehensive understanding of Databricks, its core concepts, and its applications, addressing the problem space of big data processing, analytics, and machine learning. The purpose of this documentation is to serve as a single source of truth for Databricks, providing a clear and concise explanation of its capabilities, features, and best practices.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope

**In scope:**
* Overview of Databricks and its architecture
* Core concepts, such as Apache Spark, Delta Lake, and Databricks Notebooks
* Standard usage and best practices for Databricks

**Out of scope:**
* Tool-specific implementations, such as Databricks on AWS or Azure
* Vendor-specific behavior, such as custom or proprietary features
* Detailed tutorials or step-by-step guides for using Databricks

## Definitions

| Term | Definition |
|------|------------|
| Databricks | A unified analytics platform that enables data engineers, data scientists, and data analysts to collaborate and work together on big data analytics |
| Apache Spark | An open-source data processing engine that provides high-level APIs in Java, Python, Scala, and R for large-scale data processing |
| Delta Lake | An open-source storage layer that provides a scalable and reliable way to store and manage data in a data lake |
| Databricks Notebook | A web-based interface for writing, executing, and sharing code in a variety of programming languages, including Python, R, Scala, and SQL |

> [!TIP]
> Definitions should be stable over time; avoid contextual language.

## Core Concepts

### Apache Spark
Apache Spark is a fundamental component of Databricks, providing a high-level API for large-scale data processing. Spark is designed to handle massive amounts of data and provides a flexible and efficient way to process data in parallel.

### Delta Lake
Delta Lake is a scalable and reliable storage layer that provides a way to store and manage data in a data lake. Delta Lake is designed to provide high-performance and low-latency data access, making it ideal for real-time analytics and machine learning workloads.

### Databricks Notebooks
Databricks Notebooks provide a web-based interface for writing, executing, and sharing code in a variety of programming languages. Notebooks are designed to facilitate collaboration and provide a flexible way to work with data, making it easy to prototype, test, and deploy data-driven applications.

## Standard Model

The standard model for Databricks involves using Apache Spark as the primary data processing engine, Delta Lake as the storage layer, and Databricks Notebooks as the primary interface for working with data. This model provides a scalable, reliable, and flexible way to work with big data, making it ideal for a wide range of use cases, from data warehousing and business intelligence to machine learning and real-time analytics.

> [!IMPORTANT]
> Deviations from the standard model should be explicitly documented and justified.

## Common Patterns

* Data ingestion and processing using Apache Spark and Delta Lake
* Data transformation and analysis using Databricks Notebooks and Spark SQL
* Machine learning model training and deployment using Databricks and popular machine learning libraries such as scikit-learn and TensorFlow

## Anti-Patterns

* Using Databricks as a replacement for a traditional data warehouse, without considering the trade-offs and limitations
* Not optimizing Spark jobs for performance, leading to slow and inefficient data processing
* Not using version control and collaboration tools, such as Git and Databricks Notebooks, to manage and share code

> [!WARNING]
> These anti-patterns often lead to maintenance or scalability issues, and can limit the effectiveness of Databricks in supporting big data analytics and machine learning workloads.

## Edge Cases

* Handling large and complex datasets that exceed the limits of Spark or Delta Lake
* Integrating Databricks with other data sources and systems, such as relational databases or messaging queues
* Supporting real-time analytics and streaming data processing using Databricks and Spark

> [!CAUTION]
> Edge cases are frequently overlooked and may cause incorrect assumptions, so it's essential to carefully consider and plan for these scenarios when working with Databricks.

## Related Topics

* Big data analytics and processing
* Machine learning and artificial intelligence
* Data warehousing and business intelligence
* Cloud computing and infrastructure

## References

* Apache Spark documentation: <https://spark.apache.org/docs/latest/>
* Delta Lake documentation: <https://delta.io/>
* Databricks documentation: <https://docs.databricks.com/>

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial documentation |
| 1.1 | 2026-01-12 | Added section on anti-patterns and edge cases |
| 1.2 | 2026-01-13 | Updated references and added link to Databricks documentation |
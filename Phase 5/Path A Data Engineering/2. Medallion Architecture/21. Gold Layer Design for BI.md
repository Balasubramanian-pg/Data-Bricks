# 021 Gold Layer Design for BI

Canonical documentation for 021 Gold Layer Design for BI. This document defines concepts, terminology, and standard usage.

## Purpose
The Gold Layer represents the final stage of data transformation within a multi-layered data architecture (often referred to as the Medallion Architecture). Its primary purpose is to provide a curated, high-performance, and business-aligned presentation layer for Business Intelligence (BI) tools, data visualization, and executive reporting.

This layer addresses the gap between technical data structures and business logic. It transforms normalized or semi-structured data into consumption-ready formats that ensure consistency, accuracy, and ease of use for non-technical stakeholders.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   Architectural principles of the Gold Layer.
*   Data modeling strategies for BI consumption.
*   Governance and quality standards for presentation-ready data.
*   Logical structuring of facts, dimensions, and aggregates.

**Out of scope:**
*   Specific vendor implementations (e.g., Databricks Delta Live Tables, Snowflake-specific syntax, Power BI DAX optimization).
*   Ingestion patterns (Bronze Layer) or cleansing logic (Silver Layer).
*   Physical storage optimization (e.g., partitioning, indexing, or file format specifics).

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Gold Layer** | The final, consumption-ready layer of a data platform, optimized for BI and analytics. |
| **Semantic Layer** | A business-oriented mapping that represents data in terms of business concepts rather than technical structures. |
| **Granularity** | The level of detail or precision represented by a single row in a table. |
| **Fact Table** | A table containing quantitative measurements (metrics) and foreign keys to dimension tables. |
| **Dimension Table** | A table containing descriptive attributes (context) used to filter and group fact data. |
| **Denormalization** | The process of combining tables to reduce complexity and improve query performance for BI tools. |
| **Conformed Dimension** | A dimension that has the same meaning and content when used across different fact tables. |

## Core Concepts
The Gold Layer is governed by three fundamental pillars:

### 1. Business Logic Encapsulation
Unlike the Silver Layer, which focuses on data integrity and normalization, the Gold Layer is where complex business rules are applied. This includes calculated KPIs (e.g., Year-over-Year growth, Churn Rate), currency conversions, and specific fiscal calendar alignments.

### 2. Performance Optimization
Data in the Gold Layer is structured to minimize the computational load on BI tools. This often involves pre-aggregating data or flattening hierarchies to ensure sub-second response times for end-user dashboards.

### 3. Trust and Governance
The Gold Layer serves as the "Single Source of Truth." Data entering this layer must have passed all quality checks. It is the only layer typically exposed to the broader business community, requiring strict access controls and clear metadata documentation.

## Standard Model
The Gold Layer typically adheres to one of two primary modeling paradigms, depending on the use case:

### The Dimensional Model (Star Schema)
The most widely accepted model for BI. It consists of:
*   **Central Fact Tables:** Storing quantitative data (e.g., Sales Amount, Quantity).
*   **Surrounding Dimension Tables:** Providing context (e.g., Product, Time, Geography).
*   **Benefits:** Highly intuitive for BI tools, reduces join complexity, and supports "drill-down" capabilities.

### One Big Table (OBT)
A fully denormalized approach where all facts and dimensions are collapsed into a single, wide table.
*   **Benefits:** Eliminates joins entirely, offering maximum performance for specific high-volume analytical queries and simplified consumption for flat-file reporting.

## Common Patterns
*   **Slowly Changing Dimensions (SCD):** Implementing Type 1 (overwrite) or Type 2 (historical tracking) logic to manage how attribute changes are reflected in reports.
*   **Snapshotting:** Capturing the state of a business process at specific intervals (e.g., Monthly Inventory Snapshots) to allow for historical trend analysis.
*   **Role-Playing Dimensions:** Using a single physical dimension table for multiple logical purposes (e.g., a Date table acting as both "Order Date" and "Ship Date").
*   **Bridge Tables:** Used to handle many-to-many relationships between facts and dimensions without duplicating data.

## Anti-Patterns
*   **Exposing Silver Layer Directly:** Allowing BI tools to query the Silver Layer leads to inconsistent logic across different reports and poor performance.
*   **Circular Dependencies:** Creating Gold tables that depend on other Gold tables in a recursive loop, complicating data lineage and orchestration.
*   **Over-Aggregation:** Aggregating data so heavily that users lose the ability to perform granular analysis, forcing them back to raw data sources.
*   **Hard-Coding Logic in BI Tools:** Placing complex business logic inside the visualization layer (e.g., Power BI measures) rather than the Gold Layer, leading to "logic silos."

## Edge Cases
*   **Late-Arriving Data:** Handling scenarios where a fact record arrives before its corresponding dimension record (e.g., a sale recorded for a product not yet in the master catalog).
*   **Multi-Grain Facts:** Managing fact tables that contain data at different levels of detail (e.g., actual sales at the daily level vs. budget targets at the monthly level).
*   **Real-Time Gold Views:** Balancing the need for "up-to-the-minute" data with the performance requirements of a structured Gold Layer, often solved via "Lambda" views that union batch and stream data.

## Related Topics
*   **010 Bronze Layer Standards:** The entry point for raw data.
*   **011 Silver Layer Design:** The intermediate layer for cleansing and normalization.
*   **Data Governance Framework:** The overarching policies for data quality and access.
*   **Master Data Management (MDM):** The source of truth for conformed dimensions.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
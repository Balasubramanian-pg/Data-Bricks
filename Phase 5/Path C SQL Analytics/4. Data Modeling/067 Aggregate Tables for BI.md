# 067 Aggregate Tables for BI

Canonical documentation for 067 Aggregate Tables for BI. This document defines concepts, terminology, and standard usage.

## Purpose
Aggregate tables exist to bridge the performance gap between massive volumes of atomic-level data and the low-latency requirements of Business Intelligence (BI) applications. As datasets grow into the petabyte scale, scanning raw transaction logs for every dashboard interaction becomes computationally expensive and prohibitively slow. 

Aggregate tables address this by storing pre-calculated summaries of data at higher levels of granularity. This reduces the number of rows processed during query execution, optimizes resource utilization in data warehouses, and ensures a responsive user experience for end-state analytical consumers.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
**In scope:**
* Architectural placement of aggregate tables within a data stack.
* Strategies for dimensionality reduction and granularity selection.
* Maintenance and synchronization logic between base tables and aggregates.
* Performance optimization theory for analytical workloads.

**Out of scope:**
* Specific vendor implementations (e.g., Snowflake Materialized Views, BigQuery BI Engine, or Power BI Aggregations).
* Physical storage formats (e.g., Parquet vs. Avro).
* Specific SQL syntax for creating tables.

## Definitions
| Term | Definition |
|------|------------|
| **Atomic Data** | The lowest level of detail captured by a system (e.g., a single transaction or sensor heartbeat). |
| **Grain** | The level of detail represented by a single row in a table. |
| **Dimensionality** | The number of attributes or categories by which data can be filtered or grouped. |
| **Roll-up** | The process of aggregating data from a lower level of detail to a higher level (e.g., Daily to Monthly). |
| **Materialization** | The act of physically persisting the results of an aggregate query to storage. |
| **Query Routing** | The logic (often in a semantic layer) that determines whether to query an aggregate table or the base table. |
| **Sparsity** | A measure of how many potential combinations of dimensions actually contain data. |
| **Additive Measure** | A metric that can be meaningfully summed across any dimension (e.g., Sales Amount). |

## Core Concepts

### 1. Dimensionality Reduction
The primary mechanism of an aggregate table is the removal of high-cardinality dimensions. By excluding attributes like `Transaction_ID` or `Timestamp_Seconds` and retaining only `Store_ID` and `Date`, the row count is compressed significantly.

### 2. Pre-computation
Aggregate tables shift the "computational tax" from the query-time (read) to the transformation-time (write). Instead of calculating a `SUM()` or `AVG()` every time a user opens a report, the calculation is performed once during the ETL/ELT process.

### 3. The Performance-Storage Trade-off
While aggregate tables improve query speed, they introduce storage overhead and management complexity. An effective BI strategy balances the cost of storing multiple aggregate variations against the performance gains realized by the end users.

### 4. Transparency and Routing
In an ideal architecture, the existence of aggregate tables is transparent to the end user. A "Semantic Layer" or "Query Optimizer" should automatically intercept a query and route it to the smallest (most aggregated) table capable of answering the request.

## Standard Model

The standard model for BI aggregates follows a hierarchical structure based on the **Base Fact Table**.

1.  **Base Fact Table (Grain: Atomic):** Contains every transaction.
2.  **Aggregate Fact Table (Grain: Dimensional):** Contains data grouped by specific dimensions (e.g., Product, Region, Month).
3.  **Navigation Logic:**
    *   If the user requests `Sales by Year`, the system routes to the `Yearly_Aggregate`.
    *   If the user requests `Sales by Transaction_ID`, the system routes to the `Base_Fact`.

### The "Shrunken Dimension" Pattern
Aggregate tables often use "shrunken dimensions"—subsets of full dimension tables that only contain the attributes relevant to the aggregate grain. For example, a `Date_Aggregate` table would link to a `Calendar_Dimension` but would ignore time-of-day attributes.

## Common Patterns

### 1. Time-Series Aggregates
The most common pattern where data is rolled up into fixed time buckets (Hourly, Daily, Monthly, Yearly). This is highly effective for trend analysis.

### 2. Categorical/Attribute Aggregates
Aggregating by high-level business hierarchies, such as `Product_Category` instead of `SKU`, or `State` instead of `Zip_Code`.

### 3. Materialized Views
A pattern where the database engine manages the synchronization between the base table and the aggregate table automatically, often refreshing the aggregate as new data arrives in the base table.

### 4. Incremental Refresh
Instead of rebuilding the entire aggregate table, only the periods affected by new data (e.g., the current day's transactions) are recalculated and merged.

## Anti-Patterns

*   **Aggregate Explosion:** Creating too many aggregate tables for every possible combination of dimensions, leading to "management hell" and excessive storage costs.
*   **Manual Synchronization:** Relying on manual scripts to update aggregates, which often leads to "Data Drift" where the aggregate totals do not match the base table totals.
*   **Aggregating Non-Additive Metrics:** Attempting to pre-calculate and sum metrics like `Unique_Visitors` or `Ratios`. These must be calculated from the base or via specific sketches (like HyperLogLog).
*   **Over-Aggregation:** Creating an aggregate so high-level (e.g., Total Sales by Year only) that it serves almost no real-world business questions, forcing users back to the base table.

## Edge Cases

### 1. Late-Arriving Data
If a transaction from three days ago is processed today, the aggregate tables for the last three days must be invalidated and recalculated to maintain accuracy.

### 2. Non-Additive Measures (The "Distinct Count" Problem)
You cannot sum a "Daily Unique Users" count to get a "Monthly Unique Users" count. For these cases, aggregate tables must store raw IDs or use probabilistic data structures (sketches) rather than simple scalars.

### 3. Changing Hierarchies (SCD Type 2)
If a product moves from "Category A" to "Category B," historical aggregate tables may need to be restated if the business requirement is to reflect the current hierarchy across all time.

## Related Topics
*   **012 Dimensional Modeling:** The foundational strategy for organizing data into facts and dimensions.
*   **045 Semantic Layers:** The technology that handles query routing between base and aggregate tables.
*   **089 Materialized Views:** A specific technical implementation of the aggregate concept.
*   **102 Data Latency and Freshness:** The study of how quickly data moves from source to aggregate.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
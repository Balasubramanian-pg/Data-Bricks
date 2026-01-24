# 064 Caching for BI Performance

Canonical documentation for 064 Caching for BI Performance. This document defines concepts, terminology, and standard usage.

## Purpose
The primary purpose of caching in Business Intelligence (BI) is to bridge the latency gap between massive data volumes and the requirement for sub-second interactive analysis. BI workloads are characterized by complex aggregations, multi-table joins, and repetitive query patterns. Without caching, every user interaction (filtering, drilling down, or pivoting) requires a full execution cycle on the underlying data source, which often leads to resource exhaustion and poor user experience.

Caching provides a high-speed intermediary storage layer that serves pre-computed or previously retrieved results, thereby reducing the computational load on source systems and minimizing the "Time to Insight" for end-users.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   **Architectural Layers:** Client-side, application-tier, and database-tier caching mechanisms.
*   **Data Granularity:** Result-set caching vs. metadata and schema caching.
*   **Lifecycle Management:** Invalidation strategies, Time-to-Live (TTL), and cache warming.
*   **Performance Metrics:** Hit ratios, latency reduction, and throughput.

**Out of scope:**
*   Specific vendor implementations (e.g., Power BI Premium Capacity, Tableau Server Cache, Snowflake Result Cache).
*   General web caching (CDN, browser assets) not specific to analytical data.
*   Hardware-level CPU/L1/L2 caching.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Cache Hit** | A state where requested data is found in the cache, allowing the system to bypass the primary data source. |
| **Cache Miss** | A state where requested data is not found in the cache, necessitating a query to the underlying data source. |
| **TTL (Time to Live)** | A setting that dictates the duration for which a cached object remains valid before it is considered stale. |
| **Invalidation** | The process of removing or refreshing cached data when the underlying source data changes. |
| **Warm-up** | The proactive process of populating the cache with anticipated queries before users request them. |
| **Result-set Caching** | Storing the final output of a specific query (e.g., a formatted table or chart data). |
| **Pre-aggregation** | Storing summarized data (e.g., monthly totals) to avoid calculating them from raw grains at runtime. |
| **Cold Start** | The performance degradation experienced when a system starts with an empty cache. |

## Core Concepts

### The Latency-Freshness Trade-off
The fundamental tension in BI caching is between **Data Freshness** (how closely the data reflects the current state of the source) and **Performance** (how quickly the user sees the data). A more aggressive caching strategy improves performance but increases the risk of presenting stale information.

### Cache Granularity
Caching can occur at various levels of abstraction:
1.  **Metadata Cache:** Stores schema information, table structures, and security permissions.
2.  **Query Plan Cache:** Stores the optimized execution path for a SQL or MDX statement.
3.  **Data Fragment Cache:** Stores specific columns or filtered subsets of data.
4.  **Result-set Cache:** Stores the final, fully-formed response to a specific user request.

### Determinism
For a cache to be effective, queries must be deterministic. If a query contains volatile functions (e.g., `GETDATE()` or `RAND()`), the cache engine must determine if the result is reusable or if the function forces a cache miss to ensure accuracy.

## Standard Model
The standard model for BI caching follows a multi-tier hierarchy:

1.  **Presentation/Client Layer:** Localized caching within the user's browser or desktop application. This handles immediate UI interactions like sorting a rendered table.
2.  **Application/Semantic Layer:** The BI server maintains a cache of shared query results. If User A and User B request the same dashboard, User B receives the cached result from the application tier.
3.  **Acceleration/Materialization Layer:** An intermediary storage (often in-memory or on high-speed SSDs) where pre-aggregated "cubes" or materialized views reside.
4.  **Source Layer:** The underlying data warehouse or lake. This is the "Source of Truth" accessed only on a cache miss.

## Common Patterns

### 1. Scheduled Refresh (Pull-based)
The cache is updated on a fixed schedule (e.g., every hour). This is common for executive dashboards where data does not change minute-to-minute.

### 2. Event-Driven Invalidation (Push-based)
The completion of an ETL (Extract, Transform, Load) process triggers a signal to the BI layer to purge or refresh the cache. This ensures the cache is never more "stale" than the last data load.

### 3. Lazy Loading (Demand-based)
The cache is populated only when a user requests data. The first user experiences a "Cold Start" (high latency), while subsequent users benefit from the "Warm" cache.

### 4. Proactive Warming
The system runs a set of "heavy" queries automatically after a data refresh to ensure that by the time users log in, the cache is already populated with high-priority data.

## Anti-Patterns

*   **Caching Sensitive Data without Security Context:** Storing a result set that includes sensitive rows and serving it to a user who lacks the permissions to see that data (ignoring Row-Level Security).
*   **Infinite TTL:** Setting a cache to never expire, leading to "Data Drift" where the BI report becomes completely disconnected from reality.
*   **Over-Caching High-Cardinality Data:** Attempting to cache results for queries with infinite variations (e.g., unique ID lookups), which leads to low hit ratios and "Cache Thrashing."
*   **Ignoring Cache Invalidation on Schema Change:** Failing to clear the cache when the underlying table structure changes, resulting in query errors or corrupted visualizations.

## Edge Cases

*   **Personalized Data:** When a report uses "Current User" filters, the cache must be partitioned by User ID. This can lead to a massive increase in cache volume and a decrease in the global hit ratio.
*   **Near Real-Time Requirements:** In environments where data changes every few seconds, traditional caching may be detrimental. In these cases, "Micro-caching" (TTL of 1–5 seconds) is often used to prevent "Query Storms" from multiple users hitting the same dashboard simultaneously.
*   **Non-Deterministic Parameters:** Queries that rely on external variables (like current exchange rates or weather data) require sophisticated invalidation logic to ensure the cached value remains relevant.

## Related Topics
*   **021 Data Modeling for Aggregates:** How to design tables specifically for pre-computation.
*   **045 Query Optimization:** Techniques to improve the performance of the "Cache Miss" path.
*   **088 Row-Level Security (RLS):** The impact of security filters on cache reusability.
*   **102 Materialized Views:** Database-level persistence of query results.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
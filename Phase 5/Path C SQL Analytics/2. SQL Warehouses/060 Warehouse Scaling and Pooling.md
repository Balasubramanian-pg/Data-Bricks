# 060 Warehouse Scaling and Pooling

Canonical documentation for 060 Warehouse Scaling and Pooling. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of Warehouse Scaling and Pooling is to manage the tension between computational performance, concurrency requirements, and cost efficiency in modern data environments. As data volumes and user demands fluctuate, static resource allocation leads to either performance degradation during peaks or wasteful expenditure during troughs. Scaling and pooling provide the mechanisms to dynamically adjust compute capacity and logically isolate workloads to ensure predictable service levels.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Mechanisms for adjusting compute capacity (Vertical and Horizontal scaling).
* Logical grouping of compute resources (Pooling).
* Workload isolation and concurrency management strategies.
* Resource allocation logic and elasticity principles.

**Out of scope:**
* Specific vendor implementations (e.g., Snowflake Warehouses, Databricks SQL Warehouses, BigQuery Slots).
* Storage-layer scaling (this document focuses on compute).
* Network-level load balancing.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Compute Warehouse** | A logical grouping of processing resources (CPU, RAM) dedicated to executing data tasks. |
| **Vertical Scaling** | Increasing or decreasing the power (size) of a single compute unit to handle more complex queries. |
| **Horizontal Scaling** | Increasing or decreasing the number of identical compute units (clusters) to handle higher concurrency. |
| **Resource Pooling** | The practice of segregating compute resources into distinct groups to prevent workload interference. |
| **Concurrency** | The number of queries or tasks executing simultaneously on a warehouse. |
| **Elasticity** | The ability of a system to automatically scale resources up and down in response to real-time demand. |
| **Cold Start** | The latency incurred when initializing new compute resources that were previously inactive. |
| **Workload Isolation** | The guarantee that the resource consumption of one process does not impact the performance of another. |

## Core Concepts

### 1. Separation of Storage and Compute
The foundation of modern scaling and pooling is the decoupling of persistent storage from ephemeral compute. This allows compute resources to be spun up, shut down, or resized without affecting the underlying data integrity or availability.

### 2. Dimensionality of Scaling
Scaling occurs across two primary axes:
*   **Scale-Up (Vertical):** Addresses "Large" problems. Used when a single query is too memory-intensive or computationally heavy for the current resource size.
*   **Scale-Out (Horizontal):** Addresses "Many" problems. Used when the volume of incoming queries exceeds the throughput capacity of a single cluster.

### 3. Logical Pooling
Pooling is the strategic organization of compute resources into "silos" based on business function, priority, or technical requirements. This prevents a "noisy neighbor" effect where a massive batch transformation job starves a high-priority executive dashboard of resources.

## Standard Model

The standard model for Warehouse Scaling and Pooling follows a tiered architecture:

1.  **The Request Layer:** Incoming queries are analyzed for complexity and origin.
2.  **The Routing Layer:** Queries are assigned to a specific **Resource Pool** based on predefined policies (e.g., User Group, Application ID).
3.  **The Execution Layer (The Warehouse):**
    *   If the pool is idle, it triggers an **Auto-Resume**.
    *   If the pool is at capacity, the **Scaling Policy** determines whether to queue the query or trigger **Horizontal Scaling** (adding a cluster).
    *   If the query is identified as "Large," it may be routed to a pool pre-configured for **Vertical Scaling**.
4.  **The Optimization Layer:** Once demand subsides, the system monitors for a period of inactivity before triggering an **Auto-Suspend** to minimize costs.

## Common Patterns

### The Multi-Tenant Isolation Pattern
Assigning dedicated pools to different departments (e.g., Finance Pool, Marketing Pool, Data Science Pool). This ensures that the Marketing team’s ad-hoc queries never slow down the Finance team’s end-of-month reporting.

### The "Follow the Sun" Pattern
A scheduled scaling approach where compute capacity is increased during business hours in specific time zones and reduced to a minimum footprint during overnight hours.

### The Burst-to-Batch Pattern
Utilizing a highly elastic pool for ETL/ELT processes that scales out aggressively during the data loading window and shuts down completely once the transformation pipeline is finished.

### The Query-Size Routing Pattern
Routing short, high-concurrency "point lookups" to a small, multi-cluster pool, while routing complex "full-table scans" to a single, large-scale warehouse.

## Anti-Patterns

*   **The Monolithic Warehouse:** Using a single large warehouse for all company workloads. This leads to resource contention and makes cost attribution impossible.
*   **Over-Provisioning for Peak:** Maintaining a permanent high-capacity warehouse to handle a peak that only occurs 5% of the time, leading to significant waste.
*   **Aggressive Auto-Suspend:** Setting the suspension timer too low (e.g., 1 second), which causes "thrashing"—the constant cycle of suspending and resuming that incurs "cold start" latency penalties for every query.
*   **Ignoring Query Limits:** Allowing a single unoptimized query to consume all clusters in a horizontally scaled pool, blocking all other users.

## Edge Cases

*   **The "Flash Crowd":** A sudden, massive spike in users (e.g., a viral marketing event) that exceeds the "Scaling Lead Time." The system may experience a momentary bottleneck while new clusters are provisioned.
*   **The Zombie Query:** A query that enters an infinite loop or a cross-join that never finishes. Without "Statement Timeouts," this query can keep a warehouse active and scaling indefinitely.
*   **Metadata-Only Queries:** Queries that can be answered by the storage layer's metadata (e.g., `COUNT(*)` on some systems) may not trigger a warehouse resume, leading to inconsistent expectations of when a warehouse "should" be active.

## Related Topics
*   **042 Workload Management (WLM):** The logic used to prioritize and schedule queries within a pool.
*   **075 FinOps and Cost Attribution:** The practice of monitoring and assigning the costs generated by scaling and pooling.
*   **012 Data Governance:** Policies defining who has the authority to trigger scaling events.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
# 048 Online vs Offline Feature Serving

Canonical documentation for 048 Online vs Offline Feature Serving. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of feature serving is to provide machine learning (ML) models with the input data (features) required to generate predictions. Because ML models are utilized in diverse environments—ranging from real-time web applications to massive batch processing jobs—the infrastructure must support two distinct delivery modes: **Online** and **Offline**. 

This topic addresses the fundamental challenge of maintaining consistency between these two modes to prevent "training-serving skew," while optimizing for the conflicting requirements of low-latency retrieval (Online) and high-throughput historical analysis (Offline).

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Architectural differences between online and offline data delivery.
* Performance characteristics (latency, throughput, freshness).
* Data consistency and point-in-time correctness.
* The role of the feature store in bridging both modes.

**Out of scope:**
* Specific vendor implementations (e.g., Feast, Tecton, AWS SageMaker Feature Store).
* Specific database engine internals (e.g., Redis vs. Cassandra).
* Feature engineering logic (the "how" of calculation).

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Feature** | An individual measurable property or characteristic of a phenomenon being observed, used as an input to an ML model. |
| **Online Serving** | The real-time delivery of feature values to a model for immediate inference, typically via an API. |
| **Offline Serving** | The high-throughput delivery of historical feature values, typically used for model training or batch scoring. |
| **Training-Serving Skew** | A discrepancy between the feature values used during model training and those provided during real-time inference. |
| **Point-in-Time Correctness** | The ability to retrieve the exact value of a feature as it existed at a specific historical timestamp, preventing "data leakage." |
| **Feature Freshness** | The delay between an event occurring in the real world and the updated feature value being available for serving. |
| **Online Store** | A low-latency database (usually Key-Value) optimized for single-record lookups. |
| **Offline Store** | A high-capacity storage system (usually columnar or data lake) optimized for large-scale scans and joins. |

## Core Concepts

### 1. Latency vs. Throughput
*   **Online Serving** prioritizes **latency**. When a user interacts with an application, the model must receive features and return a prediction in milliseconds. This requires indexed, row-based storage.
*   **Offline Serving** prioritizes **throughput**. When training a model on millions of rows, the system must deliver massive volumes of data efficiently. This requires partitioned, columnar storage.

### 2. Data Freshness
Online serving often requires "fresh" data (e.g., "number of clicks in the last 30 seconds"). Offline serving deals with historical "state" which may be updated in batches (e.g., daily or hourly).

### 3. The Dual-Store Architecture
To satisfy both requirements, a standard architecture utilizes a dual-store approach. A single feature pipeline typically writes to both an Online Store (for inference) and an Offline Store (for training). This ensures that the logic used to calculate the feature is identical for both paths.

## Standard Model
The standard model for feature serving involves a centralized **Feature Store** acting as the interface between data engineering and model execution.

1.  **Ingestion Layer:** Raw data is processed via streaming (for online) or batch (for offline) engines.
2.  **Storage Layer:**
    *   **Online Store:** Holds the "latest" value for every entity (e.g., User ID) for fast lookup.
    *   **Offline Store:** Holds the historical log of all feature values over time.
3.  **Serving Layer:**
    *   **Online API:** Accepts an entity ID and returns a vector of the most recent features.
    *   **Offline SDK:** Accepts a list of entity IDs and timestamps and returns a historical dataframe (Point-in-time Join).

## Common Patterns

### Batch-to-Online (The "Look-up" Pattern)
Features are calculated in a daily batch job (e.g., "Average spend over 30 days") and ingested into the Online Store. During inference, the model simply looks up this pre-computed value.

### Stream-to-Online (The "Real-time" Pattern)
Events are processed as they arrive (e.g., via a message bus). The feature value is updated in the Online Store immediately, allowing the model to react to events that happened seconds ago.

### On-Demand Transformation
Features that cannot be pre-computed (e.g., "Distance between user and current GPS coordinate") are calculated at the moment of the request using inputs provided directly by the application.

## Anti-Patterns

*   **Dual Pipeline Implementation:** Writing one piece of code for training data (SQL) and a different piece of code for online serving (Python/Java). This is the primary cause of training-serving skew.
*   **Online Scanning:** Attempting to perform aggregations (e.g., `SUM`, `AVG`) over millions of rows in the Online Store at request time. This leads to unacceptable latency.
*   **Ignoring Time-Travel:** Using the "current" value of a feature for historical training. This introduces data leakage, as the model "sees the future" during training.

## Edge Cases

*   **Cold Starts:** When a new entity (e.g., a new user) appears, the Online Store has no features. The system must have a strategy for default values or "warm-up" periods.
*   **Late-Arriving Data:** In streaming contexts, data may arrive out of order. The Offline Store must be able to reconcile these values to maintain point-in-time correctness.
*   **Large Feature Vectors:** When a model requires thousands of features, the network overhead of retrieving them from the Online Store can exceed the latency budget.

## Related Topics
*   **047 Feature Engineering:** The process of creating the features that are served.
*   **049 Training-Serving Skew:** The specific failure mode resulting from serving inconsistencies.
*   **050 Point-in-Time Joins:** The mechanism used in offline serving to ensure data integrity.
*   **MLOps:** The broader discipline of managing ML lifecycles.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
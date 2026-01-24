# 022 Gold Layer Design for ML

Canonical documentation for 022 Gold Layer Design for ML. This document defines concepts, terminology, and standard usage.

## Purpose
The Gold Layer for Machine Learning (ML) represents the final stage of data refinement within a multi-tier data architecture (often referred to as the Medallion Architecture). Its primary purpose is to provide "model-ready" data that is optimized for training, validation, and inference. 

While the Silver layer focuses on enterprise-wide consistency and normalization, the Gold layer is specialized for specific ML use cases. It addresses the critical need for feature engineering, label generation, and the enforcement of point-in-time correctness, ensuring that ML models are built on high-quality, reproducible, and non-leaked data.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   **Feature Engineering:** The transformation of raw or silver-tier data into predictive signals.
*   **Label Engineering:** The definition and generation of target variables for supervised learning.
*   **Temporal Integrity:** Standards for point-in-time joins and preventing data leakage.
*   **Data Formatting:** Optimization of data structures for ML frameworks (e.g., tensors, flattened tables).
*   **Metadata and Versioning:** Tracking the lineage and state of datasets used for specific model versions.

**Out of scope:**
*   **Specific vendor implementations:** (e.g., Databricks Feature Store, AWS SageMaker Feature Store, Google Vertex AI).
*   **Model Architecture:** The design of neural networks or algorithms themselves.
*   **Infrastructure Provisioning:** The underlying compute or storage hardware.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Feature** | An individual measurable property or characteristic of a phenomenon being observed, used as an input to an ML model. |
| **Label** | The ground truth or target variable that a model attempts to predict. |
| **Point-in-Time Correctness** | The practice of ensuring that features used for training reflect only the information available at the specific time the event occurred, preventing "look-ahead" bias. |
| **Data Leakage** | The inclusion of information in the training dataset that would not be available at the time of prediction, leading to unrealistically high performance metrics. |
| **Feature Store** | A centralized repository that stores, documents, and serves features for both training and online inference. |
| **Observation Window** | The specific timeframe from which features are extracted relative to an event. |
| **Inference Set** | The Gold-layer data structure used by a model during production deployment to generate predictions. |

## Core Concepts

### 1. Model-Readiness
Data in the Gold layer must be consumable by ML algorithms without further heavy lifting. This includes handling missing values (imputation), encoding categorical variables, and scaling numerical features.

### 2. Immutability and Versioning
Gold datasets used for training must be immutable. If a feature logic changes, a new version of the dataset or feature must be created. This ensures that any model deployment can be audited and reproduced by referencing the exact state of the Gold layer at the time of training.

### 3. Temporal Consistency
ML models are sensitive to time. The Gold layer must support "time-travel" or point-in-time joins, where features are joined to labels based on the timestamp of the observation, ensuring no future data is leaked into the past.

### 4. Feature Reusability
While Gold layers are often use-case specific, a well-designed Gold layer promotes the reuse of features across different models (e.g., a "customer_lifetime_value" feature used by both a churn model and a cross-sell model).

## Standard Model
The standard model for a Gold Layer for ML follows a structured pipeline from the Silver layer to the consumption tier:

1.  **Selection:** Identifying relevant entities and events from the Silver layer.
2.  **Transformation (Feature Engineering):** Applying aggregations (e.g., 30-day rolling average), encodings, and mathematical transformations.
3.  **Labeling:** Generating the target variable based on business outcomes (e.g., "did the user click?").
4.  **Point-in-Time Join:** Merging features and labels based on the event timestamp.
5.  **Materialization:** Storing the resulting dataset in a format optimized for high-throughput reading (e.g., Parquet for batch training) or low-latency lookup (e.g., Key-Value store for online inference).

## Common Patterns

### The Feature Store Pattern
Centralizing the Gold layer logic into a Feature Store. This ensures that the same feature logic is used for both training (offline) and serving (online), eliminating "training-serving skew."

### The Flattened Analytical Table (FAT)
Creating a wide, denormalized table where each row represents a single observation (an entity at a point in time) and each column represents a feature or label.

### The Sliding Window Pattern
Generating features based on fixed temporal windows (e.g., last 1 hour, last 24 hours, last 7 days) to capture trends and velocity.

## Anti-Patterns

*   **Online-Only Logic:** Defining feature transformations in the application code rather than the Gold layer, making it impossible to recreate the same features for offline training.
*   **Hardcoding Imputation:** Hardcoding mean or median values into the Gold layer pipeline rather than calculating them based on the training distribution.
*   **Leakage via Aggregation:** Calculating global averages (e.g., average price over the whole year) and using them as features for events that occurred mid-year.
*   **Overwriting Gold Tables:** Updating a Gold table in place without versioning, which destroys the ability to retrain or debug previous model versions.

## Edge Cases

### Cold Start Problem
When a new entity (e.g., a new user) enters the system, the Gold layer may lack historical data to generate features. The design must account for default values or "warm-up" periods.

### Late-Arriving Data
Data that arrives in the Silver layer after the Gold layer has already processed a specific time window. The Gold layer must have a strategy for either reprocessing or ignoring late data to maintain consistency.

### High-Cardinality Categoricals
Features like "User ID" or "Product SKU" can create millions of dimensions. The Gold layer design must decide between hashing, embedding, or grouping these values before materialization.

## Related Topics
*   **021 Silver Layer Data Normalization:** The prerequisite layer providing cleaned, enterprise-standard data.
*   **023 Feature Store Architecture:** The technical implementation of Gold layer serving.
*   **045 Model Evaluation Metrics:** How the quality of Gold layer data is indirectly measured through model performance.
*   **088 Data Lineage and Governance:** Tracking the flow from Raw to Gold.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
# 078 Data Quality for Inference

Canonical documentation for 078 Data Quality for Inference. This document defines concepts, terminology, and standard usage.

## Purpose
Data Quality for Inference addresses the integrity, reliability, and statistical validity of input data at the moment a machine learning model generates a prediction (inference). Unlike training-time data quality, which focuses on historical accuracy and label density, inference-time quality focuses on the immediate fitness of data for consumption by a live model.

The primary objective of this domain is to mitigate "Garbage In, Garbage Out" (GIGO) scenarios where degraded input data leads to silent failures, incorrect predictions, or systemic bias in production environments. It ensures that the operational environment mirrors the assumptions made during the model’s development phase.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   **Input Validation:** Verification of schema, types, and ranges at runtime.
*   **Statistical Consistency:** Detection of drift and skew between training and inference distributions.
*   **Systemic Reliability:** Handling of missing values, latency-induced data gaps, and upstream pipeline failures.
*   **Observability:** Monitoring and alerting frameworks for inference-time data health.

**Out of scope:**
*   **Training Data Quality:** The cleaning and labeling of historical datasets (except where used as a baseline for skew detection).
*   **Model Architecture:** The internal logic or weights of the neural network or algorithm.
*   **Hardware Performance:** GPU/CPU utilization metrics, unless they directly cause data corruption.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Training-Serving Skew** | A difference between the performance or data distribution of a model during training and its performance/distribution in production. |
| **Feature Drift** | A change in the statistical properties of independent variables over time, rendering the model less accurate. |
| **Schema Enforcement** | The process of ensuring that incoming inference requests strictly adhere to predefined data types, formats, and constraints. |
| **Imputation (Inference)** | The strategy of filling in missing input values at runtime using predefined defaults or statistical heuristics. |
| **Out-of-Distribution (OOD)** | Input data that significantly differs from the data the model was exposed to during training, making predictions unreliable. |
| **Data Integrity** | The accuracy, completeness, and consistency of data as it moves from the source to the inference endpoint. |

## Core Concepts

### 1. Temporal Relevance
Inference data is transient. Its quality is often tied to its "freshness." Data that was accurate five minutes ago (e.g., a stock price or sensor reading) may be invalid at the moment of inference. Quality frameworks must account for the timestamp of the feature relative to the inference request.

### 2. Distributional Alignment
A model is a mathematical representation of a specific data distribution. Data quality for inference requires that the "live" data remains within the bounds of the "learned" data. When the live distribution shifts (Concept Drift or Covariate Shift), the quality of the inference degrades, even if the data is technically "clean" (e.g., correct types and no nulls).

### 3. Structural Integrity
This refers to the physical characteristics of the data payload. It includes:
*   **Completeness:** Are all required features present?
*   **Conformity:** Does the data follow the expected format (e.g., ISO 8601 for dates)?
*   **Validity:** Do values fall within logical ranges (e.g., age cannot be negative)?

## Standard Model
The standard model for Inference Data Quality follows a three-stage gatekeeper architecture:

1.  **The Validation Gate (Pre-Inference):**
    *   Synchronous checks performed before the data reaches the model.
    *   Includes schema validation and range checking.
    *   Action: Reject request or apply default imputation.

2.  **The Comparison Layer (In-Inference):**
    *   Asynchronous or synchronous comparison of the current input against a "Reference Profile" (derived from training data).
    *   Calculates distance metrics (e.g., KL Divergence, Jensen-Shannon Distance).
    *   Action: Log warnings or trigger "Uncertainty" flags.

3.  **The Feedback Loop (Post-Inference):**
    *   Monitoring the relationship between the input data and the resulting prediction.
    *   Analyzing prediction distributions to infer upstream data issues.

## Common Patterns

### The "Circuit Breaker" Pattern
If the input data quality falls below a specific threshold (e.g., >20% features missing), the system trips a circuit breaker, bypassing the model and returning a safe default value or an error code rather than an unreliable prediction.

### The "Reference Baseline" Pattern
Maintaining a statistical summary (mean, standard deviation, histograms) of the training data. Every batch of inference data is compared against this baseline to detect sudden shifts in data behavior.

### Shadow Validation
Running a secondary data validation pipeline in parallel with the production pipeline. This allows for testing new quality constraints without impacting the latency of the primary inference path.

## Anti-Patterns

### Silent Failure
Allowing a model to generate a prediction on corrupted or missing data without logging a warning. This leads to downstream business logic making decisions based on "hallucinated" or low-confidence outputs.

### Hard-Coded Imputation
Using hard-coded zeros or "magic numbers" for missing values at inference time without considering if those values were present during training. This can push the input into a specific region of the feature space that triggers biased model behavior.

### Training-Only Validation
Assuming that because the data was cleaned during the ETL (Extract, Transform, Load) process for training, it will remain clean when coming from a live API or streaming source.

## Edge Cases

*   **Cold Start Features:** When a new entity (e.g., a new user) enters the system, they lack historical data. The inference quality must handle the transition from "missing data" to "populated data" without triggering false drift alerts.
*   **Adversarial Inputs:** Maliciously crafted data designed to look "clean" to a schema validator but intended to exploit model weaknesses.
*   **Upstream Schema Evolution:** When a source system changes its data format (e.g., changing a currency string to a float) without updating the inference service, leading to type-mismatch failures.

## Related Topics
*   **042 Model Monitoring:** The broader practice of observing model performance.
*   **105 Feature Stores:** Centralized repositories that often handle data quality and consistency between training and serving.
*   **012 Data Observability:** The general field of monitoring data pipelines.
*   **089 Drift Detection:** Specific algorithms used to identify statistical shifts.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
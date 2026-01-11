# 063 Alerts and Anomaly Detection

Canonical documentation for 063 Alerts and Anomaly Detection. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of Alerts and Anomaly Detection is to provide automated oversight of system health, performance, and security. In modern complex environments, manual monitoring of telemetry data is impossible due to the volume and velocity of information. This topic addresses the need for systems to autonomously identify deviations from expected behavior (anomalies) and communicate those findings to the appropriate stakeholders or automated response systems (alerts). 

The ultimate goal is to minimize the Mean Time to Detect (MTTD) and Mean Time to Resolve (MTTR) by transforming raw data into actionable intelligence.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Detection Logic:** The mathematical and logical frameworks used to identify outliers.
* **Notification Lifecycle:** The process of routing, escalating, and acknowledging signals.
* **Baseline Methodology:** How "normal" behavior is defined and maintained.
* **Signal Quality:** Strategies for reducing noise and ensuring alert relevance.

**Out of scope:**
* **Specific vendor implementations:** (e.g., specific configurations for Prometheus, Datadog, or Splunk).
* **Remediation Actions:** The specific technical steps taken to fix a system (e.g., specific code patches).
* **Data Storage:** The underlying database architecture used to store telemetry.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Anomaly** | A data point or pattern that deviates significantly from the established baseline or expected behavior. |
| **Alert** | A notification triggered by a specific condition or anomaly intended to prompt human or automated intervention. |
| **Baseline** | The historical or statistical representation of "normal" system behavior over a specific period. |
| **False Positive** | An alert triggered by a condition that does not actually represent a problem or a significant anomaly. |
| **False Negative** | A failure to trigger an alert when a significant anomaly or system failure has occurred. |
| **Threshold** | A predefined limit (upper or lower) that, when crossed, triggers a state change or alert. |
| **Seasonality** | Predictable fluctuations in data that occur at regular intervals (e.g., daily, weekly, or seasonally). |
| **Alert Fatigue** | The desensitization of responders due to a high volume of low-priority or irrelevant alerts. |
| **Flapping** | A state where a metric rapidly oscillates above and below a threshold, causing multiple redundant alerts. |

## Core Concepts

### Signal vs. Noise
The fundamental challenge of detection is distinguishing between "noise" (random, non-critical fluctuations) and "signals" (meaningful indicators of change). High-quality detection systems maximize signal-to-noise ratios to ensure that every alert is actionable.

### Deterministic vs. Probabilistic Detection
*   **Deterministic:** Based on fixed rules (e.g., "If CPU > 90% for 5 minutes"). These are predictable but brittle in dynamic environments.
*   **Probabilistic:** Based on statistical models or machine learning (e.g., "If traffic is 3 standard deviations from the 30-day mean"). these are adaptable but can be harder to interpret.

### Severity Levels
Alerts must be categorized by their impact on the business or system. Standard tiers include:
*   **Critical/P1:** Immediate threat to core services; requires instant response.
*   **Warning/P2:** Potential issue or degraded performance; requires investigation during business hours.
*   **Info/P3:** Notable change in state; no immediate action required, used for context.

## Standard Model

The standard model for Alerts and Anomaly Detection follows a linear pipeline:

1.  **Data Ingestion:** Collection of metrics, logs, or traces from the target system.
2.  **Observation:** The system monitors the data stream against defined parameters.
3.  **Analysis/Detection:** 
    *   *Static Analysis:* Comparison against fixed thresholds.
    *   *Dynamic Analysis:* Comparison against a calculated baseline (Anomaly Detection).
4.  **Evaluation (Suppression/Grouping):** The system checks if the alert should be suppressed (e.g., during a maintenance window) or grouped with existing related alerts to prevent duplication.
5.  **Notification:** Delivery of the alert to the appropriate medium (Email, SMS, Chat, Webhook).
6.  **Incident Lifecycle:** Acknowledgment, investigation, resolution, and post-mortem.

## Common Patterns

### Static Thresholding
The simplest form of alerting where a notification is sent when a metric crosses a hard-coded value. Best used for "known-bad" states (e.g., Disk Full).

### Moving Averages (Smoothing)
Using a rolling window of data to calculate an average, which reduces the impact of brief, non-critical spikes.

### Holt-Winters / Exponential Smoothing
A statistical pattern that accounts for both trends and seasonality in data, making it highly effective for business metrics (e.g., login rates, transaction volumes).

### Heartbeat/Dead Man's Switch
A pattern where the *absence* of a signal triggers an alert. This is critical for detecting system total failures where the monitoring agent itself might be offline.

## Anti-Patterns

*   **Threshold Smearing:** Setting thresholds so wide that they never trigger, or so narrow that they trigger constantly.
*   **Alerting on Symptoms, Not Causes:** Alerting on high CPU (symptom) when the actual issue is a slow database query (cause).
*   **Lack of Actionability:** Sending an alert that does not include context or instructions on how to resolve the issue.
*   **Global Baselines:** Applying the same anomaly detection parameters to all services regardless of their unique traffic patterns.
*   **The "Boy Who Cried Wolf":** Allowing low-priority alerts to go unacknowledged, which trains responders to ignore high-priority alerts.

## Edge Cases

### Cold Starts
When a new service is deployed, there is no historical data to build a baseline. Anomaly detection may be unreliable or overly sensitive during this "learning phase."

### Step Changes (The New Normal)
A permanent shift in system behavior (e.g., a successful marketing campaign doubling baseline traffic). The detection system must be able to "re-baseline" quickly to avoid continuous false positives.

### Black Swan Events
Rare, unpredictable events that fall outside of statistical models. Anomaly detection may identify these, but the lack of historical precedent makes automated response difficult.

### Clock Drift
In distributed systems, time synchronization issues can lead to "out-of-order" data, which can confuse time-series-based anomaly detection algorithms.

## Related Topics
*   **042 Observability and Telemetry:** The underlying data sources for detection.
*   **089 Incident Management:** The process that follows a triggered alert.
*   **112 Service Level Objectives (SLOs):** The business-level targets that define what thresholds should be.
*   **154 Machine Learning Operations (MLOps):** For advanced anomaly detection model maintenance.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
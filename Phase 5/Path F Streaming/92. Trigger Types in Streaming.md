# 092 Trigger Types in Streaming

Canonical documentation for 092 Trigger Types in Streaming. This document defines concepts, terminology, and standard usage.

## Purpose
In stream processing, data is theoretically infinite. To perform aggregations or produce discrete outputs, the system must determine exactly when to materialize the results of a computation. Trigger types define the temporal or logical conditions under which a window of data is "fired" or emitted. 

The purpose of the 092 Trigger classification is to provide a standardized framework for balancing the trade-offs between **latency** (how quickly a result is seen), **completeness** (how much of the data is included), and **cost** (the computational overhead of frequent updates).

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Logical classification of trigger mechanisms (Time, Count, Data-driven).
* The relationship between triggers, watermarks, and windowing.
* Accumulation modes and their impact on triggered output.
* Theoretical boundaries of non-deterministic triggering.

**Out of scope:**
* Specific API syntax for commercial streaming engines (e.g., Apache Flink, Google Cloud Dataflow, Spark Streaming).
* Physical storage layer implementation of state backends.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Trigger** | A condition that, when met, signals the system to produce an output based on the current state of a window. |
| **Watermark** | A moving threshold that represents the system's progress in event time, signaling that no data with a timestamp earlier than the watermark is expected to arrive. |
| **Pane** | A discrete version of a window’s result produced by a single trigger firing. A window may consist of multiple panes. |
| **Event Time** | The time at which an event actually occurred on the producer side. |
| **Processing Time** | The time at which an event is observed by the stream processing system. |
| **Lateness** | The delta between the event time of a record and the current watermark. |
| **Accumulation** | The strategy used to handle state after a trigger fires (e.g., discarding state vs. accumulating it for the next pane). |

## Core Concepts
The 092 Trigger model operates on the principle that "When" results are produced is independent of "How" those results are calculated.

### The Triggering Lifecycle
A trigger governs the transition of data from an internal state to an externalized result. This lifecycle is typically divided into three phases:
1.  **Early Firings:** Results produced before the watermark reaches the end of the window, primarily used to decrease latency.
2.  **On-time Firings:** The primary result produced when the watermark passes the window boundary, representing the "complete" result based on the system's heuristic.
3.  **Late Firings:** Results produced after the watermark has passed the window boundary, accounting for data that arrived out of order.

### Determinism vs. Non-Determinism
Triggers based on **Event Time** are deterministic relative to the data stream; given the same data and watermark logic, the output remains consistent. Triggers based on **Processing Time** or **Element Count** are non-deterministic, as they depend on the arrival velocity and system clock of the processing cluster.

## Standard Model
The standard model for 092 Trigger Types categorizes triggers into four primary functional groups:

### 1. Time-Based Triggers
*   **Event-Time Triggers:** Fire based on the progression of the watermark. These are the most common for ensuring correctness in temporal aggregations.
*   **Processing-Time Triggers:** Fire based on the wall-clock time of the processing engine. Useful for regular heartbeats or monitoring.

### 2. Data-Driven Triggers
*   **Element-Count Triggers:** Fire after a specific number of records have been observed within a window.
*   **Delta Triggers:** Fire when the difference between the current element and the last triggered element exceeds a predefined threshold (e.g., a price change of > 5%).

### 3. Composite Triggers
*   **OR (At Least One):** Fires when any of the sub-triggers meet their condition.
*   **AND (All):** Fires only when all sub-triggers have met their conditions.
*   **Repeated:** Re-arms a trigger after it fires, allowing for multiple panes within a single window.

### 4. Special Triggers
*   **Never:** Data is only materialized when the window is explicitly closed by the system (often used in batch-fallback scenarios).
*   **Always:** Fires after every single element.

## Common Patterns
*   **The "Speculative" Pattern:** Using an early trigger (Processing Time) combined with an on-time trigger (Event Time). This allows a dashboard to show "rough" data immediately, which is then corrected when the watermark arrives.
*   **The "Final-Only" Pattern:** Using only the on-time trigger and discarding any late data. This is used when the cost of updating downstream systems outweighs the value of incremental accuracy.
*   **The "Continuous Update" Pattern:** Using a repeated element-count trigger of 1. This transforms a windowed aggregation into a continuous stream of updates.

## Anti-Patterns
*   **Over-Triggering:** Setting a processing-time trigger with too high a frequency (e.g., every 1ms). This can saturate downstream sinks and cause backpressure.
*   **Unbounded Lateness without Discarding:** Allowing late triggers to fire indefinitely without a "maximum lateness" bound, leading to infinite state growth.
*   **Relying on Count Triggers for Correctness:** Using element-count triggers for financial or audit-grade reporting, as counts are non-deterministic in the event of system failure and retry.

## Edge Cases
*   **Empty Windows:** If a trigger is set to fire based on a watermark, but no data arrived for that window, the system must decide whether to emit an empty pane or suppress the output entirely.
*   **Late Data after State Cleanup:** If data arrives for a window after the "allowed lateness" period has expired and the state has been purged, the trigger will typically ignore the data, or it must be handled by a side-output.
*   **Watermark Stalls:** If an upstream partition stops sending data, the watermark may stop advancing. Event-time triggers will cease to fire, potentially causing a "hang" in the pipeline unless a processing-time "idle" timeout is implemented.

## Related Topics
*   **084 Windowing Strategies:** Defines the temporal boundaries of data sets.
*   **102 Watermark Generation:** Defines how the system calculates the progress of event time.
*   **115 State Management:** How the system stores data between trigger firings.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
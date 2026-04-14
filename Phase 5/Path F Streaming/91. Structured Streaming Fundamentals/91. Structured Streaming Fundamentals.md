# 091 Structured Streaming Fundamentals

Canonical documentation for 091 Structured Streaming Fundamentals. This document defines concepts, terminology, and standard usage.

## Purpose
Structured Streaming exists to provide a high-level, declarative framework for building end-to-end streaming applications. It addresses the historical fragmentation between batch and real-time processing by treating a live data stream as a table that is being continuously appended. This model allows developers to apply the same relational logic to streaming data that they would apply to static data, ensuring consistency, fault tolerance, and simplified reasoning about time-based data.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* The conceptual model of streams as unbounded tables.
* Fault tolerance mechanisms (checkpointing and write-ahead logs).
* Time-handling semantics (Event Time, Processing Time, and Watermarking).
* Output modes and delivery guarantees.
* Stateful vs. stateless processing logic.

**Out of scope:**
* Specific vendor-specific API syntax (e.g., specific Spark, Flink, or Kafka Streams code).
* Hardware-level performance tuning.
* Specific connector configurations for third-party messaging systems.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Unbounded Table** | A conceptual representation of a data stream where new data is treated as new rows appended to an infinitely growing table. |
| **Event Time** | The timestamp embedded within the data itself, representing when the event actually occurred at the source. |
| **Processing Time** | The time at which a specific data record is processed by the streaming engine. |
| **Watermark** | A moving threshold used to define how long the system waits for late-arriving data before discarding it or finalizing a state. |
| **Checkpointing** | The process of persisting the current state and metadata of a stream to reliable storage to enable recovery from failures. |
| **Trigger** | A policy that defines the timing of data processing (e.g., fixed intervals, one-time execution, or continuous). |
| **Stateful Transformation** | An operation that requires information from previous events to process the current event (e.g., aggregations, joins). |
| **Idempotency** | The property of an operation where multiple executions produce the same result as a single execution, critical for exactly-once semantics. |

## Core Concepts

### The Unbounded Table Model
The fundamental premise of structured streaming is that a stream is not a sequence of discrete messages, but a table to which data is continuously appended. Every query on a stream is logically an incremental query that runs against this unbounded table.

### Incremental Execution
The engine is responsible for determining what new data has arrived since the last trigger and incrementally updating the result. This eliminates the need for the user to manually manage offsets or state transitions.

### Handling Latency and Late Data
Structured streaming distinguishes between the arrival of data and the time the data was generated. By using **Event Time**, the system can correctly categorize data even if it arrives out of order due to network latency or source delays.

### Fault Tolerance and Reliability
To ensure "Exactly-Once" semantics, the system relies on two pillars:
1.  **Checkpoints:** Storing the current offset range and state.
2.  **Write-Ahead Logs (WAL):** Recording the intent of an operation before execution.
3.  **Replayable Sources and Idempotent Sinks:** The ability to re-read data and the ability to write data multiple times without side effects.

## Standard Model

The standard model for structured streaming follows a linear pipeline:

1.  **Input Source:** An external system (e.g., a message queue or file system) that provides a continuous flow of data.
2.  **Incremental Query:** A declarative logic (SQL or DataFrame-like) applied to the input.
3.  **Trigger:** A mechanism that decides when to execute the next incremental batch.
4.  **Output Sink:** An external system where the processed results are persisted.
5.  **Output Mode:** Defines what is written to the sink:
    *   **Append Mode:** Only new rows added to the result table since the last trigger.
    *   **Update Mode:** Only the rows that were updated in the result table since the last trigger.
    *   **Complete Mode:** The entire result table is rewritten to the sink.

## Common Patterns

### Windowed Aggregations
Grouping data into time-based buckets (e.g., "count events every 5 minutes").
*   **Tumbling Windows:** Fixed-size, non-overlapping intervals.
*   **Sliding Windows:** Overlapping intervals (e.g., "every 5 minutes, look at the last 10 minutes").

### Stream-Static Joins
Enriching a live stream by joining it with a static reference table (e.g., joining a stream of transactions with a static table of user metadata).

### Stream-Stream Joins
Joining two live streams based on a common key and a shared time constraint. This requires the system to buffer state for both streams until a match is found or a watermark is passed.

## Anti-Patterns

*   **Unbounded State Growth:** Failing to define watermarks on stateful operations, leading to the system keeping data in memory indefinitely and eventually crashing (OOM).
*   **Non-Deterministic Logic:** Using functions that produce different results on retry (e.g., `current_timestamp()` or random number generators) within the streaming logic, which breaks recovery consistency.
*   **Over-Frequent Triggers:** Setting trigger intervals shorter than the time required to process a batch, leading to "queueing" and increased latency.
*   **Ignoring Sink Idempotency:** Writing to a sink that does not support idempotent writes while expecting exactly-once results.

## Edge Cases

*   **Late Data Past Watermark:** Data arriving with an event time older than the current watermark is typically dropped. Systems must be monitored for high rates of dropped data.
*   **Schema Evolution:** When the structure of the input data changes (e.g., a new field is added). The streaming engine must have a policy for handling or ignoring these changes without failing the pipeline.
*   **Restart with Logic Changes:** Changing the processing logic (e.g., changing an aggregation formula) and restarting from an existing checkpoint can lead to state incompatibility or inconsistent results.
*   **Clock Skew:** Significant differences between the clocks of different data producers can interfere with watermark accuracy.

## Related Topics
*   088 Data Partitioning Strategies
*   092 Message Queue Integration
*   104 Distributed State Management
*   115 Exactly-Once Delivery Semantics

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
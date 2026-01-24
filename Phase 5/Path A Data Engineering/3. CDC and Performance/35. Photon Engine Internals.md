# 035 Photon Engine Internals

Canonical documentation for 035 Photon Engine Internals. This document defines concepts, terminology, and standard usage.

## Purpose
The 035 Photon Engine represents a specialized execution layer designed to maximize computational throughput by aligning software execution patterns with modern hardware architectures. It addresses the performance bottleneck traditionally found in interpreted execution engines—specifically the overhead of virtual machine dispatch, poor memory locality, and underutilization of CPU instruction sets.

The engine exists to provide a high-performance, vectorized execution environment that translates logical query plans into hardware-native instructions, ensuring that data processing speeds are limited by hardware bandwidth rather than software abstraction layers.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Vectorized Execution:** The mechanics of processing batches of data rather than individual rows.
* **Hardware Abstraction:** How the engine interacts with SIMD (Single Instruction, Multiple Data) and memory hierarchies.
* **Memory Management:** Internal memory layout and buffer handling strategies.
* **Execution Primitives:** The fundamental building blocks of the engine’s computational logic.

**Out of scope:**
* **Specific Vendor Implementations:** Proprietary optimizations specific to individual cloud service providers.
* **Storage Layer Specifics:** The physical format of data on disk (e.g., Parquet, ORC), except where it interfaces with memory buffers.
* **Query Optimization:** The high-level logical planning (CBO/RBO) that occurs before the plan reaches the execution engine.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Vectorization** | A computing method where a single instruction is applied to a batch (vector) of data simultaneously. |
| **SIMD** | Single Instruction, Multiple Data; a class of parallel computing that allows one instruction to process multiple data points. |
| **Columnar Batch** | A memory-resident structure where data is organized by column rather than by row to facilitate contiguous memory access. |
| **Kernel** | A highly optimized, low-level function designed to perform a specific operation (e.g., addition, comparison) on a vector. |
| **Late Materialization** | The practice of delaying the joining or fetching of full rows until the latest possible stage in the execution pipeline. |
| **Instruction Cache** | A specialized cache used by the CPU to store frequently used instructions, which the engine aims to keep "hot." |

## Core Concepts

### 1. Vectorized Processing Model
Unlike traditional "Volcano" iteration models that process one row at a time (next() calls), the 035 Photon Engine operates on batches of data. This reduces the number of function calls and allows the CPU to stay within tight loops, significantly improving instruction cache hits.

### 2. Hardware-Conscious Design
The engine is architected to be "hardware-aware." This involves:
* **Data Alignment:** Ensuring data structures align with CPU cache lines (typically 64 bytes).
* **Branch Minimization:** Using predication and bitmasking to avoid CPU branch mispredictions, which are costly in high-speed pipelines.
* **Pipelining:** Organizing operations so that the CPU can overlap the execution of multiple instructions.

### 3. Native Execution
The engine bypasses intermediate runtimes (such as the JVM or Python interpreter) for the actual data processing. By using a native language (typically C++ or Rust), the engine gains direct control over memory allocation and assembly-level optimizations.

## Standard Model
The standard model for the 035 Photon Engine follows a three-stage lifecycle:

1.  **Plan Translation:** The logical query plan is decomposed into a series of native operators.
2.  **Vectorized Pipeline Construction:** Operators are linked into a pipeline where data flows in columnar batches. Each operator consumes one or more input vectors and produces an output vector.
3.  **Kernel Execution:** The engine selects the most efficient hardware kernel based on the CPU's capabilities (e.g., AVX-512, NEON) to execute the operation.

## Common Patterns

### Predicate Pushdown (Vectorized)
Filtering data as early as possible. In the 035 model, this is done by generating a "selection vector" (a bitmask) that indicates which rows in a batch satisfy a condition, avoiding the need to copy data.

### Hash Aggregation
Using specialized hash table implementations that fit within the L2/L3 cache. The engine uses linear probing or other cache-friendly collision resolution strategies to maintain high throughput during "Group By" operations.

### Zero-Copy Buffering
Where possible, the engine passes pointers to memory buffers between operators rather than copying the underlying data, minimizing memory bus saturation.

## Anti-Patterns

* **Row-Based UDFs:** Introducing User Defined Functions that operate on a row-by-row basis breaks the vectorized pipeline and forces the engine to "spill" back to an interpreted state.
* **Excessive Branching:** Writing logic with complex `if-else` structures inside the inner loops of a kernel, which defeats the purpose of SIMD.
* **Small Batch Sizes:** Processing batches that are too small to amortize the overhead of the operator call or too large to fit in the CPU cache.

## Edge Cases

* **Data Skew:** When a specific key or value dominates a dataset, it can lead to uneven distribution in hash-based operators. The engine must implement "skew-join" logic or fallback mechanisms to prevent single-thread bottlenecks.
* **Null Handling in SIMD:** SIMD instructions often do not have native "null" concepts. The engine must manage nullability via separate bit-vectors, which adds complexity to the kernel logic.
* **Type Coercion:** Performing operations between mismatched types (e.g., Float64 and Int32) requires explicit "widening" or "narrowing" kernels, which can introduce overhead if not handled at the batch level.

## Related Topics
* **034 Query Optimization:** The layer that precedes the Photon Engine.
* **036 Memory Management Framework:** The system governing the allocation of buffers used by the engine.
* **SIMD Instruction Sets (AVX/SSE/NEON):** The hardware-level specifications the engine targets.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
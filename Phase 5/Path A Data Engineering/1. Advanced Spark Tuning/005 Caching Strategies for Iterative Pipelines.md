# 005 Caching Strategies for Iterative Pipelines

Canonical documentation for 005 Caching Strategies for Iterative Pipelines. This document defines concepts, terminology, and standard usage.

## Purpose
Iterative pipelines—common in machine learning, data engineering, and continuous integration—frequently execute repetitive tasks where only a subset of inputs or logic changes between runs. Without effective caching, these pipelines incur a "recomputation tax," leading to excessive latency, wasted computational resources, and increased operational costs.

The purpose of caching in iterative pipelines is to persist intermediate state and computational artifacts so that subsequent executions can skip redundant work. This documentation establishes a framework for determining when to cache, how to identify reusable artifacts, and how to maintain cache integrity.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   **Artifact Persistence:** Logic for storing and retrieving intermediate outputs.
*   **Invalidation Logic:** Criteria for determining when a cached item is no longer valid.
*   **Dependency Tracking:** Mapping inputs, code versions, and environments to specific cache entries.
*   **State Management:** Handling the transition between cached and computed states.

**Out of scope:**
*   **Specific Storage Backends:** (e.g., S3, Redis, local filesystem implementation details).
*   **Hardware-level Caching:** (e.g., CPU L1/L2/L3 caches).
*   **Network Protocol Optimization:** (e.g., CDN edge caching).

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Artifact** | Any discrete output produced by a pipeline stage (e.g., a compiled binary, a transformed dataset, a trained model). |
| **Cache Key** | A unique identifier, typically a cryptographic hash, derived from all inputs, configurations, and code versions that influence an artifact. |
| **Determinism** | The property where a pipeline stage, given the exact same inputs and environment, consistently produces the identical output. |
| **Downstream Consumer** | A pipeline stage that relies on the output of a previous (upstream) stage. |
| **Hit/Miss Ratio** | The frequency with which requested artifacts are found in the cache versus the frequency with which they must be recomputed. |
| **Invalidation** | The process of marking a cached artifact as obsolete or deleting it because its underlying inputs have changed. |
| **Side Effect** | An unintended or external change caused by a pipeline stage (e.g., a database write) that is not captured in the cached artifact. |

## Core Concepts

### The Determinism Requirement
Caching is only safe when a pipeline stage is deterministic. If a stage relies on non-deterministic factors (e.g., current system time, random seeds without fixed initialization, or unversioned external APIs), the cache may return an artifact that does not accurately represent the current state of the system.

### Content-Addressable Storage (CAS)
A standard approach where artifacts are stored based on their content (or the hash of their inputs) rather than a fixed filename or location. This ensures that if two different iterations produce the same result, they share the same cache entry.

### Dependency Graph Awareness
Iterative pipelines are represented as Directed Acyclic Graphs (DAGs). A caching strategy must be "graph-aware," meaning that if an upstream node's cache is invalidated, all descendant (downstream) nodes must also be re-evaluated, regardless of whether their specific logic changed.

## Standard Model

The standard model for caching in iterative pipelines follows the **Hash-Check-Execute-Store** cycle:

1.  **Key Generation:** Collect all inputs (data, parameters, environment variables, and code version) for a specific stage and generate a unique **Cache Key**.
2.  **Lookup:** Query the cache repository for the existence of the Cache Key.
3.  **Decision Branch:**
    *   **Cache Hit:** Retrieve the artifact and skip the execution of the stage.
    *   **Cache Miss:** Execute the stage logic.
4.  **Persistence:** Upon successful execution, store the resulting artifact in the cache associated with the Cache Key for future use.

## Common Patterns

### Layered Caching
Common in containerization and build systems. Each step in a sequence is cached independently. If step 2 changes, step 1 remains cached, but step 2 and all subsequent steps are recomputed.

### Partial Result Caching
In large-scale data processing, a single stage may be broken into "chunks." Caching occurs at the chunk level, allowing a pipeline to resume from the exact point of failure or to only process new data increments.

### Metadata-Only Caching
Instead of storing the entire artifact, the system stores only the metadata or a "fingerprint" of the result. This is used when the cost of transferring the artifact from the cache exceeds the cost of recomputing it, but the system still needs to verify if the state has changed.

### Speculative Execution
The pipeline begins computing a stage while simultaneously checking the cache. If a cache hit is confirmed, the computation is aborted. This is used in high-performance environments where latency is more critical than compute cost.

## Anti-Patterns

### Over-Caching (The "Hoarding" Problem)
Caching every intermediate step without regard for the size of the artifact or the cost of recomputation. This leads to "cache bloat," where storage costs and retrieval latency outweigh the benefits of skipping computation.

### Non-Deterministic Keys
Generating a cache key that excludes critical dependencies (e.g., failing to include the version of a library used in a transformation). This results in "poisoned" cache hits where the system returns an incorrect, stale artifact.

### Manual Invalidation
Relying on human intervention to "clear the cache" when logic changes. A robust iterative pipeline should automatically invalidate entries based on changes to the input hash.

### Caching Side Effects
Attempting to cache a stage that performs external actions (like sending an email or updating a production database). Since the cache skip bypasses the execution, the side effect will not occur on a cache hit.

## Edge Cases

### Hash Collisions
While statistically rare with modern algorithms (e.g., SHA-256), a hash collision would result in the system retrieving the wrong artifact. Canonical systems must assume a negligible probability or implement secondary validation.

### Environment Drift
A pipeline may produce different results on two different machines despite having the same code and data (e.g., different CPU architectures or OS-level libraries). If the environment is not part of the Cache Key, the cache becomes unreliable across a distributed fleet.

### Corrupted Artifacts
An artifact may be successfully stored but become corrupted in the cache storage. Systems should implement checksum verification upon retrieval to ensure the integrity of the cached data.

### Race Conditions in Parallel Pipelines
Two identical pipeline instances running simultaneously may both experience a cache miss and attempt to write to the same cache key at once. Implementation must handle atomic writes or "first-writer-wins" logic.

## Related Topics
*   **001 Pipeline Orchestration:** The high-level management of DAGs.
*   **012 Data Lineage:** Tracking the origin and transformations of data.
*   **024 Reproducibility in Computation:** Standards for ensuring consistent outputs.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
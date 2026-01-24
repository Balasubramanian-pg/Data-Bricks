# 059 Serverless vs Pro vs Classic

Canonical documentation for 059 Serverless vs Pro vs Classic. This document defines concepts, terminology, and standard usage.

## Purpose
The 059 classification system exists to categorize computational resource management and deployment models based on their abstraction level, scaling mechanics, and operational responsibility. This framework addresses the problem of aligning technical architecture with organizational requirements regarding cost, performance predictability, and maintenance overhead.

By distinguishing between Serverless, Pro (Managed/Dedicated), and Classic (Infrastructure-centric) models, stakeholders can make informed decisions about where a workload should reside based on its lifecycle stage and performance characteristics.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   Architectural paradigms for resource allocation.
*   Theoretical boundaries of scaling and state management.
*   Operational responsibility shifts between the provider and the consumer.
*   Cost-efficiency models relative to utilization.

**Out of scope:**
*   Specific vendor product names or pricing tiers.
*   Step-by-step migration tutorials.
*   Language-specific runtime optimizations.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Classic** | A model characterized by persistent, fixed-capacity infrastructure where the user manages the operating system, scaling, and resource lifecycle. |
| **Pro (Professional)** | A managed service model providing dedicated, pre-provisioned resources with high predictability and reduced operational overhead, often including advanced features like high availability and specialized tuning. |
| **Serverless** | An event-driven execution model where the provider dynamically manages the allocation and provisioning of servers, scaling automatically from zero to peak demand. |
| **Cold Start** | The latency incurred in a Serverless model when an environment must be initialized before executing a request. |
| **Provisioned Capacity** | The baseline amount of resources reserved in a Pro or Classic model to ensure immediate availability. |
| **Ephemeral Compute** | Short-lived execution environments that do not persist state between invocations, typical of Serverless. |

## Core Concepts

### Abstraction of Infrastructure
The primary differentiator between these models is the "Line of Responsibility." 
*   In **Classic**, the line is low; the user manages almost everything above the hardware. 
*   In **Pro**, the line is mid-level; the provider manages the platform, but the user manages the capacity and configuration. 
*   In **Serverless**, the line is high; the user manages only the application logic and triggers.

### Scaling Mechanics
*   **Classic:** Manual or reactive scaling (adding/removing discrete units of infrastructure).
*   **Pro:** Predictive or scheduled scaling of dedicated resource pools.
*   **Serverless:** Transparent, request-based scaling that responds to instantaneous demand.

### State Management
*   **Classic/Pro:** Generally support stateful applications where data can be stored locally on the persistent disk of the compute unit.
*   **Serverless:** Inherently stateless. Any persistence must be offloaded to external storage or database layers.

## Standard Model
The standard model for selecting a tier is based on the **Workload Profile**:

1.  **Serverless (The Utility Model):** Best for asynchronous tasks, event-driven architectures, and workloads with highly variable or unpredictable traffic. It optimizes for "Time to Market" and "Cost of Idle."
2.  **Pro (The Performance Model):** Best for production-grade applications requiring consistent latency, high throughput, and dedicated resource isolation. It optimizes for "Predictability" and "Sustained Load."
3.  **Classic (The Control Model):** Best for legacy migrations, specialized kernel requirements, or scenarios where the user requires total transparency of the underlying stack. It optimizes for "Customization" and "Fixed-Cost Budgeting."

## Common Patterns

### The Hybrid Lifecycle
An application begins in **Serverless** during the MVP phase to minimize costs. As traffic stabilizes and grows, it is migrated to a **Pro** tier to achieve better unit economics and eliminate cold-start latencies.

### The Burst Buffer
A core application runs on **Pro** or **Classic** infrastructure to handle the baseline load, while **Serverless** functions are utilized to handle sudden, massive spikes in traffic that exceed the baseline capacity.

### Scheduled Batch Processing
Using **Classic** or **Pro** for long-running, compute-intensive batch jobs that would exceed the timeout limits or cost-efficiency thresholds of a Serverless environment.

## Anti-Patterns

### The "Always-On" Serverless
Running a constant, high-volume stream of requests through a Serverless model. This is often significantly more expensive than a Pro or Classic model where the cost per request is lower at high utilization.

### The "Micro-Managed" Classic
Using a Classic model for a simple, low-traffic API. The operational overhead of patching, securing, and monitoring the server outweighs the benefits of the control it provides.

### State Storage in Serverless
Attempting to store session data or temporary files on the local file system of a Serverless function, assuming it will be available for the next request.

## Edge Cases

### Long-Running Executions
Serverless environments typically have a hard timeout (e.g., 15 minutes). Processes that require hours of continuous computation must use Pro or Classic models.

### Specialized Hardware Requirements
Workloads requiring specific GPU architectures, FPGA access, or legacy hardware instructions often cannot run in Serverless or Pro environments and must default to Classic.

### Extreme Low-Latency Requirements
Applications where even a 100ms cold start is unacceptable (e.g., high-frequency trading or real-time gaming) are unsuitable for Serverless and require the pre-warmed, dedicated nature of Pro or Classic.

## Related Topics
*   **012 Event-Driven Architecture:** The primary design pattern for Serverless.
*   **045 Infrastructure as Code (IaC):** Essential for managing Classic and Pro tiers at scale.
*   **088 Cost Optimization:** The financial framework for choosing between these tiers.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
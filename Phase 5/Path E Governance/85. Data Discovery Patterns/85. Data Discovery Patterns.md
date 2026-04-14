# 085 Data Discovery Patterns

Canonical documentation for 085 Data Discovery Patterns. This document defines concepts, terminology, and standard usage.

## Purpose
Data Discovery Patterns address the fundamental challenge of information asymmetry within modern data ecosystems. As organizations scale, the volume, velocity, and variety of data often outpace the ability of human actors to maintain an accurate inventory of available assets. 

The purpose of these patterns is to provide a structured framework for identifying, classifying, and understanding data assets across heterogeneous environments. By establishing standardized discovery patterns, organizations can reduce "time-to-insight," ensure regulatory compliance (e.g., identifying PII), and prevent the formation of "data swamps" where assets exist but remain inaccessible or untrusted.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Metadata Extraction:** Mechanisms for harvesting structural and descriptive metadata.
* **Classification Logic:** Patterns for identifying data types and sensitivity levels.
* **Relationship Mapping:** Identifying connections between disparate data entities.
* **Discovery Lifecycle:** The stages from initial ingestion to active cataloging.

**Out of scope:**
* **Specific vendor implementations:** (e.g., specific configurations for Collibra, Alation, or AWS Glue).
* **Data Storage Technologies:** The physical layer of how data is stored (e.g., Parquet vs. Avro).
* **Data Visualization:** The end-user reporting layer, except where it pertains to metadata exploration.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Metadata** | Data that provides information about other data, categorized into structural, descriptive, and administrative types. |
| **Data Profiling** | The statistical analysis and assessment of data values within a dataset to determine quality, integrity, and characteristics. |
| **Data Lineage** | The lifecycle of data that includes its origins, transformations, and ultimate destination. |
| **Semantic Discovery** | The process of identifying the meaning and context of data beyond its technical schema (e.g., recognizing a string is a "Customer ID"). |
| **Crawler/Scanner** | An automated process that traverses data sources to extract metadata and structural information. |
| **Fingerprinting** | The creation of a unique mathematical representation of a dataset's characteristics to identify duplicates or similar sets. |

## Core Concepts
The 085 Data Discovery framework relies on three fundamental pillars:

1.  **Visibility:** The ability to see that a data asset exists, regardless of its physical location (Cloud, On-premise, SaaS).
2.  **Contextualization:** Moving beyond technical schemas to understand the business relevance, ownership, and usage constraints of the data.
3.  **Automation:** The use of programmatic methods to maintain the discovery index, as manual inventory is insufficient for modern data scales.

## Standard Model
The standard model for Data Discovery follows a four-stage cyclical process:

1.  **Ingestion/Scanning:** Connectors interface with data sources (Databases, File Systems, APIs) to pull raw technical metadata.
2.  **Profiling & Analysis:** The system performs statistical analysis (min/max, null counts, distribution) and pattern matching (Regex, ML models) to understand the data's shape.
3.  **Classification & Tagging:** Data is mapped to a business glossary or taxonomy. Sensitivity labels (e.g., "Public," "Confidential," "PII") are applied here.
4.  **Persistence (The Catalog):** The enriched metadata is stored in a centralized, searchable repository (The Data Catalog) for consumption by data citizens.

## Common Patterns

### 1. The Pull-Based Crawler Pattern
A centralized discovery engine initiates connections to registered data sources on a scheduled basis. 
*   **Best for:** Legacy databases and file systems that do not emit events.
*   **Trade-off:** Can put a load on production systems during the scan.

### 2. The Push-Based Event Pattern
Data sources or ETL pipelines emit metadata events (e.g., "Table Created," "Schema Updated") to a central metadata bus.
*   **Best for:** Modern, event-driven architectures and real-time schema changes.
*   **Trade-off:** Requires sources to be "discovery-aware."

### 3. Sampling-Based Profiling
Instead of scanning entire multi-terabyte tables, the discovery engine analyzes a statistically significant sample to infer data types and quality.
*   **Best for:** Large-scale Data Lakes.
*   **Trade-off:** May miss "long-tail" anomalies or rare PII instances.

### 4. Social/Crowdsourced Discovery
Enriching technical metadata with human knowledge (tags, ratings, descriptions) provided by the actual users of the data.
*   **Best for:** Capturing tribal knowledge and business context.
*   **Trade-off:** Subject to human error and inconsistent participation.

## Anti-Patterns

*   **The Data Graveyard:** Collecting metadata for every possible asset without a strategy for curation or usage, leading to a "noisy" catalog that users ignore.
*   **Manual-Only Inventory:** Relying on spreadsheets or manual entry for data documentation. This is obsolete the moment it is completed.
*   **Siloed Discovery:** Implementing discovery tools within individual departments that do not communicate, leading to fragmented views of the enterprise data landscape.
*   **Security-Blind Discovery:** Crawling sensitive data sources without proper credential management or exposing PII within the metadata catalog itself.

## Edge Cases

*   **Dark Data:** Data that is collected and stored but not indexed or used. Discovery patterns must account for "unstructured" blobs where schemas are not predefined.
*   **Ephemeral Data:** Short-lived data (e.g., streaming telemetry) that may disappear before a scheduled crawler can identify it. Requires real-time metadata capture.
*   **Polyglot Persistence:** A single logical entity (e.g., "Customer") spread across a NoSQL store, a Relational DB, and a Cache. Discovery must use "Semantic Fingerprinting" to link these.
*   **Encrypted/Masked Data:** Discovery engines may see the field but cannot profile the content. Patterns must rely on upstream metadata or "side-channel" indicators.

## Related Topics
*   **084 Data Governance Frameworks:** The policy layer that discovery supports.
*   **086 Metadata Management:** The technical storage and versioning of the attributes found during discovery.
*   **090 Data Quality Standards:** The metrics used during the profiling stage of discovery.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
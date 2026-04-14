# 087 Sensitive Data Classification

Canonical documentation for 087 Sensitive Data Classification. This document defines concepts, terminology, and standard usage.

## Purpose
Sensitive Data Classification exists to provide a structured framework for identifying, categorizing, and protecting information assets based on their value, legal requirements, and the potential impact of unauthorized disclosure. The primary problem space it addresses is the "homogenization of data risk," where organizations treat all data with equal security measures, leading to either insufficient protection for critical assets or excessive costs for non-critical information.

By establishing a classification schema, organizations can apply granular security controls, meet regulatory compliance obligations (such as GDPR, HIPAA, or PCI-DSS), and optimize resource allocation for data protection.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Taxonomy and nomenclature of data sensitivity levels.
* Criteria for determining data impact and risk.
* The lifecycle of a data classification label.
* Theoretical frameworks for data discovery and mapping.

**Out of scope:**
* Specific vendor software implementations (e.g., Microsoft Purview, Google Cloud DLP).
* Physical security of paper documents (unless used as an analogy for digital logic).
* Specific legal advice or jurisdictional mandates.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Data Classification** | The process of organizing data into categories based on its sensitivity and the impact of its loss or disclosure. |
| **PII** | Personally Identifiable Information; any data that can be used to identify a specific individual. |
| **PHI** | Protected Health Information; any information about health status, provision of health care, or payment for health care. |
| **Impact Assessment** | The analysis of the potential damage (financial, reputational, legal) resulting from a data breach. |
| **Data Discovery** | The technical process of scanning data repositories to identify what data exists and where it resides. |
| **Labeling/Tagging** | The act of attaching metadata to a data object that indicates its classification level. |
| **Data Subject** | The individual to whom the sensitive data relates. |

## Core Concepts

### 1. Impact-Based Categorization
Classification is fundamentally driven by the "Impact of Compromise." This is measured across three pillars of the CIA Triad:
*   **Confidentiality:** Unauthorized access to the data.
*   **Integrity:** Unauthorized modification of the data.
*   **Availability:** Loss of access to the data.

### 2. The Data Lifecycle
Classification is not a static event but a continuous process throughout the data lifecycle:
1.  **Creation/Collection:** Initial classification applied at the point of entry.
2.  **Storage:** Encryption and access controls based on the label.
3.  **Usage:** Monitoring how classified data is accessed.
4.  **Sharing:** Controlling the movement of data across boundaries.
5.  **Archival/Destruction:** Ensuring data is purged according to its sensitivity level.

### 3. Inheritance and Aggregation
*   **Inheritance:** A child object (e.g., a row in a table) typically inherits the classification of its parent (the table), unless specified otherwise.
*   **Aggregation:** The "Mosaic Effect," where multiple pieces of non-sensitive data, when combined, result in a high-sensitivity data set.

## Standard Model
The generally accepted model for data classification utilizes a four-tier hierarchy.

| Level | Description | Example Data |
|-------|-------------|--------------|
| **Level 1: Public** | Disclosure causes no harm to the organization or individuals. | Marketing brochures, public price lists, job postings. |
| **Level 2: Internal** | Data intended for use within the organization; disclosure causes minimal disruption. | Internal memos, organizational charts, policy documents. |
| **Level 3: Confidential** | Sensitive data protected by policy or contract; disclosure causes significant harm. | Customer contracts, non-public financial reports, PII. |
| **Level 4: Restricted** | Highly sensitive data protected by law/regulation; disclosure causes catastrophic harm. | Social Security Numbers, biometric data, trade secrets, PHI. |

## Common Patterns

### Automated Discovery and Labeling
Utilizing pattern matching (Regex), checksums, and machine learning to scan large volumes of data and apply labels without manual intervention.

### User-Driven Classification
Empowering data creators to select the classification level at the time of document creation or email composition, often enforced by policy-driven prompts.

### Metadata-Persistent Tagging
Embedding the classification label directly into the file's metadata (e.g., XMP or custom headers) so that the protection follows the data even when it leaves the original environment.

## Anti-Patterns

*   **Over-Classification:** Labeling all data as "Restricted" or "Confidential." This leads to "security fatigue," where users ignore warnings, and increases operational costs unnecessarily.
*   **Static Classification:** Failing to re-evaluate data sensitivity over time. Data that was "Restricted" during a merger may become "Public" once the merger is announced.
*   **Shadow Data:** Maintaining "dark data" repositories that are not scanned or classified, creating blind spots in the organization's risk profile.
*   **Inconsistent Taxonomy:** Using different classification labels across different departments (e.g., "Private" in HR vs. "Secret" in Engineering) for the same type of data.

## Edge Cases

*   **Derived Data:** When an algorithm processes public data to predict sensitive attributes (e.g., predicting a medical condition based on public shopping habits).
*   **Anonymized vs. Pseudonymized Data:** Data that has been "scrubbed" may still be sensitive if the process is reversible or if the dataset is small enough to allow for re-identification.
*   **Legacy Systems:** Systems that do not support metadata tagging or modern discovery tools, requiring manual "wrapper" controls.
*   **Transient Data:** Data that exists only in memory or during transit (e.g., streaming telemetry) which may contain sensitive bursts but is never written to disk.

## Related Topics
*   **042 Data Sovereignty:** How classification affects where data can be stored geographically.
*   **115 Access Control Models (RBAC/ABAC):** How classification labels are used to grant or deny access.
*   **201 Data Loss Prevention (DLP):** The technical enforcement of classification policies.
*   **009 Encryption Standards:** The cryptographic requirements for different classification tiers.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
# 088 Data Masking Techniques

Canonical documentation for 088 Data Masking Techniques. This document defines concepts, terminology, and standard usage.

## Purpose
Data Masking exists to reconcile the conflicting requirements of data utility and data privacy. In modern information systems, functional testing, analytics, and third-party processing often require datasets that mirror the structure and characteristics of production data without exposing sensitive information, such as Personally Identifiable Information (PII), Protected Health Information (PHI), or Intellectual Property (IP).

The primary objective of data masking is to create a structurally valid but functionally "fake" version of a dataset, ensuring that even if the masked data is compromised, the original sensitive information cannot be reconstructed or attributed to a specific individual.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   Methodologies for de-identifying and obfuscating structured and semi-structured data.
*   Theoretical frameworks for maintaining data integrity and referential consistency during the masking process.
*   Classification of masking types (Static vs. Dynamic).
*   Mathematical and logical approaches to data transformation.

**Out of scope:**
*   Specific vendor implementations or software-specific syntax (e.g., specific SQL Server or Oracle masking functions).
*   Network-level encryption (TLS/SSL) or Disk-level encryption (TDE).
*   Physical security of data centers.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Static Data Masking (SDM)** | The process of creating a sanitized copy of a database where sensitive data is permanently replaced with masked values. |
| **Dynamic Data Masking (DDM)** | A technique where data is masked "on-the-fly" as it is queried, leaving the underlying stored data unchanged. |
| **Referential Integrity** | The requirement that masked values remain consistent across different tables and databases to maintain system functionality. |
| **Deterministic Masking** | A masking approach where a specific input value always yields the same masked output value, preserving join capabilities. |
| **Non-deterministic Masking** | A masking approach where the same input value yields different masked outputs each time, maximizing security at the cost of joinability. |
| **De-identification** | The general process of removing or obscuring identifiers from a dataset. |
| **Re-identification** | The act of using auxiliary information or statistical analysis to map masked data back to the original subject. |

## Core Concepts

### The Privacy-Utility Tradeoff
The fundamental challenge of data masking is balancing the degree of protection against the usefulness of the data. High-obfuscation techniques (like nulling out fields) provide maximum privacy but zero utility for testing or analytics. Low-obfuscation techniques (like simple character swapping) provide high utility but are vulnerable to pattern analysis.

### Irreversibility
A core tenet of robust data masking is that the process should be computationally irreversible. Unlike encryption, which is designed to be decrypted with a key, masked data should not contain the inherent logic required to reconstruct the original value.

### Data Realism
For masked data to be effective for software development and testing, it must pass validation checks. This includes:
*   **Format Preservation:** A masked credit card number must pass the Luhn algorithm.
*   **Type Consistency:** A masked date field must remain a valid date within a logical range.
*   **Semantic Accuracy:** A masked city should correspond to a valid state or postal code if those fields are present.

## Standard Model

The standard model for implementing data masking follows a four-phase lifecycle:

1.  **Discovery and Classification:** Scanning data environments to identify sensitive attributes (PII, PHI, PCI) and mapping their relationships across the schema.
2.  **Policy Definition:** Determining which masking technique (see Common Patterns) applies to which data class based on the required level of protection and the target use case.
3.  **Execution:**
    *   *Static:* Extracting, transforming, and loading (ETL) data into a lower-environment sink.
    *   *Dynamic:* Intercepting queries via a proxy or database engine to apply masks at runtime.
4.  **Validation:** Auditing the output to ensure no sensitive data leaked and that the dataset remains functionally viable for its intended purpose.

## Common Patterns

### 1. Substitution
Replacing the original value with a value from a look-up table of realistic but unrelated data.
*   *Example:* Replacing "John Smith" with "Robert Chen" using a pre-defined library of common names.

### 2. Shuffling
Randomizing values within a single column across different records.
*   *Example:* Swapping the "Salary" values between employees in a table. The aggregate totals remain correct, but individual privacy is protected.

### 3. Nulling / Deletion
Replacing sensitive data with a NULL value or a generic string like "REDACTED".
*   *Example:* Removing the "Comments" field from a customer support log.

### 4. Masking Out (Partial Masking)
Obfuscating only a portion of the data string while leaving the rest visible for context.
*   *Example:* Displaying a credit card as `XXXX-XXXX-XXXX-1234`.

### 5. Number and Date Variance
Applying a random percentage or fixed offset to numeric or date values.
*   *Example:* Shifting a birth date by +/- 30 days to protect the specific identity while maintaining the age demographic.

### 6. Format-Preserving Encryption (FPE)
Using cryptographic algorithms that produce output in the same format as the input. While technically encryption, it is often categorized under masking when used to generate realistic test data.

## Anti-Patterns

*   **Masking the UI Only:** Applying masking at the application layer while leaving the database accessible to developers/testers. This provides a false sense of security.
*   **Inconsistent Masking:** Masking a UserID as "123" in the Users table but "999" in the Orders table, breaking the relational integrity of the database.
*   **Reversible Logic:** Using simple ciphers (like ROT13) or predictable offsets that can be easily reversed by an observer.
*   **Masking Only "Direct" Identifiers:** Obfuscating names and SSNs but leaving "Quasi-identifiers" (Birth date, ZIP code, Gender) untouched, which can lead to re-identification via "Linkage Attacks."

## Edge Cases

*   **Small Datasets:** In a dataset with only 10 records, shuffling or variance may not provide enough entropy to prevent an observer from guessing the original values based on outliers.
*   **Circular Dependencies:** When two tables reference each other's sensitive keys, masking must be performed simultaneously or via a deterministic mapping to avoid orphaned records.
*   **Unstructured Data:** Masking PII within free-text fields (e.g., medical notes or chat logs) requires Natural Language Processing (NLP) to identify entities, which is prone to higher error rates than structured masking.
*   **Aggregated Data Leaks:** Even if individual records are masked, the sum or average of a small group might reveal information about a specific individual if the group size is too small (K-Anonymity concerns).

## Related Topics
*   **042 Data Sovereignty:** Regulations governing where data can be stored and processed.
*   **105 Differential Privacy:** A mathematical framework for adding noise to datasets to prevent individual identification.
*   **112 Synthetic Data Generation:** The creation of entirely artificial datasets from scratch rather than transforming existing data.
*   **015 Tokenization:** Replacing sensitive data with non-sensitive "tokens" that have no extrinsic value.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
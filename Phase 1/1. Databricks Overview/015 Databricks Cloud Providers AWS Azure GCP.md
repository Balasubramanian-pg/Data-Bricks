# Databricks Cloud Providers AWS Azure GCP

Canonical documentation for Databricks Cloud Providers AWS Azure GCP. This document defines concepts, terminology, and standard usage.

## Purpose

The purpose of this documentation is to provide a comprehensive understanding of Databricks Cloud Providers, specifically Amazon Web Services (AWS), Microsoft Azure, and Google Cloud Platform (GCP). This topic exists to address the problem space of deploying, managing, and optimizing Databricks workloads on various cloud providers. It aims to provide a unified understanding of the concepts, benefits, and challenges associated with using Databricks on different cloud platforms. This documentation will help users, administrators, and architects make informed decisions when designing and implementing Databricks solutions on AWS, Azure, and GCP.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope

The scope of this topic includes the concepts, terminology, and standard usage of Databricks Cloud Providers, specifically AWS, Azure, and GCP.

**In scope:**
* Databricks deployment models on AWS, Azure, and GCP
* Cloud provider-specific features and integrations
* Security, governance, and compliance considerations
* Performance optimization and cost management strategies

**Out of scope:**
* Tool-specific implementations, such as Databricks CLI or API
* Vendor-specific behavior, such as AWS-specific or Azure-specific features
* On-premises Databricks deployments

## Definitions

The following definitions are used throughout this documentation:

| Term | Definition |
|------|------------|
| Databricks | A cloud-based data engineering platform for building, deploying, and managing data pipelines and analytics workloads |
| Cloud Provider | A company that offers cloud computing services, such as AWS, Azure, or GCP |
| Deployment Model | A specific way of deploying Databricks on a cloud provider, such as a managed or unmanaged deployment |
| Cluster | A group of virtual machines or nodes that run Databricks workloads |
| Node | A single virtual machine or instance that runs Databricks workloads |

> [!TIP]
> Definitions should be stable over time; avoid contextual language.

## Core Concepts

The following core concepts are fundamental to understanding Databricks Cloud Providers:

### Cloud Deployment Models
Databricks can be deployed on cloud providers using various models, including managed and unmanaged deployments. Managed deployments provide a simplified experience, while unmanaged deployments offer more control and flexibility.

### Cloud Provider Integrations
Databricks integrates with cloud providers to leverage their services, such as storage, networking, and security. These integrations enable features like data encryption, access control, and monitoring.

### Security and Governance
Databricks on cloud providers requires careful consideration of security and governance aspects, including data encryption, access control, and compliance with regulatory requirements.

## Standard Model

The standard model for Databricks Cloud Providers involves the following components:

1. **Databricks Control Plane**: The control plane provides a centralized management interface for Databricks deployments.
2. **Cloud Provider Infrastructure**: The cloud provider infrastructure includes virtual machines, storage, and networking resources.
3. **Databricks Clusters**: Databricks clusters run on the cloud provider infrastructure and execute workloads.

> [!IMPORTANT]
> Deviations from the standard model should be explicitly documented and justified.

## Common Patterns

The following patterns are commonly observed when deploying Databricks on cloud providers:

* **Lift and Shift**: Migrating existing on-premises Databricks workloads to the cloud with minimal changes.
* **Cloud-Native**: Designing Databricks workloads from scratch to take advantage of cloud provider features and services.
* **Hybrid**: Combining on-premises and cloud-based Databricks deployments to achieve a balanced architecture.

## Anti-Patterns

The following anti-patterns are common mistakes or discouraged practices when deploying Databricks on cloud providers:

* **Over-Provisioning**: Allocating excessive resources, leading to unnecessary costs and waste.
* **Under-Provisioning**: Allocating insufficient resources, resulting in performance issues and delays.
* **Inconsistent Security**: Failing to apply consistent security controls and governance across Databricks deployments.

> [!WARNING]
> These anti-patterns often lead to maintenance or scalability issues.

## Edge Cases

The following edge cases are unusual, ambiguous, or boundary scenarios related to Databricks Cloud Providers:

* **Multi-Cloud Deployments**: Deploying Databricks on multiple cloud providers, requiring careful consideration of security, governance, and cost management.
* **Hybrid Cloud Deployments**: Combining on-premises and cloud-based Databricks deployments, requiring careful planning and management.
* **Disaster Recovery**: Implementing disaster recovery strategies for Databricks deployments on cloud providers, requiring careful consideration of data replication and failover mechanisms.

> [!CAUTION]
> Edge cases are frequently overlooked and may cause incorrect assumptions.

## Related Topics

The following topics are related to Databricks Cloud Providers:

* **Databricks Architecture**: Understanding the architecture of Databricks and its components.
* **Cloud Provider Security**: Understanding the security features and best practices of cloud providers.
* **Databricks Cost Optimization**: Optimizing costs for Databricks deployments on cloud providers.

## References

The following external references provide additional information on Databricks Cloud Providers:

* Databricks Documentation: <https://docs.databricks.com/>
* AWS Documentation: <https://docs.aws.amazon.com/>
* Azure Documentation: <https://docs.microsoft.com/en-us/azure/>
* GCP Documentation: <https://cloud.google.com/docs>

## Change Log

The following changes have been made to this topic:

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial documentation |
| 1.1 | 2026-01-12 | Added section on edge cases |
| 1.2 | 2026-01-13 | Updated references to include cloud provider documentation |
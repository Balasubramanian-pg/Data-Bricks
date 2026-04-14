# 099 Quality Gates Before Prod

Canonical documentation for 099 Quality Gates Before Prod. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of 099 Quality Gates Before Prod is to establish a rigorous, standardized framework for evaluating software artifacts and infrastructure changes before they are promoted to a production environment. This topic addresses the inherent risk of deploying unverified changes by enforcing a set of mandatory checkpoints. These gates ensure that all releases meet minimum thresholds for stability, security, performance, and functional correctness, thereby protecting the end-user experience and organizational reputation.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Validation Criteria:** The qualitative and quantitative metrics used to evaluate readiness.
* **Governance Frameworks:** The logic governing the transition between pre-production and production states.
* **Risk Mitigation Strategies:** Methods for identifying and blocking high-risk changes.
* **Decision Logic:** The binary (Pass/Fail) nature of gate evaluation.

**Out of scope:**
* **Specific Vendor Implementations:** Detailed configurations for specific CI/CD tools (e.g., Jenkins, GitLab, GitHub Actions).
* **Development-Phase Testing:** Unit testing or linting that occurs during the initial coding phase (Shift-Left), except where those results are aggregated for final approval.
* **Post-Production Monitoring:** Activities occurring after the gate has been passed and the deployment is complete.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| Quality Gate | A formal checkpoint in the delivery pipeline where a change is evaluated against predefined criteria; failure to meet criteria halts the promotion. |
| Promotion | The act of moving a validated artifact or configuration from a lower environment (e.g., Staging) to a higher environment (Production). |
| Threshold | The minimum acceptable value or state for a specific metric (e.g., < 5% error rate) required to pass a gate. |
| Artifact | The versioned, immutable package (container image, binary, or script) being evaluated for production readiness. |
| Deterministic Gate | A gate that relies on objective, machine-readable data to provide a consistent Pass/Fail result without human intervention. |
| Advisory Gate | A non-blocking checkpoint used for data collection or awareness, which does not prevent deployment but flags potential issues. |

## Core Concepts
### Binary Evaluation
Quality gates operate on a binary principle. A change is either "Ready" or "Not Ready." This removes ambiguity from the release process and ensures that subjective "gut feelings" do not override established safety standards.

### Evidence-Based Promotion
Promotion to production must be supported by cryptographic or logged evidence. This includes test execution logs, security scan reports, and approval signatures. If the evidence is missing or incomplete, the gate is considered failed.

### Environment Parity
For a quality gate to be valid, the environment in which the gate is evaluated (e.g., UAT or Staging) must closely mirror the production environment in configuration, scale, and data architecture.

### Traceability and Auditability
Every gate execution must be recorded. This provides a historical record of who approved a change, what data was used to justify the approval, and when the transition occurred.

## Standard Model
The standard model for 099 Quality Gates involves a multi-layered evaluation process:

1.  **Verification Layer:** Automated checks including integration tests, regression suites, and security vulnerability scans (SAST/DAST).
2.  **Validation Layer:** User Acceptance Testing (UAT) or synthetic monitoring to ensure the change meets business requirements.
3.  **Operational Layer:** Performance testing (load/stress) and infrastructure-as-code (IaC) validation to ensure the production environment can support the change.
4.  **Compliance Layer:** Final check for regulatory requirements, licensing, and documentation completeness.

The gate is only "Passed" when all layers return a positive result.

## Common Patterns
*   **The Stop-the-Line Pattern:** Any failure in a mandatory quality gate immediately halts the pipeline, preventing any further automated or manual promotion until the issue is remediated.
*   **The Canary Gate:** A pattern where a gate is applied to a small subset of production traffic. If metrics remain within thresholds, the gate "opens" for the remainder of the deployment.
*   **The Multi-Signature Gate:** Requiring approval from distinct stakeholders (e.g., Security, Product, and Engineering) before the final production transition.
*   **Automated Policy-as-Code:** Defining gate criteria in version-controlled files that are automatically evaluated by the deployment engine.

## Anti-Patterns
*   **The "Rubber Stamp":** Approving gates without reviewing the underlying data or evidence.
*   **The Exception Culture:** Frequently bypassing gates for "urgent" fixes, which eventually leads to the gates being ignored entirely.
*   **Moving the Goalposts:** Lowering the threshold of a gate (e.g., allowing 10 known vulnerabilities instead of 0) simply to meet a release deadline.
*   **Manual-Only Gates:** Relying entirely on human intervention, which introduces bottlenecks, inconsistency, and fatigue.
*   **Late-Stage Discovery:** Placing critical gates so late in the process that a failure causes massive rework and project delays.

## Edge Cases
*   **Emergency Hotfixes:** In scenarios where a production outage requires an immediate fix, a "Break Glass" procedure may bypass certain non-critical gates. However, these must be retroactively audited and validated.
*   **Third-Party Dependencies:** When a gate depends on a service or API outside of the organization's control. The gate must account for the "unknown" state of external dependencies.
*   **Data-Only Changes:** Changes to database schemas or reference data that may not trigger traditional code-based gates but require specialized "Data Quality Gates."
*   **Legacy Systems:** Systems that lack automated testing capabilities. In these cases, gates may rely on manual observational checklists and "burn-in" periods in staging.

## Related Topics
*   **042 Continuous Integration:** The foundation upon which quality gates are built.
*   **115 Incident Management:** The process triggered when a quality gate fails to catch a production issue.
*   **078 Infrastructure as Code (IaC):** Ensuring the environment itself passes quality gates.
*   **021 Security Compliance:** Specific gate criteria related to legal and regulatory standards.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
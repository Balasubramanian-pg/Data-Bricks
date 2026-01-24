# 055 Bias Audits

Canonical documentation for 055 Bias Audits. This document defines concepts, terminology, and standard usage.

## Purpose
The purpose of a Bias Audit is to systematically evaluate an automated decision system (ADS) or algorithmic model to identify, quantify, and mitigate systematic errors that result in unfair treatment of specific groups. Bias audits address the risk of "algorithmic discrimination," where historical prejudices or data imbalances are codified into software, leading to disparate impacts on protected classes or demographic groups.

By providing a structured framework for scrutiny, bias audits ensure that systems remain compliant with legal requirements, align with ethical standards, and maintain public trust.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Methodological Frameworks:** The theoretical approach to measuring fairness and identifying bias.
* **Metric Categories:** Statistical and mathematical definitions of parity and error rates.
* **Audit Lifecycle:** The phases of an audit from data preparation to remediation.
* **Regulatory Alignment:** General principles required for compliance with global standards (e.g., NYC Local Law 144, EU AI Act).

**Out of scope:**
* **Specific Vendor Implementations:** Proprietary software tools or specific cloud-provider fairness modules.
* **Code-level Implementation:** Specific programming language libraries (e.g., Python's Scikit-learn or Aequitas).
* **General Security Audits:** Penetration testing or vulnerability scanning, unless directly impacting bias.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Automated Decision System (ADS)** | A computational process, including those derived from machine learning or AI, that issues a simplified output (score, classification, recommendation) used to support or replace human decision-making. |
| **Protected Attribute** | A characteristic (e.g., race, gender, age, disability) that is legally protected from discrimination in specific contexts like employment, housing, or credit. |
| **Disparate Impact** | A condition where a neutral policy or algorithm disproportionately affects members of a protected group, even if there is no discriminatory intent. |
| **Proxy Variable** | A non-protected attribute that correlates strongly with a protected attribute (e.g., zip code as a proxy for race), potentially introducing "hidden" bias. |
| **Fairness Metric** | A mathematical measurement used to quantify the level of bias or equity within a model's output. |
| **Intersectionality** | The study of overlapping or social identities (e.g., Black women) and the unique forms of discrimination that occur at these intersections. |

## Core Concepts

### 1. The Nature of Bias
Bias in algorithmic systems typically originates from three sources:
*   **Data Bias:** Reflects historical inequities or sampling errors in the training data.
*   **Algorithmic Bias:** Introduced by the design of the model, such as optimization functions that prioritize majority-group accuracy.
*   **Human/Contextual Bias:** Introduced by the way humans interpret or apply the model's output in real-world scenarios.

### 2. Fairness Definitions
Fairness is not a single mathematical state but a choice of trade-offs. Core concepts include:
*   **Group Fairness:** Ensuring that outcomes (e.g., selection rates) are equal across different demographic groups.
*   **Individual Fairness:** Ensuring that similar individuals receive similar outcomes.
*   **Counterfactual Fairness:** Ensuring a decision would remain the same if a protected attribute were changed, holding all other factors constant.

### 3. Independence vs. Separation vs. Sufficiency
*   **Independence:** The outcome is independent of the protected attribute (Statistical Parity).
*   **Separation:** The error rates (False Positives/False Negatives) are equal across groups (Equalized Odds).
*   **Sufficiency:** The probability of a positive outcome given a specific score is the same across groups (Calibration).

## Standard Model
The generally accepted model for a Bias Audit follows a five-phase lifecycle:

1.  **Preparation and Scoping:** Identify the "Impacted Population," define the "Protected Attributes," and select the "Fairness Criteria" relevant to the specific use case.
2.  **Data Integrity Assessment:** Evaluate the training and validation datasets for representativeness, missingness, and the presence of proxy variables.
3.  **Quantitative Analysis:** Apply statistical tests to the model outputs to calculate disparate impact and error rate disparities.
4.  **Qualitative Review:** Assess the "Human-in-the-loop" processes, documentation, and the logic of the model's features to identify non-mathematical sources of bias.
5.  **Reporting and Remediation:** Document findings in a "Bias Audit Report" and implement mitigation strategies (e.g., re-weighting data, adjusting thresholds, or decommissioning the model).

## Common Patterns

### The Third-Party Audit
To ensure objectivity and regulatory compliance, organizations often engage independent, external entities to conduct the audit. This pattern minimizes "confirmation bias" and provides a higher level of assurance to stakeholders.

### Pre-deployment vs. Post-deployment
*   **Pre-deployment (Ex-ante):** Auditing during the development phase to prevent harm before the system is live.
*   **Post-deployment (Ex-post):** Continuous monitoring of the system in production to detect "drift" or unforeseen consequences in real-world data.

### Disparate Impact Ratio (The 4/5ths Rule)
A common pattern in employment contexts where a selection rate for any group that is less than 80% (or four-fifths) of the rate for the group with the highest rate is regarded as evidence of adverse impact.

## Anti-Patterns

### Fairness Washing
The practice of selecting the one fairness metric that a model passes while ignoring several others where the model fails, in order to present a false image of equity.

### The "Omission" Fallacy
The belief that removing protected attributes (e.g., deleting the "Gender" column) eliminates bias. This ignores the reality of proxy variables and often makes it impossible to audit the system for bias later.

### One-and-Done Auditing
Treating a bias audit as a static, one-time checkbox. Algorithmic systems and the data they process are dynamic; a system that is "fair" today may become biased as societal patterns or data distributions shift.

## Edge Cases

### Small Sample Sizes
In scenarios with very small populations (e.g., executive hiring), statistical significance is difficult to achieve. Audits in these cases must rely more heavily on qualitative process reviews rather than mathematical parity.

### Privacy-Preserving Auditing
Auditors often face the "Privacy-Fairness Paradox," where they need access to sensitive protected attributes to check for bias, but privacy regulations (like GDPR) may restrict the collection of that very data. Synthetic data or "Trusted Third Parties" are often used as workarounds.

### Conflicting Metrics
It is mathematically proven that in most cases, it is impossible to satisfy all fairness metrics simultaneously (e.g., one cannot achieve both Equalized Odds and Predictive Parity if base rates differ). The audit must document the justification for prioritizing one metric over another.

## Related Topics
*   **042 Algorithmic Transparency:** The disclosure of how an ADS functions.
*   **088 Data Governance:** The management of data quality and availability.
*   **102 Model Explainability (XAI):** Techniques to make the "black box" of AI understandable to humans.
*   **Regulatory Compliance (NYC LL144, EU AI Act):** Specific legal frameworks requiring bias audits.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
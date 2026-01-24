# 053 Model Explainability Techniques

Canonical documentation for 053 Model Explainability Techniques. This document defines concepts, terminology, and standard usage.

## Purpose
Model Explainability Techniques exist to bridge the gap between the mathematical complexity of machine learning models and human comprehension. As models—particularly deep neural networks and ensemble methods—become more sophisticated, they often function as "black boxes" where the logic driving a specific output is opaque.

This topic addresses the requirement for transparency, accountability, and trust in automated decision-making. It provides the framework for debugging models, identifying bias, ensuring regulatory compliance (such as the "right to explanation"), and validating that models are making decisions based on relevant features rather than noise or artifacts.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
*   **Taxonomy of Interpretability:** Classification of techniques by scope (global/local) and applicability (model-specific/agnostic).
*   **Post-hoc Explanation Methods:** Techniques applied after a model has been trained.
*   **Intrinsic Interpretability:** Characteristics of models that are transparent by design.
*   **Evaluation Metrics for Explanations:** Criteria such as fidelity, stability, and robustness.

**Out of scope:**
*   **Specific vendor implementations:** Proprietary software or specific library syntax (e.g., specific Python or R package documentation).
*   **General Data Visualization:** Standard charting techniques not directly related to model internals.
*   **Model Training Algorithms:** The process of building models, except where it pertains to intrinsic interpretability.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Interpretability** | The degree to which a human can understand the cause of a decision or predict the model's result. |
| **Explainability (XAI)** | The suite of techniques and methods used to make the results of machine learning algorithms understandable by humans. |
| **Black Box** | A system where the internal logic is hidden from the user, providing only inputs and outputs. |
| **White Box** | A model that is inherently transparent, such as a shallow decision tree or linear regression. |
| **Global Explanation** | An explanation that describes the average behavior of the model across the entire dataset. |
| **Local Explanation** | An explanation that justifies a specific prediction for a single instance or observation. |
| **Model-Agnostic** | Techniques that can be applied to any machine learning model regardless of its internal architecture. |
| **Model-Specific** | Techniques limited to a specific class of models (e.g., gradient-based methods for neural networks). |
| **Fidelity** | The degree to which an explanation accurately reflects the underlying model's behavior. |

## Core Concepts

### 1. The Interpretability-Flexibility Trade-off
Generally, as the predictive power and flexibility of a model increase (e.g., Deep Learning), its inherent interpretability decreases. Explainability techniques aim to recover understanding without sacrificing performance.

### 2. Taxonomy of Techniques
*   **Intrinsic vs. Post-hoc:** Intrinsic refers to models that are simple enough to be understood on their own. Post-hoc refers to methods applied to a trained model to extract insights.
*   **Model-Agnostic vs. Model-Specific:** Agnostic methods treat the model as a black box (analyzing input/output pairs), while specific methods inspect internal weights, gradients, or structures.

### 3. Feature Attribution
The process of assigning a "score" or "weight" to each input feature, indicating its relative contribution to a specific prediction or the model's overall logic.

## Standard Model

The standard model for explainability follows a three-stage pipeline:

1.  **The Model Stage:** A machine learning model is trained on an input dataset to produce a prediction function $f(x)$.
2.  **The Explainer Stage:** An explanation method $g$ is applied. This method may probe the model $f(x)$ with perturbed inputs or inspect the internal state of $f$.
3.  **The Human Interface Stage:** The output of $g$ is converted into a human-readable format, such as a feature importance plot, a heat map (saliency), or a natural language summary.

## Common Patterns

### Feature Importance (Global)
Quantifying which variables have the most significant impact on the model's performance across the entire population.
*   *Example:* Permutation Feature Importance, which measures the increase in prediction error after permuting the feature's values.

### Local Surrogate Models
Approximating a complex model's behavior in the immediate vicinity of a specific data point using a simpler, interpretable model (like a linear regression).
*   *Example:* LIME (Local Interpretable Model-agnostic Explanations).

### Shapley Values (Game Theory)
Distributing the "payout" (the prediction) among the "players" (the features) based on their average marginal contribution across all possible combinations of features.
*   *Example:* SHAP (SHapley Additive exPlanations).

### Visual Explanations (Computer Vision)
Identifying which pixels or regions of an image contributed most to a classification.
*   *Example:* Saliency Maps, Grad-CAM.

### Counterfactual Explanations
Describing the smallest change to the input features that would result in a different model output (e.g., "If your income had been $5,000 higher, your loan would have been approved").

## Anti-Patterns

*   **Explanation Washing:** Using explainability techniques to provide a false sense of security or to justify a biased model without addressing the underlying issues.
*   **Confusing Correlation with Causation:** Assuming that because a feature is highly "important" to a model, it is the causal driver of the real-world outcome.
*   **Over-reliance on a Single Method:** Relying solely on one technique (e.g., Saliency Maps) which may be prone to noise or "shattered gradients," leading to misleading visualizations.
*   **Ignoring Fidelity:** Providing an explanation that is easy to understand but does not actually track with how the model makes decisions.

## Edge Cases

*   **High-Dimensional Data:** In datasets with thousands of features (e.g., genomics), individual feature attributions may become too granular to be meaningful to humans.
*   **Adversarial Explanations:** The possibility of "gaming" an explanation method to hide a model's bias or to make a model appear to rely on different features than it actually does.
*   **Multicollinearity:** When features are highly correlated, many explainability techniques (like SHAP or Permutation Importance) may struggle to assign credit accurately, often splitting the importance between the correlated variables.
*   **Non-Tabular Data:** Explaining decisions on audio or unstructured text requires different abstractions than tabular feature importance.

## Related Topics

*   **042 Model Bias and Fairness:** Techniques for detecting and mitigating disparate impact.
*   **061 Model Monitoring:** Tracking how feature importance shifts over time (Data Drift).
*   **015 Supervised Learning:** The foundational model types that require these techniques.
*   **Regulatory Compliance (GDPR/AIA):** Legal frameworks requiring model transparency.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
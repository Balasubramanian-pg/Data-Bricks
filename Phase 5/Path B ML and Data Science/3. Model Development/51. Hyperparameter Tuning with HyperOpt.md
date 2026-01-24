# 051 Hyperparameter Tuning with HyperOpt

Canonical documentation for 051 Hyperparameter Tuning with HyperOpt. This document defines concepts, terminology, and standard usage.

## Purpose
Hyperparameter tuning is the process of optimizing the configuration of a machine learning algorithm to maximize performance on a given dataset. HyperOpt exists to address the inefficiencies of manual, grid, and random search methods by providing a framework for Bayesian optimization. It is designed to navigate high-dimensional, non-convex, and conditional search spaces where evaluating the objective function is computationally expensive.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Bayesian Optimization Logic:** The theoretical application of Tree-structured Parzen Estimators (TPE).
* **Search Space Topology:** Definition of stochastic expressions and conditional dependencies.
* **Optimization Lifecycle:** The iterative process of objective evaluation, loss reporting, and state persistence.
* **Asynchronous Execution:** The conceptual framework for parallelizing evaluations.

**Out of scope:**
* **Specific Library Versions:** Syntax specific to minor version releases of the HyperOpt Python library.
* **Vendor-Specific Integrations:** Managed services (e.g., Databricks, AWS SageMaker) that wrap HyperOpt.
* **Model Architecture:** The internal logic of the models being tuned (e.g., XGBoost, PyTorch).

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Objective Function** | A scalar-valued function that the optimizer minimizes. It maps a point in the search space to a "loss" value. |
| **Search Space** | The multi-dimensional domain of hyperparameters, defined by probability distributions, over which the optimization occurs. |
| **TPE (Tree-structured Parzen Estimator)** | A Bayesian optimization algorithm that models the distribution of "good" and "bad" hyperparameters to suggest the next point to evaluate. |
| **Stochastic Expression** | A definition within the search space that specifies a random variable (e.g., `hp.uniform`, `hp.choice`). |
| **Trials Object** | A database or data structure that stores the history of all evaluations, including hyperparameters, results, and metadata. |
| **Loss** | The metric used to evaluate the performance of a specific hyperparameter configuration; HyperOpt always seeks to minimize this value. |
| **Acquisition Function** | The logic used to decide which point in the search space to sample next, balancing exploration and exploitation. |

## Core Concepts

### Bayesian Optimization
Unlike grid search (which is exhaustive) or random search (which is uninformed), HyperOpt uses Bayesian optimization. It constructs a surrogate model of the objective function based on past results. As more evaluations are completed, the surrogate model becomes more accurate at predicting which regions of the search space are likely to yield a lower loss.

### Tree-structured Parzen Estimator (TPE)
TPE is the default optimization logic in HyperOpt. It works by modeling two distributions:
1.  $l(x)$: The distribution of hyperparameters that yielded a loss lower than a specific quantile.
2.  $g(x)$: The distribution of hyperparameters that yielded a loss higher than that quantile.
The algorithm chooses the next point $x$ that maximizes the ratio $l(x) / g(x)$, effectively searching for points that are likely to be in the "good" distribution and unlikely to be in the "bad" distribution.

### Conditional Search Spaces
One of HyperOpt's primary strengths is the ability to define nested or conditional spaces. For example, if an algorithm has a choice between two optimizers (e.g., SGD vs. Adam), the hyperparameters specific to Adam (like `beta1`) are only sampled if Adam is selected.

## Standard Model

The standard model for HyperOpt implementation follows a four-stage lifecycle:

1.  **Space Definition:** Define the hyperparameters using stochastic expressions. This defines the "priors" for the optimization.
2.  **Objective Formulation:** Create a function that accepts a configuration from the search space, trains a model, and returns a dictionary containing at least a `'loss'` and a `'status'`.
3.  **Optimization Execution (`fmin`):** Invoke the minimization function, specifying the objective, the space, the algorithm (TPE), and the maximum number of evaluations.
4.  **Result Analysis:** Inspect the `Trials` object to determine the global minimum and analyze the sensitivity of the model to various hyperparameters.

## Common Patterns

### The Minimization Requirement
HyperOpt is designed to minimize. When the primary metric is a maximization metric (like Accuracy or F1-score), the standard pattern is to return the negative value of that metric (e.g., `loss = -1 * accuracy`).

### Checkpointing with Trials
For long-running optimizations, the `Trials` object (or its persistent variants like `MongoTrials`) is used to save the state of the search. This allows for resuming an optimization after a failure or extending the number of iterations.

### Early Stopping within Objective
To save computational resources, the objective function often incorporates early stopping logic. If a model's performance is significantly worse than the current best in the `Trials` object during the early stages of training, the evaluation can be terminated early and reported as a failure or a high loss.

## Anti-Patterns

### Over-Constraining the Search Space
Defining search spaces that are too narrow prevents the Bayesian optimizer from discovering non-intuitive configurations. Conversely, using a uniform distribution for parameters that operate on a logarithmic scale (like learning rates) is inefficient; `hp.loguniform` should be used instead.

### Data Leakage in the Objective Function
Performing preprocessing or feature selection inside the objective function without proper cross-validation can lead to overfitting the hyperparameters to the validation set, resulting in poor generalization on test data.

### Ignoring the "Cold Start" Problem
TPE requires a minimum number of random samples (usually 20) before it can begin modeling the distributions effectively. Running too few total iterations (e.g., 25) does not allow the Bayesian logic enough data to outperform random search.

## Edge Cases

### Non-Convergent Objective Functions
If the objective function encounters a NaN or an exception (e.g., a specific hyperparameter combination causes a memory overflow), the status must be set to `STATUS_FAIL` in the return dictionary. This prevents the optimizer from attempting to model that point as a valid loss.

### Discrete-Continuous Hybrids
When a search space mixes discrete choices (integers) with continuous ranges (floats), TPE may struggle if the discrete choices significantly alter the meaning of the continuous ranges. In such cases, splitting the optimization into separate runs or using nested conditional spaces is required.

### High-Dimensionality Saturation
In extremely high-dimensional spaces (hundreds of hyperparameters), the surrogate model's efficiency may degrade, and the overhead of calculating the next point may become significant relative to the time required to evaluate the objective function.

## Related Topics
* **050 Automated Machine Learning (AutoML):** The broader field of automating the ML pipeline.
* **052 Bayesian Optimization Theory:** The mathematical foundations of surrogate modeling.
* **053 Tree-structured Parzen Estimators:** Deep dive into the TPE algorithm.
* **054 Distributed Computing for Optimization:** Scaling HyperOpt across clusters.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-11 | Initial AI-generated canonical documentation |
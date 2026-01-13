# 097 Notebook Tests in CI

Canonical documentation for 097 Notebook Tests in CI. This document defines concepts, terminology, and standard usage.

## Purpose
The integration of computational notebooks (e.g., Jupyter, IPython) into Continuous Integration (CI) pipelines addresses the inherent fragility of literate programming artifacts. Notebooks often suffer from "bit rot" due to their dependency on specific environments, external data sources, and the non-linear nature of their development. 

The primary purpose of Notebook Tests in CI is to ensure that notebooks remain executable, reproducible, and mathematically accurate over time. This process validates that the narrative, code, and output remain synchronized, preventing the distribution of broken tutorials, invalid research, or failing production pipelines.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* **Automated Execution:** The systematic running of notebook cells in a headless environment.
* **Validation Logic:** Criteria for determining "success" (e.g., zero exit codes, output matching).
* **Environment Parity:** Ensuring the CI environment mirrors the intended execution environment.
* **State Management:** Handling the linear flow and persistence of variables during the test lifecycle.

**Out of scope:**
* **Specific Vendor Implementations:** Detailed guides for GitHub Actions, GitLab CI, or Jenkins.
* **Tool-Specific Syntax:** Documentation for specific libraries like `nbconvert`, `papermill`, or `pytest-notebook`.
* **IDE Integration:** Local development environment configurations.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Headless Execution** | The process of running a notebook programmatically without a Graphical User Interface (GUI) or interactive browser session. |
| **Kernel** | The computational engine that executes the code contained within a notebook document. |
| **Cell** | An individual unit of a notebook that contains either code, markdown text, or raw data. |
| **Linear Execution** | The requirement that a notebook must run successfully from the first cell to the last cell in sequential order. |
| **Kernel Restart** | The act of clearing the current execution state and memory to ensure a "clean slate" for testing. |
| **Output Regression** | A failure state where the current execution output differs from the "source of truth" stored in the notebook metadata. |
| **Idempotency** | The property where a notebook can be executed multiple times without changing the result beyond the initial application. |

## Core Concepts

### 1. The Notebook as a Testable Artifact
Unlike standard source code files (.py, .js, .r), a notebook is a JSON-based document containing both code and cached results. Testing in CI treats the notebook as a functional unit where the input is the code cells and the expected output is the successful completion of the kernel's execution.

### 2. Ephemeral State
In a CI context, every notebook test must begin with a fresh kernel. This eliminates "hidden state" errors where a notebook appears to work locally only because variables remain in memory from deleted or out-of-order cell executions.

### 3. Verification Levels
* **Smoke Testing:** Validates only that the notebook executes from top to bottom without throwing an exception.
* **Functional Testing:** Validates that specific values or dataframes produced within the notebook meet defined assertions.
* **Visual/Output Testing:** Validates that the generated plots, tables, or logs match the previously saved versions within the `.ipynb` file.

## Standard Model

The standard model for Notebook Tests in CI follows a four-stage lifecycle:

1.  **Environment Provisioning:** The CI runner initializes a container or virtual environment containing the required kernel and all software dependencies.
2.  **Kernel Injection:** The system identifies the metadata within the notebook to select the correct kernel (e.g., Python 3.10, R, Julia).
3.  **Sequential Execution:** The runner executes cells in a top-down, non-interactive fashion. If any cell produces an unhandled exception, the test is marked as failed.
4.  **Artifact Comparison (Optional):** The resulting notebook is compared against the source notebook to identify regressions in data or visualizations.

## Common Patterns

### The "Clean Room" Pattern
Executing the notebook in a completely isolated container where no local files exist except those explicitly defined in the repository. This identifies missing data dependencies or hardcoded local paths.

### Parameterization
Injecting specific variables (e.g., `TRAIN_EPOCHS=1` or `DATA_PATH='test/data'`) into a notebook at runtime to allow for faster execution during CI without modifying the original "production" notebook.

### Documentation Testing
Using CI to ensure that code snippets in educational notebooks remain valid as the underlying API evolves. This is common in SDK and library maintenance.

## Anti-Patterns

*   **Manual Check-ins of Execution State:** Committing notebooks with large, binary, or sensitive output data, which bloats the repository and makes diffing difficult.
*   **Non-Linear Dependencies:** Writing notebooks that require cells to be run out of order (e.g., running cell 10 before cell 5).
*   **Network Dependency:** Relying on live, external APIs or databases that may be flaky or unavailable during CI runs, leading to "false negative" test failures.
*   **Infinite Loops/Long-Running Cells:** Failing to set execution timeouts, causing CI runners to hang and consume excessive resources.

## Edge Cases

*   **Non-Deterministic Output:** Machine learning models or random number generators may produce different results on each run. CI tests must use fixed seeds or "fuzzy" matching for assertions.
*   **Interactive Widgets:** Notebooks containing `ipywidgets` or other interactive elements often fail in headless CI environments because there is no front-end to render the JavaScript components.
*   **Hardware Acceleration:** Notebooks requiring GPUs or TPUs may fail in standard CI environments that only provide CPU resources. These require specialized runners or "mocking" of hardware-dependent code.
*   **Large Data Handling:** Notebooks that process multi-gigabyte datasets may exceed the memory limits of standard CI runners.

## Related Topics

*   **042 Containerization in DevOps:** For defining the environments where notebooks run.
*   **115 Regression Testing:** For comparing notebook outputs over time.
*   **088 Literate Programming:** For the theoretical framework of combining code and prose.
*   **Data Version Control (DVC):** For managing the data dependencies required for notebook tests.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-13 | Initial AI-generated canonical documentation |
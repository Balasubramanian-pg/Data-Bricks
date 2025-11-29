# Phase 4: Workflow Management and Production (The “Orchestration”)

**Goal:** Move beyond experimenting in notebooks and learn how to design, schedule, observe and maintain production-grade data pipelines. This is where your work stops being “projects” and becomes “systems”.

# Key Concepts

## 1. Databricks Jobs

Jobs are the backbone of production workloads. They define **what** runs, **when** it runs and **on what compute**.

### What You Must Understand

* **Job Types:**

  * Notebook jobs
  * JAR/Scala jobs
  * Python scripts
  * SQL tasks
* **Task Graphs:**

  * Multiple tasks in one job
  * Upstream and downstream dependencies
  * Automatic retries, failure handling
* **Cluster Types for Jobs:**

  * Job clusters (ephemeral)
  * All-purpose clusters (only for dev/test)
* **Scheduling:**

  * Cron expressions
  * Weekly, hourly, minute-level triggers
* **Notifications & Alerts:**

  * Email on success
  * Email/slack/webhook on failure
* **Parameters:**

  * Notebook widgets
  * Databricks job-level parameters
  * JSON configuration for complex workflows

### Why They Matter

* Jobs replace “manual notebook runs”
* Consistent execution
* Isolated compute
* Controlled dependency management

## 2. Workflows (formerly Delta Live Tables)

Workflows let you define pipelines using declarative logic. Instead of writing manual ETL plumbing, you declare what each table should be and Databricks handles the orchestration.

### What You Must Learn

* **Core Concepts:**

  * Declarative pipeline definitions
  * Managed vs advanced mode
  * Live vs triggered pipelines
  * Data quality controls baked in

* **DLT Syntax and Decorators:**

  * `@dlt.table` to define tables
  * `@dlt.view` to define upstream logical views
  * `@dlt.expect()` for validations
  * `@dlt.expect_or_drop()`
  * `@dlt.expect_or_fail()`
  * `@dlt.table(comment="...")` for documentation

* **Data Quality Monitoring:**

  * Automatic metrics tracking
  * Failed record capture
  * Automated lineage and visualization

* **Pipeline Behavior:**

  * Incremental updates
  * Schema evolution
  * How DLT optimizes physical data layout (OPTIMIZE, ZORDER)
  * What autoscaling and compute choices mean for performance

### Why DLT Matters

* Reduces pipeline fragility
* Adds observability and lineage automatically
* Forces best practices in data engineering
* Perfect for enterprise-grade pipelines

## 3. Orchestration with External Tools

Databricks integrates with popular orchestrators used by enterprises.

### Tools to Know

* **Azure Data Factory (ADF):**

  * Pipeline activities to trigger Databricks jobs
  * Passing parameters to notebooks
  * Linked services
* **Airflow:**

  * Using Databricks Operators
  * Triggering runs via API
  * Managing DAG dependencies
* **Prefect, Dagster:**

  * Modern orchestrators
  * Clean API integration
* **CI/CD Tools (GitHub Actions, Azure DevOps):**

  * Build, test and deploy pipelines
  * Automated notebook formatting
  * Automated packaging of jobs

### When to Use External Orchestrators

* Multi-system workflows
* Complex dependencies across services
* Event-driven pipelines
* Need for enterprise monitoring and alerting

# Actionable Steps

## 1. Schedule Your Notebook as a Job

* Open Jobs UI
* Create a new Job
* Attach your ETL notebook from earlier phases
* Choose a **Job Cluster** for compute
* Set a schedule (for example, every 6 hours)
* Add email notifications on failure
* Run the job manually and inspect logs
* Explore the Run History page to understand:

  * Start time
  * End time
  * Logs
  * Cluster usage
  * Error stack traces if any

## 2. Build a Simple DLT Pipeline

* Create a pipeline in the Workflows UI
* Choose Pipeline Mode: **Triggered**
* Point to a notebook containing:

  * A Bronze table defined using `@dlt.table`
  * A Silver table applying a transformation
  * A Gold table using an aggregation
* Add expectations, for example:

  ```python
  @dlt.expect("valid_price", "price >= 0")
  ```
* Run the pipeline
* View lineage auto-generated
* Explore metrics like:

  * Row counts
  * Number of dropped rows
  * Alerting from expectations

## 3. Optional: Trigger the Job Externally

**Azure Data Factory:**

* Create a pipeline
* Add “Delete activity” to ensure fresh start
* Add “Databricks Notebook” or “Job Activity”
* Parameterize your inputs

**Airflow:**

* Use the DatabricksSubmitRunOperator
* Build a simple DAG with 2 tasks
* Trigger your Databricks notebook job

**REST API Script:**

* Use Python or CURL to trigger job
* Pass job parameters dynamically
* Monitor run status via API

In the next phase can focus on advanced engineering: CI/CD, Unity Catalog security models, SQL Warehousing, feature stores, streaming pipelines and performance tuning.

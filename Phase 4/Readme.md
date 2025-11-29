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

## Additional Topics You Can Add

### Testing and Validation in Production

* Unit tests for transformations
* Data quality gates
* Smoke tests on upstream data
* Canary runs before full pipeline execution

### Observability and Monitoring

* Metrics tracking
* Pipeline health dashboards
* SLA monitoring
* Alert rules for data freshness or job delays
* Logging patterns for notebooks and scripts

### Dependency and Environment Management

* Using Databricks Repos for versioning
* Packaging Python code as wheels for jobs
* Managing secrets with Databricks Secret Scopes
* Handling environment files and cluster init scripts

### Deployment and CI/CD

* Notebook tests in CI
* Automatic deployment of jobs via YAML configs
* Promotion workflow: Dev to Test to Prod
* Using Git branches as environments

### Data Contracts and Interface Stability

* Schema-validation logic
* Backward-compatible changes
* Communicating schema changes across teams

### Advanced Scheduling Logic

* Event-driven runs
* Trigger jobs when upstream tables update
* Chaining pipelines that run on dependencies instead of clocks

### Error Recovery and Resilience

* Automatic retries
* Dead-letter queues
* Backfilling logic for missed days
* Replay strategies for late-arriving data

### Cost Governance in Production

* Cluster size policies
* Spot instance strategies
* Cost alerts for runaway jobs
* Optimizing job cluster configs

### Security and Access Controls

* Unity Catalog permissions on production pipelines
* Token-based job triggers
* Service principals for orchestrators
* Audit logging

### Multi-Hop Architecture Patterns

* Bronze/Silver/Gold organizational design
* Idempotent pipeline patterns
* Slowly changing dimensions
* Surrogate keys and merge strategies

### Streaming-Oriented Orchestration

* How streaming jobs differ from scheduled ones
* Trigger intervals
* Checkpoint management
* Failover patterns

### Productionizing ML Workflows

* Feature pipelines
* Batch inference jobs
* Drift detection
* Model re-training schedules

### Additional Topics You Can Add

* **Cluster Policies in Jobs**

  * Enforcing allowed node types.
  * Limiting auto-scaling behavior.
  * Cost control through job cluster templates.

* **Job Clusters vs All-Purpose Clusters**

  * When to use which.
  * Performance and cost tradeoffs.

* **Notifications & Alerting**

  * Email alerts.
  * PagerDuty and Slack hooks.
  * Failure-only vs every-run notifications.

* **Retry Logic & Timeout Management**

  * Setting max retries.
  * Setting timeout per task.
  * Handling intermittent API failures.

* **Task Dependencies in Multi-Task Jobs**

  * Using the Jobs UI to create DAG-like flows.
  * Running tasks in parallel vs sequential.
  * Reusing cluster configuration across tasks.

* **Parameterization**

  * Passing parameters to notebooks.
  * Using widgets to control logic.
  * Environment-based configs: dev, test, prod.

* **Secrets & Credential Management**

  * Databricks Secrets.
  * Using secret scopes in jobs.
  * Rotating tokens and credentials.

* **Monitoring & Logging Best Practices**

  * Using the Job Run dashboard.
  * Tracking pipeline lineage.
  * Logging custom metrics to Delta tables.

* **Error Handling Patterns**

  * Try/except patterns in notebooks.
  * Failing early vs failing gracefully.
  * Writing error logs to storage.

* **Versioned Deployments**

  * Using Git Repos for code.
  * CI/CD with Azure DevOps or GitHub Actions.
  * Releasing versioned updates of pipelines.

* **Data Lineage & Impact Analysis**

  * Unity Catalog lineage view.
  * Tracing upstream/downstream dependencies.

* **Quality Gates Before Publishing to Prod**

  * Schema validation tests.
  * Unit tests with pytest.
  * Sanity checks on row counts and nulls.

* **Blue-Green Deployment for Pipelines**

  * Deploying to a shadow pipeline first.
  * Comparing outputs before switching traffic.

* **Cost Observability**

  * Tracking job cost per run.
  * Detecting expensive clusters.
  * Estimating cost per data pipeline.

* **SLAs & SLOs for Data Pipelines**

  * Defining expectations for data availability.
  * Monitoring SLIs like latency and freshness.

* **Backfilling Strategies**

  * Historical loads without breaking prod.
  * Idempotent pipeline design.

* **Concurrency & Isolation**

  * Running multiple pipelines without conflict.
  * Handling table locks.
  * Avoiding cluster overload.

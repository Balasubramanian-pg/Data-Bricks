# Phase 5: Specialization Paths (The "Expertise")

Pick the direction that matches where you want to stand out. Each path goes deeper than the basics and makes you genuinely valuable.

## Path A: Data Engineering (The Builders)

* **Advanced Spark Tuning**

  * Shuffle management and spill prevention.
  * Skew handling using hints, salting, repartitioning.
  * Caching strategies for iterative pipelines.
  * Understanding the Catalyst Optimizer internals.
* **Medallion Architecture Mastery**

  * Designing proper Bronze ingestion patterns.
  * Creating schema evolution rules in Silver.
  * Designing reusable Gold models for BI and ML.
  * Governance patterns across all three layers.
* **Change Data Capture**

  * SCD Type 0, 1, 2 implementations with Delta.
  * Deletes, updates, inserts with `MERGE`.
  * Watermarking for incremental loads.
* **Performance Optimization**

  * Z-Ordering, data skipping, optimize transactions.
  * Photon Engine internals and cost efficiency.
  * Debugging slow jobs with `EXPLAIN EXTENDED`.
  * File size tuning (128 MB sweet spots and exceptions).
* **Reliability Engineering**

  * Idempotent pipelines.
  * Retry and checkpointing patterns.
  * Backfill designs for large historical data.
* **Infrastructure & DevOps**

  * dbx for deployment automation.
  * CI/CD pipelines using GitHub or Azure DevOps.
  * Terraform for workspace, cluster, and Unity Catalog provisioning.
  * Secrets and service principals for secure automation.
* **Governance**

  * Unity Catalog data lineage.
  * Table ACLs and fine-grained permissions.
  * Managing data quality expectations at scale.

## Path B: Data Science & Machine Learning (The Experimenters)

* **MLflow Deep Dive**

  * Experiment tracking across teams.
  * Model registry with staging, prod, and rollback.
  * Packaging models for batch or real-time inference.
* **Databricks Runtime for ML**

  * Built-in GPU acceleration.
  * Pre-tuned libraries: XGBoost, LightGBM, TensorFlow, PyTorch.
* **Feature Engineering & Feature Store**

  * Building reusable, versioned features.
  * Online vs offline feature serving.
  * Point-in-time correctness for training sets.
* **Model Development**

  * AutoML for rapid baseline models.
  * Hyperparameter tuning using HyperOpt.
  * Distributed training with Spark and Horovod.
* **Responsible AI**

  * Model explainability.
  * Drift detection and monitoring.
  * Bias audits.
* **Collaborative ML Workflows**

  * Git-based model development.
  * Repo-driven end-to-end ML systems.
  * CI/CD for model deployment.

## Path C: Data Analytics & SQL (The Insight People)

* **Databricks SQL (DBSQL) Mastery**

  * Designing high performance SQL queries.
  * Query plans and cost-based optimization.
  * Warehouse cluster tuning for concurrency.
* **SQL Warehouses**

  * Serverless vs Pro vs Classic.
  * Scaling, pooling, and cost controls.
* **Dashboards & Analytics Workflows**

  * Interactive dashboards with parameters.
  * Alerts based on thresholds and anomalies.
  * Caching layers for faster BI.
* **Data Modeling on Delta**

  * Star schema design.
  * Slowly changing dimensions for analytics.
  * Building aggregate tables for BI.
* **Delta Engine & Photon**

  * Understanding vectorized execution.
  * Query acceleration techniques.
  * Optimizing Delta tables for analytics loads.
* **Governed Self-Service Analytics**

  * Unity Catalog for access control.
  * Certified and semantic tables for analysts.
  * Discovery patterns for organization-wide reuse.

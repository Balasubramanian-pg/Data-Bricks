### Phase 2: Core Platform Proficiency (The "How")
**Goal:** Become comfortable with the day-to-day tools and execution environment.

**Key Concepts:**
*   **Workspace & Navigation:** Projects, Repos, Folders.
*   **Clusters:** Difference between All-Purpose and Job Clusters. Understanding driver vs. worker nodes, cluster configurations, and policies.
*   **Notebooks:** Primary tool for interactive development. Learn magic commands (`%python`, `%sql`, `%md`), attaching a notebook to a cluster, and running cells.
*   **Databricks File System (DBFS):** The distributed file system layered over cloud object storage (S3, ADLS, GCS). Learn how to read from and write to it.
*   **Basic Data Operations:** Reading/writing data in various formats (CSV, JSON, Parquet, Delta Lake).

**Actionable Steps:**
1.  **Create your own All-Purpose Cluster** in the Community Edition.
2.  **Create a new Notebook** and practice using different languages (Scala, Python, SQL, R) in different cells.
3.  **Upload a small CSV file to DBFS** and write PySpark code in a notebook to read it and display the first few rows.
4.  **Write the data back to DBFS in Parquet format.**
5.  **Explore the Cluster UI** to understand metrics and event logs.

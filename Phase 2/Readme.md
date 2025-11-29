# Phase 2: Core Platform Proficiency

**The How: Building real comfort with Databricks as an environment, not just a place to run Spark jobs**

**Goal:** Gain practical fluency with every core workspace element you will touch daily. This is where Databricks stops feeling abstract and starts feeling like your home turf.

## Key Concepts to Master

### Workspace and Navigation

Learn the structure deeply because every project, repo and job you create will live here.

* How folders, repos and shared workspaces are organized
* Difference between user workspace and shared workspace
* Importing notebooks from files or Git
* Managing permissions and understanding access boundaries
* How Databricks stores notebooks (internal revision history, versions)

### Repos

Even though CI/CD shows up later, Repos matter early.

* Connecting Databricks Repos to GitHub, GitLab or Azure DevOps
* Pull, commit and push workflows inside the UI
* Working with multi-branch workflows
* Understanding limitations like large file handling and merge conflicts

### Clusters

You need to be fluent with compute because every job depends on it.

* Difference between All-Purpose Clusters and Job Clusters
* When to use which one
* Spark driver vs worker nodes
* How executors scale and why cluster size affects performance
* Spot vs on-demand nodes
* Cluster policies and why enterprises enforce them
* Autopilot options like autoscaling and termination settings
* Reading cluster metrics dashboards to diagnose performance bottlenecks
* Using Ganglia event logs to understand execution patterns

### Notebooks

Notebooks are your cockpit.

* Using multiple languages in one notebook with magic commands like `%python`, `%sql`, `%scala`, `%md`
* How notebooks attach to clusters and what happens when they detach
* Running cells individually vs running all cells
* Using widgets for parameterization
* Exporting notebooks as HTML, JAR or DBC format
* Version history and notebook diffing
* How to modularize work by calling one notebook from another

### Databricks File System (DBFS)

DBFS sits on top of cloud object storage but behaves like a distributed filesystem.

* How `/dbfs/`, `dbfs:/` and underlying S3/ADLS paths relate
* Uploading files through the UI and CLI
* Listing directories using `%fs ls`
* Limitations of DBFS root vs mounting external storage
* Handling metadata files, schema files and intermediate outputs
* Best practice: writing to mounted external storage instead of DBFS root

### Basic Data Operations

These are your baseline Spark skills.

* Reading data using `spark.read` for CSV, JSON, Parquet, Delta
* Handling schema inference vs providing a schema
* Writing in different formats with modes like overwrite, append and ignore
* Using DataFrame actions like `show`, `count`, `describe`
* Saving DataFrames to Delta Lake tables for long-term use
* Understanding partitions created when writing data
* Evaluating file sizes and how they impact downstream performance


## Actionable Steps

1. **Create your own All-Purpose Cluster**
   Set cluster size, enable autoscaling, explore runtime versions and understand why some runtimes include ML libraries and others do not.

2. **Create a new Notebook**
   Practice running Python, SQL, Scala and R cells. Understand how execution context switches when you use language magics. Experiment with Markdown cells to document your code.

3. **Upload a CSV to DBFS**
   Use the UI file uploader or `%fs cp` to place the file under `/FileStore/tables`. Read it with PySpark using `spark.read.csv` with relevant options like header, inferSchema and delimiter.

4. **Write outputs in multiple formats**
   Store the processed dataset in Parquet and Delta. Compare file size, partitioning and schema. Validate the write with `%fs ls` and read it back with `spark.read.parquet`.

5. **Explore the Cluster UI deeply**

   * Check Spark UI tabs: Jobs, Stages, Storage, SQL
   * Review event logs
   * Inspect executors and see how tasks distribute
   * Look at driver and worker memory usage to understand resource consumption

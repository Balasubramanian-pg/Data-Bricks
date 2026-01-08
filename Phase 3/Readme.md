# Phase 3: Deep Dive into Key Technologies (The “Power”)

**Goal:** Build serious technical strength by mastering the engines and storage layers that make Databricks a high-performance data and AI platform. This phase transforms you from someone who can “use Databricks” into someone who can **engineer systems on Databricks**.

# 3.1 Apache Spark

The backbone of Databricks. Almost everything you do - ETL, data prep, ML feature engineering, analytics - runs on Spark.

### Core Concepts to Understand

**1. Spark Architecture**

* Driver vs Executor processes
* Logical plan vs physical plan
* Tasks, stages, shuffle mechanics
* Why more partitions can speed up parallelism but too many can hurt performance

**2. Spark APIs**

* RDD (low-level, rarely used today)
* DataFrames (the standard for ETL, analytics)
* Datasets (type-safe API used mostly in Scala)

**3. Transformations vs Actions**

* Transformations (lazy): `select`, `filter`, `withColumn`, `groupBy`, `join`
* Actions (trigger execution): `count()`, `collect()`, `show()`, `write` operations

**4. Lazy Evaluation**

* Spark builds a logical plan instead of executing immediately
* Execution happens only when an action is called
* This enables optimisation using the Catalyst optimizer

**5. Partitioning & Performance**

* Repartition vs Coalesce
* When to partition your data
* Skew handling
* File size considerations
* Shuffle operations and their cost

### Essential Skills

* Convert SQL logic into Spark transformations
* Analyse query plans using `explain()`
* Optimise slow transformations
* Understand how joins affect shuffles

# 3.2 Delta Lake (CRITICAL)

The Lakehouse backbone. This is where reliability, performance and governance come from.

### Core Principles

**1. Storage Format + Transaction Log**

* Delta is Parquet plus a transaction log
* Provides ACID guarantees on data lake storage
* Guarantees correctness under concurrency and failures

**2. Time Travel**

* Query older versions of data using `VERSION AS OF` or `TIMESTAMP AS OF`
* Perfect for debugging, auditing and reproducing experiments

**3. Schema Enforcement & Evolution**

* Prevents dirty data from corrupting tables
* Allows controlled evolution when schemas change

**4. Key Delta Lake Features**

* `MERGE INTO` for SCD logic and upserts
* `UPDATE` and `DELETE` without rewriting entire tables
* Auto-optimize features
* Change Data Feed (CDF) for incremental downstream pipelines

**5. Table Maintenance Commands**

* `OPTIMIZE` to compact small files
* `ZORDER` to improve selective query performance
* `VACUUM` for cleaning old files and saving storage

### Essential Skills

* Build Bronze, Silver and Gold Delta tables
* Read table history: `DESCRIBE HISTORY tableName`
* Implement SCD1 and SCD2 using MERGE
* Write streaming Delta pipelines
* Optimise Delta tables for analytics performance

# 3.3 Databricks Runtime

Databricks does not run stock Spark. Its runtime is heavily optimized.

### Runtime Types

**1. Standard Runtime**

* General-purpose Spark runtime
* Supports most workloads

**2. ML Runtime**

* Includes pre-installed ML libraries
* Supports MLflow, GPU acceleration, TensorFlow, PyTorch
* Provides optimized training performance

**3. Photon Runtime**

* High-performance vectorized execution engine
* Extremely fast for SQL queries on Delta tables
* Great for warehousing workloads

### Why You Should Care

* Using the right runtime changes performance drastically
* Photon can make BI workloads 2x–12x faster
* ML Runtime saves hours of dependency headaches

### Essential Skills

* Choose correct runtime per workload
* Benchmark performance between runtimes
* Understand how upgrades affect pipeline behavior

## Actionable Steps

### Step 1: Convert Your Parquet File into a Delta Table

Use PySpark or SQL:

```python
df.write.format("delta").save("/FileStore/mydata/delta/")
```

Or SQL:

```sql
CREATE TABLE my_delta_table
USING DELTA
LOCATION '/FileStore/mydata/delta/';
```

### Step 2: Practice Delta Operations

Run:

**UPDATE**

```sql
UPDATE my_delta_table SET columnA = 'new_value' WHERE columnB = 5;
```

**DELETE**

```sql
DELETE FROM my_delta_table WHERE columnB < 0;
```

**MERGE (Upsert)**

```sql
MERGE INTO target t
USING source s
ON t.id = s.id
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *;
```

**Time Travel**

```sql
SELECT * FROM my_delta_table VERSION AS OF 0;
```

**Table History**

```sql
DESCRIBE HISTORY my_delta_table;
```

### Step 3: Explore Maintenance Commands

Run:

```sql
OPTIMIZE my_delta_table;
```

```sql
VACUUM my_delta_table RETAIN 168 HOURS;
```

```sql
OPTIMIZE my_delta_table ZORDER BY (columnA);
```

### Step 4: Complete Courses

* Databricks Academy: *Intro to DataFrames*
* Databricks Academy: *Delta Lake Fundamentals*
* Optional: Databricks Academy: *Optimizing Delta and Apache Spark*


In the next phase can break down Medallion architecture, Auto Loader, job orchestration and designing production pipelines.

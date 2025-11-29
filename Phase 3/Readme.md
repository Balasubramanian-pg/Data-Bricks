### Phase 3: Deep Dive into Key Technologies (The "Power")
**Goal:** Master the core technologies that make Databricks powerful.

#### 3.1. Apache Spark
*   **Concept:** The core computation engine. Understand RDDs, DataFrames, and Datasets.
*   **Focus:** Spark SQL and DataFrames API (this is the most common way to use Spark today).
*   **Learn:** Transformations vs. Actions, Lazy Evaluation, Partitioning.

#### 3.2. Delta Lake (***CRITICAL***)
*   **Concept:** The storage layer that brings reliability (ACID transactions) to data lakes.
*   **Key Features:** `MERGE`, `UPDATE`, `DELETE`, Time Travel, Schema Evolution, Change Data Feed.
*   **Why it's important:** This is the foundation of the Lakehouse. Most tables in Databricks should be Delta tables.

#### 3.3. Databricks Runtime
*   **Concept:** The optimized version of Spark that Databricks provides.
*   **Understand:** The difference between Standard, ML, and Photon runtimes.

**Actionable Steps:**
1.  **Convert the Parquet file from Phase 2 into a Delta Table** using `df.write.format("delta").save(...)` or `CREATE TABLE USING DELTA`.
2.  **Practice Delta Lake Operations:**
    *   Perform an `UPDATE` on your Delta table.
    *   Use `DESCRIBE HISTORY` to see the table history.
    *   **Time Travel:** Query the data as it was at a previous version.
3.  **Complete the official "Intro to DataFrames" and "Delta Lake" tutorials** on Databricks Academy.

# Week 12: Apache Spark and Databricks — Hands-On Crash Course

> **The big idea:** You already know SQL. Spark runs the *same SQL* on a distributed cluster. Today you'll prove it.  
> **Phase 3 reminder:** Your Spark comparison is due Week 14. Everything you build today is directly usable.

---

## Part A: Setup — Your First Spark Cluster (20 min)

We're doing this together, step by step. Open your laptop and follow along.

### Step 1: Sign Up for Databricks Community Edition

1. Go to [https://community.cloud.databricks.com/](https://community.cloud.databricks.com/)
2. Click **Sign Up** (top right)
3. Fill in your name, email (use your CSULB email), and create a password
4. On the plan selection page, click **Get started with Community Edition** (the small link at the bottom — not the trial)
5. Verify your email
6. Log in

!!! warning "Community Edition, not Free Trial"
    The free trial gives you a 14-day AWS account that requires a credit card. **Community Edition** is free forever with no credit card. Look for the small link at the bottom of the sign-up page.

### Step 2: Create a Cluster

A cluster is your distributed computer — a group of machines that work together.

1. In the left sidebar, click **Compute**
2. Click **Create Cluster**
3. Name it: `IS480-Spring2026`
4. Leave all defaults (Community Edition gives you a single small node — that's fine for learning)
5. Click **Create Cluster**
6. Wait 3-5 minutes for it to start (status goes from "Pending" to "Running" with a green dot)

!!! info "What just happened?"
    Databricks just provisioned a virtual machine with Apache Spark pre-installed. In industry, a "cluster" might be 10-1000 machines. Your Community Edition cluster is one machine, but the API and SQL are identical to a 1000-node cluster.

### Step 3: Create a Notebook

1. In the left sidebar, click **Workspace**
2. Click the dropdown arrow next to your name → **Create** → **Notebook**
3. Name it: `IS480_Phase3_Workspace`
4. Default Language: **Python**
5. Cluster: Select `IS480-Spring2026`

### Step 4: Test It

In the first cell, type and run (Shift+Enter):

```python
spark.sql("SELECT 1 AS test").show()
```

You should see:

```
+----+
|test|
+----+
|   1|
+----+
```

!!! tip "Congratulations!"
    You just ran a SQL query on Apache Spark. That's the same engine Netflix uses to process petabytes of viewing data. The SQL is identical — only the scale is different.

---

## Part B: Getting Your Data In (20 min)

You need to get your Oracle star schema data into Databricks. The workflow is:

```
Oracle RDS → Export as CSV → Upload to Databricks → Load into Spark
```

### Export from Oracle (3 Options)

See the [Oracle CSV Export Guide](oracle-csv-export.md) for detailed instructions on all three methods.

**Quick summary:**

| Method | Best For | Difficulty |
|--------|---------|-----------|
| SQL Developer GUI | Small tables, quick exports | Easy |
| Python script | Automation, large tables | Medium |
| SQL*Plus SPOOL | Command-line users | Easy |

Export each of your star schema tables as a separate CSV:

- `dim_student.csv`
- `dim_course.csv`
- `dim_date.csv`
- `dim_class.csv`
- `fact_enrollments.csv`

### Upload CSVs to Databricks

1. In the left sidebar, click **Data**
2. Click **Create Table**
3. Click **Upload File**
4. Drag and drop your CSV files (or click Browse)
5. They'll be stored at `/FileStore/tables/your_file.csv`

!!! tip "Upload all CSVs at once"
    You can drag multiple files in one go. Databricks stores them in `/FileStore/tables/`.

### Load CSVs into Spark DataFrames

In your notebook, create a new cell for each table:

```python
# ============================================
# Cell 1: Load all CSV files into DataFrames
# ============================================

# Load dimension tables
df_dim_student = spark.read.csv(
    "/FileStore/tables/dim_student.csv",
    header=True,
    inferSchema=True
)

df_dim_course = spark.read.csv(
    "/FileStore/tables/dim_course.csv",
    header=True,
    inferSchema=True
)

df_dim_date = spark.read.csv(
    "/FileStore/tables/dim_date.csv",
    header=True,
    inferSchema=True
)

df_dim_class = spark.read.csv(
    "/FileStore/tables/dim_class.csv",
    header=True,
    inferSchema=True
)

# Load fact table
df_fact_enrollments = spark.read.csv(
    "/FileStore/tables/fact_enrollments.csv",
    header=True,
    inferSchema=True
)

print("All tables loaded!")
```

**Verify each table:**

```python
# Check what Spark inferred
df_dim_student.printSchema()
df_dim_student.show(5)
print(f"dim_student rows: {df_dim_student.count()}")
```

You'll see output like:

```
root
 |-- student_key: integer (nullable = true)
 |-- snum: integer (nullable = true)
 |-- sname: string (nullable = true)
 |-- standing: integer (nullable = true)
 |-- majorid: string (nullable = true)
 |-- gpa: double (nullable = true)
 |-- load_date: string (nullable = true)

+----------+----+-------+--------+-------+----+----------+
|student_key|snum| sname |standing|majorid| gpa| load_date|
+----------+----+-------+--------+-------+----+----------+
|         1| 101|  Smith|       3|   CECS| 3.2|2026-03-...|
...
```

!!! warning "Schema check!"
    If `inferSchema=True` misidentifies a column type (e.g., reads a number as string), you can fix it:
    ```python
    from pyspark.sql.types import IntegerType
    df_dim_student = df_dim_student.withColumn("snum", df_dim_student["snum"].cast(IntegerType()))
    ```

---

## Part C: Spark SQL — Same Queries, Bigger Engine (30 min)

Here's the payoff: the SQL you wrote last week in Oracle works **identically** in Spark SQL.

### Register DataFrames as SQL Tables

```python
# ============================================
# Cell 2: Register DataFrames as SQL tables
# ============================================

df_dim_student.createOrReplaceTempView("dim_student")
df_dim_course.createOrReplaceTempView("dim_course")
df_dim_date.createOrReplaceTempView("dim_date")
df_dim_class.createOrReplaceTempView("dim_class")
df_fact_enrollments.createOrReplaceTempView("fact_enrollments")

print("All tables registered as SQL views!")
```

Now you can use `spark.sql()` with the exact same SQL:

### Query 1: Enrollments by Department with ROLLUP

```python
# ============================================
# Cell 3: Analytical Query 1 — ROLLUP
# ============================================

result = spark.sql("""
    SELECT
        dc.dept,
        dd.semester_name,
        COUNT(*) AS enrollment_count
    FROM fact_enrollments f
    JOIN dim_course dc ON f.course_key = dc.course_key
    JOIN dim_date dd   ON f.date_key = dd.date_key
    GROUP BY ROLLUP(dc.dept, dd.semester_name)
    ORDER BY dc.dept, dd.semester_name
""")

result.show(50)
```

!!! tip "Notice anything?"
    That's the **exact same SQL** from Week 11. Copy-paste from your Oracle script. It just works.

### Query 2: Student Ranking by GPA

```python
# ============================================
# Cell 4: Analytical Query 2 — RANK()
# ============================================

result2 = spark.sql("""
    SELECT
        ds.majorid,
        ds.sname,
        ds.gpa,
        RANK() OVER (PARTITION BY ds.majorid ORDER BY ds.gpa DESC) AS rank_gpa,
        DENSE_RANK() OVER (PARTITION BY ds.majorid ORDER BY ds.gpa DESC) AS dense_rank_gpa
    FROM dim_student ds
    ORDER BY ds.majorid, rank_gpa
""")

result2.show(20)
```

### Query 3: Year-over-Year Enrollment Change

```python
# ============================================
# Cell 5: Analytical Query 3 — LAG
# ============================================

result3 = spark.sql("""
    SELECT
        dc.dept,
        dd.class_year,
        COUNT(*) AS enrollments,
        LAG(COUNT(*), 1) OVER (PARTITION BY dc.dept ORDER BY dd.class_year) AS prev_year,
        COUNT(*) - LAG(COUNT(*), 1) OVER (PARTITION BY dc.dept ORDER BY dd.class_year) AS yoy_change
    FROM fact_enrollments f
    JOIN dim_course dc ON f.course_key = dc.course_key
    JOIN dim_date dd   ON f.date_key = dd.date_key
    GROUP BY dc.dept, dd.class_year
    ORDER BY dc.dept, dd.class_year
""")

result3.show(20)
```

### Create Persistent Hive Tables (Optional but Recommended)

Temp views disappear when the cluster restarts. Save as permanent Hive tables:

```python
# Save as persistent Hive tables
df_dim_student.write.mode("overwrite").saveAsTable("is480.dim_student")
df_dim_course.write.mode("overwrite").saveAsTable("is480.dim_course")
df_dim_date.write.mode("overwrite").saveAsTable("is480.dim_date")
df_dim_class.write.mode("overwrite").saveAsTable("is480.dim_class")
df_fact_enrollments.write.mode("overwrite").saveAsTable("is480.fact_enrollments")

print("All tables saved to Hive metastore!")
```

!!! info "Hive tables persist"
    Once saved as Hive tables, your data survives cluster restarts. You can query them directly with `spark.sql("SELECT * FROM is480.dim_student")` without re-loading CSVs.

### The Side-by-Side Comparison (Phase 3 Format)

For Phase 3, you need to show the same query in Oracle and Spark. Here's the format:

**Create a markdown cell in your notebook** (change cell type to Markdown):

```markdown
## Query 1: Department Enrollment with ROLLUP

### Oracle SQL (ran on Oracle RDS)
| dept | semester_name | enrollment_count |
|------|--------------|-----------------|
| CECS | Fall | 45 |
| CECS | Spring | 52 |
| CECS | NULL | 97 |
| ... | ... | ... |

*Execution time: 0.23 seconds*

### Spark SQL (ran on Databricks)
[output from spark.sql().show() above]

*Execution time: 1.8 seconds*

### Observations
- Identical results — SQL syntax is fully compatible
- Spark is slower on this small dataset (overhead of distributed engine)
- At scale (millions of rows), Spark would be faster due to parallelism
```

---

## Part D: PySpark — The Python API (20 min)

Spark SQL is great, but Spark also has a Python API called **PySpark**. Same engine, different interface.

### Why PySpark?

| Use SQL When... | Use PySpark When... |
|----------------|-------------------|
| Ad-hoc analysis | Building data pipelines |
| You know exactly what you want | Logic requires loops/conditions |
| Quick exploration | Integrating with ML libraries |
| Sharing with non-programmers | Chaining complex transformations |

### Rewrite Queries in PySpark

```python
# ============================================
# Cell 6: PySpark version of Query 1 — Aggregation
# ============================================
from pyspark.sql import functions as F

pyspark_result1 = (
    df_fact_enrollments
    .join(df_dim_course, "course_key")
    .join(df_dim_date, "date_key")
    .groupBy("dept", "semester_name")
    .agg(F.count("*").alias("enrollment_count"))
    .orderBy("dept", "semester_name")
)

pyspark_result1.show()
```

!!! info "Note"
    PySpark doesn't have a direct `ROLLUP` in the DataFrame API for all versions. For ROLLUP/CUBE, use `spark.sql()`. For most other queries, PySpark works great.

```python
# ============================================
# Cell 7: PySpark version of Query 2 — Window Functions
# ============================================
from pyspark.sql.window import Window

window_spec = Window.partitionBy("majorid").orderBy(F.desc("gpa"))

pyspark_result2 = (
    df_dim_student
    .withColumn("rank_gpa", F.rank().over(window_spec))
    .withColumn("dense_rank_gpa", F.dense_rank().over(window_spec))
    .select("majorid", "sname", "gpa", "rank_gpa", "dense_rank_gpa")
    .orderBy("majorid", "rank_gpa")
)

pyspark_result2.show(20)
```

```python
# ============================================
# Cell 8: PySpark version of Query 3 — LAG
# ============================================

# First, aggregate by dept and year
dept_year = (
    df_fact_enrollments
    .join(df_dim_course, "course_key")
    .join(df_dim_date, "date_key")
    .groupBy("dept", "class_year")
    .agg(F.count("*").alias("enrollments"))
)

# Then apply LAG window function
lag_window = Window.partitionBy("dept").orderBy("class_year")

pyspark_result3 = (
    dept_year
    .withColumn("prev_year", F.lag("enrollments", 1).over(lag_window))
    .withColumn("yoy_change", F.col("enrollments") - F.col("prev_year"))
    .orderBy("dept", "class_year")
)

pyspark_result3.show(20)
```

!!! tip "SQL vs PySpark — Same Result"
    Run both versions and compare the output. They should be identical. For your Phase 3, pick whichever you're more comfortable with — or show both to impress.

---

## Part E: Complete Databricks Notebook Template

Copy this entire template into a new Databricks notebook. Replace the CSV paths with your project's files.

```python
# ============================================
# IS480 Phase 3 — Spark Analysis Notebook
# YOUR NAME | Spring 2026
# ============================================
```

### Cell 1: Load Data

```python
# --- Cell 1: Load CSVs into DataFrames ---

# Update these paths with your actual file names
df_dim_student = spark.read.csv("/FileStore/tables/dim_student.csv", header=True, inferSchema=True)
df_dim_course = spark.read.csv("/FileStore/tables/dim_course.csv", header=True, inferSchema=True)
df_dim_date = spark.read.csv("/FileStore/tables/dim_date.csv", header=True, inferSchema=True)
df_dim_class = spark.read.csv("/FileStore/tables/dim_class.csv", header=True, inferSchema=True)
df_fact = spark.read.csv("/FileStore/tables/fact_enrollments.csv", header=True, inferSchema=True)

# Verify counts
for name, df in [("dim_student", df_dim_student), ("dim_course", df_dim_course),
                  ("dim_date", df_dim_date), ("dim_class", df_dim_class),
                  ("fact_enrollments", df_fact)]:
    print(f"{name}: {df.count()} rows")
```

### Cell 2: Create SQL Tables

```python
# --- Cell 2: Register as SQL tables ---

df_dim_student.createOrReplaceTempView("dim_student")
df_dim_course.createOrReplaceTempView("dim_course")
df_dim_date.createOrReplaceTempView("dim_date")
df_dim_class.createOrReplaceTempView("dim_class")
df_fact.createOrReplaceTempView("fact_enrollments")

# Verify with a quick query
spark.sql("SELECT COUNT(*) AS total_facts FROM fact_enrollments").show()
```

### Cells 3-5: Analytical Queries in Spark SQL

```python
# --- Cell 3: Spark SQL Query 1 (ROLLUP) ---

spark.sql("""
    SELECT
        dc.dept,
        dd.semester_name,
        COUNT(*) AS enrollment_count
    FROM fact_enrollments f
    JOIN dim_course dc ON f.course_key = dc.course_key
    JOIN dim_date dd ON f.date_key = dd.date_key
    GROUP BY ROLLUP(dc.dept, dd.semester_name)
    ORDER BY dc.dept, dd.semester_name
""").show(50)
```

```python
# --- Cell 4: Spark SQL Query 2 (RANK) ---

spark.sql("""
    SELECT
        ds.majorid,
        ds.sname,
        ds.gpa,
        RANK() OVER (PARTITION BY ds.majorid ORDER BY ds.gpa DESC) AS rank_gpa
    FROM dim_student ds
    ORDER BY ds.majorid, rank_gpa
""").show(30)
```

```python
# --- Cell 5: Spark SQL Query 3 (LAG) ---

spark.sql("""
    SELECT
        dc.dept,
        dd.class_year,
        COUNT(*) AS enrollments,
        LAG(COUNT(*), 1) OVER (PARTITION BY dc.dept ORDER BY dd.class_year) AS prev_year
    FROM fact_enrollments f
    JOIN dim_course dc ON f.course_key = dc.course_key
    JOIN dim_date dd ON f.date_key = dd.date_key
    GROUP BY dc.dept, dd.class_year
    ORDER BY dc.dept, dd.class_year
""").show(30)
```

### Cells 6-8: Same Queries in PySpark

```python
# --- Cell 6: PySpark Query 1 (Aggregation) ---
from pyspark.sql import functions as F

(df_fact
 .join(df_dim_course, "course_key")
 .join(df_dim_date, "date_key")
 .groupBy("dept", "semester_name")
 .agg(F.count("*").alias("enrollment_count"))
 .orderBy("dept", "semester_name")
 .show(50))
```

```python
# --- Cell 7: PySpark Query 2 (RANK) ---
from pyspark.sql.window import Window

w = Window.partitionBy("majorid").orderBy(F.desc("gpa"))

(df_dim_student
 .withColumn("rank_gpa", F.rank().over(w))
 .select("majorid", "sname", "gpa", "rank_gpa")
 .orderBy("majorid", "rank_gpa")
 .show(30))
```

```python
# --- Cell 8: PySpark Query 3 (LAG) ---

dept_yr = (df_fact
           .join(df_dim_course, "course_key")
           .join(df_dim_date, "date_key")
           .groupBy("dept", "class_year")
           .agg(F.count("*").alias("enrollments")))

w2 = Window.partitionBy("dept").orderBy("class_year")

(dept_yr
 .withColumn("prev_year", F.lag("enrollments", 1).over(w2))
 .orderBy("dept", "class_year")
 .show(30))
```

### Cell 9: Side-by-Side Comparison (Markdown Cell)

Change this cell type to **Markdown** in Databricks:

```markdown
## Side-by-Side Comparison: Oracle vs Spark

### Query 1: Department Enrollment Summary (ROLLUP)

| Aspect | Oracle | Spark SQL | PySpark |
|--------|--------|-----------|---------|
| **SQL Syntax** | Identical | Identical | DataFrame API |
| **Results** | [paste Oracle output] | [paste Spark output] | [paste PySpark output] |
| **Execution Time** | X.XX sec | X.XX sec | X.XX sec |

### Query 2: Student GPA Ranking (RANK)

| Aspect | Oracle | Spark SQL | PySpark |
|--------|--------|-----------|---------|
| **SQL Syntax** | Identical | Identical | Window API |
| **Results** | [paste] | [paste] | [paste] |
| **Execution Time** | X.XX sec | X.XX sec | X.XX sec |

### Query 3: Year-over-Year Enrollment (LAG)

| Aspect | Oracle | Spark SQL | PySpark |
|--------|--------|-----------|---------|
| **SQL Syntax** | Identical | Identical | lag() function |
| **Results** | [paste] | [paste] | [paste] |
| **Execution Time** | X.XX sec | X.XX sec | X.XX sec |

### Key Observations
1. **SQL Compatibility:** Spark SQL syntax is nearly identical to Oracle for analytical queries
2. **Scale:** Our dataset is small, so Oracle is faster (no distributed overhead). At millions+ rows, Spark's parallelism wins
3. **PySpark:** More verbose but more flexible — better for pipelines and ML integration
4. **Ecosystem:** Spark integrates with Python ML libraries (scikit-learn, TensorFlow) that Oracle cannot
```

---

## Common Issues and Fixes

| Problem | Solution |
|---------|---------|
| Cluster won't start | Wait 5 min. If still pending, delete and recreate |
| CSV upload fails | Check file size (Community Edition limit: 2GB). Try smaller files |
| `inferSchema` gets wrong types | Use `.withColumn("col", col.cast("integer"))` to fix |
| Query returns 0 rows | Check `df.count()` — did your CSV load correctly? Check column names match JOINs |
| `AnalysisException: Table not found` | Run the `createOrReplaceTempView` cell first |
| Cluster auto-terminates | Community Edition shuts down after 2 hours idle. Just restart it |

!!! warning "Save your work!"
    Databricks auto-saves notebooks, but export a backup: **File → Export → DBC Archive**. Upload the `.dbc` file with your Phase 3 submission.

---

**Next week:** We talk about what happens at real scale, the modern data stack, and you get presentation prep time. 🎯

# PySpark in 20 Minutes — Student Cheat Sheet

> Pure reference. No prose. Copy-paste and go.

---

## Setup

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window
```

---

## Load Data

```python
# From CSV
df = spark.read.csv("/FileStore/tables/my_table.csv", header=True, inferSchema=True)

# From Hive table
df = spark.table("is480.dim_student")

# Register as SQL table
df.createOrReplaceTempView("dim_student")
```

---

## Inspect Schema

| Action | Code |
|--------|------|
| Show schema tree | `df.printSchema()` |
| Column names | `df.columns` |
| Data types | `df.dtypes` |
| Row count | `df.count()` |
| First N rows | `df.show(10)` |
| Summary stats | `df.describe().show()` |
| Distinct count | `df.select("col").distinct().count()` |

---

## Select and Rename

```python
# Select columns
df.select("sname", "gpa")

# Select with alias
df.select(F.col("sname").alias("student_name"), "gpa")

# Add/replace column
df.withColumn("gpa_x100", F.col("gpa") * 100)

# Rename column
df.withColumnRenamed("sname", "student_name")

# Drop column
df.drop("load_date")
```

---

## Filter

```python
# Single condition
df.filter(F.col("gpa") > 3.5)
df.where("gpa > 3.5")          # SQL-style string also works

# Multiple conditions (use & for AND, | for OR)
df.filter((F.col("gpa") > 3.5) & (F.col("standing") == 4))

# IN list
df.filter(F.col("majorid").isin("CECS", "IS", "MATH"))

# NULL checks
df.filter(F.col("majorid").isNotNull())
df.filter(F.col("majorid").isNull())

# LIKE
df.filter(F.col("sname").like("S%"))
```

---

## Aggregations

```python
# Basic GROUP BY
df.groupBy("majorid").count()

# Multiple aggregations
df.groupBy("majorid").agg(
    F.count("*").alias("student_count"),
    F.avg("gpa").alias("avg_gpa"),
    F.min("gpa").alias("min_gpa"),
    F.max("gpa").alias("max_gpa"),
    F.sum("standing").alias("total_standing")
)

# Without grouping (whole table)
df.agg(F.count("*"), F.avg("gpa")).show()
```

| Function | PySpark |
|----------|---------|
| COUNT | `F.count("*")` or `F.count("col")` |
| SUM | `F.sum("col")` |
| AVG | `F.avg("col")` or `F.mean("col")` |
| MIN | `F.min("col")` |
| MAX | `F.max("col")` |
| COUNT DISTINCT | `F.countDistinct("col")` |

---

## Joins

```python
# Inner join (default)
df1.join(df2, "shared_key")
df1.join(df2, df1.course_key == df2.course_key)

# Left join
df1.join(df2, "shared_key", "left")

# Right join
df1.join(df2, "shared_key", "right")

# Full outer join
df1.join(df2, "shared_key", "full")

# Multi-column join
df1.join(df2, (df1.year == df2.year) & (df1.sem == df2.sem))
```

!!! warning "Ambiguous columns"
    If both DataFrames have a column with the same name (other than the join key), reference them as `df1.col` and `df2.col` or rename before joining.

---

## Window Functions

```python
from pyspark.sql.window import Window

# Define window
w = Window.partitionBy("majorid").orderBy(F.desc("gpa"))

# RANK
df.withColumn("rank", F.rank().over(w))

# DENSE_RANK
df.withColumn("dense_rank", F.dense_rank().over(w))

# ROW_NUMBER
df.withColumn("row_num", F.row_number().over(w))

# LAG (previous row)
df.withColumn("prev_gpa", F.lag("gpa", 1).over(w))

# LEAD (next row)
df.withColumn("next_gpa", F.lead("gpa", 1).over(w))

# Running SUM
w_running = Window.partitionBy("dept").orderBy("class_year").rowsBetween(Window.unboundedPreceding, Window.currentRow)
df.withColumn("running_total", F.sum("enrollments").over(w_running))

# NTILE
df.withColumn("quartile", F.ntile(4).over(w))
```

---

## Sorting

```python
# Ascending (default)
df.orderBy("gpa")

# Descending
df.orderBy(F.desc("gpa"))

# Multiple columns
df.orderBy("majorid", F.desc("gpa"))
```

---

## Output

```python
# Display in notebook
df.show()           # Default 20 rows, truncated
df.show(50, False)  # 50 rows, no truncation

# Collect to Python list
rows = df.collect()

# Convert to Pandas DataFrame
pdf = df.toPandas()

# Write to CSV
df.write.csv("/FileStore/output/results.csv", header=True, mode="overwrite")

# Save as Hive table
df.write.mode("overwrite").saveAsTable("is480.my_results")
```

---

## Common Gotchas

| Gotcha | Explanation | Fix |
|--------|------------|-----|
| **Lazy evaluation** | Transformations (select, filter, join) don't execute until an action (show, count, collect) | This is by design — Spark optimizes the whole chain |
| **Column references** | `"gpa"` (string) vs `F.col("gpa")` vs `df.gpa` | All three work in most cases; use `F.col()` when chaining |
| **Boolean operators** | Python `and`/`or` don't work with columns | Use `&` (and), `|` (or), `~` (not) with parentheses |
| **NULL sorting** | NULLs sort last by default in Spark | Use `F.col("x").asc_nulls_first()` if needed |
| **Column name conflicts** | After join, duplicate column names cause errors | Rename before joining or use `df1["col"]` syntax |
| **Show vs collect** | `show()` prints; `collect()` returns Python list | Use `show()` for inspection, `collect()` for processing |

---

## Oracle SQL vs PySpark — Side by Side

### 1. Filter Rows

=== "Oracle SQL"

    ```sql
    SELECT * FROM dim_student WHERE gpa > 3.5;
    ```

=== "PySpark"

    ```python
    df_dim_student.filter(F.col("gpa") > 3.5).show()
    ```

### 2. Aggregate with GROUP BY

=== "Oracle SQL"

    ```sql
    SELECT majorid, COUNT(*) AS cnt, AVG(gpa) AS avg_gpa
    FROM dim_student
    GROUP BY majorid;
    ```

=== "PySpark"

    ```python
    df_dim_student.groupBy("majorid").agg(
        F.count("*").alias("cnt"),
        F.avg("gpa").alias("avg_gpa")
    ).show()
    ```

### 3. JOIN Two Tables

=== "Oracle SQL"

    ```sql
    SELECT ds.sname, dc.ctitle
    FROM fact_enrollments f
    JOIN dim_student ds ON f.student_key = ds.student_key
    JOIN dim_course dc ON f.course_key = dc.course_key;
    ```

=== "PySpark"

    ```python
    (df_fact
     .join(df_dim_student, "student_key")
     .join(df_dim_course, "course_key")
     .select("sname", "ctitle")
     .show())
    ```

### 4. Ranking

=== "Oracle SQL"

    ```sql
    SELECT majorid, sname, gpa,
           RANK() OVER (PARTITION BY majorid ORDER BY gpa DESC) AS rnk
    FROM dim_student;
    ```

=== "PySpark"

    ```python
    w = Window.partitionBy("majorid").orderBy(F.desc("gpa"))
    df_dim_student.withColumn("rnk", F.rank().over(w)).show()
    ```

### 5. LAG (Previous Row)

=== "Oracle SQL"

    ```sql
    SELECT dept, class_year, enrollments,
           LAG(enrollments, 1) OVER (PARTITION BY dept ORDER BY class_year) AS prev
    FROM dept_yearly;
    ```

=== "PySpark"

    ```python
    w = Window.partitionBy("dept").orderBy("class_year")
    dept_yr.withColumn("prev", F.lag("enrollments", 1).over(w)).show()
    ```

# Apache Spark & Hive

!!! note "Coming in Weeks 13-15"
    This module covers the concepts and techniques needed for **Phase 3** of the semester project.

## What You'll Learn

- **Why distributed computing?** — What happens when your data outgrows a single database
- **Apache Spark** — Distributed processing engine, 100x faster than MapReduce
- **Spark SQL** — Write SQL that runs on distributed data
- **Apache Hive** — SQL interface over distributed file systems
- **PySpark** — Python API for Spark (connects to ML pipelines)
- **Databricks** — Free cloud platform for Spark (Community Edition)

## The Big Picture

```
Your Oracle DB                    The Hadoop Ecosystem
┌──────────────┐                  ┌──────────────────────────┐
│ Single server│                  │  Hundreds of machines    │
│ ~10K rows    │                  │  Billions of rows        │
│ Milliseconds │                  │                          │
│              │   Same SQL!      │  Spark SQL / HiveQL      │
│ SELECT ...   │ ─────────────►   │  SELECT ...              │
│ GROUP BY ... │                  │  GROUP BY ...             │
│              │                  │                          │
│ Oracle engine│                  │  Spark/Hive engine       │
│ (1 machine)  │                  │  (distributed cluster)   │
└──────────────┘                  └──────────────────────────┘
```

## Key Concepts

### Same Query, Three Ways

**Oracle PL/SQL:**
```sql
SELECT department_name, SUM(total_cost) as revenue
FROM fact_appointments f
JOIN dim_department d ON f.department_key = d.department_key
GROUP BY department_name
ORDER BY revenue DESC;
```

**Spark SQL (on Databricks):**
```sql
-- Looks identical! But runs on a distributed cluster
SELECT department_name, SUM(total_cost) as revenue
FROM fact_appointments f
JOIN dim_department d ON f.department_key = d.department_key
GROUP BY department_name
ORDER BY revenue DESC;
```

**PySpark (Python API):**
```python
from pyspark.sql import functions as F

(fact_df
    .join(dept_df, "department_key")
    .groupBy("department_name")
    .agg(F.sum("total_cost").alias("revenue"))
    .orderBy(F.desc("revenue"))
    .show())
```

### Why Spark Over Oracle at Scale?

| Aspect | Oracle (Single Node) | Spark (Distributed) |
|--------|---------------------|-------------------|
| **Data size** | GBs | PBs (petabytes) |
| **Processing** | 1 machine | Hundreds of machines in parallel |
| **Speed** | Fast for small data | Fast for massive data (in-memory) |
| **Cost** | Expensive licenses | Open source, pay for compute only |
| **Use case** | OLTP + small OLAP | Big data analytics, ML pipelines |

### Databricks Setup (Free)

1. Sign up at [community.cloud.databricks.com](https://community.cloud.databricks.com/)
2. Create a notebook (Python + SQL)
3. Upload CSV files from your Oracle star schema
4. Create Hive tables:

```sql
CREATE TABLE fact_appointments
USING CSV
OPTIONS (path '/FileStore/tables/fact_appointments.csv', 
         header 'true', inferSchema 'true');
```

5. Query with SparkSQL — same syntax as Oracle!

## Career Connection

!!! info "These skills are in demand"
    Netflix ($466K-$750K), Spotify, Uber, Airbnb — all list **Spark** and **Hive** as required or preferred skills for ML and data engineering roles. Learning them now puts you ahead.

## Resources

- [Databricks Community Edition](https://community.cloud.databricks.com/) (free)
- [Spark SQL Guide](https://spark.apache.org/docs/latest/sql-programming-guide.html)
- [Hive Language Manual](https://cwiki.apache.org/confluence/display/Hive/LanguageManual)
- [Project Spec — Phase 3](project-spec.md#phase-3-big-data-with-apache-spark-hive-30-points)

---

*Detailed content, hands-on tutorials, and Databricks walkthrough will be added as we cover this material in class.*

# IS480 — Semester Project: Enterprise Database & Analytics Platform

## Why This Project Matters

Before we talk about requirements, let's talk about **where this can take you**.

Here's a real job posting from **Netflix**, open right now:

!!! info "Research Scientist 5/6 – AI for Member Systems | Netflix | Remote"

    **Compensation: $466,000 – $750,000/year**

    *"Nice to have: Java, Scala, **Spark**, **Hive**, Jax, **Flink**, **Hadoop**"*

    The role involves building production-ready ML systems for personalization and recommendations at global scale. The data infrastructure that powers Netflix's recommendations? It runs on **the exact technologies you'll use in this project** — Oracle databases, Spark for large-scale processing, and Hive for analytical queries.

This isn't an anomaly. Data engineering and ML infrastructure roles at **Google, Meta, Amazon, Apple, and Spotify** all list Spark, Hive, and SQL expertise as core or preferred skills. These positions pay $200K–$750K+.

**The gap between where you are now and those roles is smaller than you think.** You already know SQL. You're learning PL/SQL (procedural database programming). This project adds the modern data stack — the same tools used at the companies listed above.

By the end of this project, you will have built a system that spans:

- **OLTP** (transactional database with PL/SQL business logic)
- **OLAP** (data warehouse with star schema and analytical queries)
- **Big Data** (Spark SQL and Hive on distributed infrastructure)

That's the full data pipeline. That's what gets you hired.

---

## Project Overview

**Teams:** 3–4 students

**Submission Deadline:** Week of April 20, 2026

**Deliverables:** Code + Documentation + 15-min Live Demo

Your team will build an **end-to-end database application** for a real or realistic organization. The project has three phases, each building on the last.

---

## Phase 1: PL/SQL Business Logic Package (40 points)

### Choose Your Organization

Pick a domain that interests your team. Examples:

| Domain | Key Entities | Sample Operations |
|--------|-------------|-------------------|
| Hospital | Patients, Doctors, Appointments, Prescriptions | Schedule appointment, assign doctor, check drug interactions |
| E-Commerce | Customers, Products, Orders, Inventory | Place order, process return, update stock, apply discount |
| University | Students, Courses, Professors, Rooms | Register student, assign grades, check prerequisites |
| Airline | Flights, Passengers, Bookings, Crew | Book flight, cancel reservation, assign crew |
| Gym/Fitness | Members, Trainers, Classes, Equipment | Register member, book class, track attendance |
| Restaurant Chain | Locations, Menu, Staff, Orders, Suppliers | Place order, manage inventory, schedule shifts |

You may choose your own domain — be creative. **The schema itself is not graded.** What's graded is the quality of your PL/SQL package.

### Requirements

Build a PL/SQL **package** (specification + body) that demonstrates mastery of every concept we covered:

| Requirement | Min. Count | What We're Looking For |
|-------------|:----------:|----------------------|
| **Procedures** | 3 | Real business operations (not trivial INSERT wrappers) |
| **Functions** | 3 | At least one computational function (like our `Grading()` function) |
| **Explicit Cursors** | 2 | `OPEN` / `FETCH` / `EXIT WHEN` / `CLOSE` pattern |
| **FOR Loops** | 2 | Cursor FOR loop or numeric FOR loop with SQL inside |
| **Exception Handling** | 3 | Mix of: predefined (`NO_DATA_FOUND`, `TOO_MANY_ROWS`), user-defined, and `RAISE_APPLICATION_ERROR` |
| **`%TYPE` / `%ROWTYPE`** | Throughout | All parameters and variables should use `%TYPE` where applicable |
| **`COUNT(*)` checks** | 2 | Existence validation before INSERT/UPDATE/DELETE |
| **Conditional Logic** | Throughout | `IF/ELSIF/ELSE` and/or `CASE` statements |
| **`DBMS_OUTPUT` + `RAISE_APPLICATION_ERROR`** | Both | Show you understand the difference: diagnostic output vs. error propagation |

### Grading Rubric — Phase 1

| Criteria | Points |
|----------|:------:|
| Package compiles and runs without errors | 8 |
| All required PL/SQL features present and correctly used | 12 |
| Business logic is meaningful (not trivial) | 8 |
| Code quality: naming, comments, organization | 6 |
| Test script that calls every procedure/function with sample output | 6 |
| **Total** | **40** |

### Deliverables

1. `schema.sql` — DDL to create your tables + seed data
2. `package_spec.sql` — Package specification
3. `package_body.sql` — Package body
4. `test_script.sql` — Calls every procedure/function, shows output
5. `phase1_report.md` — Brief write-up: domain description, what each procedure/function does, any design decisions

---

## Phase 2: Data Warehouse & Analytics (30 points)

### What You'll Build

Transform your OLTP database into an **analytical data warehouse** using the star schema pattern.

### Concepts

```
OLTP (Phase 1)                    OLAP (Phase 2)
┌──────────────┐                  ┌──────────────────┐
│ Normalized   │   ETL Process    │ Star Schema      │
│ Tables       │ ──────────────►  │                  │
│ (3NF)        │   Extract        │  ┌──────┐        │
│              │   Transform      │  │ FACT │        │
│ Patients     │   Load           │  │ table│        │
│ Doctors      │                  │  └──┬───┘        │
│ Appointments │                  │  ┌──┼──┐         │
│ Prescriptions│                  │  DIM DIM DIM     │
└──────────────┘                  └──────────────────┘
```

### Requirements

| Requirement | Details |
|-------------|---------|
| **Star Schema** | 1 fact table + 3–5 dimension tables |
| **ETL Procedures** | PL/SQL procedures that extract from your OLTP tables, transform (aggregate, clean, derive), and load into the warehouse |
| **Analytical Queries** | 5+ queries using: `GROUP BY`, `ROLLUP` or `CUBE`, `RANK()`/`DENSE_RANK()`, subqueries, and `CASE` |
| **Documentation** | Star schema diagram + explanation of each dimension |

### Example: Hospital Domain

**Fact Table:** `fact_appointments`
```sql
CREATE TABLE fact_appointments (
    appointment_key   NUMBER PRIMARY KEY,
    patient_key       NUMBER REFERENCES dim_patient,
    doctor_key        NUMBER REFERENCES dim_doctor,
    date_key          NUMBER REFERENCES dim_date,
    department_key    NUMBER REFERENCES dim_department,
    appointment_count NUMBER,
    total_cost        NUMBER(10,2),
    prescription_count NUMBER
);
```

**Dimension Tables:** `dim_patient`, `dim_doctor`, `dim_date`, `dim_department`

**Analytical Queries:**
```sql
-- Top 5 doctors by revenue, by department
SELECT d.department_name, doc.doctor_name,
       SUM(f.total_cost) as revenue,
       RANK() OVER (PARTITION BY d.department_name ORDER BY SUM(f.total_cost) DESC) as rank
FROM fact_appointments f
JOIN dim_doctor doc ON f.doctor_key = doc.doctor_key
JOIN dim_department d ON f.department_key = d.department_key
GROUP BY d.department_name, doc.doctor_name;
```

### Grading Rubric — Phase 2

| Criteria | Points |
|----------|:------:|
| Star schema design (fact + dimensions) | 8 |
| ETL procedures work correctly | 8 |
| 5+ analytical queries with variety | 8 |
| Documentation: schema diagram + explanations | 6 |
| **Total** | **30** |

### Deliverables

1. `warehouse_schema.sql` — DDL for star schema
2. `etl_procedures.sql` — PL/SQL procedures for ETL
3. `analytical_queries.sql` — 5+ analytical queries with comments
4. `phase2_report.md` — Star schema diagram, dimension descriptions, sample query results

---

## Phase 3: Big Data with Apache Spark & Hive (30 points)

### Why Spark and Hive?

Your Oracle database handles thousands of rows beautifully. But what happens when your hospital has **100 million appointment records**? Or your e-commerce platform processes **1 billion transactions per year**?

That's where **Apache Spark** and **Apache Hive** come in:

| Technology | What It Is | Why It Matters |
|-----------|-----------|----------------|
| **Apache Spark** | Distributed computing engine | Processes massive datasets across clusters of machines. 100x faster than traditional MapReduce. |
| **Spark SQL** | SQL interface for Spark | Write SQL queries that run on distributed data. Same syntax you know, infinite scale. |
| **Apache Hive** | Data warehouse on Hadoop | SQL-like queries (`HiveQL`) over massive datasets stored in distributed file systems. |
| **Databricks** | Cloud platform for Spark | Free Community Edition. Notebooks, clusters, storage — all managed. |

These are **production technologies at Netflix, Uber, Airbnb, Spotify, and every major tech company.** Netflix's recommendation engine processes petabytes of viewing data using Spark. Uber's surge pricing runs on Spark Streaming. Airbnb's search ranking uses Spark ML pipelines.

### What You'll Build

Take your data warehouse from Phase 2 and **port it to Spark SQL and Hive** on Databricks.

### Setup (Free)

1. Sign up for [Databricks Community Edition](https://community.cloud.databricks.com/) (free, no credit card)
2. Create a notebook (Python + SQL)
3. Upload your warehouse data as CSV files

### Requirements

| Requirement | Details |
|-------------|---------|
| **Load Data into Spark** | Read your CSV exports into Spark DataFrames |
| **Create Hive Tables** | Register your fact + dimension tables as Hive tables |
| **Rewrite 3+ Analytical Queries** | Take 3+ queries from Phase 2 and rewrite them in SparkSQL/HiveQL |
| **Side-by-Side Comparison** | Show the same query in PL/SQL vs. SparkSQL — explain what's different and why |
| **Scale Discussion** | Written analysis: What changes when your dataset grows from 1,000 rows to 1 billion? Why does Spark handle it better than Oracle on a single machine? |

### Example: Same Query, Three Ways

**PL/SQL (Oracle):**
```sql
-- Runs on a single Oracle instance
SELECT department_name, SUM(total_cost) as revenue
FROM fact_appointments f
JOIN dim_department d ON f.department_key = d.department_key
GROUP BY department_name
ORDER BY revenue DESC;
```

**Spark SQL (Databricks):**
```sql
-- Runs distributed across a Spark cluster
SELECT department_name, SUM(total_cost) as revenue
FROM fact_appointments f
JOIN dim_department d ON f.department_key = d.department_key
GROUP BY department_name
ORDER BY revenue DESC;
```

**PySpark (programmatic):**
```python
# Same query, Python API — enables ML pipeline integration
(fact_df
    .join(dept_df, "department_key")
    .groupBy("department_name")
    .agg(F.sum("total_cost").alias("revenue"))
    .orderBy(F.desc("revenue"))
    .show())
```

Notice: **the SQL is almost identical.** The power isn't in the syntax — it's in the execution engine. Spark distributes that query across hundreds of machines automatically.

### Bonus Points (up to +5)

Ambitious teams can earn bonus points for:

- **Spark Structured Streaming:** Simulate a real-time data stream (e.g., incoming orders) and process it with Spark Streaming
- **NoSQL Integration:** Load data into MongoDB or Cassandra alongside Spark, compare query patterns
- **Visualization:** Build charts/dashboards from your Spark query results using matplotlib or Databricks built-in visualizations
- **Apache Flink:** Implement a streaming pipeline with Flink (this is cutting-edge — Netflix and Uber use this)

### Grading Rubric — Phase 3

| Criteria | Points |
|----------|:------:|
| Data successfully loaded into Spark + Hive tables created | 6 |
| 3+ analytical queries rewritten in SparkSQL/HiveQL | 8 |
| Side-by-side PL/SQL vs. SparkSQL comparison with explanation | 8 |
| Scale discussion (what happens at 1B rows?) | 4 |
| Notebook is clean, documented, and reproducible | 4 |
| **Total** | **30** |
| **Bonus** | **+5** |

### Deliverables

1. Databricks notebook (exported as `.html` or `.ipynb`)
2. CSV data files used
3. `phase3_report.md` — Comparison analysis, scale discussion, what you learned

---

## Final Presentation (Required, not separately graded)

**15 minutes per team** in the last week of class.

Structure:

1. **Domain & Schema** (2 min) — What organization, what problem
2. **PL/SQL Demo** (5 min) — Run your package, show it working
3. **Data Warehouse** (3 min) — Star schema, ETL, analytical insights
4. **Spark/Hive** (3 min) — Live Databricks notebook, side-by-side comparison
5. **Reflections** (2 min) — What was hard, what you learned, what you'd do differently

---

## Submission

**Due: Week of April 20, 2026**

Submit via Canvas as a **single ZIP file** containing:

```
team_name/
├── phase1/
│   ├── schema.sql
│   ├── package_spec.sql
│   ├── package_body.sql
│   ├── test_script.sql
│   └── phase1_report.md
├── phase2/
│   ├── warehouse_schema.sql
│   ├── etl_procedures.sql
│   ├── analytical_queries.sql
│   └── phase2_report.md
├── phase3/
│   ├── notebook.html (or .ipynb)
│   ├── data/ (CSV files)
│   └── phase3_report.md
└── README.md (team members, domain, setup instructions)
```

---

## Grading Summary

| Phase | Weight | Focus |
|-------|:------:|-------|
| Phase 1: PL/SQL Package | 40 pts | Everything from Weeks 1-6 |
| Phase 2: Data Warehouse | 30 pts | Star schema, ETL, analytical SQL |
| Phase 3: Spark & Hive | 30 pts | Modern data stack, distributed computing |
| Bonus | +5 pts | Streaming, NoSQL, visualization, Flink |
| **Total** | **100 (+5)** |

---

## Resources

### PL/SQL (Phase 1)
- [Course Wiki — Lab Solutions](lab1-solutions.md)
- [Schema & Dataset](lab-schema.md)
- Oracle PL/SQL documentation

### Data Warehousing (Phase 2)
- Kimball, R. — *The Data Warehouse Toolkit* (star schema bible)
- Oracle `ROLLUP`, `CUBE`, and analytical functions documentation

### Spark & Hive (Phase 3)
- [Databricks Community Edition](https://community.cloud.databricks.com/) (free signup)
- [Spark SQL Guide](https://spark.apache.org/docs/latest/sql-programming-guide.html)
- [Hive Language Manual](https://cwiki.apache.org/confluence/display/Hive/LanguageManual)
- PySpark DataFrame API documentation

### Career
- [Netflix Research Scientist role — $466K-$750K](https://explore.jobs.netflix.net/careers/job/790300763295) — Lists Spark, Hive, Flink, Hadoop as preferred skills
- Search "data engineer" or "ML infrastructure" on LinkedIn — notice how often Spark and SQL appear

---

## A Note From Prof. Pineda

The technologies in this project aren't academic exercises. They're the foundation of how the world's most valuable companies handle data at scale. The students who master this material — PL/SQL for business logic, star schemas for analytics, Spark/Hive for scale — are the ones who land the roles that pay $200K+ right out of the gate.

You're closer than you think. Build something you're proud of.

---

*IS480 — Advanced Database Management | CSULB | Spring 2026 | Prof. Jose Pineda*

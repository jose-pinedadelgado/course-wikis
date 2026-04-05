# Week 13: Scale, Architecture and Presentations

> **Phase 3 due next week.** Today: a short lecture on distributed systems, a tour of the modern data stack, and presentation prep.

---

## Part A: What Happens at Scale? (20 min)

You've now built the same queries in Oracle and Spark. On your lab data (hundreds of rows), Oracle is faster. So why does Spark exist?

### The Single-Node Bottleneck

Your Oracle RDS instance is one machine. Every query runs on:

- **1 CPU** (or a few cores)
- **Limited RAM** (8-64 GB typically)
- **1 disk** (even SSDs have throughput limits)

This works great for thousands or even millions of rows. But consider:

| Company | Data Volume | Queries Per Day |
|---------|-----------|----------------|
| CSULB registrar | ~500K enrollments | Dozens |
| Netflix | 15+ PB of viewing data | Millions |
| Google | Exabytes (1 EB = 1M TB) | Billions |
| Walmart | 2.5 PB of transactions/hour | Continuous |

At Netflix scale, your single Oracle server would take **days** to answer one analytical query.

### Vertical vs Horizontal Scaling

| Strategy | What It Means | Limit |
|----------|--------------|-------|
| **Vertical scaling** (scale up) | Bigger machine: more CPU, RAM, faster disk | There's a biggest machine you can buy (~$100K) |
| **Horizontal scaling** (scale out) | More machines working together | Virtually unlimited — just add nodes |

Oracle = vertical scaling (one powerful machine).  
Spark = horizontal scaling (many machines in parallel).

### How Spark Distributes Work

When you run `spark.sql("SELECT dept, COUNT(*) FROM fact GROUP BY dept")`:

```
  Your Query
      │
      ▼
  ┌─────────────────────────────────────────┐
  │            DRIVER (coordinator)          │
  │  Parses SQL → Creates execution plan    │
  └──────────────────┬──────────────────────┘
                     │ distributes work
         ┌───────────┼───────────┐
         ▼           ▼           ▼
   ┌──────────┐ ┌──────────┐ ┌──────────┐
   │ Worker 1 │ │ Worker 2 │ │ Worker 3 │
   │          │ │          │ │          │
   │ Partition│ │ Partition│ │ Partition│
   │ A-F data │ │ G-M data │ │ N-Z data │
   │          │ │          │ │          │
   │ Local    │ │ Local    │ │ Local    │
   │ COUNT(*) │ │ COUNT(*) │ │ COUNT(*) │
   └────┬─────┘ └────┬─────┘ └────┬─────┘
        │             │             │
        └──────┬──────┘─────────────┘
               ▼
         Final merge
         (combine counts)
               │
               ▼
          Result to you
```

**Key concepts:**

- **Partitions** — data split into chunks across machines
- **Executors** — worker processes that run on each machine
- **Shuffle** — redistribution of data between workers (expensive!)

### MapReduce in 30 Seconds

Spark evolved from **MapReduce**, the original distributed processing model:

1. **Map** — each worker processes its chunk independently (filter, transform)
2. **Shuffle** — data is redistributed by key (e.g., group by department)
3. **Reduce** — each worker aggregates its group (COUNT, SUM)

Spark improves on MapReduce by keeping data **in memory** instead of writing to disk between steps. This makes it 10-100x faster than original Hadoop MapReduce.

### Real Numbers

| Metric | Oracle (single node) | Spark (100-node cluster) |
|--------|---------------------|-------------------------|
| Data capacity | GBs to low TBs | Petabytes |
| Query on 1B rows | Minutes to hours | Seconds to minutes |
| Concurrent users | Dozens | Thousands |
| Cost to scale | Buy bigger server ($$$) | Add more commodity machines ($) |
| Best for | OLTP + moderate OLAP | Large-scale OLAP, ML, ETL |

### The CAP Theorem (60-Second Version)

In any distributed system, you can only guarantee two out of three:

- **C**onsistency — every read gets the latest write
- **A**vailability — every request gets a response
- **P**artition tolerance — system works even if network splits

| System | Chooses | Sacrifices |
|--------|---------|-----------|
| Oracle (single node) | C + A | P (not distributed) |
| Spark | A + P | C (eventual consistency OK for analytics) |
| Banking systems | C + P | A (rather deny than be wrong) |

!!! info "Why this matters"
    You don't need to memorize CAP for this class. But if you ever interview for a data engineering role, knowing this exists puts you ahead of 90% of candidates.

---

## Part B: The Modern Data Stack (10 min)

Where do all these tools fit? Here's the landscape you'll encounter in job descriptions:

### The Architecture

```
  Data Sources          Processing           Storage            Consumption
 ┌────────────┐      ┌──────────────┐     ┌──────────────┐    ┌────────────┐
 │ PostgreSQL │─┐    │              │     │              │    │ Dashboards │
 │ MySQL      │ │    │  Spark       │     │  Snowflake   │    │ (Tableau)  │
 │ Oracle     │─┤───▶│  Flink       │────▶│  BigQuery    │───▶│            │
 │ APIs       │ │    │  dbt         │     │  Redshift    │    │ ML Models  │
 │ Files      │─┘    │              │     │  Delta Lake  │    │ Reports    │
 └────────────┘      └──────────────┘     └──────────────┘    └────────────┘
                           │
                     ┌─────┴─────┐
                     │ Airflow   │  (Orchestration —
                     │ Prefect   │   schedules the
                     │ Dagster   │   whole pipeline)
                     └───────────┘
```

### Tool Categories

| Category | Tools | What It Does | You Learned |
|----------|-------|-------------|------------|
| **OLTP** | PostgreSQL, MySQL, Oracle | Run the business (transactions) | Weeks 1-9 |
| **OLAP / Warehouse** | Snowflake, BigQuery, Redshift | Analyze the business (queries) | Weeks 10-11 |
| **Processing** | Spark, Flink, dbt | Transform data at scale | Weeks 12-13 |
| **Orchestration** | Airflow, Prefect, Dagster | Schedule and monitor pipelines | (Mentioned) |
| **Storage** | S3, HDFS, Delta Lake | Store raw and processed data | (Mentioned) |
| **Visualization** | Tableau, Power BI, Looker | Dashboards and reports | (Not covered) |

### Reading Job Descriptions

When you see a job posting like this:

> *"Data Engineer — Experience with SQL, Python, Spark, Airflow. Snowflake preferred. dbt a plus."*

You now know what **every one of those tools does**:

- **SQL** — you've written it for 13 weeks
- **Python** — you used PySpark
- **Spark** — you ran queries on it in Databricks
- **Airflow** — schedules ETL jobs (like your PL/SQL procedures, but automated)
- **Snowflake** — cloud data warehouse (like Oracle OLAP, but cloud-native)
- **dbt** — transforms data using SQL templates (like your ETL, but version-controlled)

!!! tip "You're more prepared than you think"
    After this course, you understand the fundamentals behind every tool in the modern data stack. The specific tools change every few years — the concepts (star schemas, ETL, analytical SQL, distributed processing) have been stable for decades.

---

## Part C: Presentation Guidelines

### Format Reminder

From the [project specification](project-spec.md):

- **15 minutes** per team (10 min presentation + 5 min Q&A)
- **Live demo** required — show your actual system working
- All team members must present

### Tips for a Great Demo

**Before presenting:**

- [ ] Test your Oracle RDS connection from the classroom (sometimes campus WiFi blocks ports)
- [ ] Have your Databricks notebook loaded and cluster running **before** your slot
- [ ] Export key outputs as screenshots in case of connection issues
- [ ] Practice the 10-minute version and time yourselves

**During the presentation:**

| Do | Don't |
|----|-------|
| Start with the business problem your schema solves | Start with technical setup details |
| Show your star schema diagram early | Read code line by line |
| Run 2-3 queries live (have them ready to go) | Type queries live from scratch |
| Explain results in business terms | Just show raw output |
| Show the Oracle vs Spark comparison side by side | Switch back and forth confusingly |
| Have a backup plan (screenshots) if connection fails | Panic if something breaks |

**Presentation structure (suggested):**

1. **Problem statement** (1 min) — What domain? What questions?
2. **Star schema design** (2 min) — Show diagram, explain fact + dimensions
3. **ETL demo** (2 min) — Show one ETL procedure running, verify data
4. **Oracle analytical queries** (2 min) — Run 2-3 queries, explain results
5. **Spark comparison** (2 min) — Same queries in Databricks, show side-by-side
6. **Key observations** (1 min) — What did you learn about Oracle vs Spark?

### Common Mistakes to Avoid

!!! warning "These will cost you points"
    - **No live demo** — showing only slides and screenshots
    - **Unequal participation** — one person presents everything
    - **Going over time** — practice and trim; 15 minutes is strict
    - **No star schema diagram** — this is the conceptual foundation
    - **Queries that return errors** — test everything before presenting
    - **No Oracle vs Spark comparison** — this is the whole point of Phase 3

### Grading Emphasis

| Component | Weight | What We're Looking For |
|-----------|--------|----------------------|
| Star schema design | 25% | Proper fact/dimension structure, justified choices |
| ETL implementation | 20% | Working procedures, error handling, idempotent |
| Analytical queries | 25% | Correct use of ROLLUP/CUBE/RANK/LAG, meaningful results |
| Spark comparison | 20% | Side-by-side output, thoughtful observations |
| Presentation quality | 10% | Clear delivery, time management, team participation |

---

## Rest of Class: Project Work Time

Use the remaining time to:

1. **Finalize your star schema** — make sure DDL runs cleanly
2. **Test ETL procedures** — load data, verify counts
3. **Run all queries** — in Oracle AND Spark
4. **Build your comparison table** — side-by-side format
5. **Rehearse your presentation** — time it, assign parts

!!! tip "Office hours"
    I'm available this week and next for project help. Come with specific questions — "my ETL procedure gives wrong counts" is better than "I need help."

---

**Next week:** Presentations begin. Be ready. You've got this. 💪

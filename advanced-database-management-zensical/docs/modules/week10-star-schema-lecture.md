# Week 10: Data Warehousing & Star Schema Design

> **Reading due this week:** Kimball, *The Data Warehouse Toolkit* — Chapter 1 (pp. 1-35)  
> **Reading for next week:** Kimball Ch 2 (pp. 37-68) + Ch 3 (pp. 69-100)

---

## Part 1: Why Data Warehouses? (20 min)

### The Problem

You've spent 9 weeks building transactional databases with PL/SQL. These OLTP systems are great at their job — processing one transaction at a time, fast.

But try answering this question from your CEO:

> *"Show me revenue by department, by quarter, for the last 3 years — and compare it to the same quarters last year."*

In your normalized 3NF database, this requires:

- 5+ JOINs across normalized tables
- Aggregations across millions of rows
- The query competes with real-time transactions for resources
- It takes minutes (or hours) on large datasets

**This is the wrong tool for the job.** You need a system designed for analysis, not transactions.

### Two Worlds of Data (Kimball Ch 1, pp. 2-3)

> *"The operational systems are where you put the data in, and the DW/BI system is where you get the data out."* — Kimball

| | OLTP (Operational) | OLAP (Data Warehouse) |
|--|--------------------|-----------------------|
| **Purpose** | Run the business day-to-day | Analyze the business for decisions |
| **Users** | Clerks, apps, front-line workers | Managers, analysts, executives |
| **Operations** | INSERT/UPDATE/DELETE one row | SELECT + GROUP BY over millions |
| **Schema** | Normalized (3NF) | Denormalized (Star Schema) |
| **Data** | Current state — overwrites old data | Historical snapshots — never deletes |
| **Optimization** | Fast writes, data integrity | Fast reads, query performance |
| **Volume** | GBs (current data) | TBs to PBs (years of history) |

*(This maps to Slide 2 of Chapter 9 textbook slides and Kimball Ch 1 "Different Worlds")*

### Data Characteristics: Transient vs. Periodic

*(From Chapter 9 slides, Slides 13-16)*

- **Transient data (OLTP):** Changes are written over previous values. When a customer updates their address, the old address is gone.
- **Periodic data (Warehouse):** Data is **never overwritten**. Every state is preserved as a historical snapshot. You can always go back and see what the data looked like at any point in time.

### Real-World Architecture

*(Expanded from Chapter 9 Slide 6 — 3-Layer Architecture)*

```
         OLTP Source Systems                    Data Warehouse
    ┌──────────────────────┐               ┌─────────────────────┐
    │  Hospital EHR        │──┐            │                     │
    └──────────────────────┘  │   ETL      │   Star Schema       │
    ┌──────────────────────┐  ├───────────►│                     │
    │  Billing System      │──┤  Extract   │  ┌──────────┐       │
    └──────────────────────┘  │  Transform │  │  FACT    │       │──► Reports
    ┌──────────────────────┐  │  Load      │  │  table   │       │──► Dashboards
    │  Scheduling App      │──┘            │  └────┬─────┘       │──► ML Models
    └──────────────────────┘               │  ┌────┼────┐        │
                                           │  DIM  DIM  DIM      │
                                           └─────────────────────┘
```

**Netflix** processes petabytes of viewing data in their warehouse for recommendations.  
**Walmart** loads 1+ million transactions/hour into their warehouse.  
**Amazon** uses warehousing to analyze billions of purchase events for pricing and logistics.

### Warehouse vs. Data Mart

*(From Chapter 9 Slides 8-10)*

| | Data Warehouse | Data Mart |
|--|---------------|-----------|
| **Scope** | Enterprise-wide | One department or subject |
| **Data** | All subjects, many sources | One subject, few sources |
| **Size** | Large | Starts small |
| **Lifecycle** | Long-lived | Shorter-lived |
| **Planning** | Centralized, planned | Decentralized, organic |

A data mart is essentially a **mini-warehouse focused on one business area** (e.g., just sales, just HR).

---

## Part 2: Star Schema Design (25 min)

### Kimball's 4-Step Design Process

*(Kimball Ch 2, pp. 38-40 — the foundation of every dimensional model)*

| Step | Question to Answer | Example (Hospital) |
|------|-------------------|--------------------|
| **1. Select the business process** | What activity generates measurable events? | Patient appointments |
| **2. Declare the grain** | What does ONE row represent? | One appointment |
| **3. Identify the dimensions** | Who, what, where, when, how? | Patient, Doctor, Date, Department |
| **4. Identify the facts** | What numbers do we measure? | Cost, wait time, prescription count |

> ⚠️ **Kimball's #1 warning:** The most common mistake is skipping Step 2 (grain). If the grain isn't defined, the entire design rests on quicksand.

### Star Schema Anatomy

*(From Chapter 9 Slide 19-20 and Kimball Ch 1, pp. 16-17)*

```
                        ┌─────────────────┐
                        │   dim_date      │
                        │─────────────────│
                        │ date_key     PK │
                        │ full_date       │
                        │ day_of_week     │
                        │ month, quarter  │
                        │ year            │
                        │ is_holiday      │
                        └────────┬────────┘
                                 │
┌─────────────────┐   ┌─────────┴──────────────┐   ┌─────────────────┐
│  dim_patient    │   │   fact_appointments    │   │ dim_department  │
│─────────────────│   │────────────────────────│   │─────────────────│
│ patient_key  PK ├───┤ appointment_key    PK  ├───┤ dept_key     PK │
│ name            │   │ patient_key        FK  │   │ dept_name       │
│ age_group       │   │ doctor_key         FK  │   │ building        │
│ gender          │   │ date_key           FK  │   │ floor           │
│ insurance_type  │   │ dept_key           FK  │   └─────────────────┘
│ zip_code        │   │                        │
└─────────────────┘   │ ── Measures ──         │   ┌─────────────────┐
                      │ visit_count            │   │  dim_doctor     │
                      │ total_cost             │   │─────────────────│
                      │ wait_time_min          │───┤ doctor_key   PK │
                      │ prescription_count     │   │ doctor_name     │
                      └────────────────────────┘   │ specialty       │
                                                   │ hire_year       │
                                                   └─────────────────┘
```

It's called a **"star"** because the fact table sits in the center with dimensions radiating outward.

### Fact Table Rules

*(Kimball Ch 2, pp. 41-45)*

| Rule | Details |
|------|---------|
| One row = one event | Each row represents a real-world measurement |
| Foreign keys to dimensions | One FK per dimension table |
| Numeric measures only | Numbers you want to SUM, AVG, COUNT |
| Uniform grain | Never mix different grains in the same table |
| No descriptive text | Text belongs in dimensions, not facts |

**Three types of facts:**

| Type | Can SUM across... | Example |
|------|-------------------|---------|
| **Additive** | All dimensions | Revenue, quantity, cost |
| **Semi-additive** | Some (not time) | Account balance |
| **Non-additive** | None — compute from parts | Unit price, ratios, margins |

> 💡 **Kimball's rule:** *"Store the ratio's components, not the ratio. Calculate ratio of the sums, not sum of the ratios."*

**Example:** You sold 1 widget at $1.00 and 4 widgets at $0.50 each.
- ✅ Correct average price: Total revenue ($3.00) ÷ Total quantity (5) = **$0.60**
- ❌ Wrong: Average of unit prices ($1.00 + $0.50) ÷ 2 = $0.75

### Dimension Table Rules

*(Kimball Ch 2, pp. 46-50)*

| Rule | Details |
|------|---------|
| **Surrogate keys** | Use simple integers (1, 2, 3...) — NOT business keys |
| **Wide and flat** | Many descriptive columns |
| **Denormalized** | Flatten hierarchies (product + brand + category in one table) |
| **Text-heavy** | Use "Cardiology", not "DEPT-04" |
| **Relatively small** | Thousands to low millions of rows |

**Why surrogate keys?** *(Chapter 9 Slide 22)*
- Business keys change (employee rehired, product renumbered)
- Multiple source systems have conflicting keys
- Need multiple rows per entity for change tracking (SCD)

### The Date Dimension: Everyone Needs One

*(Kimball Ch 3, pp. 79-83)*

Pre-populate with 10-20 years of days (~7,300 rows). Every star schema has one.

```sql
CREATE TABLE dim_date (
    date_key       NUMBER PRIMARY KEY,    -- YYYYMMDD (e.g., 20260318)
    full_date      DATE NOT NULL,
    day_of_week    VARCHAR2(10),           -- 'Monday', 'Tuesday', ...
    day_of_month   NUMBER,
    month_num      NUMBER,
    month_name     VARCHAR2(10),
    quarter        NUMBER,                 -- 1, 2, 3, 4
    year           NUMBER,
    is_weekend     CHAR(1),               -- 'Y' or 'N'
    is_holiday     CHAR(1)
);
```

**Why not just use a DATE column in the fact table?**

Because you can't `GROUP BY is_holiday` or `WHERE fiscal_quarter = 'FQ3'` with a raw date. The dimension pre-computes these attributes for fast, flexible analysis.

### Estimating Fact Table Size

*(Chapter 9 Slide 26 & Kimball Ch 3, p. 79)*

```
Total rows = unique values per dimension₁ × dimension₂ × ... × fill rate

Example (Hospital):
  500 doctors × 10,000 patients × 365 days × 20% fill rate
  = 365,000,000 rows/year

Size in bytes = rows × columns × avg bytes/column
  = 365M × 8 columns × 15 bytes = ~44 GB/year
```

This is why warehouses get BIG — and why we'll need Spark in a few weeks.

---

## Part 3: Star vs Snowflake (10 min)

*(Chapter 9 Slides 31-33 & Kimball Ch 2, p. 50)*

**Snowflaking** = normalizing dimension tables by breaking them into sub-tables.

```
STAR (recommended):                SNOWFLAKE:
┌──────────────────┐               ┌──────────────────┐
│   dim_product    │               │   dim_product    │
│──────────────────│               │──────────────────│
│ product_key   PK │               │ product_key   PK │
│ product_name     │               │ product_name     │
│ brand_name       │ ←flattened    │ brand_key      FK│──→ dim_brand
│ category_name    │ ←flattened    │ category_key   FK│──→ dim_category
│ department_name  │ ←flattened    └──────────────────┘
└──────────────────┘                        ↓ More JOINs!
```

| | Star | Snowflake |
|--|------|-----------|
| Query speed | Faster ✅ | Slower (more JOINs) |
| Ease of use | Simpler ✅ | More complex |
| Storage | More (redundancy) | Less |
| Kimball says | **Use this** | Use sparingly |

> 💡 The storage savings from snowflaking are negligible — dimension tables are tiny compared to the massive fact table. Don't sacrifice query performance for trivial storage savings.

---

## Part 4: Slowly Changing Dimensions (10 min)

*(Chapter 9 Slide 35 & Kimball Ch 2, pp. 53-56)*

What happens when a doctor changes departments? A customer moves to a new state?

| SCD Type | Strategy | History? | Example |
|----------|----------|----------|---------|
| **Type 1** | Overwrite old value | ❌ Lost | Update Dr. Park's dept to Neurology |
| **Type 2** | Add new row (new surrogate key) | ✅ Full | New row for Dr. Park in Neurology; old row preserved |
| **Type 3** | Add "previous_value" column | Partial | `current_dept` = Neurology, `previous_dept` = Cardiology |

**Type 2 is the most common** — it preserves full history:

| doctor_key | doctor_id | name | department | effective_date | expiration_date | is_current |
|-----------|-----------|------|------------|---------------|----------------|-----------|
| 101 | D-5432 | Dr. Park | Cardiology | 2020-01-15 | 2025-06-30 | N |
| 247 | D-5432 | Dr. Park | Neurology | 2025-07-01 | 9999-12-31 | Y |

Historical appointments link to key **101** (Cardiology era). New appointments link to key **247** (Neurology era). History is preserved!

---

## Part 5: ETL — Extract, Transform, Load (15 min)

*(Chapter 9 Slides 42-50 & Kimball Ch 1, pp. 19-21)*

### The ETL Pipeline

```
  EXTRACT                TRANSFORM                    LOAD
┌──────────────┐    ┌───────────────────┐     ┌────────────────┐
│ Read from    │    │ Clean & reshape   │     │ Insert into    │
│ OLTP tables  │───►│ • Deduplicate     │────►│ star schema    │
│ (SELECT)     │    │ • Standardize     │     │ • Dims FIRST   │
└──────────────┘    │ • Derive fields   │     │ • Facts SECOND │
                    │ • Handle NULLs    │     │ • Index        │
                    │ • Map surr. keys  │     └────────────────┘
                    └───────────────────┘
```

### Data Quality Issues

*(Chapter 9 Slides 46-50 — Transformation Functions)*

| Problem | Example | Solution |
|---------|---------|----------|
| Inconsistent keys | Same customer, different IDs across systems | Surrogate key mapping |
| Synonyms | "CA" vs "California" vs "Calif." | Standardize during Transform |
| Missing data | NULL insurance type | Default "Unknown" dimension row |
| Duplicate records | Same appointment entered twice | Deduplication logic |
| Format mismatches | "03/18/2026" vs "2026-03-18" vs "18-MAR-26" | Canonical format conversion |

### ETL in PL/SQL (Your Phase 2!)

For this course, ETL = PL/SQL procedures you already know how to write:

```sql
-- Load a dimension table
CREATE OR REPLACE PROCEDURE load_dim_doctor AS
BEGIN
    DELETE FROM dim_doctor;
    
    INSERT INTO dim_doctor (doctor_key, doctor_name, specialty, department, hire_year)
    SELECT d.doctor_id,
           d.name,
           s.spec_name,        -- JOIN to denormalize
           dept.dept_name,     -- JOIN to denormalize
           EXTRACT(YEAR FROM d.hire_date)
    FROM Doctors d
    JOIN Specialties s ON d.specialty_id = s.spec_id
    JOIN Departments dept ON d.department_id = dept.dept_id;
    
    COMMIT;
    DBMS_OUTPUT.PUT_LINE('Loaded ' || SQL%ROWCOUNT || ' rows into dim_doctor');
END;
/

-- Load the fact table (AFTER dimensions)
CREATE OR REPLACE PROCEDURE load_fact_appointments AS
BEGIN
    DELETE FROM fact_appointments;
    
    INSERT INTO fact_appointments (
        appointment_key, patient_key, doctor_key, date_key, dept_key,
        visit_count, total_cost
    )
    SELECT a.appt_id,
           a.patient_id,
           a.doctor_id,
           TO_NUMBER(TO_CHAR(a.appt_date, 'YYYYMMDD')),  -- date surrogate key
           d.department_id,
           1,                    -- each row = 1 visit
           a.cost
    FROM Appointments a
    JOIN Doctors d ON a.doctor_id = d.doctor_id
    WHERE a.status = 'COMPLETED';
    
    COMMIT;
    DBMS_OUTPUT.PUT_LINE('Loaded ' || SQL%ROWCOUNT || ' facts');
END;
/

-- Master ETL: run everything in order
CREATE OR REPLACE PROCEDURE run_full_etl AS
BEGIN
    DBMS_OUTPUT.PUT_LINE('=== Starting ETL ===');
    load_dim_date;           -- dimensions first!
    load_dim_patient;
    load_dim_doctor;
    load_dim_department;
    load_fact_appointments;  -- facts last (need dimension FKs)
    DBMS_OUTPUT.PUT_LINE('=== ETL Complete ===');
END;
/
```

**Key principle:** Always load dimensions FIRST, then facts.

---

## Part 6: Analytical SQL Preview (10 min)

Once the warehouse is loaded, you unlock powerful analytical queries. We'll go deep on these **next week** — here's a taste:

### ROLLUP — Automatic Subtotals

```sql
SELECT NVL(d.dept_name, '** ALL **') AS department,
       NVL(doc.doctor_name, '** All Doctors **') AS doctor,
       COUNT(*) AS appointments,
       SUM(f.total_cost) AS revenue
FROM fact_appointments f
JOIN dim_department d ON f.dept_key = d.dept_key
JOIN dim_doctor doc ON f.doctor_key = doc.doctor_key
GROUP BY ROLLUP(d.dept_name, doc.doctor_name);
```

### RANK — Top-N Analysis

```sql
-- Top 3 doctors by revenue per department
SELECT * FROM (
    SELECT d.dept_name, doc.doctor_name, SUM(f.total_cost) AS revenue,
           RANK() OVER (PARTITION BY d.dept_name ORDER BY SUM(f.total_cost) DESC) AS rnk
    FROM fact_appointments f
    JOIN dim_department d ON f.dept_key = d.dept_key
    JOIN dim_doctor doc ON f.doctor_key = doc.doctor_key
    GROUP BY d.dept_name, doc.doctor_name
) WHERE rnk <= 3;
```

**Next week:** Full deep-dive into ROLLUP, CUBE, RANK, DENSE_RANK, LAG, and window functions.

---

## Part 7: Connecting to Your Project (5 min)

**Phase 2 of your semester project** requires exactly what we covered today:

| Deliverable | What you need |
|-------------|---------------|
| Star schema | 1 fact table + 3-5 dimension tables for YOUR domain |
| ETL procedures | PL/SQL procedures to extract from Phase 1 tables → load into star schema |
| Analytical queries | 5+ queries using ROLLUP, RANK, CUBE, etc. |
| Documentation | Star schema diagram + explanation of each dimension |

### Homework

1. **Read:** Kimball Ch 1 (pp. 1-35) if you haven't yet
2. **Read for next week:** Kimball Ch 2 (pp. 37-68) + Ch 3 (pp. 69-100)
3. **With your team:** Start thinking about your star schema using the 4-step process:
    - What business process will you model?
    - What's the grain?
    - What are the dimensions?
    - What are the facts/measures?
4. **Download:** [Kimball Cheat Sheet](kimball-cheat-sheet.md) — print it, keep it handy

---

## Key Takeaways

1. **OLTP ≠ OLAP** — Different purposes, different designs, different systems
2. **Star Schema = Fact + Dimensions** — Facts are events/measures, dimensions are context
3. **4-Step Process** — Business process → Grain → Dimensions → Facts (never skip grain!)
4. **ETL is PL/SQL** — You already have the skills
5. **Surrogate keys, not business keys** — For flexibility and history tracking
6. **This is what gets you hired** — Star schemas and ETL appear in every data engineering job posting

---

*IS480 — Advanced Database Management | Week 10 | CSULB Spring 2026 | Prof. Jose Pineda*

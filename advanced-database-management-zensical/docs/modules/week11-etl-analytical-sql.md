# Week 11: ETL with PL/SQL + Analytical SQL

> **Phase 2 reminder:** Your star schema DDL + ETL procedures are due Week 13.  
> **Today:** We build a complete ETL pipeline together, then unlock analytical SQL to query the warehouse.

---

## Part A: PL/SQL ETL — Building the Pipeline (45 min)

### What is ETL?

ETL stands for **Extract, Transform, Load** — the process of moving data from your normalized OLTP system into a denormalized star schema for analysis.

```
  3NF Source (OLTP)              Star Schema (OLAP)
 ┌───────────────┐              ┌──────────────────┐
 │  Students     │   EXTRACT    │  dim_student      │
 │  Courses      │──────────►   │  dim_course       │
 │  SchClasses   │   TRANSFORM  │  dim_date         │
 │  Enrollments  │──────────►   │  dim_class        │
 │               │   LOAD       │  fact_enrollments  │
 └───────────────┘──────────►   └──────────────────┘
```

| Phase | What It Does | Example |
|-------|-------------|---------|
| **Extract** | Read data from source tables | `SELECT * FROM Students` |
| **Transform** | Clean, reshape, derive new columns | Combine `class_year` + `semester` into a date key |
| **Load** | Insert into star schema tables | `INSERT INTO dim_student ...` |

!!! tip "Career Connection"
    Every data engineering job description mentions ETL. Tools like **Informatica**, **Talend**, **dbt**, and **Apache Airflow** automate what you're doing manually with PL/SQL today. The concepts are identical — the tools just add scheduling, logging, and error handling.

---

### Step 1: Design the Star Schema

Last week we learned star schema theory. Now let's build one from our lab schema.

**Source tables (3NF):**

- `Students` — snum, sname, standing, majorid, major_gpa, gpa
- `Courses` — dept, cnum, ctitle, crhr, standing
- `SchClasses` — classnum, cnum, class_year, semester, section, class_day, class_time, room, instructor, capacity
- `Enrollments` — classnum, snum

**Target star schema:**

```
              dim_student
              ┌──────────────┐
              │ student_key   │
              │ snum          │
              │ sname         │
              │ standing      │
              │ majorid       │
              │ gpa           │
              └──────┬───────┘
                     │
  dim_course         │         dim_date
  ┌─────────────┐    │    ┌──────────────┐
  │ course_key   │    │    │ date_key      │
  │ cnum         │    │    │ class_year    │
  │ dept         │    │    │ semester      │
  │ ctitle       │    │    │ semester_name │
  │ crhr         │    │    │ acad_year     │
  └──────┬──────┘    │    └──────┬───────┘
         │           │           │
         └─────┬─────┴─────┬─────┘
               │           │
          ┌────┴───────────┴────┐
          │  fact_enrollments    │
          │  enrollment_key      │
          │  student_key (FK)    │
          │  course_key (FK)     │
          │  date_key (FK)       │
          │  class_key (FK)      │
          │  enrollment_count    │
          └─────────────────────┘
               │
          ┌────┴──────────┐
          │ dim_class      │
          │ class_key      │
          │ classnum       │
          │ section        │
          │ class_day      │
          │ class_time     │
          │ room           │
          │ instructor     │
          │ capacity       │
          └───────────────┘
```

---

### Step 2: Create the Star Schema Tables (DDL)

!!! warning "Run these in order"
    Create dimension tables first, then the fact table — the fact table has foreign keys to all dimensions.

```sql
-- ============================================
-- DIMENSION TABLES
-- ============================================

CREATE TABLE dim_student (
    student_key   NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    snum          NUMBER        NOT NULL,
    sname         VARCHAR2(30)  NOT NULL,
    standing      NUMBER,
    majorid       VARCHAR2(10),
    gpa           NUMBER(3,2),
    load_date     DATE DEFAULT SYSDATE
);

CREATE TABLE dim_course (
    course_key    NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    cnum          VARCHAR2(10)  NOT NULL,
    dept          VARCHAR2(10)  NOT NULL,
    ctitle        VARCHAR2(60),
    crhr          NUMBER,
    standing      NUMBER,
    load_date     DATE DEFAULT SYSDATE
);

CREATE TABLE dim_date (
    date_key       NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    class_year     NUMBER        NOT NULL,
    semester       VARCHAR2(10)  NOT NULL,
    semester_name  VARCHAR2(20),
    acad_year      VARCHAR2(9),
    load_date      DATE DEFAULT SYSDATE
);

CREATE TABLE dim_class (
    class_key     NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    classnum      NUMBER        NOT NULL,
    section       VARCHAR2(10),
    class_day     VARCHAR2(10),
    class_time    VARCHAR2(20),
    room          VARCHAR2(20),
    instructor    VARCHAR2(30),
    capacity      NUMBER,
    load_date     DATE DEFAULT SYSDATE
);

-- ============================================
-- FACT TABLE
-- ============================================

CREATE TABLE fact_enrollments (
    enrollment_key   NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    student_key      NUMBER NOT NULL REFERENCES dim_student(student_key),
    course_key       NUMBER NOT NULL REFERENCES dim_course(course_key),
    date_key         NUMBER NOT NULL REFERENCES dim_date(date_key),
    class_key        NUMBER NOT NULL REFERENCES dim_class(class_key),
    enrollment_count NUMBER DEFAULT 1
);
```

!!! info "Why surrogate keys?"
    Notice we use `student_key` (auto-generated) instead of the natural key `snum`. Surrogate keys protect the warehouse from source system changes — if the registrar renumbers students, your warehouse history stays intact. This is Kimball's Rule #2.

---

### Step 3: Write the ETL Procedures

Each procedure handles one dimension or the fact table. The pattern is always the same: **Extract → Transform → Load**.

#### Procedure 1: Load Student Dimension

```sql
CREATE OR REPLACE PROCEDURE load_dim_student AS
    v_count NUMBER;
BEGIN
    -- Check what's already loaded (avoid duplicates)
    SELECT COUNT(*) INTO v_count FROM dim_student;
    DBMS_OUTPUT.PUT_LINE('Existing dim_student rows: ' || v_count);

    -- Insert only students not already in the dimension
    INSERT INTO dim_student (snum, sname, standing, majorid, gpa)
    SELECT s.snum, s.sname, s.standing, s.majorid, s.gpa
    FROM Students s
    WHERE s.snum NOT IN (SELECT snum FROM dim_student);

    v_count := SQL%ROWCOUNT;
    DBMS_OUTPUT.PUT_LINE('Loaded ' || v_count || ' new students into dim_student');

    COMMIT;
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('ERROR in load_dim_student: ' || SQLERRM);
        ROLLBACK;
END;
/
```

#### Procedure 2: Load Course Dimension

```sql
CREATE OR REPLACE PROCEDURE load_dim_course AS
    v_count NUMBER;
BEGIN
    INSERT INTO dim_course (cnum, dept, ctitle, crhr, standing)
    SELECT c.cnum, c.dept, c.ctitle, c.crhr, c.standing
    FROM Courses c
    WHERE c.cnum NOT IN (SELECT cnum FROM dim_course);

    v_count := SQL%ROWCOUNT;
    DBMS_OUTPUT.PUT_LINE('Loaded ' || v_count || ' new courses into dim_course');

    COMMIT;
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('ERROR in load_dim_course: ' || SQLERRM);
        ROLLBACK;
END;
/
```

#### Procedure 3: Load Date Dimension

```sql
CREATE OR REPLACE PROCEDURE load_dim_date AS
    v_count NUMBER;
BEGIN
    -- Extract distinct year/semester combinations and transform
    INSERT INTO dim_date (class_year, semester, semester_name, acad_year)
    SELECT DISTINCT
        sc.class_year,
        sc.semester,
        -- Transform: create readable semester name
        CASE sc.semester
            WHEN 'SP' THEN 'Spring'
            WHEN 'FA' THEN 'Fall'
            WHEN 'SU' THEN 'Summer'
            ELSE sc.semester
        END AS semester_name,
        -- Transform: create academic year label
        sc.class_year || '-' || (sc.class_year + 1) AS acad_year
    FROM SchClasses sc
    WHERE NOT EXISTS (
        SELECT 1 FROM dim_date d
        WHERE d.class_year = sc.class_year
        AND d.semester = sc.semester
    );

    v_count := SQL%ROWCOUNT;
    DBMS_OUTPUT.PUT_LINE('Loaded ' || v_count || ' new date records into dim_date');

    COMMIT;
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('ERROR in load_dim_date: ' || SQLERRM);
        ROLLBACK;
END;
/
```

#### Procedure 4: Load Fact Table

!!! warning "Always load dimensions first"
    The fact table procedure looks up surrogate keys from the dimensions. If a dimension row is missing, the enrollment will be skipped.

```sql
CREATE OR REPLACE PROCEDURE load_fact_enrollments AS
    v_count NUMBER;
    v_skipped NUMBER := 0;
BEGIN
    -- Join source tables and look up dimension keys
    INSERT INTO fact_enrollments (student_key, course_key, date_key, class_key, enrollment_count)
    SELECT
        ds.student_key,
        dc.course_key,
        dd.date_key,
        dcl.class_key,
        1 AS enrollment_count
    FROM Enrollments e
    JOIN SchClasses sc   ON e.classnum = sc.classnum
    JOIN dim_student ds  ON e.snum = ds.snum
    JOIN dim_course dc   ON sc.cnum = dc.cnum
    JOIN dim_date dd     ON sc.class_year = dd.class_year
                        AND sc.semester = dd.semester
    JOIN dim_class dcl   ON sc.classnum = dcl.classnum
    -- Avoid duplicate loads
    WHERE NOT EXISTS (
        SELECT 1 FROM fact_enrollments f
        WHERE f.student_key = ds.student_key
        AND f.class_key = dcl.class_key
    );

    v_count := SQL%ROWCOUNT;
    DBMS_OUTPUT.PUT_LINE('Loaded ' || v_count || ' enrollment facts');

    COMMIT;
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('ERROR in load_fact_enrollments: ' || SQLERRM);
        ROLLBACK;
END;
/
```

---

### Step 4: Run the Full ETL Pipeline

```sql
-- Enable output
SET SERVEROUTPUT ON;

-- Run ETL in order: dimensions first, then facts
EXEC load_dim_student;
EXEC load_dim_course;
EXEC load_dim_date;
EXEC load_dim_class;      -- (you'll write this one!)
EXEC load_fact_enrollments;
```

**Verify the results:**

```sql
-- Check row counts
SELECT 'dim_student' AS tbl, COUNT(*) AS rows FROM dim_student
UNION ALL
SELECT 'dim_course', COUNT(*) FROM dim_course
UNION ALL
SELECT 'dim_date', COUNT(*) FROM dim_date
UNION ALL
SELECT 'dim_class', COUNT(*) FROM dim_class
UNION ALL
SELECT 'fact_enrollments', COUNT(*) FROM fact_enrollments;
```

```sql
-- Sample the fact table with dimension labels
SELECT
    ds.sname,
    dc.ctitle,
    dd.semester_name || ' ' || dd.class_year AS term,
    dcl.instructor,
    dcl.class_day || ' ' || dcl.class_time AS schedule
FROM fact_enrollments f
JOIN dim_student ds ON f.student_key = ds.student_key
JOIN dim_course dc ON f.course_key = dc.course_key
JOIN dim_date dd   ON f.date_key = dd.date_key
JOIN dim_class dcl ON f.class_key = dcl.class_key
WHERE ROWNUM <= 10;
```

!!! tip "Re-running ETL safely"
    Notice every procedure checks for existing data before inserting. This means you can run the ETL pipeline multiple times without getting duplicates. In industry, this is called **idempotent loading** — a key design principle.

---

### Common ETL Pitfalls

| Pitfall | What Happens | How to Avoid |
|---------|-------------|-------------|
| **Duplicate loads** | Same data inserted twice, inflating counts | Use `NOT EXISTS` or `NOT IN` checks |
| **NULL handling** | JOINs fail silently when keys are NULL | Add `WHERE key IS NOT NULL` in extracts |
| **Type mismatches** | `VARCHAR2` vs `NUMBER` comparison fails | Use explicit `TO_NUMBER()` / `TO_CHAR()` |
| **Load order** | Fact insert fails — FK dimension row missing | Always load dimensions before facts |
| **No error handling** | Procedure fails silently | Use `EXCEPTION WHEN OTHERS` + `ROLLBACK` |

---

### How This Maps to Phase 2

Your Phase 2 deliverables require exactly what we just did:

| What We Did | Your Phase 2 Task |
|------------|------------------|
| Designed star schema from lab data | Design star schema for YOUR domain |
| Wrote DDL for 4 dims + 1 fact | Write DDL for your dims + fact(s) |
| Wrote 4 ETL procedures | Write ETL procedures for each table |
| Verified with queries | Show your loaded data works |

!!! info "Phase 2 tip"
    Start from your ER diagram (Phase 1). Ask: *What questions does management want answered?* That tells you what your fact table measures and which dimensions you need.

---

## Part B: Analytical SQL — Querying the Warehouse (45 min)

Now that we have a star schema loaded with data, let's write the kinds of queries that make data warehouses valuable. These are the queries your Phase 3 will require.

### GROUP BY Review

You already know this — just a quick reminder:

```sql
-- How many students per major?
SELECT ds.majorid, COUNT(*) AS student_count
FROM fact_enrollments f
JOIN dim_student ds ON f.student_key = ds.student_key
GROUP BY ds.majorid
ORDER BY student_count DESC;
```

This gives you one row per group. But what if you want **subtotals and grand totals**?

---

### ROLLUP — Hierarchical Subtotals

`ROLLUP` adds subtotal rows automatically, rolling up from right to left.

```sql
-- Enrollments by department and semester, with subtotals
SELECT
    dc.dept,
    dd.semester_name,
    COUNT(*) AS enrollment_count
FROM fact_enrollments f
JOIN dim_course dc ON f.course_key = dc.course_key
JOIN dim_date dd   ON f.date_key = dd.date_key
GROUP BY ROLLUP(dc.dept, dd.semester_name)
ORDER BY dc.dept, dd.semester_name;
```

**What ROLLUP produces:**

| dept | semester_name | enrollment_count | What is this row? |
|------|--------------|------------------|--------------------|
| CECS | Fall | 45 | Detail row |
| CECS | Spring | 52 | Detail row |
| CECS | *NULL* | 97 | **Subtotal for CECS** |
| IS | Fall | 38 | Detail row |
| IS | Spring | 41 | Detail row |
| IS | *NULL* | 79 | **Subtotal for IS** |
| *NULL* | *NULL* | 176 | **Grand total** |

!!! tip "How to read ROLLUP"
    `ROLLUP(a, b)` produces: all (a,b) combos + subtotals per `a` + grand total. It "rolls up" from the rightmost column.

---

### CUBE — Cross-Tabulation Totals

`CUBE` gives you subtotals for **every possible combination** of the grouped columns.

```sql
-- Full cross-tabulation of department × semester
SELECT
    dc.dept,
    dd.semester_name,
    COUNT(*) AS enrollment_count
FROM fact_enrollments f
JOIN dim_course dc ON f.course_key = dc.course_key
JOIN dim_date dd   ON f.date_key = dd.date_key
GROUP BY CUBE(dc.dept, dd.semester_name)
ORDER BY dc.dept, dd.semester_name;
```

**CUBE adds rows that ROLLUP doesn't:**

| dept | semester_name | enrollment_count | Row type |
|------|--------------|------------------|----------|
| CECS | Fall | 45 | Detail |
| CECS | Spring | 52 | Detail |
| CECS | *NULL* | 97 | Subtotal by dept |
| IS | Fall | 38 | Detail |
| IS | Spring | 41 | Detail |
| IS | *NULL* | 79 | Subtotal by dept |
| *NULL* | Fall | 83 | **Subtotal by semester** (CUBE only!) |
| *NULL* | Spring | 93 | **Subtotal by semester** (CUBE only!) |
| *NULL* | *NULL* | 176 | Grand total |

!!! info "ROLLUP vs CUBE"
    - `ROLLUP(a, b)` = detail + subtotals for `a` + grand total → **(n+1) levels**
    - `CUBE(a, b)` = detail + subtotals for `a` + subtotals for `b` + grand total → **all 2^n combinations**

---

### Detecting Subtotal Rows with GROUPING()

NULL in a subtotal row means "all values" — but what if your data has real NULLs? Use `GROUPING()`:

```sql
SELECT
    CASE WHEN GROUPING(dc.dept) = 1 THEN 'ALL DEPTS' ELSE dc.dept END AS dept,
    CASE WHEN GROUPING(dd.semester_name) = 1 THEN 'ALL SEMESTERS' ELSE dd.semester_name END AS semester,
    COUNT(*) AS enrollment_count
FROM fact_enrollments f
JOIN dim_course dc ON f.course_key = dc.course_key
JOIN dim_date dd   ON f.date_key = dd.date_key
GROUP BY CUBE(dc.dept, dd.semester_name)
ORDER BY GROUPING(dc.dept), dc.dept, GROUPING(dd.semester_name), dd.semester_name;
```

`GROUPING(column)` returns **1** if the NULL is from a subtotal, **0** if it's a real value.

---

### RANK and DENSE_RANK — Ranking Within Partitions

```sql
-- Rank students by GPA within each major
SELECT
    ds.majorid,
    ds.sname,
    ds.gpa,
    RANK()       OVER (PARTITION BY ds.majorid ORDER BY ds.gpa DESC) AS rank_gpa,
    DENSE_RANK() OVER (PARTITION BY ds.majorid ORDER BY ds.gpa DESC) AS dense_rank_gpa
FROM dim_student ds
ORDER BY ds.majorid, rank_gpa;
```

**RANK vs DENSE_RANK:**

| majorid | sname | gpa | RANK | DENSE_RANK |
|---------|-------|-----|------|------------|
| CECS | Alice | 3.9 | 1 | 1 |
| CECS | Bob | 3.9 | 1 | 1 |
| CECS | Carol | 3.7 | **3** | **2** |

- `RANK()` — skips numbers after ties (1, 1, **3**)
- `DENSE_RANK()` — no gaps (1, 1, **2**)

---

### ROW_NUMBER — Unique Numbering

```sql
-- Number each enrollment per student (chronologically)
SELECT
    ds.sname,
    dc.ctitle,
    dd.class_year,
    dd.semester,
    ROW_NUMBER() OVER (PARTITION BY ds.snum ORDER BY dd.class_year, dd.semester) AS enrollment_seq
FROM fact_enrollments f
JOIN dim_student ds ON f.student_key = ds.student_key
JOIN dim_course dc ON f.course_key = dc.course_key
JOIN dim_date dd   ON f.date_key = dd.date_key
ORDER BY ds.sname, enrollment_seq;
```

`ROW_NUMBER()` always produces unique sequential numbers — no ties.

---

### Window Functions — Running Totals and Averages

The `OVER()` clause turns any aggregate into a **window function** — it computes across a "window" of related rows without collapsing them.

```sql
-- Running total of enrollments per department, by year
SELECT
    dc.dept,
    dd.class_year,
    COUNT(*) AS yearly_enrollments,
    SUM(COUNT(*)) OVER (PARTITION BY dc.dept ORDER BY dd.class_year) AS running_total
FROM fact_enrollments f
JOIN dim_course dc ON f.course_key = dc.course_key
JOIN dim_date dd   ON f.date_key = dd.date_key
GROUP BY dc.dept, dd.class_year
ORDER BY dc.dept, dd.class_year;
```

| dept | class_year | yearly_enrollments | running_total |
|------|-----------|-------------------|---------------|
| CECS | 2024 | 45 | 45 |
| CECS | 2025 | 52 | 97 |
| CECS | 2026 | 48 | 145 |

!!! tip "Window function anatomy"
    ```
    SUM(amount) OVER (
        PARTITION BY dept        -- start a new window per dept
        ORDER BY class_year      -- define row order within window
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW  -- window frame
    )
    ```
    Think of it as: "for each row, compute SUM across all preceding rows in the same partition."

---

### LAG and LEAD — Comparing Adjacent Rows

```sql
-- Compare each year's enrollments to the previous year
SELECT
    dc.dept,
    dd.class_year,
    COUNT(*) AS enrollments,
    LAG(COUNT(*), 1) OVER (PARTITION BY dc.dept ORDER BY dd.class_year) AS prev_year,
    COUNT(*) - LAG(COUNT(*), 1) OVER (PARTITION BY dc.dept ORDER BY dd.class_year) AS year_over_year_change
FROM fact_enrollments f
JOIN dim_course dc ON f.course_key = dc.course_key
JOIN dim_date dd   ON f.date_key = dd.date_key
GROUP BY dc.dept, dd.class_year
ORDER BY dc.dept, dd.class_year;
```

| dept | class_year | enrollments | prev_year | year_over_year_change |
|------|-----------|------------|-----------|----------------------|
| CECS | 2024 | 45 | *NULL* | *NULL* |
| CECS | 2025 | 52 | 45 | 7 |
| CECS | 2026 | 48 | 52 | -4 |

- `LAG(column, n)` — looks **back** n rows
- `LEAD(column, n)` — looks **forward** n rows

!!! tip "Year-over-year analysis"
    This pattern is everywhere in business analytics. Netflix tracks subscriber growth YoY. Amazon tracks revenue QoQ. You just wrote that query.

---

### Practice Queries

Try these on your star schema. Each one uses a different analytical technique.

!!! example "Practice 1: Top 3 courses by enrollment per department"
    ```sql
    SELECT dept, ctitle, enrollment_count, rnk
    FROM (
        SELECT
            dc.dept,
            dc.ctitle,
            COUNT(*) AS enrollment_count,
            RANK() OVER (PARTITION BY dc.dept ORDER BY COUNT(*) DESC) AS rnk
        FROM fact_enrollments f
        JOIN dim_course dc ON f.course_key = dc.course_key
        GROUP BY dc.dept, dc.ctitle
    )
    WHERE rnk <= 3;
    ```

!!! example "Practice 2: Enrollment trend with running average"
    ```sql
    SELECT
        dd.class_year,
        dd.semester_name,
        COUNT(*) AS enrollments,
        ROUND(AVG(COUNT(*)) OVER (ORDER BY dd.class_year, dd.semester ROWS BETWEEN 2 PRECEDING AND CURRENT ROW), 1) AS moving_avg_3term
    FROM fact_enrollments f
    JOIN dim_date dd ON f.date_key = dd.date_key
    GROUP BY dd.class_year, dd.semester, dd.semester_name
    ORDER BY dd.class_year, dd.semester;
    ```

!!! example "Practice 3: Department enrollment with ROLLUP"
    ```sql
    SELECT
        NVL(dc.dept, 'TOTAL') AS department,
        NVL(dd.semester_name, 'ALL') AS semester,
        COUNT(*) AS enrollments
    FROM fact_enrollments f
    JOIN dim_course dc ON f.course_key = dc.course_key
    JOIN dim_date dd   ON f.date_key = dd.date_key
    GROUP BY ROLLUP(dc.dept, dd.semester_name);
    ```

!!! example "Practice 4: Student enrollment count with quartile buckets"
    ```sql
    SELECT
        ds.sname,
        COUNT(*) AS total_enrollments,
        NTILE(4) OVER (ORDER BY COUNT(*) DESC) AS quartile
    FROM fact_enrollments f
    JOIN dim_student ds ON f.student_key = ds.student_key
    GROUP BY ds.sname
    ORDER BY total_enrollments DESC;
    ```

!!! example "Practice 5: Instructor workload comparison"
    ```sql
    SELECT
        dcl.instructor,
        dd.semester_name || ' ' || dd.class_year AS term,
        COUNT(*) AS enrollment_count,
        LAG(COUNT(*)) OVER (PARTITION BY dcl.instructor ORDER BY dd.class_year, dd.semester) AS prev_term,
        RANK() OVER (PARTITION BY dd.class_year, dd.semester ORDER BY COUNT(*) DESC) AS workload_rank
    FROM fact_enrollments f
    JOIN dim_class dcl ON f.class_key = dcl.class_key
    JOIN dim_date dd   ON f.date_key = dd.date_key
    GROUP BY dcl.instructor, dd.semester_name, dd.class_year, dd.semester
    ORDER BY dcl.instructor, dd.class_year, dd.semester;
    ```

!!! example "Practice 6: CUBE cross-tab with GROUPING labels"
    ```sql
    SELECT
        CASE WHEN GROUPING(ds.majorid) = 1 THEN '-- ALL --' ELSE ds.majorid END AS major,
        CASE WHEN GROUPING(dd.semester_name) = 1 THEN '-- ALL --' ELSE dd.semester_name END AS semester,
        COUNT(*) AS enrollments,
        ROUND(AVG(ds.gpa), 2) AS avg_gpa
    FROM fact_enrollments f
    JOIN dim_student ds ON f.student_key = ds.student_key
    JOIN dim_date dd   ON f.date_key = dd.date_key
    GROUP BY CUBE(ds.majorid, dd.semester_name)
    ORDER BY GROUPING(ds.majorid), ds.majorid, GROUPING(dd.semester_name), dd.semester_name;
    ```

---

## Part C: Week 11 Exercise

**Due:** Before next class (Week 12)

### Task 1: Design Your Project Star Schema

Using your Phase 1 ER diagram, design a star schema for your project domain:

1. Identify **one fact table** — what business process are you measuring?
2. Identify **3-4 dimension tables** — what context describes each fact?
3. Draw the star schema diagram (use draw.io, Lucidchart, or hand-drawn)
4. Write the **complete DDL** (CREATE TABLE statements) for all tables

!!! tip "Deliverable"
    Include this DDL in your Phase 2 submission. It should be runnable — test it on your Oracle RDS instance.

### Task 2: Write ETL Procedures

Write at least **2 PL/SQL ETL procedures** that:

- Extract data from your 3NF source tables
- Transform as needed (derive new columns, clean data, handle NULLs)
- Load into your dimension or fact tables
- Include error handling (`EXCEPTION` block)
- Are idempotent (safe to run multiple times)

### Task 3: Write Analytical Queries

Write **3 analytical queries** against your star schema using techniques from today:

1. One query using `ROLLUP` or `CUBE`
2. One query using `RANK()`, `DENSE_RANK()`, or `ROW_NUMBER()`
3. One query using `LAG` or `LEAD`

!!! info "These queries become part of Phase 3"
    The analytical queries you write now will be the Oracle baseline for your Phase 3 Spark comparison. Choose queries that reveal interesting patterns in your data.

---

**Next week:** We take these queries to Apache Spark and Databricks — same SQL, massive scale. 🚀

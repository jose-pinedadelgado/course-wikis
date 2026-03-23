# Data Warehousing & Star Schema

!!! note "Coming in Weeks 10-12"
    This module covers the concepts and techniques needed for **Phase 2** of the semester project.

## What You'll Learn

- **OLTP vs OLAP** — Why transactional databases aren't ideal for analytics
- **Star Schema Design** — Fact tables, dimension tables, and how they fit together
- **ETL with PL/SQL** — Extract from your 3NF tables, Transform, and Load into star schema
- **Analytical SQL** — `GROUP BY`, `ROLLUP`, `CUBE`, `RANK()`, `DENSE_RANK()`, window functions

## Key Concepts

### OLTP vs OLAP

| | OLTP (Phase 1) | OLAP (Phase 2) |
|--|----------------|-----------------|
| **Purpose** | Day-to-day operations | Analysis & reporting |
| **Schema** | Normalized (3NF) | Denormalized (Star) |
| **Queries** | Short, targeted (INSERT/UPDATE) | Complex, aggregated (GROUP BY, JOIN) |
| **Example** | "Enroll student 101 in class 10125" | "What's the average GPA by department over 3 years?" |

### Star Schema

```
           ┌─────────────┐
           │  dim_date    │
           └──────┬──────┘
                  │
┌──────────┐   ┌──┴───────────┐   ┌──────────────┐
│dim_student├───┤  fact_table  ├───┤ dim_category  │
└──────────┘   └──┬───────────┘   └──────────────┘
                  │
           ┌──────┴──────┐
           │ dim_location │
           └─────────────┘
```

- **Fact table** — Contains measurable events (transactions, enrollments, sales) with foreign keys to dimensions and numeric measures
- **Dimension tables** — Descriptive context (who, what, when, where)

### ETL Process

```sql
-- Example: PL/SQL ETL procedure
CREATE OR REPLACE PROCEDURE load_fact_enrollments AS
BEGIN
    INSERT INTO fact_enrollments (student_key, course_key, date_key, grade_value, credit_hours)
    SELECT s.snum, c.cnum, TO_NUMBER(TO_CHAR(SYSDATE, 'YYYYMMDD')),
           CASE e.grade WHEN 'A' THEN 4 WHEN 'B' THEN 3 WHEN 'C' THEN 2 WHEN 'D' THEN 1 ELSE 0 END,
           c.crhr
    FROM Enrollments e
    JOIN SchClasses sc ON e.classnum = sc.classnum
    JOIN Courses c ON sc.cnum = c.cnum
    JOIN Students s ON e.snum = s.snum
    WHERE e.grade IS NOT NULL;
    
    COMMIT;
    DBMS_OUTPUT.PUT_LINE('Loaded ' || SQL%ROWCOUNT || ' rows into fact_enrollments');
END;
/
```

### Three Types of Facts

Understanding fact types is critical — it determines how you can aggregate your data.

#### ✅ Additive Facts

Can be **summed across ALL dimensions**. These are the most common and most useful.

| Example | Why Additive |
|---------|-------------|
| Revenue | $100 (Store A) + $200 (Store B) = $300 total ✅ |
| Quantity Sold | 10 (Monday) + 20 (Tuesday) = 30 total ✅ |
| Cost | Sum by product, by store, by time — all valid ✅ |

#### ⚠️ Semi-Additive Facts

Can be summed across **some dimensions, but NOT across time**. These are typically *snapshot* measurements — what something *is* at a moment, not what *happened*.

| Example | Why Semi-Additive |
|---------|------------------|
| Account Balance | $5,000 Monday + $5,000 Tuesday ≠ $10,000! Use AVG or point-in-time. |
| Inventory Level | 100 units today + 100 units yesterday ≠ 200 units. But SUM across warehouses works. |
| Headcount | 50 employees in Dept A + 30 in Dept B = 80 total ✅. But NOT across months. |

#### ❌ Non-Additive Facts

Can **never be meaningfully summed** across any dimension. Must be computed from components.

| Example | Why Non-Additive |
|---------|-----------------|
| Unit Price | Can't sum or average prices without weighting by quantity |
| Profit Margin % | Ratios can't be summed |
| Temperature | 70°F + 80°F = 150°F makes no sense |

#### The Unit Price Trap

This is the classic mistake students make:

| | Qty | Unit Price | Revenue |
|--|-----|-----------|---------|
| Sale 1 | 1 | $1.00 | $1.00 |
| Sale 2 | 4 | $0.50 | $2.00 |
| **Total** | **5** | **???** | **$3.00** |

- ❌ Simple average: ($1.00 + $0.50) / 2 = **$0.75** — WRONG
- ✅ Weighted average: $3.00 / 5 = **$0.60** — CORRECT

!!! warning "Kimball's Golden Rule"
    **"Ratio of the sums, not sum of the ratios."**
    
    Store the **components** (revenue + quantity) as additive facts in the fact table, then **compute** the ratio in the query. Never store a pre-computed ratio as a fact if you can avoid it.

```sql
-- ❌ WRONG: Averaging a non-additive fact
SELECT AVG(unit_price) FROM fact_sales;  -- Unweighted, misleading!

-- ✅ RIGHT: Compute from additive components
SELECT SUM(revenue) / SUM(quantity) AS weighted_avg_price 
FROM fact_sales;
```

#### Applying This to Your Exercises

When designing your star schemas, ask: **"Can I SUM this across every dimension?"**

| Fact | Type | Why |
|------|------|-----|
| CourseGrade (Millennium) | ⚠️ Semi-additive | AVG across students, not SUM |
| FaceValue (Fitchwood) | ✅ Additive | Total face value across agents, territories, time |
| Commission % (Fitchwood) | ❌ Non-additive | Store dollar amounts instead |
| InitComm % (Fitchwood) | ❌ Non-additive | Compute: FaceValue × InitComm% |

### Analytical Queries

```sql
-- ROLLUP: Subtotals by department and course
SELECT c.dept, c.ctitle, COUNT(*) as enrollments, AVG(s.gpa) as avg_gpa
FROM fact_enrollments f
JOIN dim_course c ON f.course_key = c.cnum
JOIN dim_student s ON f.student_key = s.snum
GROUP BY ROLLUP(c.dept, c.ctitle);

-- RANK: Top students by GPA within each major
SELECT sname, majorid, gpa,
       RANK() OVER (PARTITION BY majorid ORDER BY gpa DESC) as rank_in_major
FROM Students
WHERE majorid IS NOT NULL;
```

## Resources

- Kimball, R. — *The Data Warehouse Toolkit* (the star schema bible)
- Oracle documentation: `ROLLUP`, `CUBE`, analytical functions
- [Project Spec — Phase 2](project-spec.md#phase-2-data-warehouse-analytics-30-points)

---

*Detailed content will be added as we cover this material in class.*

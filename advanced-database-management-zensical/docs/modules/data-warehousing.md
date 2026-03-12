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

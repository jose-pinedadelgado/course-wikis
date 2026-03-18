# Data Warehousing & Star Schema

!!! tip "Week 10-12 Module"
    This module covers the concepts and techniques needed for **Phase 2** of the semester project.

## Module Resources

| Resource | Description |
|----------|-------------|
| [📖 Lecture Notes: Star Schema Design](week10-star-schema-lecture.md) | Full lecture content with examples, SQL, and diagrams |
| [🖥️ Interactive Slides](../slides/index.html) | Reveal.js presentation for in-class use |
| [✏️ In-Class Exercise](week10-exercise.md) | 3-part exercise: Warm-up, Millennium College, Project Design |
| [📋 Kimball Cheat Sheet](kimball-cheat-sheet.md) | Quick reference for dimensional modeling |

## Reading Assignments

| Week | Reading | Focus |
|------|---------|-------|
| **10** | Kimball Ch 1 (pp. 1-35) | Why warehouses? OLTP vs OLAP. The bus architecture. |
| **10-11** | Kimball Ch 2 (pp. 37-68) | 4-step process, fact/dimension rules, SCD types |
| **11** | Kimball Ch 3 (pp. 69-100) | Retail sales case study — a complete star schema walkthrough |

## What You'll Learn

By the end of this module, you should be able to:

1. **Explain** why transactional databases (OLTP) aren't suitable for analytics
2. **Apply** Kimball's 4-step process to design a star schema from business requirements
3. **Distinguish** between fact tables and dimension tables, and their respective rules
4. **Identify** the three types of facts (additive, semi-additive, non-additive) and avoid the "unit price trap"
5. **Design** SCD Type 2 dimensions to preserve historical changes
6. **Write** PL/SQL ETL procedures to extract, transform, and load data into a star schema
7. **Use** analytical SQL (ROLLUP, CUBE, RANK) to query the warehouse

## Key Concepts at a Glance

### OLTP vs OLAP

| | OLTP (Phase 1) | OLAP (Phase 2) |
|--|----------------|-----------------|
| **Purpose** | Run the business | Analyze the business |
| **Schema** | Normalized (3NF) | Denormalized (Star) |
| **Queries** | Short, targeted (INSERT/UPDATE) | Complex, aggregated (GROUP BY, JOIN) |
| **Data** | Current state | Historical snapshots |
| **Redundancy** | Eliminated | Intentional |

!!! info "The Big Irony"
    In Phase 1, you fought to eliminate redundancy through normalization. In Phase 2, you'll **add it back on purpose** — because redundancy speeds up analytical reads.

### Star Schema

```
           ┌─────────────┐
           │  dim_date    │  ← DIMENSION: descriptive context
           └──────┬──────┘     (wide, denormalized, ~thousands of rows)
                  │
┌──────────┐   ┌──┴───────────┐   ┌──────────────┐
│dim_student├───┤  fact_table  ├───┤ dim_category  │
└──────────┘   └──┬───────────┘   └──────────────┘
                  │               ← FACT: events + measures
           ┌──────┴──────┐          (narrow, tall, millions of rows)
           │ dim_location │
           └─────────────┘
```

- **Fact table** — Contains measurable events (transactions, enrollments, sales) with foreign keys to dimensions and numeric measures
- **Dimension tables** — Descriptive context answering who, what, when, where, how

### Kimball's 4-Step Process

| Step | Question | Example |
|------|----------|---------|
| 1. Business Process | What activity generates events? | Student enrollment |
| 2. Declare the Grain | What is ONE row? | "One enrollment" |
| 3. Dimensions | Who/what/when/where? | Student, Course, Instructor, Semester |
| 4. Facts (Measures) | What numbers to aggregate? | grade_value, credit_hours |

!!! warning "Never Skip Step 2"
    Kimball's #1 design rule: "If the grain isn't defined, the rest of the design rests on quicksand."

### Three Types of Facts

| Type | Can SUM across... | Example |
|------|-------------------|---------|
| **Additive** | ALL dimensions ✅ | Revenue, quantity, cost |
| **Semi-additive** | Some (not time) ⚠️ | Account balance |
| **Non-additive** | NONE ❌ | Unit price, ratios |

💡 **"Ratio of the sums, not sum of the ratios."** — Store components, compute in query.

### SCD Types

| Type | Strategy | History? |
|------|----------|----------|
| **Type 1** | Overwrite | ❌ Lost |
| **Type 2** | New row + new surrogate key | ✅ Full |
| **Type 3** | Add "previous" column | ⚠️ Partial |

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
END;
/
```

**Key rule:** Always load dimensions FIRST, then facts (facts need dimension FKs to exist).

### Analytical SQL Preview

```sql
-- ROLLUP: Automatic subtotals
SELECT dept, ctitle, COUNT(*) as enrollments, AVG(gpa) as avg_gpa
FROM fact_enrollments f
JOIN dim_course c ON f.course_key = c.cnum
GROUP BY ROLLUP(dept, ctitle);

-- RANK: Top students by GPA within each major
SELECT sname, majorid, gpa,
       RANK() OVER (PARTITION BY majorid ORDER BY gpa DESC) as rank_in_major
FROM Students;
```

## Phase 2 Project Deliverables

| Deliverable | What You Need |
|-------------|---------------|
| Star Schema | 1 fact table + 3-5 dimension tables for YOUR domain |
| ETL Procedures | PL/SQL to extract from Phase 1 → load star schema |
| Analytical Queries | 5+ queries using ROLLUP, RANK, CUBE, etc. |
| Documentation | Star schema diagram + grain statement + explanations |

---

*For detailed content, see the [full lecture notes](week10-star-schema-lecture.md) and the [Kimball Cheat Sheet](kimball-cheat-sheet.md).*

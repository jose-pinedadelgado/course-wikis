# Analytical SQL Quick Reference

> Pure reference card. Syntax + example + expected output. All examples use the enrollment star schema.

---

## GROUP BY Patterns

```sql
-- Basic grouping
SELECT dept, COUNT(*) AS cnt
FROM fact_enrollments f
JOIN dim_course dc ON f.course_key = dc.course_key
GROUP BY dept;
```

| dept | cnt |
|------|-----|
| CECS | 97 |
| IS | 79 |

---

## ROLLUP

**Syntax:**

```sql
GROUP BY ROLLUP(col_a, col_b)
```

**Produces:** detail rows + subtotals for `col_a` + grand total. Rolls up right to left.

```sql
SELECT dc.dept, dd.semester_name, COUNT(*) AS cnt
FROM fact_enrollments f
JOIN dim_course dc ON f.course_key = dc.course_key
JOIN dim_date dd ON f.date_key = dd.date_key
GROUP BY ROLLUP(dc.dept, dd.semester_name);
```

| dept | semester_name | cnt | Row Type |
|------|--------------|-----|----------|
| CECS | Fall | 45 | Detail |
| CECS | Spring | 52 | Detail |
| CECS | NULL | 97 | Subtotal (dept) |
| IS | Fall | 38 | Detail |
| IS | Spring | 41 | Detail |
| IS | NULL | 79 | Subtotal (dept) |
| NULL | NULL | 176 | Grand total |

**Levels produced by `ROLLUP(a, b, c)`:** (a,b,c) → (a,b) → (a) → ()

---

## CUBE

**Syntax:**

```sql
GROUP BY CUBE(col_a, col_b)
```

**Produces:** all possible subtotal combinations (2^n groups for n columns).

```sql
SELECT dc.dept, dd.semester_name, COUNT(*) AS cnt
FROM fact_enrollments f
JOIN dim_course dc ON f.course_key = dc.course_key
JOIN dim_date dd ON f.date_key = dd.date_key
GROUP BY CUBE(dc.dept, dd.semester_name);
```

| dept | semester_name | cnt | Row Type |
|------|--------------|-----|----------|
| CECS | Fall | 45 | Detail |
| CECS | Spring | 52 | Detail |
| CECS | NULL | 97 | Subtotal by dept |
| IS | Fall | 38 | Detail |
| IS | Spring | 41 | Detail |
| IS | NULL | 79 | Subtotal by dept |
| NULL | Fall | 83 | Subtotal by semester |
| NULL | Spring | 93 | Subtotal by semester |
| NULL | NULL | 176 | Grand total |

**ROLLUP vs CUBE:** ROLLUP = hierarchical subtotals. CUBE = all cross-tabulation subtotals.

---

## GROUPING Function

Detects whether a NULL is from a subtotal (returns 1) or real data (returns 0).

```sql
SELECT
    CASE WHEN GROUPING(dc.dept) = 1 THEN 'ALL' ELSE dc.dept END AS dept,
    CASE WHEN GROUPING(dd.semester_name) = 1 THEN 'ALL' ELSE dd.semester_name END AS semester,
    COUNT(*) AS cnt
FROM fact_enrollments f
JOIN dim_course dc ON f.course_key = dc.course_key
JOIN dim_date dd ON f.date_key = dd.date_key
GROUP BY CUBE(dc.dept, dd.semester_name);
```

| dept | semester | cnt |
|------|---------|-----|
| CECS | Fall | 45 |
| CECS | ALL | 97 |
| ALL | Fall | 83 |
| ALL | ALL | 176 |

---

## RANK

**Syntax:**

```sql
RANK() OVER (PARTITION BY col ORDER BY col2 [ASC|DESC])
```

Ties get the same rank. Next rank skips (1, 1, **3**).

```sql
SELECT majorid, sname, gpa,
       RANK() OVER (PARTITION BY majorid ORDER BY gpa DESC) AS rnk
FROM dim_student;
```

| majorid | sname | gpa | rnk |
|---------|-------|-----|-----|
| CECS | Alice | 3.9 | 1 |
| CECS | Bob | 3.9 | 1 |
| CECS | Carol | 3.7 | 3 |

---

## DENSE_RANK

Same as RANK but no gaps after ties (1, 1, **2**).

```sql
DENSE_RANK() OVER (PARTITION BY majorid ORDER BY gpa DESC)
```

| majorid | sname | gpa | dense_rnk |
|---------|-------|-----|-----------|
| CECS | Alice | 3.9 | 1 |
| CECS | Bob | 3.9 | 1 |
| CECS | Carol | 3.7 | 2 |

---

## ROW_NUMBER

Unique sequential number per partition. Ties broken arbitrarily.

```sql
ROW_NUMBER() OVER (PARTITION BY majorid ORDER BY gpa DESC)
```

| majorid | sname | gpa | row_num |
|---------|-------|-----|---------|
| CECS | Alice | 3.9 | 1 |
| CECS | Bob | 3.9 | 2 |
| CECS | Carol | 3.7 | 3 |

---

## Running Aggregates (SUM, AVG, COUNT OVER)

**Syntax:**

```sql
SUM(col) OVER (PARTITION BY p ORDER BY o ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
```

```sql
SELECT dept, class_year,
       COUNT(*) AS yearly,
       SUM(COUNT(*)) OVER (PARTITION BY dept ORDER BY class_year) AS running_total,
       ROUND(AVG(COUNT(*)) OVER (PARTITION BY dept ORDER BY class_year), 1) AS running_avg
FROM fact_enrollments f
JOIN dim_course dc ON f.course_key = dc.course_key
JOIN dim_date dd ON f.date_key = dd.date_key
GROUP BY dept, class_year;
```

| dept | class_year | yearly | running_total | running_avg |
|------|-----------|--------|---------------|-------------|
| CECS | 2024 | 45 | 45 | 45.0 |
| CECS | 2025 | 52 | 97 | 48.5 |
| CECS | 2026 | 48 | 145 | 48.3 |

---

## LAG

Access a value from a previous row in the same partition.

**Syntax:**

```sql
LAG(column, offset, default) OVER (PARTITION BY p ORDER BY o)
```

- `offset` — how many rows back (default 1)
- `default` — value if no previous row exists (default NULL)

```sql
SELECT dept, class_year, COUNT(*) AS enrollments,
       LAG(COUNT(*), 1) OVER (PARTITION BY dept ORDER BY class_year) AS prev_year
FROM fact_enrollments f
JOIN dim_course dc ON f.course_key = dc.course_key
JOIN dim_date dd ON f.date_key = dd.date_key
GROUP BY dept, class_year;
```

| dept | class_year | enrollments | prev_year |
|------|-----------|------------|-----------|
| CECS | 2024 | 45 | NULL |
| CECS | 2025 | 52 | 45 |
| CECS | 2026 | 48 | 52 |

---

## LEAD

Access a value from a following row. Same syntax as LAG but looks forward.

```sql
LEAD(column, offset, default) OVER (PARTITION BY p ORDER BY o)
```

```sql
SELECT dept, class_year, COUNT(*) AS enrollments,
       LEAD(COUNT(*), 1) OVER (PARTITION BY dept ORDER BY class_year) AS next_year
FROM fact_enrollments f
JOIN dim_course dc ON f.course_key = dc.course_key
JOIN dim_date dd ON f.date_key = dd.date_key
GROUP BY dept, class_year;
```

| dept | class_year | enrollments | next_year |
|------|-----------|------------|-----------|
| CECS | 2024 | 45 | 52 |
| CECS | 2025 | 52 | 48 |
| CECS | 2026 | 48 | NULL |

---

## NTILE

Divides rows into N roughly equal buckets.

**Syntax:**

```sql
NTILE(n) OVER (ORDER BY col)
```

```sql
SELECT sname, gpa,
       NTILE(4) OVER (ORDER BY gpa DESC) AS quartile
FROM dim_student;
```

| sname | gpa | quartile |
|-------|-----|----------|
| Alice | 3.9 | 1 |
| Bob | 3.8 | 1 |
| Carol | 3.5 | 2 |
| Dave | 3.2 | 3 |
| Eve | 2.8 | 4 |

Quartile 1 = top 25%, quartile 4 = bottom 25%.

---

## FIRST_VALUE and LAST_VALUE

Get the first or last value in a window frame.

**Syntax:**

```sql
FIRST_VALUE(col) OVER (PARTITION BY p ORDER BY o)
LAST_VALUE(col)  OVER (PARTITION BY p ORDER BY o
                       ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
```

!!! warning "LAST_VALUE gotcha"
    The default window frame ends at `CURRENT ROW`, not the partition end. Always specify `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` for LAST_VALUE.

```sql
SELECT majorid, sname, gpa,
       FIRST_VALUE(sname) OVER (PARTITION BY majorid ORDER BY gpa DESC) AS top_student,
       LAST_VALUE(sname) OVER (PARTITION BY majorid ORDER BY gpa DESC
           ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS bottom_student
FROM dim_student;
```

| majorid | sname | gpa | top_student | bottom_student |
|---------|-------|-----|-------------|----------------|
| CECS | Alice | 3.9 | Alice | Eve |
| CECS | Bob | 3.5 | Alice | Eve |
| CECS | Eve | 2.8 | Alice | Eve |

---

## LISTAGG

Concatenate values from multiple rows into a single string.

**Syntax:**

```sql
LISTAGG(column, delimiter) WITHIN GROUP (ORDER BY col)
```

```sql
SELECT dc.dept,
       LISTAGG(dc.ctitle, ', ') WITHIN GROUP (ORDER BY dc.ctitle) AS course_list
FROM dim_course dc
GROUP BY dc.dept;
```

| dept | course_list |
|------|------------|
| CECS | Data Structures, Database Systems, Operating Systems |
| IS | Business Analytics, Info Systems, Project Management |

!!! info "String length limit"
    LISTAGG has a 4000-character limit in Oracle (32K in 12.2+). For long lists, use `LISTAGG(...) ON OVERFLOW TRUNCATE` (Oracle 12.2+).

---

## Window Frame Summary

| Frame Clause | Meaning |
|-------------|---------|
| `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` | All rows from partition start to current (default for ORDER BY) |
| `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW` | Current row + 2 rows before (3-row moving window) |
| `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` | Entire partition |
| `ROWS BETWEEN CURRENT ROW AND 1 FOLLOWING` | Current row + next row |

---

## Quick Comparison: RANK vs DENSE_RANK vs ROW_NUMBER

| gpa | RANK | DENSE_RANK | ROW_NUMBER |
|-----|------|------------|------------|
| 3.9 | 1 | 1 | 1 |
| 3.9 | 1 | 1 | 2 |
| 3.7 | 3 | 2 | 3 |
| 3.5 | 4 | 3 | 4 |

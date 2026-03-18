# Week 10: In-Class Exercise — Designing Star Schemas

> **Time:** ~50 minutes in-class + team homework  
> **Format:** Individual (Parts A-B), then team (Part C)  
> **References:** [Kimball Cheat Sheet](kimball-cheat-sheet.md), [Lecture Notes](week10-star-schema-lecture.md)

---

## Part A: Warm-Up — Identify the Components (10 min)

For each scenario, identify the **business process**, **grain**, **dimensions**, and **measures**. Don't draw a schema yet — just list them.

### Scenario 1: University Registrar

A university wants to analyze student enrollment patterns. The OLTP system tracks students registering for course sections each semester.

| Component | Your Answer |
|-----------|-------------|
| **Business process** | |
| **Grain** (one row per ___) | |
| **Dimensions** (who/what/when/where) | |
| **Measures** (what numbers to aggregate) | |

### Scenario 2: Online Streaming Service

A streaming platform (think Netflix) wants to analyze viewing patterns. Every time a user watches content, the system logs the event.

| Component | Your Answer |
|-----------|-------------|
| **Business process** | |
| **Grain** | |
| **Dimensions** | |
| **Measures** | |

### Scenario 3: Hospital Emergency Room

A hospital wants to analyze ER visit patterns to optimize staffing and reduce wait times.

| Component | Your Answer |
|-----------|-------------|
| **Business process** | |
| **Grain** | |
| **Dimensions** | |
| **Measures** | |

??? tip "💡 Click for suggested answers"

    **Scenario 1 — University:**

    - **Process:** Student enrollment
    - **Grain:** One row per student per course section per semester
    - **Dimensions:** dim_student (name, major, standing), dim_course (title, dept, credits), dim_instructor (name, title, dept), dim_date/period (semester, year)
    - **Measures:** grade_value, credit_hours, enrollment_count (1 per row)

    **Scenario 2 — Streaming:**

    - **Process:** Content viewing
    - **Grain:** One row per user per content item per viewing session
    - **Dimensions:** dim_user (age_group, subscription_tier, country), dim_content (title, genre, rating, duration), dim_date (date, day_of_week, month), dim_device (type, OS, screen_size)
    - **Measures:** watch_duration_min, completion_pct, pause_count

    **Scenario 3 — ER:**

    - **Process:** ER patient visits
    - **Grain:** One row per ER visit
    - **Dimensions:** dim_patient (age_group, gender, insurance), dim_provider (name, specialty), dim_date (date, hour, shift), dim_diagnosis (category, severity), dim_department
    - **Measures:** wait_time_min, treatment_duration_min, total_charges, num_procedures

---

## Part B: Millennium College Star Schema (20 min)

*(Adapted from Chapter 9 textbook exercises)*

Millennium College wants to analyze **course grades** over time. There are four dimension tables:

| Dimension | Attributes | Size |
|-----------|-----------|------|
| **CourseSection** | CourseID, SectionNumber, CourseName, Units, RoomID, RoomCapacity | ~500 sections/semester |
| **Professor** | ProfID, ProfName, Title, DepartmentID, DepartmentName | ~200 professors |
| **Student** | StudentID, StudentName, Major | ~40 students/section, 5 courses/student |
| **Period** | SemesterID, Year | 30 periods (10 years) |

The facts to record: **Course Grade** and **CompletionDate**.

### Questions

**Q1.** Draw the star schema. Show the fact table in the center with FKs to each dimension, and list the measures.

**Q2.** Estimate the number of rows in the fact table.

!!! hint
    500 sections × 40 students/section × 30 periods = ?

**Q3.** Estimate the total size of the fact table in bytes, assuming each field averages 15 bytes.

!!! hint
    rows × number of fields per row × 15 bytes

**Q4.** If you didn't have to use a strict star schema, how would you change the design? Why?

!!! hint "Think about..."
    Does Professor need to connect directly to the fact table? Could it connect through CourseSection instead? What about normalizing Course info out of CourseSection?

**Q5.** Various characteristics of sections, professors, and students change over time (e.g., a professor changes departments, a room assignment changes). How would you handle this?

!!! hint
    Which SCD type makes sense here?

??? tip "💡 Click for answers"

    **Q2:** 500 × 40 × 30 = **600,000 rows**

    **Q3:** 600,000 × 6 fields (4 FKs + 2 facts) × 15 bytes = **54,000,000 bytes (~54 MB)**

    **Q4:** Possible changes:

    - **Snowflake Professor from CourseSection** — professors teach sections, so Professor could connect through CourseSection instead of directly to the fact table. This removes ProfID FK from the fact table.
    - **Normalize Course from Section** — separate CourseName and Units into a dim_course table, linked from CourseSection.
    - **Normalize Department from Professor** — DepartmentID and DepartmentName could be a separate table.

    **Q5:** Use **SCD Type 2**. Split each dimension into stable attributes and changing attributes:

    - **Student:** StudentID and StudentName are stable; Major can change → SCD Type 2 with effective dates and current_flag
    - **Professor:** ProfName is stable; Department and Title can change → SCD Type 2
    - **CourseSection:** RoomID can change → SCD Type 2

    This preserves the historical context of each fact row.

---

## Part C: Design Your Project's Star Schema (20 min + homework)

Now apply the 4-step process to **your team's project domain** (the one from Phase 1).

### Step 1: Select the Business Process

What is the central measurable activity in your domain?

> Your answer: _______________

### Step 2: Declare the Grain

What does one row in your fact table represent?

> "One row per _______________"

### Step 3: Identify the Dimensions

| Dimension | Key Attributes | Source OLTP Table(s) |
|-----------|---------------|---------------------|
| **dim_date** | date_key, full_date, day_of_week, month, quarter, year, is_weekend | (pre-populated) |
| | | |
| | | |
| | | |
| | | |

### Step 4: Identify the Facts/Measures

| Measure | Additive? | Example Analytical Question |
|---------|-----------|---------------------------|
| | | |
| | | |
| | | |
| | | |

### Draw Your Star Schema

```
                        ┌──────────────────┐
                        │   dim_?????      │
                        │──────────────────│
                        │                  │
                        └────────┬─────────┘
                                 │
┌──────────────────┐  ┌──────────┴───────────────┐  ┌──────────────────┐
│   dim_?????      │  │    fact_?????            │  │   dim_?????      │
│──────────────────│  │──────────────────────────│  │──────────────────│
│                  ├──┤  FKs:                    ├──┤                  │
│                  │  │                          │  │                  │
└──────────────────┘  │  Measures:               │  └──────────────────┘
                      │                          │
                      └──────────┬───────────────┘
                                 │
                        ┌────────┴─────────┐
                        │   dim_?????      │
                        │──────────────────│
                        │                  │
                        └──────────────────┘
```

### Estimate Your Fact Table Size

- Unique values per dimension: ___ × ___ × ___ × ___ = ___
- Fill rate estimate: ___% 
- Estimated rows: ___
- Estimated size: ___ rows × ___ columns × 15 bytes = ___ bytes

### Write 3 Analytical Questions

Your star schema should be able to answer questions like these:

1. "_______________?"
2. "_______________?"
3. "_______________?"

---

## Homework (Due Next Week)

1. **Finalize your star schema** with your team
2. **Write the CREATE TABLE DDL** for your fact + dimension tables
3. **Read:** Kimball Ch 2 (pp. 37-68) — Techniques Overview, and Ch 3 (pp. 69-100) — Retail Sales case study
4. **Think about:** What 5 analytical questions would a manager in your domain want answered? (You'll need 5+ analytical queries for Phase 2)
5. **Download and review:** [Kimball Cheat Sheet](kimball-cheat-sheet.md)

---

*IS480 — Advanced Database Management | Week 10 Exercise | CSULB Spring 2026 | Prof. Jose Pineda*

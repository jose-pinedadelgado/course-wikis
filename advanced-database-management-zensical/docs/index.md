# IS480 — Advanced Database Management

> From Oracle PL/SQL to Data Warehousing to Spark — the full data pipeline.

This wiki covers the hands-on content for IS480 at California State University, Long Beach (Spring 2026).

## Course Topics

| Week | Topic | Status |
|------|-------|--------|
| 1–2 | [Intro to Oracle PL/SQL](modules/intro-plsql.md) | ✅ Complete |
| 3 | [Functions, Procedures & Conditionals](modules/functions-procedures.md) | ✅ Complete |
| 4 | [SQL in PL/SQL](modules/sql-in-plsql.md) | ✅ Complete |
| 5 | [PL/SQL Cursors](modules/cursors.md) | ✅ Complete |
| 6–7 | Exceptions & Packages | ✅ Complete |
| 8–9 | [Midterm](study-guide.md) & Project Kickoff | ✅ Complete |
| 10 | [Star Schema Design](modules/week10-star-schema-lecture.md) / [Exercise](modules/week10-exercise.md) | ✅ Complete |
| — | **Spring Break (Mar 30 – Apr 5)** | — |
| 11 | ETL with PL/SQL + Analytical SQL | 🔜 Coming Soon |
| 12 | Spark & Databricks Crash Course | 📅 Apr 13 |
| 13 | Scale & Presentations | 📅 Apr 20 |
| 14–15 | Project Presentations & Finals Prep | 📅 Apr 27+ |

### Reference Materials

- [📋 Kimball Cheat Sheet](modules/kimball-cheat-sheet.md) — Star schema design reference
- *More references will be published as we progress*

### Semester Project

- [📝 Project Specification](modules/project-spec.md) — Full requirements (Phases 1–3)
- [📝 Sample: BambooBudget](modules/project-sample.md) — Worked example
- **Phase 2 Due:** Week 13 (Apr 20) — Star schema DDL + ETL procedures
- **Phase 3 Due:** Week 14 (Apr 27) — Spark comparison
- **Live Demos:** Weeks 13–14

## Tools & Setup

| Tool | Purpose |
|------|---------|
| **Oracle Database** | Course database server |
| **SQL Developer** | IDE for Oracle SQL and PL/SQL |
| **VS Code + SQL Developer Extension** | Alternative IDE |
| **DataCamp** | Supplementary SQL exercises |

### Getting Connected

1. Download [SQL Developer](https://www.oracle.com/database/sqldeveloper/technologies/download/) or the VS Code extension
2. Use your credentials from the class spreadsheet
3. Test your connection with: `SELECT * FROM Students;`

## Lab Schema

All exercises use the **Student Enrollment** schema:

- **Students** — Student records (snum, name, standing, GPA)
- **Courses** — Course catalog (dept, cnum, title, credits, standing requirement)
- **SchClasses** — Scheduled class sections (classnum, semester, instructor, capacity)
- **Enrollments** — Student enrollments with grades

See [Schema & Dataset](modules/lab-schema.md) for the full DDL and seed data.

---

*Course: IS480 — Advanced Database Management | CSULB | Spring 2026 | Prof. Jose Pineda*

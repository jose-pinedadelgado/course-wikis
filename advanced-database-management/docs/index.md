# IS480 — Advanced Database Management

> Oracle PL/SQL programming: from basics to cursors and beyond.

This wiki covers the hands-on PL/SQL content for IS480 at California State University, Long Beach (Spring 2026).

## Course Topics

| Week | Topic | Status |
|------|-------|--------|
| 1-2 | [Intro to Oracle PL/SQL](modules/intro-plsql.md) | ✅ |
| 3 | [Functions, Procedures & Conditionals](modules/functions-procedures.md) | ✅ |
| 4 | [SQL in PL/SQL](modules/sql-in-plsql.md) | ✅ |
| 5 | [PL/SQL Cursors](modules/cursors.md) | ✅ |

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

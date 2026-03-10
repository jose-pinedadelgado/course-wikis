# Intro to Oracle PL/SQL

## What is PL/SQL??

PL/SQL (Procedural Language/SQL) is Oracle's procedural extension to SQL. It combines the data manipulation power of SQL with the processing power of procedural languages.

### Why PL/SQL?

- **SQL alone** can query and manipulate data, but can't do conditional logic, loops, or error handling
- **PL/SQL** adds programming constructs: variables, conditions, loops, procedures, functions, cursors, and exception handling
- Runs **inside the database server** — less network overhead than sending individual SQL statements from a client

## PL/SQL Block Structure

Every PL/SQL program is made of **blocks**. A block has three sections:

```sql
DECLARE
    -- Variable declarations (optional)
BEGIN
    -- Executable statements (required)
EXCEPTION
    -- Error handling (optional)
END;
/
```

!!! tip
    The `/` on its own line tells SQL Developer to execute the block.

### Anonymous Block Example

```sql
SET SERVEROUTPUT ON;

DECLARE
    v_message VARCHAR2(50) := 'Hello, PL/SQL!';
BEGIN
    DBMS_OUTPUT.PUT_LINE(v_message);
END;
/
```

!!! note "`SET SERVEROUTPUT ON`"
    Always run this first in your session — otherwise `DBMS_OUTPUT.PUT_LINE` output won't display.

## Variables and Data Types

### Declaring Variables

```sql
DECLARE
    v_name      VARCHAR2(50) := 'Andy';
    v_age       NUMBER := 21;
    v_gpa       NUMBER(3,2) := 3.85;
    v_is_active BOOLEAN := TRUE;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Student: ' || v_name || ', GPA: ' || v_gpa);
END;
/
```

### Anchored Types with `%TYPE`

Instead of hardcoding data types, anchor them to table columns:

```sql
DECLARE
    v_sname Students.sname%TYPE;
    v_gpa   Students.gpa%TYPE;
BEGIN
    SELECT sname, gpa INTO v_sname, v_gpa
    FROM Students
    WHERE snum = 101;
    
    DBMS_OUTPUT.PUT_LINE(v_sname || ' has GPA: ' || v_gpa);
END;
/
```

!!! tip "Why `%TYPE`?"
    If the column type changes in the table, your code automatically adapts. No need to update variable declarations.

## SELECT INTO

Use `SELECT INTO` to fetch data from a table into PL/SQL variables:

```sql
DECLARE
    v_sname   Students.sname%TYPE;
    v_standing Students.standing%TYPE;
BEGIN
    SELECT sname, standing INTO v_sname, v_standing
    FROM Students
    WHERE snum = 101;
    
    DBMS_OUTPUT.PUT_LINE(v_sname || ' is standing ' || v_standing);
END;
/
```

!!! warning
    `SELECT INTO` must return **exactly one row**. If it returns zero rows, you get `NO_DATA_FOUND`. If it returns multiple rows, you get `TOO_MANY_ROWS`. Use exception handling or cursors for multi-row queries.

## Oracle Database Architecture (Overview)

### Key Concepts

- **Schema** — A named collection of database objects (tables, views, procedures) owned by a user
- **Tablespace** — Logical storage container for data files
- **Instance** — The running Oracle database (memory + processes)
- **Data Dictionary** — System tables that describe the database structure

### Useful System Queries

```sql
-- See your current schema
SELECT SYS_CONTEXT('USERENV', 'CURRENT_SCHEMA') FROM dual;

-- List all tables you own
SELECT table_name FROM user_tables;

-- List columns in a table
SELECT column_name, data_type, data_length
FROM user_tab_columns
WHERE table_name = 'STUDENTS';
```

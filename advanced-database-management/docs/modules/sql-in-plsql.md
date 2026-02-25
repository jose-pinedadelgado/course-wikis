# SQL in PL/SQL

## Overview

PL/SQL can embed SQL statements directly. This is one of its most powerful features — you can mix procedural logic with data manipulation seamlessly.

### What SQL Can You Use in PL/SQL?

| SQL Type | Supported | Notes |
|----------|-----------|-------|
| `SELECT INTO` | ✅ | Must return exactly one row |
| `INSERT` | ✅ | Direct use |
| `UPDATE` | ✅ | Direct use |
| `DELETE` | ✅ | Direct use |
| `COMMIT` / `ROLLBACK` | ✅ | Transaction control |
| DDL (`CREATE`, `ALTER`, `DROP`) | ⚠️ | Must use `EXECUTE IMMEDIATE` |

## Loops in PL/SQL

### Basic LOOP

```sql
DECLARE
    v_counter NUMBER := 1;
BEGIN
    LOOP
        DBMS_OUTPUT.PUT_LINE('Counter: ' || v_counter);
        v_counter := v_counter + 1;
        EXIT WHEN v_counter > 5;
    END LOOP;
END;
/
```

### WHILE LOOP

```sql
DECLARE
    v_counter NUMBER := 1;
BEGIN
    WHILE v_counter <= 5 LOOP
        DBMS_OUTPUT.PUT_LINE('Counter: ' || v_counter);
        v_counter := v_counter + 1;
    END LOOP;
END;
/
```

### FOR LOOP

```sql
BEGIN
    FOR i IN 1..5 LOOP
        DBMS_OUTPUT.PUT_LINE('Iteration: ' || i);
    END LOOP;
END;
/
```

!!! tip "FOR loop variable"
    The loop variable (`i`) is declared automatically — you don't need to declare it in the `DECLARE` section. It's read-only inside the loop.

### Reverse FOR LOOP

```sql
BEGIN
    FOR i IN REVERSE 1..5 LOOP
        DBMS_OUTPUT.PUT_LINE('Countdown: ' || i);
    END LOOP;
END;
/
```

## Combining SQL and Loops

### Example: Print All Student Names

```sql
SET SERVEROUTPUT ON;

DECLARE
    v_sname Students.sname%TYPE;
    v_snum  Students.snum%TYPE;
BEGIN
    FOR i IN 101..103 LOOP
        SELECT sname INTO v_sname
        FROM Students
        WHERE snum = i;
        
        DBMS_OUTPUT.PUT_LINE('Student ' || i || ': ' || v_sname);
    END LOOP;
END;
/
```

!!! warning
    This approach only works if you know the exact range of student numbers. For dynamic multi-row queries, use **cursors** (covered in Week 5).

## Procedures with SQL

### Example: AddMe with Credit Limit Check

Enroll a student only if adding the class keeps their semester credits ≤ 15:

```sql
CREATE OR REPLACE PROCEDURE AddMe3 (
    p_snum     IN NUMBER,
    p_classnum IN NUMBER
) AS
    v_student_standing Students.standing%TYPE;
    v_course_standing  Courses.standing%TYPE;
    v_semester         SchClasses.semester%TYPE;
    v_year             SchClasses.class_year%TYPE;
    v_new_credits      Courses.crhr%TYPE;
    v_total_credits    NUMBER := 0;
BEGIN
    -- Get student standing
    SELECT s.standing INTO v_student_standing
    FROM Students s
    WHERE s.snum = p_snum;
    
    -- Get course requirements and term info
    SELECT c.standing, c.crhr, sc.semester, sc.class_year
    INTO v_course_standing, v_new_credits, v_semester, v_year
    FROM SchClasses sc
    JOIN Courses c ON sc.cnum = c.cnum
    WHERE sc.classnum = p_classnum;
    
    -- Check standing
    IF v_student_standing < v_course_standing THEN
        DBMS_OUTPUT.PUT_LINE('Failed: Student standing too low.');
        RETURN;
    END IF;
    
    -- Get current credits for that semester
    SELECT NVL(SUM(c.crhr), 0) INTO v_total_credits
    FROM Enrollments e
    JOIN SchClasses sc ON e.classnum = sc.classnum
    JOIN Courses c ON sc.cnum = c.cnum
    WHERE e.snum = p_snum
    AND sc.semester = v_semester
    AND sc.class_year = v_year;
    
    -- Check credit limit
    IF (v_total_credits + v_new_credits) <= 15 THEN
        INSERT INTO Enrollments (snum, classnum) VALUES (p_snum, p_classnum);
        DBMS_OUTPUT.PUT_LINE('Successfully Enrolled ' || p_snum || ' into ' || p_classnum || '.');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Failed: Adding this class would exceed the 15 credit limit.');
    END IF;
END;
/
```

## GPA Calculation

### Example: Update_GPA Procedure

Recompute a student's GPA using completed enrollments weighted by credit hours:

$$GPA = \frac{\sum (grade\_value \times credit\_hours)}{\sum credit\_hours}$$

Where: A=4, B=3, C=2, D=1

```sql
CREATE OR REPLACE PROCEDURE Update_GPA (p_snum IN NUMBER) AS
    v_total_points NUMBER := 0;
    v_total_hours  NUMBER := 0;
    v_grade_value  NUMBER;
    v_crhr         Courses.crhr%TYPE;
    v_grade        Enrollments.grade%TYPE;
BEGIN
    -- For each enrollment, calculate weighted grade points
    FOR rec IN (
        SELECT e.grade, c.crhr
        FROM Enrollments e
        JOIN SchClasses sc ON e.classnum = sc.classnum
        JOIN Courses c ON sc.cnum = c.cnum
        WHERE e.snum = p_snum
        AND e.grade IS NOT NULL
    ) LOOP
        v_grade_value := CASE rec.grade
            WHEN 'A' THEN 4
            WHEN 'B' THEN 3
            WHEN 'C' THEN 2
            WHEN 'D' THEN 1
            ELSE 0
        END;
        
        v_total_points := v_total_points + (v_grade_value * rec.crhr);
        v_total_hours := v_total_hours + rec.crhr;
    END LOOP;
    
    IF v_total_hours > 0 THEN
        UPDATE Students
        SET gpa = ROUND(v_total_points / v_total_hours, 2)
        WHERE snum = p_snum;
        
        DBMS_OUTPUT.PUT_LINE('GPA updated to: ' || ROUND(v_total_points / v_total_hours, 2));
    ELSE
        DBMS_OUTPUT.PUT_LINE('No graded enrollments found.');
    END IF;
END;
/
```

!!! note "Example Calculation"
    Student gets A (4) in a 3-credit course and D (1) in a 2-credit course:
    GPA = (4×3 + 1×2) / (3+2) = 14/5 = **2.8**

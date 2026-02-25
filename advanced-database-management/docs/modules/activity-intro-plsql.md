# Activity: Intro to Oracle & PL/SQL

Practice exercises for getting started with Oracle and PL/SQL blocks.

## Exercise 1: Your First Block

Write an anonymous PL/SQL block that:

1. Declares a variable for your name
2. Declares a variable for the current date
3. Prints: "Hello, [name]! Today is [date]."

??? example "Solution"
    ```sql
    SET SERVEROUTPUT ON;
    
    DECLARE
        v_name VARCHAR2(50) := 'Jose';
        v_today DATE := SYSDATE;
    BEGIN
        DBMS_OUTPUT.PUT_LINE('Hello, ' || v_name || '! Today is ' || TO_CHAR(v_today, 'MM/DD/YYYY') || '.');
    END;
    /
    ```

## Exercise 2: SELECT INTO

Write a block that retrieves the name and GPA of student 101 and prints them.

??? example "Solution"
    ```sql
    SET SERVEROUTPUT ON;
    
    DECLARE
        v_sname Students.sname%TYPE;
        v_gpa   Students.gpa%TYPE;
    BEGIN
        SELECT sname, gpa INTO v_sname, v_gpa
        FROM Students
        WHERE snum = 101;
        
        DBMS_OUTPUT.PUT_LINE('Student: ' || v_sname || ', GPA: ' || v_gpa);
    END;
    /
    ```

## Exercise 3: Exploring the Schema

Run these queries to explore the lab database:

```sql
-- How many students?
SELECT COUNT(*) FROM Students;

-- What courses are offered?
SELECT dept, cnum, ctitle FROM Courses;

-- Who is enrolled where?
SELECT s.sname, c.ctitle, e.grade
FROM Students s
JOIN Enrollments e ON s.snum = e.snum
JOIN SchClasses sc ON e.classnum = sc.classnum
JOIN Courses c ON sc.cnum = c.cnum;
```

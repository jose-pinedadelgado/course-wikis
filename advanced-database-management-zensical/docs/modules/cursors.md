# PL/SQL Cursors

## What is a Cursor?

A cursor is a **pointer to a result set** returned by a SQL query. While `SELECT INTO` handles single-row results, cursors let you process **multiple rows** one at a time.

### Types of Cursors

| Type | Description |
|------|-------------|
| **Implicit** | Created automatically by Oracle for single-row queries (`SELECT INTO`) |
| **Explicit** | Declared and managed by the programmer for multi-row queries |

## Explicit Cursor Lifecycle

```
DECLARE → OPEN → FETCH → CLOSE
```

### Syntax

```sql
DECLARE
    -- 1. Declare the cursor
    CURSOR cursor_name IS
        SELECT column1, column2
        FROM table_name
        WHERE condition;
    
    -- Variables to hold fetched data
    v_col1 table_name.column1%TYPE;
    v_col2 table_name.column2%TYPE;
BEGIN
    -- 2. Open the cursor
    OPEN cursor_name;
    
    -- 3. Fetch rows in a loop
    LOOP
        FETCH cursor_name INTO v_col1, v_col2;
        EXIT WHEN cursor_name%NOTFOUND;
        
        -- Process each row
        DBMS_OUTPUT.PUT_LINE(v_col1 || ' - ' || v_col2);
    END LOOP;
    
    -- 4. Close the cursor
    CLOSE cursor_name;
END;
/
```

## Cursor Attributes

| Attribute | Description |
|-----------|-------------|
| `%FOUND` | TRUE if the last FETCH returned a row |
| `%NOTFOUND` | TRUE if the last FETCH returned no row |
| `%ROWCOUNT` | Number of rows fetched so far |
| `%ISOPEN` | TRUE if the cursor is currently open |

## Example: Update Standing Based on Credits

This procedure uses a cursor to loop through all students, calculate their total credit hours, and update their standing accordingly.

```sql
CREATE OR REPLACE PROCEDURE p_update_standing AS
    v_snum         Students.snum%TYPE;
    v_credit_hours NUMBER;
    
    -- Cursor: total credit hours per student
    CURSOR c_credit_hours IS
        SELECT e.snum, SUM(c.crhr)
        FROM Enrollments e
        JOIN SchClasses sc ON e.classnum = sc.classnum
        JOIN Courses c ON sc.cnum = c.cnum
        GROUP BY e.snum;
BEGIN
    OPEN c_credit_hours;
    LOOP
        FETCH c_credit_hours INTO v_snum, v_credit_hours;
        EXIT WHEN c_credit_hours%NOTFOUND;
        
        -- Determine new standing based on credit hours
        IF v_credit_hours < 30 THEN
            UPDATE Students SET standing = 1 WHERE snum = v_snum;
        ELSIF v_credit_hours < 60 THEN
            UPDATE Students SET standing = 2 WHERE snum = v_snum;
        ELSIF v_credit_hours < 90 THEN
            UPDATE Students SET standing = 3 WHERE snum = v_snum;
        ELSIF v_credit_hours < 120 THEN
            UPDATE Students SET standing = 4 WHERE snum = v_snum;
        ELSE
            UPDATE Students SET standing = 5 WHERE snum = v_snum;
        END IF;
        
        DBMS_OUTPUT.PUT_LINE('Student ' || v_snum || ': ' || v_credit_hours || 
                             ' credits → Standing updated');
    END LOOP;
    CLOSE c_credit_hours;
    
    COMMIT;
END;
/
```

## Cursor FOR Loop (Simplified)

Oracle provides a shortcut that handles OPEN, FETCH, and CLOSE automatically:

```sql
CREATE OR REPLACE PROCEDURE p_update_standing_v2 AS
BEGIN
    FOR rec IN (
        SELECT e.snum, SUM(c.crhr) AS total_credits
        FROM Enrollments e
        JOIN SchClasses sc ON e.classnum = sc.classnum
        JOIN Courses c ON sc.cnum = c.cnum
        GROUP BY e.snum
    ) LOOP
        IF rec.total_credits < 30 THEN
            UPDATE Students SET standing = 1 WHERE snum = rec.snum;
        ELSIF rec.total_credits < 60 THEN
            UPDATE Students SET standing = 2 WHERE snum = rec.snum;
        ELSIF rec.total_credits < 90 THEN
            UPDATE Students SET standing = 3 WHERE snum = rec.snum;
        ELSE
            UPDATE Students SET standing = 4 WHERE snum = rec.snum;
        END IF;
    END LOOP;
    
    COMMIT;
END;
/
```

!!! tip "Cursor FOR Loop vs Explicit Cursor"
    - **Cursor FOR loop** is cleaner and less error-prone (no need to OPEN/FETCH/CLOSE)
    - **Explicit cursor** gives more control (you can FETCH conditionally, use attributes like `%ROWCOUNT`)
    - Use FOR loop when you just need to process all rows; use explicit when you need fine-grained control

## Parameterized Cursors

Cursors can accept parameters, making them reusable:

```sql
DECLARE
    CURSOR c_enrollments (p_snum NUMBER) IS
        SELECT sc.classnum, c.ctitle, e.grade
        FROM Enrollments e
        JOIN SchClasses sc ON e.classnum = sc.classnum
        JOIN Courses c ON sc.cnum = c.cnum
        WHERE e.snum = p_snum;
BEGIN
    DBMS_OUTPUT.PUT_LINE('--- Enrollments for Student 101 ---');
    FOR rec IN c_enrollments(101) LOOP
        DBMS_OUTPUT.PUT_LINE(rec.ctitle || ' - Grade: ' || NVL(rec.grade, 'N/A'));
    END LOOP;
    
    DBMS_OUTPUT.PUT_LINE('--- Enrollments for Student 102 ---');
    FOR rec IN c_enrollments(102) LOOP
        DBMS_OUTPUT.PUT_LINE(rec.ctitle || ' - Grade: ' || NVL(rec.grade, 'N/A'));
    END LOOP;
END;
/
```

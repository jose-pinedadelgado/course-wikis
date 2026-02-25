# Functions, Procedures & Conditionals

## Procedures

A **procedure** is a named PL/SQL block that performs an action. It can accept parameters and be called repeatedly.

### Syntax

```sql
CREATE OR REPLACE PROCEDURE procedure_name (
    p_param1 IN  data_type,
    p_param2 OUT data_type
) AS
    -- local variable declarations
BEGIN
    -- executable statements
EXCEPTION
    -- error handling
END;
/
```

### Parameter Modes

| Mode | Description |
|------|-------------|
| `IN` | Input only (default). Cannot be modified inside the procedure. |
| `OUT` | Output only. Value is returned to the caller. |
| `IN OUT` | Both input and output. Can be read and modified. |

### Example: Simple Enrollment Procedure

```sql
CREATE OR REPLACE PROCEDURE AddMe (
    p_snum     IN NUMBER,
    p_classnum IN NUMBER
) AS
BEGIN
    INSERT INTO Enrollments (snum, classnum) 
    VALUES (p_snum, p_classnum);
    
    DBMS_OUTPUT.PUT_LINE('Successfully Enrolled ' || p_snum || ' into ' || p_classnum || '.');
END;
/
```

**Calling the procedure:**

```sql
EXEC AddMe(101, 10113);
-- or in a PL/SQL block:
BEGIN
    AddMe(101, 10113);
END;
/
```

## Functions

A **function** is like a procedure but **returns a value**. Use functions when you need to compute and return something.

### Syntax

```sql
CREATE OR REPLACE FUNCTION function_name (
    p_param1 IN data_type
) RETURN return_type AS
    v_result return_type;
BEGIN
    -- compute v_result
    RETURN v_result;
END;
/
```

### Example: Get Student Name

```sql
CREATE OR REPLACE FUNCTION get_student_name (
    p_snum IN NUMBER
) RETURN VARCHAR2 AS
    v_name Students.sname%TYPE;
BEGIN
    SELECT sname INTO v_name
    FROM Students
    WHERE snum = p_snum;
    
    RETURN v_name;
END;
/
```

**Using the function:**

```sql
SELECT get_student_name(101) FROM dual;

-- or in PL/SQL:
DECLARE
    v_name VARCHAR2(50);
BEGIN
    v_name := get_student_name(101);
    DBMS_OUTPUT.PUT_LINE('Name: ' || v_name);
END;
/
```

!!! tip "Procedure vs. Function"
    - **Procedure** → does something (INSERT, UPDATE, prints output)
    - **Function** → computes and returns a value
    - Functions can be used in SQL statements; procedures cannot

## Conditional Logic: IF / ELSIF / ELSE

### Basic IF

```sql
IF condition THEN
    -- statements
END IF;
```

### IF-ELSE

```sql
IF condition THEN
    -- when true
ELSE
    -- when false
END IF;
```

### IF-ELSIF-ELSE

```sql
IF condition1 THEN
    -- when condition1 is true
ELSIF condition2 THEN
    -- when condition2 is true
ELSE
    -- when none are true
END IF;
```

### Example: Enrollment with Standing Check

```sql
CREATE OR REPLACE PROCEDURE AddMe2 (
    p_snum     IN NUMBER,
    p_classnum IN NUMBER
) AS
    v_student_standing Students.standing%TYPE;
    v_course_standing  Courses.standing%TYPE;
BEGIN
    -- Get student's standing
    SELECT s.standing INTO v_student_standing
    FROM Students s
    WHERE s.snum = p_snum;
    
    -- Get course's required standing (via SchClasses → Courses)
    SELECT c.standing INTO v_course_standing
    FROM SchClasses sc
    JOIN Courses c ON sc.cnum = c.cnum
    WHERE sc.classnum = p_classnum;
    
    -- Check standing requirement
    IF v_student_standing >= v_course_standing THEN
        INSERT INTO Enrollments (snum, classnum) VALUES (p_snum, p_classnum);
        DBMS_OUTPUT.PUT_LINE('Successfully Enrolled ' || p_snum || ' into ' || p_classnum || '.');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Failed to Enroll ' || p_snum || ' into ' || p_classnum || 
                             '. Student Standing is too low.');
    END IF;
END;
/
```

## CASE Statements

An alternative to chained IF-ELSIF for checking a single value:

```sql
CASE v_standing
    WHEN 1 THEN DBMS_OUTPUT.PUT_LINE('Freshman');
    WHEN 2 THEN DBMS_OUTPUT.PUT_LINE('Sophomore');
    WHEN 3 THEN DBMS_OUTPUT.PUT_LINE('Junior');
    WHEN 4 THEN DBMS_OUTPUT.PUT_LINE('Senior');
    ELSE DBMS_OUTPUT.PUT_LINE('Unknown');
END CASE;
```

### Searched CASE (with conditions)

```sql
CASE
    WHEN v_gpa >= 3.7 THEN v_honors := 'Summa Cum Laude';
    WHEN v_gpa >= 3.5 THEN v_honors := 'Magna Cum Laude';
    WHEN v_gpa >= 3.0 THEN v_honors := 'Cum Laude';
    ELSE v_honors := 'No honors';
END CASE;
```

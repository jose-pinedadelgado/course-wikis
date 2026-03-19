# Lab 4 Solutions — Packages & Exception Handling

!!! info "Prerequisites"
    This lab builds on all previous labs. You should be comfortable with procedures, functions, cursors, and SQL in PL/SQL before tackling exception handling and packages.

## Part 1 — Exception Handling

### Q1: Pre-defined Exception — NO_DATA_FOUND

Write `p_get_student_name(p_snum)` that handles the case when the student doesn't exist.

```sql
CREATE OR REPLACE PROCEDURE p_get_student_name(
    p_snum NUMBER
) AS
    v_name Students.sname%TYPE;
BEGIN
    SELECT sname INTO v_name
    FROM Students
    WHERE snum = p_snum;

    DBMS_OUTPUT.PUT_LINE('Student: ' || v_name);

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Error: Student ' || p_snum || ' does not exist.');
END;
/
```

!!! note "Process Flow"
    1. Try to `SELECT INTO` — if the student exists, it works fine
    2. If no row is found, Oracle raises `NO_DATA_FOUND` automatically
    3. Our `EXCEPTION` block catches it and prints a friendly message instead of crashing

**Test:**
```sql
EXEC p_get_student_name(101);   -- prints: Student: Andy
EXEC p_get_student_name(999);   -- prints: Error: Student 999 does not exist.
```

---

### Q2: Pre-defined Exception — TOO_MANY_ROWS

Write `p_find_student_by_major(p_majorid)` that handles multiple results.

```sql
CREATE OR REPLACE PROCEDURE p_find_student_by_major(
    p_majorid VARCHAR2
) AS
    v_name Students.sname%TYPE;
BEGIN
    SELECT sname INTO v_name
    FROM Students
    WHERE majorid = p_majorid;

    DBMS_OUTPUT.PUT_LINE('Found: ' || v_name);

EXCEPTION
    WHEN TOO_MANY_ROWS THEN
        DBMS_OUTPUT.PUT_LINE('Error: Multiple students found for major ' ||
                             p_majorid || '. Use a cursor instead.');
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Error: No students found for major ' || p_majorid || '.');
END;
/
```

!!! note "Why TOO_MANY_ROWS?"
    `SELECT INTO` expects **exactly one row**. If the query returns 0 rows → `NO_DATA_FOUND`. If it returns 2+ rows → `TOO_MANY_ROWS`. Both will crash the program unless handled. When you need multiple rows, use a **cursor** instead.

**Test:**
```sql
EXEC p_find_student_by_major('IS');     -- Error: Multiple students found...
EXEC p_find_student_by_major('ACCT');   -- Error: No students found...
```

---

### Q3: User-Defined Exception

Write `p_enroll_with_standing(p_snum, p_classnum)` using a user-defined exception.

```sql
CREATE OR REPLACE PROCEDURE p_enroll_with_standing(
    p_snum     NUMBER,
    p_classnum NUMBER
) AS
    v_student_standing Students.standing%TYPE;
    v_course_standing  Courses.standing%TYPE;
    v_cnum             SchClasses.cnum%TYPE;

    -- Declare user-defined exception
    e_standing_too_low EXCEPTION;
BEGIN
    -- Get student standing
    SELECT standing INTO v_student_standing
    FROM Students WHERE snum = p_snum;

    -- Get course standing requirement
    SELECT cnum INTO v_cnum
    FROM SchClasses WHERE classnum = p_classnum;

    SELECT standing INTO v_course_standing
    FROM Courses WHERE cnum = v_cnum;

    -- Check standing — raise our custom exception if too low
    IF v_student_standing < v_course_standing THEN
        RAISE e_standing_too_low;
    END IF;

    -- If we get here, standing is OK
    INSERT INTO Enrollments (classnum, snum, grade)
    VALUES (p_classnum, p_snum, NULL);
    DBMS_OUTPUT.PUT_LINE('Successfully enrolled student ' || p_snum ||
                         ' in class ' || p_classnum || '.');

EXCEPTION
    WHEN e_standing_too_low THEN
        DBMS_OUTPUT.PUT_LINE('Error: Student standing is too low to enroll in this course.');
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Error: Student or class not found.');
END;
/
```

!!! note "User-Defined Exceptions"
    1. **DECLARE** the exception: `e_standing_too_low EXCEPTION;`
    2. **RAISE** it when the business rule is violated: `RAISE e_standing_too_low;`
    3. **HANDLE** it in the EXCEPTION block: `WHEN e_standing_too_low THEN ...`

---

### Q4: RAISE_APPLICATION_ERROR

Write `p_enroll_strict(p_snum, p_classnum)` with custom error codes.

```sql
CREATE OR REPLACE PROCEDURE p_enroll_strict(
    p_snum     NUMBER,
    p_classnum NUMBER
) AS
    v_student_standing Students.standing%TYPE;
    v_course_standing  Courses.standing%TYPE;
    v_cnum             SchClasses.cnum%TYPE;
    v_total_credits    NUMBER;
    v_new_credits      Courses.crhr%TYPE;
    v_semester         SchClasses.semester%TYPE;
    v_year             SchClasses.year%TYPE;
BEGIN
    -- Validate student exists
    BEGIN
        SELECT standing INTO v_student_standing
        FROM Students WHERE snum = p_snum;
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            RAISE_APPLICATION_ERROR(-20001, 'Student does not exist.');
    END;

    -- Validate class exists
    BEGIN
        SELECT cnum, semester, year INTO v_cnum, v_semester, v_year
        FROM SchClasses WHERE classnum = p_classnum;
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            RAISE_APPLICATION_ERROR(-20002, 'Class does not exist.');
    END;

    -- Get course info
    SELECT standing, crhr INTO v_course_standing, v_new_credits
    FROM Courses WHERE cnum = v_cnum;

    -- Check standing
    IF v_student_standing < v_course_standing THEN
        RAISE_APPLICATION_ERROR(-20003, 'Student standing is too low for this course.');
    END IF;

    -- Check 15-credit limit
    SELECT NVL(SUM(co.crhr), 0) INTO v_total_credits
    FROM Enrollments e
    INNER JOIN SchClasses sc ON e.classnum = sc.classnum
    INNER JOIN Courses co ON sc.cnum = co.cnum
    WHERE e.snum = p_snum
    AND sc.semester = v_semester
    AND sc.year = v_year;

    IF (v_total_credits + v_new_credits) > 15 THEN
        RAISE_APPLICATION_ERROR(-20004, 'Enrollment would exceed 15 credit hour limit.');
    END IF;

    -- All checks passed — enroll
    INSERT INTO Enrollments (classnum, snum, grade)
    VALUES (p_classnum, p_snum, NULL);
    DBMS_OUTPUT.PUT_LINE('Enrolled student ' || p_snum || ' in class ' || p_classnum || '.');
    COMMIT;
END;
/
```

!!! warning "RAISE_APPLICATION_ERROR vs DBMS_OUTPUT"
    `DBMS_OUTPUT.PUT_LINE` prints a message to the output buffer — the calling program doesn't know an error occurred. `RAISE_APPLICATION_ERROR` sends a **real error** back to the caller (SQL Developer, your application, etc.) with a code (-20001 to -20999) and message. Use it when the caller needs to know something went wrong.

---

### Q5: SQLERRM and SQLCODE — WHEN OTHERS

Write `p_safe_update_gpa(p_snum)` with a generic error handler.

```sql
CREATE OR REPLACE PROCEDURE p_safe_update_gpa(
    p_snum NUMBER
) AS
    v_grade      Enrollments.grade%TYPE;
    v_crhr       Courses.crhr%TYPE;
    v_total_pts  NUMBER := 0;
    v_total_hrs  NUMBER := 0;
    v_gpa        NUMBER;

    CURSOR c_grades IS
        SELECT e.grade, co.crhr
        FROM Enrollments e
        INNER JOIN SchClasses sc ON e.classnum = sc.classnum
        INNER JOIN Courses co ON sc.cnum = co.cnum
        WHERE e.snum = p_snum
        AND e.grade IS NOT NULL;
BEGIN
    FOR rec IN c_grades LOOP
        v_total_pts := v_total_pts + (Grading(rec.grade) * rec.crhr);
        v_total_hrs := v_total_hrs + rec.crhr;
    END LOOP;

    v_gpa := ROUND(v_total_pts / v_total_hrs, 2);
    UPDATE Students SET gpa = v_gpa WHERE snum = p_snum;
    DBMS_OUTPUT.PUT_LINE('GPA updated to ' || v_gpa);

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Student ' || p_snum || ' not found.');
    WHEN ZERO_DIVIDE THEN
        DBMS_OUTPUT.PUT_LINE('Cannot calculate GPA: student has no graded courses.');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Unexpected error occurred.');
        DBMS_OUTPUT.PUT_LINE('Error Code: ' || SQLCODE);
        DBMS_OUTPUT.PUT_LINE('Error Message: ' || SQLERRM);
        ROLLBACK;
END;
/
```

!!! warning "WHEN OTHERS — Use as a Safety Net"
    Always handle **known** exceptions first (`NO_DATA_FOUND`, `ZERO_DIVIDE`), then use `WHEN OTHERS` as a catch-all for unexpected errors. Never use `WHEN OTHERS` as your only handler — you'd hide the real problem.

---

### Q6: Nested Blocks with Exception Handling

Write `p_full_enrollment(p_snum, p_classnum)` with inner validation blocks.

```sql
CREATE OR REPLACE PROCEDURE p_full_enrollment(
    p_snum     NUMBER,
    p_classnum NUMBER
) AS
    e_validation_failed EXCEPTION;
    v_name     Students.sname%TYPE;
    v_classnum SchClasses.classnum%TYPE;
BEGIN
    -- Inner Block #1: Validate student exists
    BEGIN
        SELECT sname INTO v_name
        FROM Students WHERE snum = p_snum;
        DBMS_OUTPUT.PUT_LINE('Student ' || v_name || ' found.');
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            DBMS_OUTPUT.PUT_LINE('Inner Block: Student not found.');
            RAISE e_validation_failed;
    END;

    -- Inner Block #2: Validate class exists
    BEGIN
        SELECT classnum INTO v_classnum
        FROM SchClasses WHERE classnum = p_classnum;
        DBMS_OUTPUT.PUT_LINE('Class ' || p_classnum || ' found.');
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            DBMS_OUTPUT.PUT_LINE('Inner Block: Class not found.');
            RAISE e_validation_failed;
    END;

    -- If both validations passed, enroll
    INSERT INTO Enrollments (classnum, snum, grade)
    VALUES (p_classnum, p_snum, NULL);
    DBMS_OUTPUT.PUT_LINE('Enrollment successful!');

EXCEPTION
    WHEN e_validation_failed THEN
        DBMS_OUTPUT.PUT_LINE('Enrollment aborted due to validation failure.');
END;
/
```

!!! note "How Nested Blocks Work"
    1. Each inner `BEGIN...EXCEPTION...END` block catches its own errors
    2. The inner block can **re-raise** a different exception to the outer block
    3. When the inner block raises `e_validation_failed`, execution jumps to the **outer** EXCEPTION handler
    4. The outer block never reaches the INSERT — the enrollment is aborted

---

### Q7: Handled vs. Unhandled Exceptions

#### Version A — No Handler (will crash)

```sql
CREATE OR REPLACE PROCEDURE p_calc_gpa_no_handler(
    p_snum NUMBER
) AS
    v_total_pts  NUMBER := 0;
    v_total_hrs  NUMBER := 0;
    v_gpa        NUMBER;

    CURSOR c_grades IS
        SELECT e.grade, co.crhr
        FROM Enrollments e
        INNER JOIN SchClasses sc ON e.classnum = sc.classnum
        INNER JOIN Courses co ON sc.cnum = co.cnum
        WHERE e.snum = p_snum
        AND e.grade IS NOT NULL;
BEGIN
    FOR rec IN c_grades LOOP
        v_total_pts := v_total_pts + (Grading(rec.grade) * rec.crhr);
        v_total_hrs := v_total_hrs + rec.crhr;
    END LOOP;

    -- This crashes with ZERO_DIVIDE if no graded courses!
    v_gpa := ROUND(v_total_pts / v_total_hrs, 2);
    DBMS_OUTPUT.PUT_LINE('GPA: ' || v_gpa);
END;
/
```

#### Version B — With Handler (graceful)

```sql
CREATE OR REPLACE PROCEDURE p_calc_gpa_with_handler(
    p_snum NUMBER
) AS
    v_name       Students.sname%TYPE;
    v_total_pts  NUMBER := 0;
    v_total_hrs  NUMBER := 0;
    v_gpa        NUMBER;

    CURSOR c_grades IS
        SELECT e.grade, co.crhr
        FROM Enrollments e
        INNER JOIN SchClasses sc ON e.classnum = sc.classnum
        INNER JOIN Courses co ON sc.cnum = co.cnum
        WHERE e.snum = p_snum
        AND e.grade IS NOT NULL;
BEGIN
    SELECT sname INTO v_name FROM Students WHERE snum = p_snum;

    FOR rec IN c_grades LOOP
        v_total_pts := v_total_pts + (Grading(rec.grade) * rec.crhr);
        v_total_hrs := v_total_hrs + rec.crhr;
    END LOOP;

    v_gpa := ROUND(v_total_pts / v_total_hrs, 2);
    DBMS_OUTPUT.PUT_LINE(v_name || '''s GPA: ' || v_gpa);

EXCEPTION
    WHEN ZERO_DIVIDE THEN
        DBMS_OUTPUT.PUT_LINE('Cannot calculate GPA: student has no graded courses.');
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Error: Student ' || p_snum || ' does not exist.');
END;
/
```

!!! tip "The Lesson"
    Without exception handling, your program **crashes** and the user sees a cryptic Oracle error. With exception handling, the program **recovers gracefully** and gives a meaningful message. Always handle the exceptions you can predict.

---

## Part 2 — Packages

### Q8: Package Specification

```sql
CREATE OR REPLACE PACKAGE student_enrollment_pkg AS

    FUNCTION f_validate_student(p_snum NUMBER) RETURN BOOLEAN;

    FUNCTION f_validate_class(p_classnum NUMBER) RETURN BOOLEAN;

    FUNCTION f_get_semester_credits(p_snum NUMBER, p_classnum NUMBER) RETURN NUMBER;

    PROCEDURE p_enroll(p_snum NUMBER, p_classnum NUMBER);

    PROCEDURE p_update_all_gpa;

    PROCEDURE p_update_all_standing;

END student_enrollment_pkg;
/
```

!!! note "Package Specification"
    The specification is the **interface** — it declares what procedures and functions are available publicly. It does NOT contain any implementation code. Think of it like a table of contents.

---

### Q9: Package Body

```sql
CREATE OR REPLACE PACKAGE BODY student_enrollment_pkg AS

    -- ================================================
    -- f_validate_student: Check if student exists
    -- ================================================
    FUNCTION f_validate_student(p_snum NUMBER) RETURN BOOLEAN AS
        v_count NUMBER;
    BEGIN
        SELECT COUNT(*) INTO v_count
        FROM Students WHERE snum = p_snum;

        RETURN (v_count > 0);
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            RETURN FALSE;
    END f_validate_student;

    -- ================================================
    -- f_validate_class: Check if class exists
    -- ================================================
    FUNCTION f_validate_class(p_classnum NUMBER) RETURN BOOLEAN AS
        v_count NUMBER;
    BEGIN
        SELECT COUNT(*) INTO v_count
        FROM SchClasses WHERE classnum = p_classnum;

        RETURN (v_count > 0);
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            RETURN FALSE;
    END f_validate_class;

    -- ================================================
    -- f_get_semester_credits: Total credits for a
    -- student in the same semester/year as the class
    -- ================================================
    FUNCTION f_get_semester_credits(
        p_snum     NUMBER,
        p_classnum NUMBER
    ) RETURN NUMBER AS
        v_semester SchClasses.semester%TYPE;
        v_year     SchClasses.year%TYPE;
        v_total    NUMBER;
    BEGIN
        -- Get the semester and year of the target class
        SELECT semester, year INTO v_semester, v_year
        FROM SchClasses WHERE classnum = p_classnum;

        -- Sum credits for that semester/year
        SELECT NVL(SUM(co.crhr), 0) INTO v_total
        FROM Enrollments e
        INNER JOIN SchClasses sc ON e.classnum = sc.classnum
        INNER JOIN Courses co ON sc.cnum = co.cnum
        WHERE e.snum = p_snum
        AND sc.semester = v_semester
        AND sc.year = v_year;

        RETURN v_total;
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            RETURN 0;
    END f_get_semester_credits;

    -- ================================================
    -- p_enroll: Full enrollment with all validations
    -- ================================================
    PROCEDURE p_enroll(
        p_snum     NUMBER,
        p_classnum NUMBER
    ) AS
        v_student_standing Students.standing%TYPE;
        v_course_standing  Courses.standing%TYPE;
        v_cnum             SchClasses.cnum%TYPE;
        v_new_credits      Courses.crhr%TYPE;
        v_total_credits    NUMBER;
    BEGIN
        -- Validate student
        IF NOT f_validate_student(p_snum) THEN
            RAISE_APPLICATION_ERROR(-20001, 'Student does not exist.');
        END IF;

        -- Validate class
        IF NOT f_validate_class(p_classnum) THEN
            RAISE_APPLICATION_ERROR(-20002, 'Class does not exist.');
        END IF;

        -- Get standings
        SELECT standing INTO v_student_standing
        FROM Students WHERE snum = p_snum;

        SELECT sc.cnum, co.standing, co.crhr
        INTO v_cnum, v_course_standing, v_new_credits
        FROM SchClasses sc
        INNER JOIN Courses co ON sc.cnum = co.cnum
        WHERE sc.classnum = p_classnum;

        -- Check standing
        IF v_student_standing < v_course_standing THEN
            RAISE_APPLICATION_ERROR(-20003, 'Student standing is too low.');
        END IF;

        -- Check credit limit
        v_total_credits := f_get_semester_credits(p_snum, p_classnum);
        IF (v_total_credits + v_new_credits) > 15 THEN
            RAISE_APPLICATION_ERROR(-20004, 'Enrollment would exceed 15 credit hour limit.');
        END IF;

        -- All checks passed
        INSERT INTO Enrollments (classnum, snum, grade)
        VALUES (p_classnum, p_snum, NULL);
        COMMIT;
        DBMS_OUTPUT.PUT_LINE('Enrolled student ' || p_snum || ' in class ' || p_classnum || '.');

    EXCEPTION
        WHEN OTHERS THEN
            ROLLBACK;
            RAISE;  -- re-raise the error to the caller
    END p_enroll;

    -- ================================================
    -- p_update_all_gpa: Cursor through all students
    -- ================================================
    PROCEDURE p_update_all_gpa AS
        v_snum       Students.snum%TYPE;
        v_total_pts  NUMBER;
        v_total_hrs  NUMBER;
        v_gpa        NUMBER;

        CURSOR c_students IS
            SELECT DISTINCT snum FROM Enrollments;

        CURSOR c_grades(p_student NUMBER) IS
            SELECT e.grade, co.crhr
            FROM Enrollments e
            INNER JOIN SchClasses sc ON e.classnum = sc.classnum
            INNER JOIN Courses co ON sc.cnum = co.cnum
            WHERE e.snum = p_student
            AND e.grade IS NOT NULL;
    BEGIN
        OPEN c_students;
        LOOP
            FETCH c_students INTO v_snum;
            EXIT WHEN c_students%NOTFOUND;

            v_total_pts := 0;
            v_total_hrs := 0;

            BEGIN
                FOR rec IN c_grades(v_snum) LOOP
                    v_total_pts := v_total_pts + (Grading(rec.grade) * rec.crhr);
                    v_total_hrs := v_total_hrs + rec.crhr;
                END LOOP;

                IF v_total_hrs > 0 THEN
                    v_gpa := ROUND(v_total_pts / v_total_hrs, 2);
                    UPDATE Students SET gpa = v_gpa WHERE snum = v_snum;
                    DBMS_OUTPUT.PUT_LINE('Student ' || v_snum || ' GPA → ' || v_gpa);
                END IF;
            EXCEPTION
                WHEN OTHERS THEN
                    DBMS_OUTPUT.PUT_LINE('Error updating GPA for student ' || v_snum ||
                                        ': ' || SQLERRM);
            END;
        END LOOP;
        CLOSE c_students;
    END p_update_all_gpa;

    -- ================================================
    -- p_update_all_standing: Update standing by credits
    -- ================================================
    PROCEDURE p_update_all_standing AS
        v_snum         Students.snum%TYPE;
        v_credit_hours NUMBER;

        CURSOR c_credits IS
            SELECT e.snum, SUM(c.crhr)
            FROM Enrollments e
            INNER JOIN SchClasses sc ON e.classnum = sc.classnum
            INNER JOIN Courses c ON sc.cnum = c.cnum
            WHERE e.grade IN ('A', 'B', 'C')
            GROUP BY e.snum;
    BEGIN
        OPEN c_credits;
        LOOP
            FETCH c_credits INTO v_snum, v_credit_hours;
            EXIT WHEN c_credits%NOTFOUND;

            BEGIN
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
                DBMS_OUTPUT.PUT_LINE('Student ' || v_snum || ' standing updated.');
            EXCEPTION
                WHEN OTHERS THEN
                    DBMS_OUTPUT.PUT_LINE('Error updating standing for student ' || v_snum ||
                                        ': ' || SQLERRM);
            END;
        END LOOP;
        CLOSE c_credits;
    END p_update_all_standing;

END student_enrollment_pkg;
/
```

!!! note "Key Package Patterns"
    - The body **implements** everything declared in the specification
    - `p_enroll` uses the **validation functions** (`f_validate_student`, `f_validate_class`) before proceeding
    - `p_update_all_gpa` and `p_update_all_standing` handle errors **per student** (inner `BEGIN...EXCEPTION...END`) so one bad record doesn't stop the entire batch
    - `WHEN OTHERS` in `p_enroll` does a `ROLLBACK` then `RAISE` — re-raises the error to the caller

---

### Q10: Using the Package

```sql
SET SERVEROUTPUT ON;

DECLARE
    v_result BOOLEAN;
BEGIN
    -- Test 1: Validate existing student
    v_result := student_enrollment_pkg.f_validate_student(101);
    IF v_result THEN
        DBMS_OUTPUT.PUT_LINE('Test 1: Student 101 exists → TRUE');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Test 1: Student 101 exists → FALSE');
    END IF;

    -- Test 2: Validate non-existing student
    v_result := student_enrollment_pkg.f_validate_student(999);
    IF v_result THEN
        DBMS_OUTPUT.PUT_LINE('Test 2: Student 999 exists → TRUE');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Test 2: Student 999 exists → FALSE');
    END IF;

    -- Test 3: Attempt enrollment
    DBMS_OUTPUT.PUT_LINE('Test 3: Enrolling student 101 into class 10112...');
    student_enrollment_pkg.p_enroll(101, 10112);

    -- Test 4: Batch update GPA
    DBMS_OUTPUT.PUT_LINE('Test 4: Updating all GPAs...');
    student_enrollment_pkg.p_update_all_gpa;

    -- Test 5: Batch update standing
    DBMS_OUTPUT.PUT_LINE('Test 5: Updating all standings...');
    student_enrollment_pkg.p_update_all_standing;
END;
/
```

!!! tip "Dot Notation"
    Package members are called using **dot notation**: `package_name.procedure_name()`. This keeps related procedures organized and avoids name conflicts with standalone procedures.

**Expected output:**
```
Test 1: Student 101 exists → TRUE
Test 2: Student 999 exists → FALSE
Test 3: Enrolling student 101 into class 10112...
Enrolled student 101 in class 10112.
Test 4: Updating all GPAs...
Student 101 GPA → ...
Student 102 GPA → ...
Test 5: Updating all standings...
Student 101 standing updated.
...
```

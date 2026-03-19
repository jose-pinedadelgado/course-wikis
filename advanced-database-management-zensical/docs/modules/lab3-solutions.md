# Lab 3 Solutions — SQL in PL/SQL & Cursors

!!! info "Prerequisites"
    This lab uses the Student Enrollment schema (Students, Courses, SchClasses, Enrollments). Make sure the tables are created and populated before running these solutions.

## Part 3 — SQL in PL/SQL

### Q1: AddMe — Basic Enrollment

Write a procedure `AddMe(p_snum, p_classnum)` that inserts a row into Enrollments.

```sql
CREATE OR REPLACE PROCEDURE AddMe(
    p_snum     IN Students.snum%TYPE,
    p_classnum IN SchClasses.classnum%TYPE
) AS
BEGIN
    INSERT INTO Enrollments (classnum, snum, grade)
    VALUES (p_classnum, p_snum, NULL);

    DBMS_OUTPUT.PUT_LINE('Successfully Enrolled ' || p_snum || ' into ' || p_classnum || '.');
END;
/
```

!!! note "Process Flow"
    1. Accept student number and class number as parameters
    2. Insert into Enrollments with grade as NULL (not yet graded)
    3. Print a confirmation message

**Test:**
```sql
EXEC AddMe(101, 10112);
SELECT * FROM Enrollments WHERE snum = 101;
ROLLBACK;  -- undo the test insert
```

---

### Q2: AddMe with Standing Check

Refine AddMe so the student may enroll only if `Students.standing >= Courses.standing`.

```sql
CREATE OR REPLACE PROCEDURE AddMe2(
    p_snum     IN Students.snum%TYPE,
    p_classnum IN SchClasses.classnum%TYPE
) AS
    v_student_standing Students.standing%TYPE;
    v_course_standing  Courses.standing%TYPE;
BEGIN
    -- Get the student's standing
    SELECT standing INTO v_student_standing
    FROM Students
    WHERE snum = p_snum;

    -- Get the course's required standing (join SchClasses → Courses)
    SELECT Courses.standing INTO v_course_standing
    FROM SchClasses, Courses
    WHERE SchClasses.classnum = p_classnum
    AND SchClasses.cnum = Courses.cnum;

    -- Check if student meets the requirement
    IF v_student_standing >= v_course_standing THEN
        INSERT INTO Enrollments (classnum, snum, grade)
        VALUES (p_classnum, p_snum, NULL);
        DBMS_OUTPUT.PUT_LINE('Successfully Enrolled ' || p_snum || ' into ' || p_classnum || '.');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Failed to Enroll ' || p_snum || ' into ' || p_classnum ||
                             '. Student Standing is too low.');
    END IF;
END;
/
```

!!! note "Process Flow"
    1. Use `SELECT INTO` to get the student's standing from the Students table
    2. Use `SELECT INTO` with a join (SchClasses → Courses) to get the course's required standing
    3. Compare: if student standing is high enough, insert; otherwise print an error
    4. Standing codes: 1=Freshman, 2=Sophomore, 3=Junior, 4=Senior

**Test:**
```sql
-- Student 101 (Andy, standing=4 Senior) enrolling in class 10112 (IS 380, standing=3 Junior) → should succeed
EXEC AddMe2(101, 10112);

-- Student 102 (Betty, standing=2 Sophomore) enrolling in class 10112 (IS 380, standing=3 Junior) → should fail
EXEC AddMe2(102, 10112);
ROLLBACK;
```

---

### Q3: AddMe with Credit Limit Check

Refine AddMe so enrollment only succeeds if total credits for that semester/year remain ≤ 15.

```sql
CREATE OR REPLACE PROCEDURE AddMe3(
    p_snum     IN Students.snum%TYPE,
    p_classnum IN SchClasses.classnum%TYPE
) AS
    v_student_standing Students.standing%TYPE;
    v_course_standing  Courses.standing%TYPE;
    v_semester         SchClasses.semester%TYPE;
    v_year             SchClasses.year%TYPE;
    v_new_credits      Courses.crhr%TYPE;
    v_total_credits    NUMBER;
BEGIN
    -- Get student standing
    SELECT standing INTO v_student_standing
    FROM Students
    WHERE snum = p_snum;

    -- Get course standing requirement, credits, semester, and year for the target class
    SELECT Courses.standing, Courses.crhr, SchClasses.semester, SchClasses.year
    INTO v_course_standing, v_new_credits, v_semester, v_year
    FROM SchClasses, Courses
    WHERE SchClasses.classnum = p_classnum
    AND SchClasses.cnum = Courses.cnum;

    -- Check standing requirement
    IF v_student_standing >= v_course_standing THEN

        -- Calculate total credits already enrolled for that semester/year
        SELECT NVL(SUM(Courses.crhr), 0) INTO v_total_credits
        FROM Enrollments
        INNER JOIN SchClasses ON Enrollments.classnum = SchClasses.classnum
        INNER JOIN Courses ON SchClasses.cnum = Courses.cnum
        WHERE Enrollments.snum = p_snum
        AND SchClasses.semester = v_semester
        AND SchClasses.year = v_year;

        -- Check if adding the new class would exceed 15 credits
        IF (v_total_credits + v_new_credits) <= 15 THEN
            INSERT INTO Enrollments (classnum, snum, grade)
            VALUES (p_classnum, p_snum, NULL);
            DBMS_OUTPUT.PUT_LINE('Successfully Enrolled ' || p_snum || ' into ' || p_classnum || '.');
        ELSE
            DBMS_OUTPUT.PUT_LINE('Failed to Enroll ' || p_snum || ' into ' || p_classnum ||
                                 '. Adding this class would exceed the 15 credit limit for the semester.');
        END IF;

    ELSE
        DBMS_OUTPUT.PUT_LINE('Failed to Enroll ' || p_snum || ' into ' || p_classnum ||
                             '. Student doesnt meet minimum standing to take course.');
    END IF;
END;
/
```

!!! note "Process Flow"
    1. Get student standing
    2. Get course info: required standing, credits, semester, year
    3. First check: does the student meet the standing requirement?
    4. If yes, calculate total credits already enrolled for that same semester/year
    5. Second check: would adding this class push total credits over 15?
    6. Only enroll if both checks pass

!!! tip "NVL for NULL handling"
    We use `NVL(SUM(...), 0)` because if the student has no enrollments for that semester, `SUM` returns `NULL`. `NVL` converts `NULL` to `0` so our arithmetic works correctly.

---

### Q4: Update_GPA — Weighted GPA Calculation

Write `Update_GPA(p_snum)` to recompute GPA. Formula: GPA = Σ(grade_value × credit_hours) / Σ(credit_hours).

```sql
CREATE OR REPLACE FUNCTION Grading(
    p_LetterGrade VARCHAR2
) RETURN NUMBER AS
BEGIN
    IF p_LetterGrade = 'A' THEN RETURN 4;
    ELSIF p_LetterGrade = 'B' THEN RETURN 3;
    ELSIF p_LetterGrade = 'C' THEN RETURN 2;
    ELSIF p_LetterGrade = 'D' THEN RETURN 1;
    ELSE RETURN 0;
    END IF;
END;
/
```

```sql
CREATE OR REPLACE PROCEDURE Update_GPA(
    p_snum Students.snum%TYPE
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
    OPEN c_grades;
    LOOP
        FETCH c_grades INTO v_grade, v_crhr;
        EXIT WHEN c_grades%NOTFOUND;

        v_total_pts := v_total_pts + (Grading(v_grade) * v_crhr);
        v_total_hrs := v_total_hrs + v_crhr;
    END LOOP;
    CLOSE c_grades;

    IF v_total_hrs > 0 THEN
        v_gpa := ROUND(v_total_pts / v_total_hrs, 2);
        UPDATE Students SET gpa = v_gpa WHERE snum = p_snum;
        DBMS_OUTPUT.PUT_LINE('GPA for student ' || p_snum || ' updated to ' || v_gpa);
    ELSE
        DBMS_OUTPUT.PUT_LINE('No graded courses found for student ' || p_snum);
    END IF;
END;
/
```

!!! note "Process Flow"
    1. The `Grading` function converts letter grades to numbers (A=4, B=3, C=2, D=1, else=0)
    2. A cursor retrieves all graded enrollments for the student (joins Enrollments → SchClasses → Courses)
    3. For each row: multiply grade value × credit hours and accumulate totals
    4. Calculate GPA = total points / total hours
    5. Update the Students table with the new GPA

**Example:** Student gets A (4) in a 3-credit course and D (1) in a 2-credit course:
GPA = (4×3 + 1×2) / (3+2) = 14/5 = **2.8**

---

### Q5: Validate_Student

```sql
CREATE OR REPLACE PROCEDURE Validate_Student(
    p_snum Students.snum%TYPE
) AS
    v_count NUMBER;
BEGIN
    SELECT COUNT(*) INTO v_count
    FROM Students
    WHERE snum = p_snum;

    IF v_count > 0 THEN
        DBMS_OUTPUT.PUT_LINE('The student number is valid.');
    ELSE
        DBMS_OUTPUT.PUT_LINE('The student number is invalid.');
    END IF;
END;
/
```

**Test:**
```sql
EXEC Validate_Student(101);  -- valid
EXEC Validate_Student(999);  -- invalid
```

---

### Q6: Validate_Credit_Limit

```sql
CREATE OR REPLACE PROCEDURE Validate_Credit_Limit(
    p_snum     Students.snum%TYPE,
    p_classnum SchClasses.classnum%TYPE
) AS
    v_semester      SchClasses.semester%TYPE;
    v_year          SchClasses.year%TYPE;
    v_new_credits   Courses.crhr%TYPE;
    v_total_credits NUMBER;
BEGIN
    -- Get semester, year, and credits for the target class
    SELECT sc.semester, sc.year, co.crhr
    INTO v_semester, v_year, v_new_credits
    FROM SchClasses sc
    INNER JOIN Courses co ON sc.cnum = co.cnum
    WHERE sc.classnum = p_classnum;

    -- Get current total credits for that semester/year
    SELECT NVL(SUM(co.crhr), 0) INTO v_total_credits
    FROM Enrollments e
    INNER JOIN SchClasses sc ON e.classnum = sc.classnum
    INNER JOIN Courses co ON sc.cnum = co.cnum
    WHERE e.snum = p_snum
    AND sc.semester = v_semester
    AND sc.year = v_year;

    -- Check the limit
    IF (v_total_credits + v_new_credits) > 15 THEN
        RAISE_APPLICATION_ERROR(-20010,
            'Student exceeds 15-credit-hour limit after enrollment.');
    END IF;
END;
/
```

!!! note "RAISE_APPLICATION_ERROR"
    Unlike `DBMS_OUTPUT.PUT_LINE`, `RAISE_APPLICATION_ERROR` sends an error back to the calling environment (SQL Developer). The error code must be between -20000 and -20999.

---

### Q7: AddMe Calling Validate_Credit_Limit

```sql
CREATE OR REPLACE PROCEDURE AddMe_Final(
    p_snum     IN Students.snum%TYPE,
    p_classnum IN SchClasses.classnum%TYPE
) AS
    v_student_standing Students.standing%TYPE;
    v_course_standing  Courses.standing%TYPE;
BEGIN
    -- Get student standing
    SELECT standing INTO v_student_standing
    FROM Students WHERE snum = p_snum;

    -- Get course standing requirement
    SELECT Courses.standing INTO v_course_standing
    FROM SchClasses, Courses
    WHERE SchClasses.classnum = p_classnum
    AND SchClasses.cnum = Courses.cnum;

    -- Check standing
    IF v_student_standing >= v_course_standing THEN
        -- Check credit limit (will raise an error if exceeded)
        Validate_Credit_Limit(p_snum, p_classnum);

        -- If we get here, both checks passed
        INSERT INTO Enrollments (classnum, snum, grade)
        VALUES (p_classnum, p_snum, NULL);
        DBMS_OUTPUT.PUT_LINE('Successfully Enrolled ' || p_snum || ' into ' || p_classnum || '.');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Failed to Enroll. Student standing is too low.');
    END IF;
END;
/
```

!!! tip "Composition"
    Notice how `AddMe_Final` **calls** `Validate_Credit_Limit` as a helper. If the credit limit is exceeded, `RAISE_APPLICATION_ERROR` in the helper procedure stops execution — the INSERT never runs. This is the same pattern used in the package version.

---

## Cursor Exercises

### Cursor Q1: Update Every Student's Standing

Update standing based on total completed credit hours.

```sql
CREATE OR REPLACE PROCEDURE p_update_standing AS
    v_snum         Students.snum%TYPE;
    v_credit_hours NUMBER;

    CURSOR c_credit_hours IS
        SELECT e.snum, SUM(c.crhr)
        FROM Enrollments e
        INNER JOIN SchClasses sc ON e.classnum = sc.classnum
        INNER JOIN Courses c ON sc.cnum = c.cnum
        WHERE e.grade IN ('A', 'B', 'C')  -- only passing grades count
        GROUP BY e.snum;
BEGIN
    OPEN c_credit_hours;
    LOOP
        FETCH c_credit_hours INTO v_snum, v_credit_hours;
        EXIT WHEN c_credit_hours%NOTFOUND;

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
    END LOOP;
    CLOSE c_credit_hours;
END;
/
```

!!! note "Process Flow"
    1. **Cursor** selects total credit hours per student (only passing grades: A, B, C)
    2. **OPEN** the cursor, then **FETCH** one row at a time
    3. Use **IF/ELSIF** to determine new standing based on credit hours
    4. **UPDATE** the student's standing
    5. **EXIT WHEN %NOTFOUND** — stop when there are no more rows
    6. **CLOSE** the cursor

---

### Cursor Q2: Calculate and Update Every Student's GPA

```sql
CREATE OR REPLACE PROCEDURE p_update_all_gpa AS
    v_snum       Students.snum%TYPE;
    v_grade      Enrollments.grade%TYPE;
    v_crhr       Courses.crhr%TYPE;
    v_total_pts  NUMBER;
    v_total_hrs  NUMBER;
    v_gpa        NUMBER;

    -- Outer cursor: loop through all students
    CURSOR c_students IS
        SELECT DISTINCT snum FROM Enrollments;

    -- Inner cursor: get grades for a specific student
    CURSOR c_grades(p_student_snum NUMBER) IS
        SELECT e.grade, co.crhr
        FROM Enrollments e
        INNER JOIN SchClasses sc ON e.classnum = sc.classnum
        INNER JOIN Courses co ON sc.cnum = co.cnum
        WHERE e.snum = p_student_snum
        AND e.grade IS NOT NULL;
BEGIN
    OPEN c_students;
    LOOP
        FETCH c_students INTO v_snum;
        EXIT WHEN c_students%NOTFOUND;

        -- Reset totals for each student
        v_total_pts := 0;
        v_total_hrs := 0;

        OPEN c_grades(v_snum);
        LOOP
            FETCH c_grades INTO v_grade, v_crhr;
            EXIT WHEN c_grades%NOTFOUND;

            v_total_pts := v_total_pts + (Grading(v_grade) * v_crhr);
            v_total_hrs := v_total_hrs + v_crhr;
        END LOOP;
        CLOSE c_grades;

        IF v_total_hrs > 0 THEN
            v_gpa := ROUND(v_total_pts / v_total_hrs, 2);
            UPDATE Students SET gpa = v_gpa WHERE snum = v_snum;
            DBMS_OUTPUT.PUT_LINE('Student ' || v_snum || ' GPA updated to ' || v_gpa);
        END IF;
    END LOOP;
    CLOSE c_students;
END;
/
```

!!! tip "Parameterized Cursor"
    The inner cursor `c_grades(p_student_snum NUMBER)` accepts a parameter — it's reused for each student from the outer loop. This is more efficient than declaring a new cursor each time.

---

### Cursor Q3: Update CurrEnroll for Each Class

Add a `CurrEnroll` column and populate it with the current enrollment count.

```sql
-- Step 1: Add the column (DDL — run this outside PL/SQL)
ALTER TABLE SchClasses ADD (CurrEnroll NUMBER);

-- Step 2: Procedure to update enrollment counts
CREATE OR REPLACE PROCEDURE p_update_curr_enroll AS
    v_classnum  SchClasses.classnum%TYPE;
    v_count     NUMBER;

    CURSOR c_classes IS
        SELECT classnum FROM SchClasses;
BEGIN
    OPEN c_classes;
    LOOP
        FETCH c_classes INTO v_classnum;
        EXIT WHEN c_classes%NOTFOUND;

        SELECT COUNT(*) INTO v_count
        FROM Enrollments
        WHERE classnum = v_classnum;

        UPDATE SchClasses
        SET CurrEnroll = v_count
        WHERE classnum = v_classnum;
    END LOOP;
    CLOSE c_classes;

    DBMS_OUTPUT.PUT_LINE('Enrollment counts updated for all classes.');
END;
/
```

!!! note "Process Flow"
    1. Cursor iterates through every class in SchClasses
    2. For each class, count enrollments using `SELECT COUNT(*) INTO`
    3. Update the `CurrEnroll` column with the count
    4. Note: the `ALTER TABLE` must run first as a separate SQL statement (DDL cannot run inside PL/SQL directly)

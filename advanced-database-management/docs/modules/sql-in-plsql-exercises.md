# SQL in PL/SQL — Exercises

These exercises build on each other, progressively refining an enrollment procedure.

---

## Q1: AddMe (Basic)

Write a procedure `AddMe(p_snum, p_classnum)` that inserts a row into Enrollments to enroll a student into a scheduled class.

??? example "Solution"
    ```sql
    CREATE OR REPLACE PROCEDURE AddMe (
        p_snum     IN Students.snum%TYPE,
        p_classnum IN SchClasses.classnum%TYPE
    ) AS
    BEGIN
        INSERT INTO Enrollments (snum, classnum) VALUES (p_snum, p_classnum);
        DBMS_OUTPUT.PUT_LINE('Successfully Enrolled ' || p_snum || ' into ' || p_classnum || '.');
    END;
    /
    ```

---

## Q2: AddMe with Standing Check

Refine AddMe so the student may enroll only if `Students.standing >= Courses.standing`.

Standing codes: 1=Freshman, 2=Sophomore, 3=Junior, 4=Senior.

??? example "Solution"
    ```sql
    CREATE OR REPLACE PROCEDURE AddMe2 (
        p_snum     IN Students.snum%TYPE,
        p_classnum IN SchClasses.classnum%TYPE
    ) AS
        v_student_standing Students.standing%TYPE;
        v_course_standing  Courses.standing%TYPE;
    BEGIN
        SELECT s.standing INTO v_student_standing
        FROM Students s WHERE s.snum = p_snum;
        
        SELECT c.standing INTO v_course_standing
        FROM SchClasses sc
        JOIN Courses c ON sc.cnum = c.cnum
        WHERE sc.classnum = p_classnum;
        
        IF v_student_standing >= v_course_standing THEN
            INSERT INTO Enrollments (snum, classnum) VALUES (p_snum, p_classnum);
            DBMS_OUTPUT.PUT_LINE('Successfully Enrolled ' || p_snum || ' into ' || p_classnum || '.');
        ELSE
            DBMS_OUTPUT.PUT_LINE('Failed: Student Standing is too low.');
        END IF;
    END;
    /
    ```

---

## Q3: AddMe with Credit Limit

Refine AddMe so the student may enroll only if their total credits for that semester/year will be ≤ 15 after adding the class.

??? example "Solution"
    See the full `AddMe3` procedure in the [SQL in PL/SQL](sql-in-plsql.md#example-addme-with-credit-limit-check) page.

---

## Q4: Update_GPA

Write `Update_GPA(p_snum)` to recompute GPA using completed enrollments, weighted by credit hours.

Grade values: A=4, B=3, C=2, D=1

??? example "Solution"
    See the full `Update_GPA` procedure in the [SQL in PL/SQL](sql-in-plsql.md#gpa-calculation) page.

---

## Q5: Validate_Student

Write `Validate_Student(p_snum)` that prints:
- "The student number is valid." if a row exists in Students
- "The student number is invalid." otherwise

??? example "Solution"
    ```sql
    CREATE OR REPLACE PROCEDURE Validate_Student (p_snum IN NUMBER) AS
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

---

## Q6: Validate_Credit_Limit

Write `Validate_Credit_Limit(p_snum, p_classnum)` that raises an error message if adding that class would push the student over 15 credits for that term.

??? example "Solution"
    ```sql
    CREATE OR REPLACE PROCEDURE Validate_Credit_Limit (
        p_snum     IN NUMBER,
        p_classnum IN NUMBER
    ) AS
        v_total    NUMBER := 0;
        v_new_crhr Courses.crhr%TYPE;
        v_semester SchClasses.semester%TYPE;
        v_year     SchClasses.class_year%TYPE;
    BEGIN
        -- Get the term and credits for the new class
        SELECT sc.semester, sc.class_year, c.crhr
        INTO v_semester, v_year, v_new_crhr
        FROM SchClasses sc
        JOIN Courses c ON sc.cnum = c.cnum
        WHERE sc.classnum = p_classnum;
        
        -- Get current credits for that term
        SELECT NVL(SUM(c.crhr), 0) INTO v_total
        FROM Enrollments e
        JOIN SchClasses sc ON e.classnum = sc.classnum
        JOIN Courses c ON sc.cnum = c.cnum
        WHERE e.snum = p_snum
        AND sc.semester = v_semester
        AND sc.class_year = v_year;
        
        IF (v_total + v_new_crhr) > 15 THEN
            RAISE_APPLICATION_ERROR(-20001, 
                'Student exceeds 15-credit-hour limit after enrollment.');
        END IF;
    END;
    /
    ```

---

## Q7: AddMe with Validate_Credit_Limit

Call `Validate_Credit_Limit` inside AddMe, so enrollment happens only when the credit limit remains ≤ 15.

??? example "Solution"
    ```sql
    CREATE OR REPLACE PROCEDURE AddMe_Final (
        p_snum     IN NUMBER,
        p_classnum IN NUMBER
    ) AS
        v_student_standing Students.standing%TYPE;
        v_course_standing  Courses.standing%TYPE;
    BEGIN
        -- Validate student exists
        Validate_Student(p_snum);
        
        -- Check standing
        SELECT s.standing INTO v_student_standing
        FROM Students s WHERE s.snum = p_snum;
        
        SELECT c.standing INTO v_course_standing
        FROM SchClasses sc
        JOIN Courses c ON sc.cnum = c.cnum
        WHERE sc.classnum = p_classnum;
        
        IF v_student_standing < v_course_standing THEN
            DBMS_OUTPUT.PUT_LINE('Failed: Standing too low.');
            RETURN;
        END IF;
        
        -- Check credit limit (raises error if exceeded)
        Validate_Credit_Limit(p_snum, p_classnum);
        
        -- All checks passed — enroll
        INSERT INTO Enrollments (snum, classnum) VALUES (p_snum, p_classnum);
        DBMS_OUTPUT.PUT_LINE('Successfully Enrolled ' || p_snum || ' into ' || p_classnum || '.');
    
    EXCEPTION
        WHEN OTHERS THEN
            DBMS_OUTPUT.PUT_LINE('Enrollment failed: ' || SQLERRM);
    END;
    /
    ```

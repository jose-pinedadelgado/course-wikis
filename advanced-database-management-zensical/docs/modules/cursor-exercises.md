# Cursor Exercises

## Exercise 1: Update Standing

Write a procedure `p_update_standing` that:

1. Uses a cursor to get total credit hours per student (via Enrollments → SchClasses → Courses)
2. Updates each student's standing based on:
    - < 30 credits → Standing 1 (Freshman)
    - < 60 credits → Standing 2 (Sophomore)
    - < 90 credits → Standing 3 (Junior)
    - < 120 credits → Standing 4 (Senior)
    - 120+ credits → Standing 5 (Graduate)

??? example "Solution (Explicit Cursor)"
    See the full solution in [PL/SQL Cursors](cursors.md#example-update-standing-based-on-credits).

??? example "Solution (Cursor FOR Loop)"
    See the simplified version in [PL/SQL Cursors](cursors.md#cursor-for-loop-simplified).

---

## Exercise 2: Enrollment Report

Write a procedure that prints a formatted report of all students and their enrolled courses, using a cursor.

??? example "Solution"
    ```sql
    SET SERVEROUTPUT ON;
    
    CREATE OR REPLACE PROCEDURE enrollment_report AS
        CURSOR c_students IS
            SELECT snum, sname FROM Students ORDER BY sname;
        
        CURSOR c_courses (p_snum NUMBER) IS
            SELECT c.ctitle, e.grade
            FROM Enrollments e
            JOIN SchClasses sc ON e.classnum = sc.classnum
            JOIN Courses c ON sc.cnum = c.cnum
            WHERE e.snum = p_snum;
        
        v_found BOOLEAN;
    BEGIN
        FOR student IN c_students LOOP
            DBMS_OUTPUT.PUT_LINE('=== ' || TRIM(student.sname) || ' (ID: ' || student.snum || ') ===');
            
            v_found := FALSE;
            FOR course IN c_courses(student.snum) LOOP
                v_found := TRUE;
                DBMS_OUTPUT.PUT_LINE('  ' || course.ctitle || ' - Grade: ' || NVL(course.grade, 'In Progress'));
            END LOOP;
            
            IF NOT v_found THEN
                DBMS_OUTPUT.PUT_LINE('  (No enrollments)');
            END IF;
            
            DBMS_OUTPUT.PUT_LINE('');
        END LOOP;
    END;
    /
    
    EXEC enrollment_report;
    ```

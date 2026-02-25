# Activity: Conditional PL/SQL & Intro to SQL in PL/SQL

## Exercise 1: Grade Classifier

Write a PL/SQL block that:

1. Accepts a student number
2. Retrieves their GPA
3. Prints their classification:
    - GPA >= 3.7 → "Dean's List"
    - GPA >= 3.0 → "Good Standing"
    - GPA >= 2.0 → "Satisfactory"
    - GPA < 2.0 → "Academic Probation"

??? example "Solution"
    ```sql
    SET SERVEROUTPUT ON;
    
    DECLARE
        v_snum Students.snum%TYPE := 101;
        v_sname Students.sname%TYPE;
        v_gpa   Students.gpa%TYPE;
    BEGIN
        SELECT sname, gpa INTO v_sname, v_gpa
        FROM Students
        WHERE snum = v_snum;
        
        DBMS_OUTPUT.PUT_LINE('Student: ' || v_sname || ', GPA: ' || v_gpa);
        
        IF v_gpa >= 3.7 THEN
            DBMS_OUTPUT.PUT_LINE('Classification: Dean''s List');
        ELSIF v_gpa >= 3.0 THEN
            DBMS_OUTPUT.PUT_LINE('Classification: Good Standing');
        ELSIF v_gpa >= 2.0 THEN
            DBMS_OUTPUT.PUT_LINE('Classification: Satisfactory');
        ELSE
            DBMS_OUTPUT.PUT_LINE('Classification: Academic Probation');
        END IF;
    END;
    /
    ```

## Exercise 2: Standing Decoder

Write a function that takes a standing number (1-4) and returns the text equivalent.

??? example "Solution"
    ```sql
    CREATE OR REPLACE FUNCTION decode_standing (
        p_standing IN NUMBER
    ) RETURN VARCHAR2 AS
    BEGIN
        RETURN CASE p_standing
            WHEN 1 THEN 'Freshman'
            WHEN 2 THEN 'Sophomore'
            WHEN 3 THEN 'Junior'
            WHEN 4 THEN 'Senior'
            ELSE 'Unknown'
        END;
    END;
    /
    
    -- Test it:
    SELECT sname, standing, decode_standing(standing) AS standing_text
    FROM Students;
    ```

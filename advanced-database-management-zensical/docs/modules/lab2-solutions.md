# Lab 2 Solutions — FOR Loops & SQL in PL/SQL

!!! info "Prerequisites"
    This lab builds on Lab 1 with more loop practice and introduces SQL operations inside PL/SQL blocks.

## Exercise 1: PrintTable — Multiplication Table

Write a procedure `PrintTable(p_BaseNumber)` that prints a multiplication table for the given number.

**Expected output:**
```
SQL> EXEC PrintTable(2);
2x1=2
2x2=4
2x3=6
2x4=8
2x5=10
2x6=12
2x7=14
2x8=16
2x9=18
2x10=20
```

### Solution

```sql
CREATE OR REPLACE PROCEDURE PrintTable(
    p_BaseNumber NUMBER
) AS
BEGIN
    FOR i IN 1..10 LOOP
        DBMS_OUTPUT.PUT_LINE(p_BaseNumber || 'x' || i || '=' || (p_BaseNumber * i));
    END LOOP;
END;
/
```

!!! note "Process Flow"
    1. Use a FOR LOOP with index `i` from 1 to 10
    2. Each iteration prints the base number × i using string concatenation (`||`)
    3. The multiplication `(p_BaseNumber * i)` is calculated inline

---

## Exercise 2: PrintWholeTable — Full Multiplication Table

Write a procedure `PrintWholeTable` that prints the entire 10×10 multiplication table.

**Expected output:**
```
1x1=1
1x2=2
...
1x10=10
2x1=2
2x2=4
...
10x10=100
```

### Solution

```sql
CREATE OR REPLACE PROCEDURE PrintWholeTable AS
BEGIN
    FOR base IN 1..10 LOOP
        FOR i IN 1..10 LOOP
            DBMS_OUTPUT.PUT_LINE(base || 'x' || i || '=' || (base * i));
        END LOOP;
        DBMS_OUTPUT.PUT_LINE('---');  -- separator between tables
    END LOOP;
END;
/
```

!!! tip "Nested Loops"
    This uses **nested FOR LOOPs** — the outer loop controls the base number (1 through 10), and the inner loop multiplies it by 1 through 10. This is the same as calling `PrintTable` for each number from 1 to 10.

**Alternative using PrintTable:**

```sql
CREATE OR REPLACE PROCEDURE PrintWholeTable AS
BEGIN
    FOR base IN 1..10 LOOP
        PrintTable(base);
        DBMS_OUTPUT.PUT_LINE('---');
    END LOOP;
END;
/
```

!!! note "Reusing Procedures"
    You can call one procedure from inside another! This version calls our `PrintTable` procedure 10 times, once for each base number.

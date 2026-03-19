# Lab 1 Solutions — Conditional PL/SQL & Intro to SQL

!!! info "Prerequisites"
    This lab covers IF/THEN/ELSIF, all three loop types (LOOP+EXIT, WHILE, FOR), and SQL in PL/SQL (INSERT, UPDATE, DELETE, SELECT..INTO).

## Exercise 1: Trace the Programs

### Program 1

```sql
DECLARE
  j NUMBER := 0;
BEGIN
  FOR i IN 1..10 LOOP
    IF i >= 3 THEN
      EXIT;
      DBMS_OUTPUT.PUT_LINE(j);
    END IF;
    j := i;
  END LOOP;
END;
/
```

**What does it print?** **Nothing.**

!!! note "Walkthrough"
    - `i=1`: `1 >= 3` is FALSE → skip IF block → `j := 1`
    - `i=2`: `2 >= 3` is FALSE → skip IF block → `j := 2`
    - `i=3`: `3 >= 3` is TRUE → enters IF → hits `EXIT` immediately → **exits the loop**
    - The `DBMS_OUTPUT.PUT_LINE(j)` is **after** the EXIT, so it never executes
    - The program prints nothing because the EXIT comes before the print statement

---

### Program 2

```sql
DECLARE
  j NUMBER := 0;
BEGIN
  FOR i IN 1..10 LOOP
    j := 0;
    IF i >= 3 THEN
      DBMS_OUTPUT.PUT_LINE(j);
    END IF;
    j := i + j;
  END LOOP;
END;
/
```

**Output:**
```
0
0
0
0
0
0
0
0
```

!!! note "Walkthrough"
    - Every iteration starts with `j := 0` (reset!)
    - The print only fires when `i >= 3`, so iterations 1 and 2 print nothing
    - When `i >= 3`: it prints `j` which was just set to `0`
    - After printing, `j := i + j` runs, but `j` gets reset to `0` at the top of the next iteration anyway
    - Result: prints `0` eight times (for `i = 3, 4, 5, 6, 7, 8, 9, 10`)

---

## Exercise 2: MyFill — The Spreadsheet Fill Function

Write a procedure `MyFill(p_start, p_step, p_times)` that imitates the fill function from Excel.

**Expected output:**
```
SQL> EXEC MyFill(1, 2, 5);
1
3
5
7
9

SQL> EXEC MyFill(1000, 10, 6);
1000
1010
1020
1030
1040
1050
```

### Version 1: LOOP with EXIT

```sql
CREATE OR REPLACE PROCEDURE MyFill(
    p_start NUMBER,
    p_step  NUMBER,
    p_times NUMBER
) AS
    v_current NUMBER := p_start;
    v_count   NUMBER := 1;
BEGIN
    LOOP
        DBMS_OUTPUT.PUT_LINE(v_current);
        v_count   := v_count + 1;
        v_current := v_current + p_step;
        EXIT WHEN v_count > p_times;
    END LOOP;
END;
/
```

!!! tip "Process Flow"
    1. Start with `v_current = p_start` and `v_count = 1`
    2. Print the current value
    3. Increment the counter and add the step
    4. Exit when we've printed `p_times` values

### Version 2: WHILE LOOP

```sql
CREATE OR REPLACE PROCEDURE MyFill_While(
    p_start NUMBER,
    p_step  NUMBER,
    p_times NUMBER
) AS
    v_current NUMBER := p_start;
    v_count   NUMBER := 1;
BEGIN
    WHILE v_count <= p_times LOOP
        DBMS_OUTPUT.PUT_LINE(v_current);
        v_count   := v_count + 1;
        v_current := v_current + p_step;
    END LOOP;
END;
/
```

### Version 3: FOR LOOP

```sql
CREATE OR REPLACE PROCEDURE MyFill_For(
    p_start NUMBER,
    p_step  NUMBER,
    p_times NUMBER
) AS
    v_current NUMBER := p_start;
BEGIN
    FOR i IN 1..p_times LOOP
        DBMS_OUTPUT.PUT_LINE(v_current);
        v_current := v_current + p_step;
    END LOOP;
END;
/
```

!!! tip "FOR LOOP Advantage"
    The FOR LOOP version is the cleanest — the index `i` handles the counting automatically, so we only need one variable (`v_current`) for the value.

---

## Exercise 3: GetChange — Cash Register

Write a procedure `GetChange(p_AmountDue, p_Pay)` that tells the cashier how many of each bill to give back.

**Expected output:**
```
SQL> EXEC GetChange(12, 200);
9 Twenty Dollar Bill
1 Five Dollar Bill
3 One Dollar Bill
```

### Solution

```sql
CREATE OR REPLACE PROCEDURE GetChange(
    p_AmountDue NUMBER,
    p_Pay       NUMBER
) AS
    v_change    NUMBER;
    v_twenties  NUMBER;
    v_tens      NUMBER;
    v_fives     NUMBER;
    v_ones      NUMBER;
BEGIN
    -- Check if customer paid enough
    IF p_Pay < p_AmountDue THEN
        DBMS_OUTPUT.PUT_LINE('You need to give me more money!');
        RETURN;
    END IF;

    -- Check for exact change
    IF p_Pay = p_AmountDue THEN
        DBMS_OUTPUT.PUT_LINE('You just gave me exact change! Thank you!');
        RETURN;
    END IF;

    -- Calculate change
    v_change := p_Pay - p_AmountDue;

    -- Break down into bills using integer division (FLOOR) and MOD
    v_twenties := FLOOR(v_change / 20);
    v_change   := MOD(v_change, 20);

    v_tens     := FLOOR(v_change / 10);
    v_change   := MOD(v_change, 10);

    v_fives    := FLOOR(v_change / 5);
    v_change   := MOD(v_change, 5);

    v_ones     := v_change;

    -- Print results (only print non-zero amounts)
    IF v_twenties > 0 THEN
        DBMS_OUTPUT.PUT_LINE(v_twenties || ' Twenty Dollar Bill');
    END IF;
    IF v_tens > 0 THEN
        DBMS_OUTPUT.PUT_LINE(v_tens || ' Ten Dollar Bill');
    END IF;
    IF v_fives > 0 THEN
        DBMS_OUTPUT.PUT_LINE(v_fives || ' Five Dollar Bill');
    END IF;
    IF v_ones > 0 THEN
        DBMS_OUTPUT.PUT_LINE(v_ones || ' One Dollar Bill');
    END IF;
END;
/
```

!!! note "Process Flow"
    1. Check edge cases first: not enough money, exact change
    2. Calculate total change: `p_Pay - p_AmountDue`
    3. Use **integer division** (`FLOOR(v_change / 20)`) to get the number of each bill
    4. Use **MOD** to get the remaining amount after each denomination
    5. Work from largest bill to smallest (greedy algorithm)

**Test cases:**
```sql
EXEC GetChange(12, 200);   -- 9 Twenty, 1 Five, 3 One
EXEC GetChange(50, 50);    -- Exact change
EXEC GetChange(100, 50);   -- Not enough money
EXEC GetChange(3, 20);     -- 1 Ten, 1 Five, 2 One
```

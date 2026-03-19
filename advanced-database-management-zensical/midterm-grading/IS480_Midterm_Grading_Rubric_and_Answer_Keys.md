# IS 480 Midterm — PL/SQL with Soccer Schema
## Detailed Grading Rubric & Instructor Answer Keys

---

## ESSAY QUESTIONS OVERVIEW

| Question | Topic | Points | Time Estimate |
|----------|-------|--------|---------------|
| Q11 | `add_match` procedure | 12 | 8–10 min |
| Q12 | `get_team_points_for_match` function | 12 | 10–12 min |
| Q13 | `add_player_rating` procedure w/ cursor | 16 | 12–15 min |
| Q16 | `pkg_team_tools` package skeleton | 12 | 10–12 min |

**Total essay points: 52** (out of 100)

---

## QUESTION 11 — `add_match` Procedure (12 points)

### Prompt (as administered)
> Write a PL/SQL procedure named add_match with these input parameters:
> p_match_id, p_season, p_stage, p_match_date, p_home_team_id, p_away_team_id
>
> Requirements:
> - Insert a new row into MATCHES.
> - Initialize (set initial value of) home_team_goal and away_team_goal to NULL.
> - If the home team and away team are the same, reject the insert using RAISE_APPLICATION_ERROR.
> - Do not issue COMMIT.
>
> You may use either explicit Oracle datatypes or %TYPE.

### Instructor Answer Key

```sql
CREATE OR REPLACE PROCEDURE add_match (
    p_match_id       IN NUMBER,
    p_season         IN VARCHAR2,
    p_stage          IN NUMBER,
    p_match_date     IN DATE,
    p_home_team_id   IN NUMBER,
    p_away_team_id   IN NUMBER
) IS
BEGIN
    -- Validate: team cannot play itself
    IF p_home_team_id = p_away_team_id THEN
        RAISE_APPLICATION_ERROR(-20001, 'A team cannot play itself.');
    END IF;

    -- Insert the match with NULL goals
    INSERT INTO MATCHES (
        match_id, season, stage, match_date,
        home_team_id, away_team_id,
        home_team_goal, away_team_goal
    ) VALUES (
        p_match_id, p_season, p_stage, p_match_date,
        p_home_team_id, p_away_team_id,
        NULL, NULL
    );
END add_match;
/
```

### Detailed Rubric (12 points)

| Criterion | Points | Full Credit | Partial Credit | No Credit |
|-----------|--------|-------------|----------------|-----------|
| **A. Procedure header** | 2 | Correct `CREATE OR REPLACE PROCEDURE add_match(...)` with all 6 parameters and reasonable datatypes (NUMBER, VARCHAR2, DATE or %TYPE). `IN` mode may be implicit. | 1 pt: Missing 1–2 parameters, or minor datatype issue (e.g., all NUMBER). | 0: Wrong construct (FUNCTION, anonymous block), or header is fundamentally broken. |
| **B. Validation logic** | 3 | Checks `p_home_team_id = p_away_team_id` BEFORE the INSERT, and calls `RAISE_APPLICATION_ERROR` with a valid error number (-20000 to -20999) and message string. | 2 pts: Correct check but wrong error procedure name (e.g., `RAISE ERROR`), or check is after INSERT. 1 pt: Conceptually right idea but badly broken syntax (e.g., `IF home = away THEN PRINT 'error'`). | 0: No validation at all, or validation has nothing to do with comparing team IDs. |
| **C. INSERT statement** | 5 | Correct `INSERT INTO MATCHES (...) VALUES (...)` with all 8 columns (match_id, season, stage, match_date, home_team_id, away_team_id, home_team_goal, away_team_goal). Goal columns explicitly set to NULL. Uses parameters correctly in VALUES. | 4 pts: Correct structure but missing 1 column or minor syntax error (trailing comma, missing paren). 3 pts: Correct concept but goals initialized to 0 instead of NULL, or missing 2 columns. 2 pts: INSERT present but significantly wrong (e.g., wrong table name, parameters in column list instead of values). 1 pt: Attempted INSERT but fundamentally broken. | 0: No INSERT statement. |
| **D. Overall PL/SQL structure** | 2 | Code is syntactically reasonable as a PL/SQL procedure. Has BEGIN/END, semicolons, IS/AS keyword. No COMMIT present. | 1 pt: Minor structural issues (missing END, missing semicolons) but recognizable as a procedure. Deduct 0.5 if COMMIT is included. | 0: Not recognizable as PL/SQL. |

### Common Student Errors to Watch For
- Using `ELSEIF` instead of `ELSIF` (SQL Server syntax) — deduct 0.5 from criterion B
- Putting parameters in the column list: `INSERT INTO MATCHES (p_match_id, p_season...)` — deduct 1 from criterion C
- Including `COMMIT` — deduct 0.5 from criterion D
- Goals initialized to 0 instead of NULL — deduct 2 from criterion C
- `RAISE_APPLICATION_ERROR` with error number outside -20000 to -20999 range — deduct 0.5 from criterion B
- Validation AFTER the INSERT — deduct 1 from criterion B (give 2 instead of 3)

---

## QUESTION 12 — `get_team_points_for_match` Function (12 points)

### Prompt (as administered)
> Write a PL/SQL function named get_team_points_for_match.
> Inputs: p_match_id, p_team_id
> Return type: NUMBER
>
> Requirements:
> - Read the corresponding row from MATCHES.
> - If no row exists for p_match_id, reject the call.
> - If p_team_id is not equal to either home_team_id or away_team_id for that match, reject the call.
> - If either home_team_goal or away_team_goal is NULL, reject the call because the match is not completed.
> - Return 3 for a win, 1 for a draw, and 0 for a loss.
>
> You may use RAISE_APPLICATION_ERROR for validation failures. Alternatively, use a user-defined exception.

### Instructor Answer Key

```sql
CREATE OR REPLACE FUNCTION get_team_points_for_match (
    p_match_id   IN NUMBER,
    p_team_id    IN NUMBER
) RETURN NUMBER
IS
    v_home_id    MATCHES.home_team_id%TYPE;
    v_away_id    MATCHES.away_team_id%TYPE;
    v_home_goal  MATCHES.home_team_goal%TYPE;
    v_away_goal  MATCHES.away_team_goal%TYPE;
BEGIN
    -- Read the match row
    BEGIN
        SELECT home_team_id, away_team_id, home_team_goal, away_team_goal
        INTO   v_home_id, v_away_id, v_home_goal, v_away_goal
        FROM   MATCHES
        WHERE  match_id = p_match_id;
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            RAISE_APPLICATION_ERROR(-20001, 'Match not found.');
    END;

    -- Validate: team must be in the match
    IF p_team_id NOT IN (v_home_id, v_away_id) THEN
        RAISE_APPLICATION_ERROR(-20002, 'Team is not part of this match.');
    END IF;

    -- Validate: match must be completed
    IF v_home_goal IS NULL OR v_away_goal IS NULL THEN
        RAISE_APPLICATION_ERROR(-20003, 'Match is not completed.');
    END IF;

    -- Determine points
    IF p_team_id = v_home_id THEN
        -- Team is home
        IF v_home_goal > v_away_goal THEN
            RETURN 3;   -- win
        ELSIF v_home_goal = v_away_goal THEN
            RETURN 1;   -- draw
        ELSE
            RETURN 0;   -- loss
        END IF;
    ELSE
        -- Team is away
        IF v_away_goal > v_home_goal THEN
            RETURN 3;   -- win
        ELSIF v_away_goal = v_home_goal THEN
            RETURN 1;   -- draw
        ELSE
            RETURN 0;   -- loss
        END IF;
    END IF;
END get_team_points_for_match;
/
```

### Detailed Rubric (12 points)

| Criterion | Points | Full Credit | Partial Credit | No Credit |
|-----------|--------|-------------|----------------|-----------|
| **A. Function header & RETURN type** | 2 | Correct `CREATE OR REPLACE FUNCTION ... RETURN NUMBER IS`. Both parameters present with reasonable types. | 1 pt: RETURN clause missing or wrong type, but otherwise correct function structure. | 0: Written as PROCEDURE, or header fundamentally wrong. |
| **B. SELECT INTO** | 3 | Correct `SELECT ... INTO` retrieving home_team_id, away_team_id, home_team_goal, away_team_goal from MATCHES where match_id = p_match_id. Variables declared or %TYPE used. | 2 pts: SELECT INTO present but missing 1–2 columns, or uses `SELECT *`. 1 pt: Attempts to read from MATCHES but syntax is badly wrong. | 0: No SELECT INTO or data retrieval. |
| **C. Validation — team in match** | 2 | Checks that p_team_id equals either home_team_id or away_team_id; rejects with RAISE_APPLICATION_ERROR or user-defined exception if not. | 1 pt: Check is present but logic is inverted, incomplete, or uses wrong rejection method (e.g., DBMS_OUTPUT instead of raising error). | 0: No team validation. |
| **D. Validation — match exists & goals not NULL** | 2 | Handles NO_DATA_FOUND (either via exception handler or explicit check) AND checks that goals are not NULL. Both validations reject the call. | 1 pt: Only one of the two validations present. 0.5 pt: Concept present but implementation broken. | 0: Neither validation present. |
| **E. Win/draw/loss logic & RETURN** | 3 | Correctly determines whether p_team_id won, drew, or lost by comparing the right goal columns based on whether the team is home or away. Returns 3, 1, or 0 respectively. | 2 pts: Logic is mostly correct but has an edge case error (e.g., doesn't distinguish home vs away correctly, or always compares from home perspective). 1 pt: Returns 3/1/0 somewhere but logic is fundamentally confused. | 0: No return logic or completely wrong. |

### Common Student Errors to Watch For
- Comparing `p_match_id` to point values instead of looking up goals — 0 pts for criteria B+E
- Using `ELSEIF` instead of `ELSIF` — deduct 0.5
- Not distinguishing home vs. away perspective (e.g., always `IF home_goal > away_goal THEN RETURN 3`) — if p_team_id is the away team, this is wrong. Deduct 1 from criterion E.
- Letting NO_DATA_FOUND propagate unhandled without any mention — acceptable IF the prompt allows it (give 1 pt for criterion D since it's implicit handling)
- Error number outside -20000 to -20999 — deduct 0.5

---

## QUESTION 13 — `add_player_rating` Procedure with Cursor (16 points)

### Prompt (as administered)
> Write a PL/SQL procedure named add_player_rating that inserts a new row into PLAYER_ATTRIBUTE_HISTORY.
> After the insert, use a cursor to print that player's rating history from newest to oldest using DBMS_OUTPUT.PUT_LINE.
> Your printed output must include at least:
> - attribute_date
> - overall_rating
>
> You may choose the full parameter list, but it should reasonably support inserting one new rating row.

### Instructor Answer Key

```sql
CREATE OR REPLACE PROCEDURE add_player_rating (
    p_player_attr_id   IN NUMBER,
    p_player_id        IN NUMBER,
    p_attribute_date   IN DATE,
    p_overall_rating   IN NUMBER,
    p_potential         IN NUMBER,
    p_preferred_foot   IN VARCHAR2
) IS
    -- Cursor to retrieve rating history for the player, newest first
    CURSOR c_history IS
        SELECT attribute_date, overall_rating
        FROM   PLAYER_ATTRIBUTE_HISTORY
        WHERE  player_id = p_player_id
        ORDER BY attribute_date DESC;
BEGIN
    -- Insert the new rating row
    INSERT INTO PLAYER_ATTRIBUTE_HISTORY (
        player_attr_id, player_id, attribute_date,
        overall_rating, potential, preferred_foot
    ) VALUES (
        p_player_attr_id, p_player_id, p_attribute_date,
        p_overall_rating, p_potential, p_preferred_foot
    );

    -- Print the full rating history
    DBMS_OUTPUT.PUT_LINE('Rating history for player ' || p_player_id || ':');
    FOR rec IN c_history LOOP
        DBMS_OUTPUT.PUT_LINE(rec.attribute_date || ' - Rating: ' || rec.overall_rating);
    END LOOP;
END add_player_rating;
/
```

**Alternative acceptable cursor patterns:**
- Implicit cursor FOR LOOP (no explicit CURSOR declaration): `FOR rec IN (SELECT ...) LOOP`
- Explicit OPEN/FETCH/CLOSE pattern
- Parameterized cursor: `CURSOR c_history(p_pid NUMBER) IS ...`

### Detailed Rubric (16 points)

| Criterion | Points | Full Credit | Partial Credit | No Credit |
|-----------|--------|-------------|----------------|-----------|
| **A. Procedure header** | 3 | Correct `CREATE OR REPLACE PROCEDURE add_player_rating(...)` with parameters that reasonably support inserting a rating row. Must include at minimum: player_id, attribute_date, overall_rating. Additional columns acceptable. | 2 pts: Missing 1–2 important parameters but structure is correct. 1 pt: Header present but severely incomplete. | 0: Not a procedure, or no header. |
| **B. INSERT statement** | 5 | Correct `INSERT INTO PLAYER_ATTRIBUTE_HISTORY (...) VALUES (...)` using the input parameters. Column list matches the parameter values. | 4 pts: INSERT correct but missing 1 column. 3 pts: INSERT present, right table, but 2+ columns missing or parameter/column mismatch. 2 pts: INSERT attempted but significantly wrong. 1 pt: INSERT keyword present, barely functional. | 0: No INSERT. |
| **C. Cursor declaration** | 3 | Cursor correctly defined to select from PLAYER_ATTRIBUTE_HISTORY WHERE player_id = [parameter], ORDER BY attribute_date DESC. Any valid cursor pattern accepted (explicit, implicit FOR LOOP, parameterized). | 2 pts: Cursor selects from right table but missing WHERE clause or ORDER BY. 1 pt: Cursor attempted but fundamentally wrong (e.g., wrong table, no SELECT). | 0: No cursor. |
| **D. Cursor loop / fetch** | 3 | Correctly iterates through cursor results. FOR LOOP, or OPEN/FETCH/CLOSE pattern. Loop body accesses cursor record fields. | 2 pts: Loop present but has errors (e.g., FETCH without EXIT WHEN, or wrong variable references). 1 pt: Loop attempted but broken. | 0: No loop/iteration. |
| **E. DBMS_OUTPUT.PUT_LINE** | 2 | Prints at least attribute_date and overall_rating inside the loop. Concatenation or multiple calls acceptable. | 1 pt: Prints something but missing one of the two required fields, OR has significant typo in DBMS_OUTPUT name but intent is clear. | 0: No output statement. |

### Common Student Errors to Watch For
- No INSERT, only cursor — give 0 for B, grade cursor parts normally
- Cursor without WHERE clause (retrieves ALL players' history) — deduct 1 from criterion C
- Cursor without ORDER BY — deduct 1 from criterion C
- Using UPDATE instead of INSERT — 0 for criterion B
- `DBMS_OUTPUT` misspellings (`dbms.outline_put.line`, `DBMS_OUTPUT_LINE`) — if intent is clear, give 1 pt for E
- Cursor declared but never opened/looped — 0 for D, keep C credit
- Empty/no answer — Sydney Mendez submitted nothing for this question

---

## QUESTION 16 — `pkg_team_tools` Package Skeleton (12 points)

### Prompt (as administered)
> Create a package named pkg_team_tools using a package specification and package body.
> The package specification must expose these public program units:
> - FUNCTION get_team_label(p_team_id IN NUMBER) RETURN VARCHAR2;
> - PROCEDURE print_team_message(p_team_id IN NUMBER);
> - FUNCTION count_team_players(p_team_id IN NUMBER) RETURN NUMBER;
>
> Requirements for the package body:
> - Implement all three public program units using placeholder bodies.
> - Create a helper procedure named log_action that remains private.
> - log_action must appear only in the package body.
> - print_team_message must call log_action.
> - You do not need to write the actual SQL statements or exception handling.
> - Use NULL; as a placeholder in procedures.
> - Use RETURN NULL; as a placeholder in functions.
>
> Write the package specification and package body skeleton.

### Instructor Answer Key

```sql
-- Package Specification
CREATE OR REPLACE PACKAGE pkg_team_tools AS
    FUNCTION get_team_label(p_team_id IN NUMBER) RETURN VARCHAR2;
    PROCEDURE print_team_message(p_team_id IN NUMBER);
    FUNCTION count_team_players(p_team_id IN NUMBER) RETURN NUMBER;
END pkg_team_tools;
/

-- Package Body
CREATE OR REPLACE PACKAGE BODY pkg_team_tools AS

    -- Private helper (NOT in spec)
    PROCEDURE log_action IS
    BEGIN
        NULL;
    END log_action;

    FUNCTION get_team_label(p_team_id IN NUMBER) RETURN VARCHAR2 IS
    BEGIN
        RETURN NULL;
    END get_team_label;

    PROCEDURE print_team_message(p_team_id IN NUMBER) IS
    BEGIN
        log_action;   -- calls the private helper
        NULL;
    END print_team_message;

    FUNCTION count_team_players(p_team_id IN NUMBER) RETURN NUMBER IS
    BEGIN
        RETURN NULL;
    END count_team_players;

END pkg_team_tools;
/
```

### Detailed Rubric (12 points)

| Criterion | Points | Full Credit | Partial Credit | No Credit |
|-----------|--------|-------------|----------------|-----------|
| **A. Package specification** | 4 | Correct `CREATE OR REPLACE PACKAGE pkg_team_tools AS/IS` with exactly the 3 public declarations (get_team_label, print_team_message, count_team_players). Functions include RETURN clause. Ends with `END pkg_team_tools;`. log_action does NOT appear in spec. | 3 pts: All 3 routines declared but minor syntax issues (missing RETURN type, missing semicolons). 2 pts: 2 of 3 routines declared correctly, OR log_action incorrectly appears in spec (key conceptual error — deduct 2). 1 pt: Attempted spec but severely incomplete. | 0: No specification, or spec is just `BEGIN ... END`. |
| **B. Package body structure** | 3 | Correct `CREATE OR REPLACE PACKAGE BODY pkg_team_tools AS/IS` with all 3 public routines implemented (even as placeholders). Each routine has header + BEGIN + (NULL or RETURN NULL) + END. Ends with `END pkg_team_tools;`. | 2 pts: Body present with most routines but 1 missing or structural errors. 1 pt: Body attempted but fundamentally broken (e.g., uses CREATE OR REPLACE inside body, or BEGIN in spec). | 0: No package body. |
| **C. Private helper placement** | 3 | `log_action` procedure is declared and implemented ONLY in the package body. It does NOT appear in the spec. It is placed before the routines that call it (or a forward declaration is used). | 2 pts: log_action is in the body but also incorrectly in the spec. 1 pt: log_action exists somewhere but placement is wrong or it's not a procedure. | 0: No log_action at all. |
| **D. Call from public to private** | 2 | `print_team_message` body contains a call to `log_action` (e.g., `log_action;`). | 1 pt: A different routine calls log_action, or the call is present but wrong syntax. | 0: No routine calls log_action. |

### Common Student Errors to Watch For
- **log_action in the spec** — This is the #1 conceptual error. It directly contradicts "must remain private." Deduct 2 from criterion A (cap at 2), and give only 1 for criterion C.
- **Using `CREATE OR REPLACE` inside the body for each routine** — This is a fundamental misconception. Deduct 1 from criterion B.
- **Using `BEGIN` in the spec** — Package spec should only have declarations, no executable section. Deduct 1 from criterion A.
- **Missing RETURN NULL for functions** — Minor; deduct 0.5 from criterion B if the function body has no RETURN at all.
- **Only writing body, no spec** — 0 for criterion A, grade body normally.
- **Only writing spec, no body** — Grade spec normally, 0 for B/C/D.
- **Forgetting `END pkg_team_tools;`** — Deduct 0.5 from the relevant section.

---

## GRADING GUIDELINES — General Principles

### Syntax vs. Logic
**Weight logic and structure more heavily than syntax.** Students wrote code by hand under exam pressure. Common acceptable variations:
- Missing `OR REPLACE` (still valid PL/SQL)
- Using `AS` vs `IS` interchangeably
- Minor semicolon omissions
- Reasonable datatype variations (`NUMBER` vs `NUMBER(10)`)
- Case variations (`Begin` vs `BEGIN`)

### What to penalize
- Wrong construct entirely (function when procedure asked, or vice versa)
- Missing required logic (no validation, no cursor, no INSERT)
- Fundamental PL/SQL misconceptions (BEGIN in a spec, COMMIT when told not to)
- Using syntax from other languages (ELSEIF, THROW, TRY/CATCH)

### Partial credit philosophy
- If a student demonstrates understanding of the concept but has syntax errors, give majority credit
- If a student has perfect syntax but wrong logic, give less credit
- An incomplete answer that shows correct approach is worth more than a complete but wrong answer

### Cross-question dependency
Q11–Q13 use the soccer schema routines. Q16 intentionally uses DIFFERENT routine names (pkg_team_tools) to avoid leaking answers. Grade each question independently — do not penalize Q16 if Q11–Q13 were wrong, and do not give Q16 credit just because Q11–Q13 were correct.

---

## QUICK REFERENCE — Point Allocation Summary

```
Q11  add_match (12 pts)
  A. Procedure header:        2
  B. Validation logic:        3
  C. INSERT statement:        5
  D. Overall PL/SQL structure: 2

Q12  get_team_points_for_match (12 pts)
  A. Function header/RETURN:  2
  B. SELECT INTO:             3
  C. Team validation:         2
  D. Match exists + NULL:     2
  E. Win/draw/loss + RETURN:  3

Q13  add_player_rating (16 pts)
  A. Procedure header:        3
  B. INSERT statement:        5
  C. Cursor declaration:      3
  D. Cursor loop/fetch:       3
  E. DBMS_OUTPUT:             2

Q16  pkg_team_tools (12 pts)
  A. Package specification:   4
  B. Package body structure:  3
  C. Private helper placement: 3
  D. Call from public→private: 2

TOTAL ESSAY: 52 points
TOTAL MC:    48 points (auto-graded)
GRAND TOTAL: 100 points
```

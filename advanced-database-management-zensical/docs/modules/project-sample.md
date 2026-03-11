# Sample Project: BambooBudget — Personal Finance Manager

!!! tip "This is a complete Phase 1 example"
    Use this as a reference for how your project should look. Your domain will be different, but the structure, code quality, and use of PL/SQL features should match this level.

## Domain: Personal Finance & Budgeting

**BambooBudget** is a personal finance platform inspired by apps like YNAB, Monarch Money, and Copilot. Users track their income and spending across bank accounts, organize transactions into budget categories using "envelope-style" budgeting, and monitor their net worth over time.

**Why this domain?** The personal finance app market serves over 83% of U.S. internet adults in some form. Apps like YNAB ($109/yr), Monarch Money ($99.99/yr), and Copilot ($12.99/mo) generate significant revenue by helping users answer two questions: *"Where did my money go?"* and *"Am I building the life I want?"* Behind every one of these apps is a relational database handling the exact operations you'll see below.

---

## Schema (`schema.sql`)

```sql
-- ============================================================
-- BambooBudget Schema — Personal Finance Manager
-- IS480 Sample Project | Spring 2026
-- ============================================================

-- Users
CREATE TABLE bb_users (
    user_id       NUMBER        PRIMARY KEY,
    username      VARCHAR2(50)  NOT NULL UNIQUE,
    email         VARCHAR2(100) NOT NULL UNIQUE,
    created_date  DATE          DEFAULT SYSDATE
);

-- Bank Accounts
CREATE TABLE bb_accounts (
    account_id    NUMBER        PRIMARY KEY,
    user_id       NUMBER        NOT NULL REFERENCES bb_users(user_id),
    account_name  VARCHAR2(100) NOT NULL,
    account_type  VARCHAR2(20)  NOT NULL
                  CHECK (account_type IN ('checking','savings','credit_card','investment','retirement','loan')),
    balance       NUMBER(12,2)  DEFAULT 0,
    is_active     CHAR(1)       DEFAULT 'Y' CHECK (is_active IN ('Y','N'))
);

-- Budget Categories (envelopes)
CREATE TABLE bb_categories (
    category_id   NUMBER        PRIMARY KEY,
    user_id       NUMBER        NOT NULL REFERENCES bb_users(user_id),
    category_name VARCHAR2(60)  NOT NULL,
    monthly_limit NUMBER(10,2)  DEFAULT 0,
    is_income     CHAR(1)       DEFAULT 'N' CHECK (is_income IN ('Y','N'))
);

-- Transactions
CREATE TABLE bb_transactions (
    txn_id        NUMBER        PRIMARY KEY,
    account_id    NUMBER        NOT NULL REFERENCES bb_accounts(account_id),
    category_id   NUMBER        REFERENCES bb_categories(category_id),
    txn_date      DATE          NOT NULL,
    merchant      VARCHAR2(100),
    amount        NUMBER(10,2)  NOT NULL,  -- positive = expense, negative = income
    notes         VARCHAR2(200)
);

-- Savings Goals
CREATE TABLE bb_goals (
    goal_id       NUMBER        PRIMARY KEY,
    user_id       NUMBER        NOT NULL REFERENCES bb_users(user_id),
    goal_name     VARCHAR2(100) NOT NULL,
    target_amount NUMBER(12,2)  NOT NULL,
    current_amount NUMBER(12,2) DEFAULT 0,
    target_date   DATE
);

-- Categorization Rules (auto-assign categories by merchant pattern)
CREATE TABLE bb_rules (
    rule_id       NUMBER        PRIMARY KEY,
    user_id       NUMBER        NOT NULL REFERENCES bb_users(user_id),
    pattern       VARCHAR2(100) NOT NULL,  -- merchant substring match
    category_id   NUMBER        NOT NULL REFERENCES bb_categories(category_id),
    priority      NUMBER        DEFAULT 1
);

-- Sequences
CREATE SEQUENCE bb_txn_seq START WITH 1000;
CREATE SEQUENCE bb_goal_seq START WITH 100;

-- ============================================================
-- Seed Data
-- ============================================================

-- Users
INSERT INTO bb_users VALUES (1, 'maria_g', 'maria@email.com', DATE '2026-01-15');
INSERT INTO bb_users VALUES (2, 'james_t', 'james@email.com', DATE '2026-02-01');

-- Accounts for Maria
INSERT INTO bb_accounts VALUES (101, 1, 'Chase Checking',    'checking',     4250.00, 'Y');
INSERT INTO bb_accounts VALUES (102, 1, 'Ally Savings',      'savings',      12000.00, 'Y');
INSERT INTO bb_accounts VALUES (103, 1, 'Visa Credit Card',  'credit_card',  -1875.50, 'Y');
INSERT INTO bb_accounts VALUES (104, 1, 'Vanguard 401k',     'retirement',   45200.00, 'Y');
INSERT INTO bb_accounts VALUES (105, 1, 'Old Checking',      'checking',     0.00,     'N');

-- Accounts for James
INSERT INTO bb_accounts VALUES (201, 2, 'BofA Checking',     'checking',     3100.00, 'Y');
INSERT INTO bb_accounts VALUES (202, 2, 'Marcus Savings',    'savings',       8500.00, 'Y');
INSERT INTO bb_accounts VALUES (203, 2, 'Amex Gold',         'credit_card',  -620.00,  'Y');

-- Categories for Maria
INSERT INTO bb_categories VALUES (10, 1, 'Salary',        0,       'Y');
INSERT INTO bb_categories VALUES (11, 1, 'Groceries',     600.00,  'N');
INSERT INTO bb_categories VALUES (12, 1, 'Rent',          2000.00, 'N');
INSERT INTO bb_categories VALUES (13, 1, 'Dining Out',    300.00,  'N');
INSERT INTO bb_categories VALUES (14, 1, 'Entertainment', 150.00,  'N');
INSERT INTO bb_categories VALUES (15, 1, 'Transportation',200.00,  'N');
INSERT INTO bb_categories VALUES (16, 1, 'Utilities',     250.00,  'N');

-- Categories for James
INSERT INTO bb_categories VALUES (20, 2, 'Freelance Income', 0,      'Y');
INSERT INTO bb_categories VALUES (21, 2, 'Groceries',        500.00, 'N');
INSERT INTO bb_categories VALUES (22, 2, 'Rent',             1800.00,'N');
INSERT INTO bb_categories VALUES (23, 2, 'Subscriptions',    100.00, 'N');

-- Transactions for Maria (February 2026)
INSERT INTO bb_transactions VALUES (bb_txn_seq.NEXTVAL, 101, 10, DATE '2026-02-01', 'Employer Direct Deposit', -5200.00, 'Monthly salary');
INSERT INTO bb_transactions VALUES (bb_txn_seq.NEXTVAL, 101, 12, DATE '2026-02-01', 'Landlord',                 2000.00, 'Feb rent');
INSERT INTO bb_transactions VALUES (bb_txn_seq.NEXTVAL, 101, 11, DATE '2026-02-03', 'Trader Joes',              87.50,   NULL);
INSERT INTO bb_transactions VALUES (bb_txn_seq.NEXTVAL, 103, 13, DATE '2026-02-05', 'Chipotle',                 14.25,   NULL);
INSERT INTO bb_transactions VALUES (bb_txn_seq.NEXTVAL, 103, 14, DATE '2026-02-07', 'Netflix',                  15.99,   'Monthly sub');
INSERT INTO bb_transactions VALUES (bb_txn_seq.NEXTVAL, 101, 11, DATE '2026-02-10', 'Whole Foods',              123.40,  NULL);
INSERT INTO bb_transactions VALUES (bb_txn_seq.NEXTVAL, 103, 13, DATE '2026-02-12', 'Olive Garden',             62.80,   'Dinner w/ friends');
INSERT INTO bb_transactions VALUES (bb_txn_seq.NEXTVAL, 101, 15, DATE '2026-02-14', 'Shell Gas Station',        48.00,   NULL);
INSERT INTO bb_transactions VALUES (bb_txn_seq.NEXTVAL, 101, 16, DATE '2026-02-15', 'SoCal Edison',             142.30,  'Electric bill');
INSERT INTO bb_transactions VALUES (bb_txn_seq.NEXTVAL, 103, 11, DATE '2026-02-18', 'Costco',                   215.60,  'Bulk groceries');
INSERT INTO bb_transactions VALUES (bb_txn_seq.NEXTVAL, 101, 13, DATE '2026-02-20', 'Starbucks',                6.75,    NULL);
INSERT INTO bb_transactions VALUES (bb_txn_seq.NEXTVAL, 103, 14, DATE '2026-02-22', 'Spotify',                  10.99,   'Monthly sub');
INSERT INTO bb_transactions VALUES (bb_txn_seq.NEXTVAL, 101, 10, DATE '2026-02-28', 'Employer Direct Deposit', -5200.00, 'Bi-weekly salary');

-- Transactions for James (February 2026)
INSERT INTO bb_transactions VALUES (bb_txn_seq.NEXTVAL, 201, 20, DATE '2026-02-05', 'Client Payment',          -3200.00, 'Web dev project');
INSERT INTO bb_transactions VALUES (bb_txn_seq.NEXTVAL, 201, 22, DATE '2026-02-01', 'Property Mgmt Co',         1800.00, 'Feb rent');
INSERT INTO bb_transactions VALUES (bb_txn_seq.NEXTVAL, 201, 21, DATE '2026-02-08', 'Walmart',                  95.20,   NULL);
INSERT INTO bb_transactions VALUES (bb_txn_seq.NEXTVAL, 203, 23, DATE '2026-02-10', 'Adobe Creative Cloud',     59.99,   NULL);

-- Goals for Maria
INSERT INTO bb_goals VALUES (bb_goal_seq.NEXTVAL, 1, 'Emergency Fund',     15000.00, 12000.00, DATE '2026-06-30');
INSERT INTO bb_goals VALUES (bb_goal_seq.NEXTVAL, 1, 'Vacation to Japan',  4000.00,  1200.00,  DATE '2026-12-01');
INSERT INTO bb_goals VALUES (bb_goal_seq.NEXTVAL, 1, 'Pay Off Credit Card',1875.50,  0.00,     DATE '2026-05-01');

-- Goals for James
INSERT INTO bb_goals VALUES (bb_goal_seq.NEXTVAL, 2, 'New Laptop',         2500.00,  800.00,   DATE '2026-08-01');

-- Rules for Maria
INSERT INTO bb_rules VALUES (1, 1, 'Trader Joe',  11, 1);
INSERT INTO bb_rules VALUES (2, 1, 'Whole Foods', 11, 1);
INSERT INTO bb_rules VALUES (3, 1, 'Costco',      11, 2);
INSERT INTO bb_rules VALUES (4, 1, 'Chipotle',    13, 1);
INSERT INTO bb_rules VALUES (5, 1, 'Starbucks',   13, 1);
INSERT INTO bb_rules VALUES (6, 1, 'Netflix',     14, 1);
INSERT INTO bb_rules VALUES (7, 1, 'Spotify',     14, 1);
INSERT INTO bb_rules VALUES (8, 1, 'Shell',       15, 1);

COMMIT;
```

---

## Package Specification (`package_spec.sql`)

```sql
-- ============================================================
-- BambooBudget PL/SQL Package — Specification
-- ============================================================
CREATE OR REPLACE PACKAGE bamboo_budget_pkg AS

    -- ---- PROCEDURES ----

    -- Record a new transaction and update the account balance
    PROCEDURE record_transaction(
        p_account_id  bb_accounts.account_id%TYPE,
        p_category_id bb_categories.category_id%TYPE,
        p_txn_date    bb_transactions.txn_date%TYPE,
        p_merchant    bb_transactions.merchant%TYPE,
        p_amount      bb_transactions.amount%TYPE,
        p_notes       bb_transactions.notes%TYPE DEFAULT NULL
    );

    -- Transfer money between two accounts for the same user
    PROCEDURE transfer_funds(
        p_from_account bb_accounts.account_id%TYPE,
        p_to_account   bb_accounts.account_id%TYPE,
        p_amount       NUMBER
    );

    -- Contribute toward a savings goal
    PROCEDURE contribute_to_goal(
        p_goal_id bb_goals.goal_id%TYPE,
        p_amount  NUMBER
    );

    -- Generate a monthly budget report for a user (cursor-based)
    PROCEDURE monthly_budget_report(
        p_user_id bb_users.user_id%TYPE,
        p_month   NUMBER,
        p_year    NUMBER
    );

    -- ---- FUNCTIONS ----

    -- Calculate budget health score (like our Grading() function)
    FUNCTION budget_health_score(
        p_user_id bb_users.user_id%TYPE,
        p_month   NUMBER,
        p_year    NUMBER
    ) RETURN VARCHAR2;

    -- Calculate net worth for a user
    FUNCTION get_net_worth(
        p_user_id bb_users.user_id%TYPE
    ) RETURN NUMBER;

    -- Auto-categorize a transaction by merchant name using rules
    FUNCTION auto_categorize(
        p_user_id  bb_users.user_id%TYPE,
        p_merchant bb_transactions.merchant%TYPE
    ) RETURN NUMBER;

END bamboo_budget_pkg;
/
```

---

## Package Body (`package_body.sql`)

```sql
-- ============================================================
-- BambooBudget PL/SQL Package — Body
-- ============================================================
CREATE OR REPLACE PACKAGE BODY bamboo_budget_pkg AS

    -- ===========================================================
    -- PROCEDURE: record_transaction
    -- Records a transaction, auto-categorizes if no category given,
    -- and updates the account balance.
    -- Features: %TYPE params, COUNT(*) existence check, exception
    --           handling, IF/ELSIF, auto-categorize function call
    -- ===========================================================
    PROCEDURE record_transaction(
        p_account_id  bb_accounts.account_id%TYPE,
        p_category_id bb_categories.category_id%TYPE,
        p_txn_date    bb_transactions.txn_date%TYPE,
        p_merchant    bb_transactions.merchant%TYPE,
        p_amount      bb_transactions.amount%TYPE,
        p_notes       bb_transactions.notes%TYPE DEFAULT NULL
    ) IS
        v_count       NUMBER;
        v_user_id     bb_accounts.user_id%TYPE;
        v_category_id bb_categories.category_id%TYPE;
        v_new_txn_id  NUMBER;
    BEGIN
        -- Validate account exists and is active
        SELECT COUNT(*) INTO v_count
        FROM bb_accounts
        WHERE account_id = p_account_id AND is_active = 'Y';

        IF v_count = 0 THEN
            RAISE_APPLICATION_ERROR(-20001,
                'Account ' || p_account_id || ' does not exist or is inactive.');
        END IF;

        -- Get user_id for this account
        SELECT user_id INTO v_user_id
        FROM bb_accounts
        WHERE account_id = p_account_id;

        -- Determine category: use provided or auto-categorize
        IF p_category_id IS NOT NULL THEN
            -- Validate category exists for this user
            SELECT COUNT(*) INTO v_count
            FROM bb_categories
            WHERE category_id = p_category_id AND user_id = v_user_id;

            IF v_count = 0 THEN
                RAISE_APPLICATION_ERROR(-20002,
                    'Category ' || p_category_id || ' not found for this user.');
            END IF;
            v_category_id := p_category_id;
        ELSE
            -- Auto-categorize based on merchant name
            v_category_id := auto_categorize(v_user_id, p_merchant);
            IF v_category_id IS NOT NULL THEN
                DBMS_OUTPUT.PUT_LINE('Auto-categorized "' || p_merchant
                    || '" to category ' || v_category_id);
            ELSE
                DBMS_OUTPUT.PUT_LINE('WARNING: No matching rule for "'
                    || p_merchant || '". Transaction saved uncategorized.');
            END IF;
        END IF;

        -- Insert the transaction
        v_new_txn_id := bb_txn_seq.NEXTVAL;
        INSERT INTO bb_transactions (txn_id, account_id, category_id, txn_date, merchant, amount, notes)
        VALUES (v_new_txn_id, p_account_id, v_category_id, p_txn_date, p_merchant, p_amount, p_notes);

        -- Update account balance
        UPDATE bb_accounts
        SET balance = balance - p_amount   -- expense (+) decreases balance; income (-) increases it
        WHERE account_id = p_account_id;

        DBMS_OUTPUT.PUT_LINE('Transaction #' || v_new_txn_id || ' recorded: '
            || p_merchant || ' $' || ABS(p_amount));

        COMMIT;

    EXCEPTION
        WHEN OTHERS THEN
            ROLLBACK;
            DBMS_OUTPUT.PUT_LINE('ERROR in record_transaction: ' || SQLERRM);
            RAISE;
    END record_transaction;


    -- ===========================================================
    -- PROCEDURE: transfer_funds
    -- Moves money between two accounts owned by the same user.
    -- Features: %TYPE params, COUNT(*) check, RAISE_APPLICATION_ERROR,
    --           multi-table validation
    -- ===========================================================
    PROCEDURE transfer_funds(
        p_from_account bb_accounts.account_id%TYPE,
        p_to_account   bb_accounts.account_id%TYPE,
        p_amount       NUMBER
    ) IS
        v_from_user bb_accounts.user_id%TYPE;
        v_to_user   bb_accounts.user_id%TYPE;
        v_from_bal  bb_accounts.balance%TYPE;
    BEGIN
        IF p_amount <= 0 THEN
            RAISE_APPLICATION_ERROR(-20003, 'Transfer amount must be positive.');
        END IF;

        IF p_from_account = p_to_account THEN
            RAISE_APPLICATION_ERROR(-20004, 'Cannot transfer to the same account.');
        END IF;

        -- Get and validate both accounts
        BEGIN
            SELECT user_id, balance INTO v_from_user, v_from_bal
            FROM bb_accounts
            WHERE account_id = p_from_account AND is_active = 'Y';
        EXCEPTION
            WHEN NO_DATA_FOUND THEN
                RAISE_APPLICATION_ERROR(-20005,
                    'Source account ' || p_from_account || ' not found or inactive.');
        END;

        BEGIN
            SELECT user_id INTO v_to_user
            FROM bb_accounts
            WHERE account_id = p_to_account AND is_active = 'Y';
        EXCEPTION
            WHEN NO_DATA_FOUND THEN
                RAISE_APPLICATION_ERROR(-20006,
                    'Destination account ' || p_to_account || ' not found or inactive.');
        END;

        -- Verify same user
        IF v_from_user != v_to_user THEN
            RAISE_APPLICATION_ERROR(-20007,
                'Cannot transfer between accounts of different users.');
        END IF;

        -- Check sufficient funds (for non-credit accounts)
        IF v_from_bal < p_amount THEN
            DBMS_OUTPUT.PUT_LINE('WARNING: Transfer will overdraw account '
                || p_from_account || ' (balance: $' || v_from_bal || ')');
        END IF;

        -- Execute transfer
        UPDATE bb_accounts SET balance = balance - p_amount
        WHERE account_id = p_from_account;

        UPDATE bb_accounts SET balance = balance + p_amount
        WHERE account_id = p_to_account;

        DBMS_OUTPUT.PUT_LINE('Transferred $' || p_amount
            || ' from account ' || p_from_account
            || ' to account ' || p_to_account);

        COMMIT;

    EXCEPTION
        WHEN OTHERS THEN
            ROLLBACK;
            DBMS_OUTPUT.PUT_LINE('ERROR in transfer_funds: ' || SQLERRM);
            RAISE;
    END transfer_funds;


    -- ===========================================================
    -- PROCEDURE: contribute_to_goal
    -- Adds money toward a savings goal. Checks if already met.
    -- Features: NO_DATA_FOUND exception, IF/ELSIF, %TYPE params
    -- ===========================================================
    PROCEDURE contribute_to_goal(
        p_goal_id bb_goals.goal_id%TYPE,
        p_amount  NUMBER
    ) IS
        v_goal_name    bb_goals.goal_name%TYPE;
        v_target       bb_goals.target_amount%TYPE;
        v_current      bb_goals.current_amount%TYPE;
        v_new_amount   NUMBER;
        v_pct_complete NUMBER;
    BEGIN
        IF p_amount <= 0 THEN
            RAISE_APPLICATION_ERROR(-20010, 'Contribution must be positive.');
        END IF;

        -- Fetch goal details (will raise NO_DATA_FOUND if invalid)
        SELECT goal_name, target_amount, current_amount
        INTO v_goal_name, v_target, v_current
        FROM bb_goals
        WHERE goal_id = p_goal_id;

        -- Check if already completed
        IF v_current >= v_target THEN
            DBMS_OUTPUT.PUT_LINE('Goal "' || v_goal_name
                || '" is already fully funded! ($'
                || v_current || ' / $' || v_target || ')');
            RETURN;
        END IF;

        -- Calculate new amount (cap at target)
        v_new_amount := LEAST(v_current + p_amount, v_target);

        UPDATE bb_goals
        SET current_amount = v_new_amount
        WHERE goal_id = p_goal_id;

        v_pct_complete := ROUND((v_new_amount / v_target) * 100, 1);

        IF v_new_amount >= v_target THEN
            DBMS_OUTPUT.PUT_LINE('🎉 GOAL REACHED! "' || v_goal_name
                || '" is fully funded at $' || v_target || '!');
        ELSIF v_pct_complete >= 75 THEN
            DBMS_OUTPUT.PUT_LINE('Almost there! "' || v_goal_name
                || '" is ' || v_pct_complete || '% funded ($'
                || v_new_amount || ' / $' || v_target || ')');
        ELSE
            DBMS_OUTPUT.PUT_LINE('Contributed $' || p_amount
                || ' to "' || v_goal_name || '" — '
                || v_pct_complete || '% complete ($'
                || v_new_amount || ' / $' || v_target || ')');
        END IF;

        COMMIT;

    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            DBMS_OUTPUT.PUT_LINE('ERROR: Goal #' || p_goal_id || ' does not exist.');
            RAISE_APPLICATION_ERROR(-20011, 'Goal ' || p_goal_id || ' not found.');
    END contribute_to_goal;


    -- ===========================================================
    -- PROCEDURE: monthly_budget_report
    -- Prints a budget-vs-actual report using an explicit cursor.
    -- Features: Explicit cursor (OPEN/FETCH/EXIT WHEN/CLOSE),
    --           %ROWTYPE-style variables, FOR loop with SQL,
    --           budget_health_score function call
    -- ===========================================================
    PROCEDURE monthly_budget_report(
        p_user_id bb_users.user_id%TYPE,
        p_month   NUMBER,
        p_year    NUMBER
    ) IS
        -- Explicit cursor: budget categories with spending
        CURSOR c_budget IS
            SELECT c.category_name,
                   c.monthly_limit,
                   NVL(SUM(t.amount), 0) AS spent
            FROM bb_categories c
            LEFT JOIN bb_transactions t
                ON c.category_id = t.category_id
                AND EXTRACT(MONTH FROM t.txn_date) = p_month
                AND EXTRACT(YEAR FROM t.txn_date) = p_year
            WHERE c.user_id = p_user_id
              AND c.is_income = 'N'
              AND c.monthly_limit > 0
            GROUP BY c.category_name, c.monthly_limit
            ORDER BY c.category_name;

        v_cat_name   bb_categories.category_name%TYPE;
        v_limit      bb_categories.monthly_limit%TYPE;
        v_spent      NUMBER;
        v_remaining  NUMBER;
        v_status     VARCHAR2(20);
        v_total_income NUMBER := 0;
        v_total_spent  NUMBER := 0;
        v_health       VARCHAR2(50);
        v_count        NUMBER;
    BEGIN
        -- Validate user exists
        SELECT COUNT(*) INTO v_count
        FROM bb_users WHERE user_id = p_user_id;

        IF v_count = 0 THEN
            RAISE_APPLICATION_ERROR(-20020,
                'User ' || p_user_id || ' does not exist.');
        END IF;

        DBMS_OUTPUT.PUT_LINE('========================================');
        DBMS_OUTPUT.PUT_LINE('  MONTHLY BUDGET REPORT');
        DBMS_OUTPUT.PUT_LINE('  User: ' || p_user_id
            || ' | ' || p_month || '/' || p_year);
        DBMS_OUTPUT.PUT_LINE('========================================');

        -- Calculate total income using FOR loop with SQL
        FOR r_income IN (
            SELECT NVL(SUM(ABS(t.amount)), 0) AS total_inc
            FROM bb_transactions t
            JOIN bb_accounts a ON t.account_id = a.account_id
            JOIN bb_categories c ON t.category_id = c.category_id
            WHERE a.user_id = p_user_id
              AND c.is_income = 'Y'
              AND EXTRACT(MONTH FROM t.txn_date) = p_month
              AND EXTRACT(YEAR FROM t.txn_date) = p_year
        ) LOOP
            v_total_income := r_income.total_inc;
        END LOOP;

        DBMS_OUTPUT.PUT_LINE('Total Income: $' || v_total_income);
        DBMS_OUTPUT.PUT_LINE('----------------------------------------');
        DBMS_OUTPUT.PUT_LINE(RPAD('Category', 18) || RPAD('Budget', 10)
            || RPAD('Spent', 10) || RPAD('Left', 10) || 'Status');
        DBMS_OUTPUT.PUT_LINE('----------------------------------------');

        -- Explicit cursor: OPEN / FETCH / EXIT WHEN / CLOSE
        OPEN c_budget;
        LOOP
            FETCH c_budget INTO v_cat_name, v_limit, v_spent;
            EXIT WHEN c_budget%NOTFOUND;

            v_remaining := v_limit - v_spent;
            v_total_spent := v_total_spent + v_spent;

            -- Determine status using IF/ELSIF
            IF v_spent > v_limit THEN
                v_status := 'OVER BUDGET';
            ELSIF v_spent >= v_limit * 0.9 THEN
                v_status := 'WARNING';
            ELSIF v_spent >= v_limit * 0.5 THEN
                v_status := 'On Track';
            ELSE
                v_status := 'Under';
            END IF;

            DBMS_OUTPUT.PUT_LINE(
                RPAD(v_cat_name, 18)
                || RPAD('$' || v_limit, 10)
                || RPAD('$' || v_spent, 10)
                || RPAD('$' || v_remaining, 10)
                || v_status
            );
        END LOOP;
        CLOSE c_budget;

        DBMS_OUTPUT.PUT_LINE('----------------------------------------');
        DBMS_OUTPUT.PUT_LINE('Total Spent: $' || v_total_spent
            || '  |  Net: $' || (v_total_income - v_total_spent));

        -- Call the budget health score function
        v_health := budget_health_score(p_user_id, p_month, p_year);
        DBMS_OUTPUT.PUT_LINE('Budget Health: ' || v_health);
        DBMS_OUTPUT.PUT_LINE('========================================');

    END monthly_budget_report;


    -- ===========================================================
    -- FUNCTION: budget_health_score
    -- Calculates a letter grade based on budget adherence.
    -- Like the Grading() function from our labs.
    -- Features: Cursor FOR loop, CASE statement, aggregation
    -- ===========================================================
    FUNCTION budget_health_score(
        p_user_id bb_users.user_id%TYPE,
        p_month   NUMBER,
        p_year    NUMBER
    ) RETURN VARCHAR2 IS
        v_total_budget   NUMBER := 0;
        v_total_spent    NUMBER := 0;
        v_pct_used       NUMBER;
        v_over_count     NUMBER := 0;
        v_cat_count      NUMBER := 0;
    BEGIN
        -- Cursor FOR loop to calculate totals
        FOR r IN (
            SELECT c.monthly_limit,
                   NVL(SUM(t.amount), 0) AS spent
            FROM bb_categories c
            LEFT JOIN bb_transactions t
                ON c.category_id = t.category_id
                AND EXTRACT(MONTH FROM t.txn_date) = p_month
                AND EXTRACT(YEAR FROM t.txn_date) = p_year
            WHERE c.user_id = p_user_id
              AND c.is_income = 'N'
              AND c.monthly_limit > 0
            GROUP BY c.category_id, c.monthly_limit
        ) LOOP
            v_total_budget := v_total_budget + r.monthly_limit;
            v_total_spent := v_total_spent + r.spent;
            v_cat_count := v_cat_count + 1;

            IF r.spent > r.monthly_limit THEN
                v_over_count := v_over_count + 1;
            END IF;
        END LOOP;

        IF v_total_budget = 0 THEN
            RETURN 'N/A — No budgets set';
        END IF;

        v_pct_used := ROUND((v_total_spent / v_total_budget) * 100, 1);

        -- Grade using CASE (like Grading() function)
        RETURN CASE
            WHEN v_pct_used <= 70 AND v_over_count = 0 THEN
                'A+ (' || v_pct_used || '% used, all categories under budget)'
            WHEN v_pct_used <= 85 AND v_over_count = 0 THEN
                'A (' || v_pct_used || '% used, all categories under budget)'
            WHEN v_pct_used <= 95 AND v_over_count <= 1 THEN
                'B (' || v_pct_used || '% used, ' || v_over_count || ' over)'
            WHEN v_pct_used <= 100 THEN
                'C (' || v_pct_used || '% used, ' || v_over_count || ' over)'
            WHEN v_pct_used <= 115 THEN
                'D (' || v_pct_used || '% used — over budget!)'
            ELSE
                'F (' || v_pct_used || '% used — significantly over budget!)'
        END;

    END budget_health_score;


    -- ===========================================================
    -- FUNCTION: get_net_worth
    -- Sums all account balances for a user.
    -- Features: Cursor FOR loop, account type classification
    -- ===========================================================
    FUNCTION get_net_worth(
        p_user_id bb_users.user_id%TYPE
    ) RETURN NUMBER IS
        v_net_worth    NUMBER := 0;
        v_assets       NUMBER := 0;
        v_liabilities  NUMBER := 0;
        v_count        NUMBER;
    BEGIN
        SELECT COUNT(*) INTO v_count
        FROM bb_users WHERE user_id = p_user_id;

        IF v_count = 0 THEN
            RAISE_APPLICATION_ERROR(-20030, 'User ' || p_user_id || ' not found.');
        END IF;

        FOR r_acct IN (
            SELECT account_name, account_type, balance
            FROM bb_accounts
            WHERE user_id = p_user_id AND is_active = 'Y'
            ORDER BY account_type
        ) LOOP
            IF r_acct.account_type IN ('credit_card', 'loan') THEN
                v_liabilities := v_liabilities + ABS(r_acct.balance);
            ELSE
                v_assets := v_assets + r_acct.balance;
            END IF;
        END LOOP;

        v_net_worth := v_assets - v_liabilities;

        DBMS_OUTPUT.PUT_LINE('--- Net Worth for User ' || p_user_id || ' ---');
        DBMS_OUTPUT.PUT_LINE('Assets:      $' || v_assets);
        DBMS_OUTPUT.PUT_LINE('Liabilities: $' || v_liabilities);
        DBMS_OUTPUT.PUT_LINE('Net Worth:   $' || v_net_worth);

        RETURN v_net_worth;

    END get_net_worth;


    -- ===========================================================
    -- FUNCTION: auto_categorize
    -- Matches a merchant name against categorization rules.
    -- Features: Explicit cursor, UPPER() matching, priority ordering
    -- ===========================================================
    FUNCTION auto_categorize(
        p_user_id  bb_users.user_id%TYPE,
        p_merchant bb_transactions.merchant%TYPE
    ) RETURN NUMBER IS
        CURSOR c_rules IS
            SELECT category_id, pattern
            FROM bb_rules
            WHERE user_id = p_user_id
            ORDER BY priority ASC;

        v_category_id bb_rules.category_id%TYPE;
        v_pattern     bb_rules.pattern%TYPE;
    BEGIN
        IF p_merchant IS NULL THEN
            RETURN NULL;
        END IF;

        OPEN c_rules;
        LOOP
            FETCH c_rules INTO v_category_id, v_pattern;
            EXIT WHEN c_rules%NOTFOUND;

            IF UPPER(p_merchant) LIKE '%' || UPPER(v_pattern) || '%' THEN
                CLOSE c_rules;
                RETURN v_category_id;
            END IF;
        END LOOP;
        CLOSE c_rules;

        RETURN NULL;  -- No matching rule found

    END auto_categorize;

END bamboo_budget_pkg;
/
```

---

## Test Script (`test_script.sql`)

```sql
-- ============================================================
-- BambooBudget Test Script
-- Run after schema.sql, package_spec.sql, package_body.sql
-- ============================================================
SET SERVEROUTPUT ON;

PROMPT ============================================
PROMPT TEST 1: Auto-Categorize Function
PROMPT ============================================
DECLARE
    v_cat NUMBER;
BEGIN
    v_cat := bamboo_budget_pkg.auto_categorize(1, 'Trader Joes Market');
    DBMS_OUTPUT.PUT_LINE('Trader Joes -> Category: ' || NVL(TO_CHAR(v_cat), 'NULL'));

    v_cat := bamboo_budget_pkg.auto_categorize(1, 'SHELL GAS #4521');
    DBMS_OUTPUT.PUT_LINE('Shell Gas   -> Category: ' || NVL(TO_CHAR(v_cat), 'NULL'));

    v_cat := bamboo_budget_pkg.auto_categorize(1, 'Random Store');
    DBMS_OUTPUT.PUT_LINE('Random Store -> Category: ' || NVL(TO_CHAR(v_cat), 'NULL'));
END;
/

PROMPT ============================================
PROMPT TEST 2: Record Transaction (with auto-categorize)
PROMPT ============================================
BEGIN
    -- This should auto-categorize to Groceries (category 11)
    bamboo_budget_pkg.record_transaction(
        p_account_id  => 101,
        p_category_id => NULL,  -- let auto-categorize decide
        p_txn_date    => DATE '2026-03-01',
        p_merchant    => 'Whole Foods Market #123',
        p_amount      => 67.30
    );
END;
/

PROMPT ============================================
PROMPT TEST 3: Record Transaction (explicit category)
PROMPT ============================================
BEGIN
    bamboo_budget_pkg.record_transaction(
        p_account_id  => 103,
        p_category_id => 14,
        p_txn_date    => DATE '2026-03-02',
        p_merchant    => 'AMC Theatres',
        p_amount      => 24.50,
        p_notes       => 'Movie night'
    );
END;
/

PROMPT ============================================
PROMPT TEST 4: Record Transaction — ERROR (invalid account)
PROMPT ============================================
BEGIN
    bamboo_budget_pkg.record_transaction(
        p_account_id  => 999,
        p_category_id => 11,
        p_txn_date    => SYSDATE,
        p_merchant    => 'Test',
        p_amount      => 10.00
    );
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Caught expected error: ' || SQLERRM);
END;
/

PROMPT ============================================
PROMPT TEST 5: Transfer Funds
PROMPT ============================================
BEGIN
    bamboo_budget_pkg.transfer_funds(
        p_from_account => 101,
        p_to_account   => 102,
        p_amount       => 500.00
    );
END;
/

PROMPT ============================================
PROMPT TEST 6: Transfer — ERROR (different users)
PROMPT ============================================
BEGIN
    bamboo_budget_pkg.transfer_funds(
        p_from_account => 101,  -- Maria
        p_to_account   => 201,  -- James
        p_amount       => 100.00
    );
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Caught expected error: ' || SQLERRM);
END;
/

PROMPT ============================================
PROMPT TEST 7: Contribute to Goal
PROMPT ============================================
BEGIN
    -- Emergency Fund (80% funded, should show "Almost there!")
    bamboo_budget_pkg.contribute_to_goal(100, 1500.00);

    -- Vacation fund
    bamboo_budget_pkg.contribute_to_goal(101, 500.00);
END;
/

PROMPT ============================================
PROMPT TEST 8: Contribute — ERROR (invalid goal)
PROMPT ============================================
BEGIN
    bamboo_budget_pkg.contribute_to_goal(9999, 100.00);
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Caught expected error: ' || SQLERRM);
END;
/

PROMPT ============================================
PROMPT TEST 9: Net Worth Calculation
PROMPT ============================================
DECLARE
    v_nw NUMBER;
BEGIN
    v_nw := bamboo_budget_pkg.get_net_worth(1);
    DBMS_OUTPUT.PUT_LINE('Maria net worth result: $' || v_nw);
END;
/

PROMPT ============================================
PROMPT TEST 10: Monthly Budget Report (Feb 2026)
PROMPT ============================================
BEGIN
    bamboo_budget_pkg.monthly_budget_report(1, 2, 2026);
END;
/

PROMPT ============================================
PROMPT TEST 11: Budget Health Score
PROMPT ============================================
DECLARE
    v_score VARCHAR2(200);
BEGIN
    v_score := bamboo_budget_pkg.budget_health_score(1, 2, 2026);
    DBMS_OUTPUT.PUT_LINE('Maria Feb 2026 health: ' || v_score);

    v_score := bamboo_budget_pkg.budget_health_score(2, 2, 2026);
    DBMS_OUTPUT.PUT_LINE('James Feb 2026 health: ' || v_score);
END;
/

PROMPT ============================================
PROMPT ALL TESTS COMPLETE
PROMPT ============================================
```

---

## Feature Checklist

How this sample maps to every Phase 1 requirement:

| Requirement | Where It Appears |
|-------------|-----------------|
| **3+ Procedures** | `record_transaction`, `transfer_funds`, `contribute_to_goal`, `monthly_budget_report` (4 total) |
| **3+ Functions** | `budget_health_score`, `get_net_worth`, `auto_categorize` (3 total) |
| **Explicit Cursors** | `monthly_budget_report` (c_budget), `auto_categorize` (c_rules) — both use OPEN/FETCH/EXIT WHEN/CLOSE |
| **FOR Loops** | `budget_health_score` (cursor FOR loop), `monthly_budget_report` (FOR loop for income), `get_net_worth` (cursor FOR loop) |
| **Exception Handling** | `NO_DATA_FOUND` in `transfer_funds` and `contribute_to_goal`; `RAISE_APPLICATION_ERROR` throughout; `WHEN OTHERS` with ROLLBACK |
| **%TYPE** | Every parameter and local variable uses `%TYPE` |
| **COUNT(*) checks** | `record_transaction` (account + category validation), `monthly_budget_report` (user check), `get_net_worth` (user check) |
| **Conditional Logic** | `IF/ELSIF/ELSE` in budget report status, `CASE` in health score grading |
| **DBMS_OUTPUT** | Diagnostic output in every procedure/function |
| **RAISE_APPLICATION_ERROR** | Custom errors -20001 through -20030 |

---

## Sample Output

When you run the test script, you should see output like:

```
============================================
TEST 1: Auto-Categorize Function
============================================
Trader Joes -> Category: 11
Shell Gas   -> Category: 15
Random Store -> Category: NULL

============================================
TEST 2: Record Transaction (with auto-categorize)
============================================
Auto-categorized "Whole Foods Market #123" to category 11
Transaction #1013 recorded: Whole Foods Market #123 $67.3

============================================
TEST 5: Transfer Funds
============================================
Transferred $500 from account 101 to account 102

============================================
TEST 6: Transfer — ERROR (different users)
============================================
Caught expected error: ORA-20007: Cannot transfer between accounts of different users.

============================================
TEST 10: Monthly Budget Report (Feb 2026)
============================================
========================================
  MONTHLY BUDGET REPORT
  User: 1 | 2/2026
========================================
Total Income: $10400
----------------------------------------
Category          Budget    Spent     Left      Status
----------------------------------------
Dining Out        $300      $83.8     $216.2    Under
Entertainment     $150      $26.98    $123.02   Under
Groceries         $600      $426.5    $173.5    On Track
Rent              $2000     $2000     $0        WARNING
Transportation    $200      $48       $152      Under
Utilities         $250      $142.3    $107.7    On Track
----------------------------------------
Total Spent: $2727.58  |  Net: $7672.42
Budget Health: A (77.9% used, all categories under budget)
========================================
```

---

*This sample demonstrates every required PL/SQL feature in a real-world personal finance context. Use it as your benchmark — your project should be at this level of quality and completeness.*

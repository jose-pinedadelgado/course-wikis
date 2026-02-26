# Lab Exercises

PacoDogShop is used for PL/SQL labs in IS480 Advanced Database Management.

## Available Files

| File | Purpose |
|---|---|
| `Lab_PLSQL_Solutions.sql` | Complete lab solutions with PL/SQL procedures, functions, cursors |
| `Reset_Class_Lab_CurrentSchema.sql` | Drops all objects for a clean reset |
| `Student_Schema_Reset_Template.sql` | Student-specific setup script |

## Lab Topics

The PacoDogShop schema supports labs covering:

### Basic SQL

- SELECT with JOINs (product + category, order + customer)
- GROUP BY with aggregates (average ratings, order totals)
- Subqueries (highest-rated products, customers with most orders)

### Advanced SQL

- Self-joins (category hierarchy traversal)
- EXISTS / NOT EXISTS (customers without orders)
- Window functions (running totals, rankings)

### PL/SQL

- Stored procedures (process order, update inventory)
- Functions (calculate discount, get customer spend)
- Cursors (iterate through orders, generate reports)
- Triggers (auto-update timestamps, validate stock)
- Exception handling (insufficient stock, invalid payment)

## Sample Exercise

??? example "Exercise: Find Products Never Ordered"
    ```sql
    -- Find all products that have never been ordered
    SELECT p.name, p.base_price
    FROM Product p
    WHERE p.product_id NOT IN (
        SELECT DISTINCT pv.product_id
        FROM Product_Variant pv
        JOIN Order_Item oi ON pv.variant_id = oi.variant_id
    );
    ```

??? example "Exercise: PL/SQL Cursor — Monthly Sales Report"
    ```sql
    DECLARE
        CURSOR c_monthly IS
            SELECT EXTRACT(MONTH FROM o.order_date) AS month,
                   SUM(o.total) AS total_sales,
                   COUNT(*) AS order_count
            FROM `Order` o
            WHERE o.status = 'delivered'
            GROUP BY EXTRACT(MONTH FROM o.order_date)
            ORDER BY month;
    BEGIN
        FOR rec IN c_monthly LOOP
            DBMS_OUTPUT.PUT_LINE('Month ' || rec.month ||
                ': $' || rec.total_sales ||
                ' (' || rec.order_count || ' orders)');
        END LOOP;
    END;
    ```

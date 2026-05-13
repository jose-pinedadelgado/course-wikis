# Advanced SQL

Advanced SQL is mostly about combining tables and comparing rows to other rows. Requests usually reveal the technique through their wording.

## Join Types At A Glance

| Join Type | What It Keeps | Use When |
|---|---|---|
| `INNER JOIN` | Only matching rows from both tables | Unmatched rows should disappear. |
| `LEFT JOIN` | All rows from the left table | You must keep all rows from the left side, even without matches. |
| `RIGHT JOIN` | All rows from the right table | You must keep all rows from the right side. |
| `FULL OUTER JOIN` | All rows from both tables | You care about unmatched rows on either side. |
| `NATURAL JOIN` | Rows matched automatically by same-named columns | You are asked specifically about natural join behavior. |
| Self join | One table joined to another copy of itself | Parent-child, referral, or hierarchy questions. |

## Join Videos

!!! tip "Video: INNER JOIN"
    <video controls preload="metadata" width="100%">
      <source src="../../assets/videos/sql_inner_join.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>


!!! tip "Video: LEFT JOIN"
    <video controls preload="metadata" width="100%">
      <source src="../../assets/videos/sql_left_join.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>


!!! tip "Video: RIGHT JOIN"
    <video controls preload="metadata" width="100%">
      <source src="../../assets/videos/sql_right_join.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>


!!! tip "Video: FULL OUTER JOIN"
    <video controls preload="metadata" width="100%">
      <source src="../../assets/videos/sql_full_outer_join.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>


## Subquery Types

| Type | How To Recognize It |
|---|---|
| Noncorrelated subquery | The inner query can run by itself. Example: above the company-wide average. |
| Correlated subquery | The inner query refers to the outer row. Example: above the average for that row's own group. |

!!! tip "Video: Noncorrelated Subquery"
    <video controls preload="metadata" width="100%">
      <source src="../../assets/videos/sql_subquery_noncorrelated.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>


!!! tip "Video: Correlated Subquery"
    <video controls preload="metadata" width="100%">
      <source src="../../assets/videos/sql_subquery_correlated.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>


## CASE Logic and Views

Use `CASE` when the output needs a label, category, or status. For this undergraduate guide, keep aggregate functions and `CASE` as separate skills unless a prompt explicitly combines them.

!!! tip "Video: CASE Logic"
    <video controls preload="metadata" width="100%">
      <source src="../../assets/videos/sql_case.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>


## Optional SQL Stretch Videos

These are useful, but not central to the current compact guide.

!!! tip "Video: CROSS JOIN"
    <video controls preload="metadata" width="100%">
      <source src="../../assets/videos/sql_cross_join.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>


!!! tip "Video: EXISTS and NOT EXISTS"
    <video controls preload="metadata" width="100%">
      <source src="../../assets/videos/sql_exists_not_exists.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>


## Compact Guide Section

# 2. Advanced SQL

## 2.1 Why Joins Matter

Relational databases split data across related tables. Joins reconnect those tables when a query needs information from more than one place.

Example relationship:

- `readings.device_id` is a foreign key.
- `devices.device_id` is the primary key.

Query:

```sql
SELECT
  r.reading_id,
  r.recorded_at,
  r.value,
  d.device_label
FROM readings r
INNER JOIN devices d
  ON r.device_id = d.device_id;
```

## 2.2 Inner Joins

An inner join keeps only rows that match on both sides.

Use it when unmatched rows should not appear.

```sql
SELECT
  l.log_id,
  l.service_date,
  d.device_label
FROM maintenance_logs l
INNER JOIN devices d
  ON l.device_id = d.device_id;
```

Recognition clue: "Show each maintenance log with its device" usually implies an inner join if every log should have a valid device.

## 2.3 Outer Joins

An outer join keeps unmatched rows from one or both sides.

### LEFT JOIN

A left join keeps every row from the left table and fills in `NULL` for missing matches from the right table.

```sql
SELECT
  d.device_id,
  d.device_label,
  r.reading_id
FROM devices d
LEFT JOIN readings r
  ON d.device_id = r.device_id;
```

This keeps every device, even devices with no readings. For those devices, the reading columns appear as `NULL`.

### RIGHT JOIN

A right join keeps every row from the right table and fills in `NULL` for missing matches from the left table.

```sql
SELECT
  room.room_id,
  room.room_name,
  d.device_id,
  d.device_label
FROM devices d
RIGHT JOIN rooms room
  ON d.room_id = room.room_id;
```

This keeps every room, even rooms with no devices. A right join can usually be rewritten as a left join by switching the table order.

### FULL OUTER JOIN

A full outer join keeps all rows from both tables. Matching rows are combined; unmatched rows from either side still appear with `NULL` values for the missing side.

```sql
SELECT
  p.asset_tag AS planned_asset,
  s.asset_tag AS scanned_asset
FROM planned_assets p
FULL OUTER JOIN scanned_assets s
  ON p.asset_tag = s.asset_tag;
```

Use a full outer join when unmatched rows from both tables matter, such as comparing two lists and finding records that exist on only one side. Some database systems do not support `FULL OUTER JOIN` directly, but the concept is still important.

Recognition clues:

- "Include all devices, even if they have no readings."
- "List all rooms, even rooms with zero devices."
- "Find records that do not have a match."
- "Include unmatched rows from both tables."

## 2.4 Natural Joins

A natural join automatically joins tables using every column name they have in common. It does not use an explicit `ON` clause.

```sql
SELECT
  device_id,
  device_label,
  room_name
FROM device_assignments
NATURAL JOIN device_locations;
```

Natural joins can be convenient, but they can also be risky. If two tables share more column names than expected, the database will use all of those shared columns as join conditions. In most real projects, an explicit `JOIN ... ON ...` is easier to read and safer.

## 2.5 Multi-Table Joins

For each relationship in the path, include a join condition.

```sql
SELECT
  d.device_label,
  r.recorded_at,
  s.sensor_name,
  r.value
FROM devices d
INNER JOIN readings r
  ON d.device_id = r.device_id
INNER JOIN sensors s
  ON r.sensor_id = s.sensor_id;
```

Mental checklist:

1. What output columns do I need?
2. Which tables contain those columns?
3. How are those tables connected?
4. Should unmatched rows be kept?
5. What is the expected grain of the result: one row per device, reading, sensor, or group?

## 2.6 Self Joins

A self join uses the same table twice. Each copy needs a different alias.

Example: product categories and their parent categories.

```sql
SELECT
  c.category_id,
  c.category_name,
  parent.category_name AS parent_category_name
FROM categories c
LEFT JOIN categories parent
  ON c.parent_category_id = parent.category_id;
```

Use `LEFT JOIN` if top-level categories without a parent should still appear.

Recognition clues:

- "Which rows point to another row in the same table?"
- "Categories and their parent categories."
- "Products and their parent products."
- "People and their referrals."

## 2.7 Noncorrelated Subqueries

A noncorrelated subquery can run by itself. Its result is used by the outer query.

Example: products priced above the overall product average.

```sql
SELECT product_id, product_name, price
FROM products
WHERE price > (
  SELECT AVG(price)
  FROM products
);
```

The inner query does not refer to the outer query. It runs once.

Recognition clues:

- "Greater than the overall average."
- "In the list of devices that have readings."
- "Equal to the maximum/minimum value."

## 2.8 Correlated Subqueries

A correlated subquery refers to the outer query. It is evaluated for each row from the outer query.

Example: products priced above the average price in their own category.

```sql
SELECT
  p.product_id,
  p.product_name,
  p.category_id,
  p.price
FROM products p
WHERE p.price > (
  SELECT AVG(p2.price)
  FROM products p2
  WHERE p2.category_id = p.category_id
);
```

The inner query uses `p.category_id` from the outer query, so it cannot run independently.

Recognition clues:

- "Compared to its own group."
- "For each device..."
- "For each product..."
- "Where no matching row exists for this same outer row."

## 2.9 CASE Logic

`CASE` creates conditional output.

```sql
SELECT
  ticket_id,
  priority_score,
  CASE
    WHEN priority_score >= 8 THEN 'High'
    WHEN priority_score >= 4 THEN 'Medium'
    ELSE 'Low'
  END AS priority_label
FROM support_tickets;
```

Use `CASE` when the prompt asks for categories, labels, statuses, or if-then logic.

Examples:

- `High`, `Medium`, `Low`
- `Open` vs `Closed`
- `High` vs `Medium` vs `Low`
- `Needs Review` vs `Ready`

For this guide, keep `CASE` focused on readable labels and categories. Conditional aggregation such as `SUM(CASE WHEN ... THEN ... END)` is beyond the expected undergraduate level here.

## 2.10 Views

A view is a saved query that can be used like a table.

```sql
CREATE VIEW category_parent_view AS
SELECT
  c.category_id,
  c.category_name,
  parent.category_name AS parent_category_name
FROM categories c
LEFT JOIN categories parent
  ON c.parent_category_id = parent.category_id;
```

Then query the view:

```sql
SELECT category_id, category_name, parent_category_name
FROM category_parent_view
WHERE parent_category_name IS NOT NULL;
```

Views are useful when:

- A query is reused often.
- A complex join should be simplified for users.
- Users should see only certain columns or rows.

## 2.11 Advanced SQL Recognition Table

| Prompt Language | Likely Technique |
|---|---|
| "Show rows and their parent rows from the same table" | Self join |
| "Keep unmatched rows from one table" | Outer join |
| "Keep all rows from the right-hand table" | `RIGHT JOIN` |
| "Keep unmatched rows from both tables" | `FULL OUTER JOIN` |
| "Join automatically by same-named columns" | `NATURAL JOIN` |
| "Rows above the average for their own group" | Correlated subquery |
| "Rows above one overall average" | Noncorrelated subquery |
| "Classify each row with a readable label" | `CASE` |
| "Save this query for later reuse" | View |
| "One row per category" | `GROUP BY` |
| "Only groups with more than N rows" | `HAVING` |

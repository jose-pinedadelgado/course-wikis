# Full IS380 Exam Study Guide

## How to Use This Guide

This guide is meant to help you review the major database topics covered after the modeling and normalization portion of IS380. Focus on being able to explain the idea, read examples, and write small SQL or SQL-like answers by hand.

For query-writing topics, minor syntax differences between database systems are less important than showing the correct logic: the right tables, the right join conditions, the right filtering level, clear aliases, and readable output columns.

## High-Level Exam Map

| Area | What You Should Be Able To Do |
|---|---|
| SQL foundations | Write single-table queries with filtering, sorting, calculations, grouping, and aggregate functions such as `COUNT`, `SUM`, and `AVG`. |
| Advanced SQL | Join multiple tables, use inner joins, left/right/full outer joins, natural joins, self joins, subqueries, `CASE`, and simple views. |
| Physical design | Explain denormalization, indexes, sequential access, hashed access, and the read/write tradeoffs of indexes. |
| Big Data | Explain the Five V's and schema-on-read vs schema-on-write at a conceptual level. |
| NoSQL | Compare key-value, document, wide-column, and graph databases; choose when relational or NoSQL is a better fit. |
| DynamoDB | Explain tables, items, attributes, partition keys, sort keys, composite keys, Query vs Scan, and read consistency. |
| Neo4j/Cypher | Explain graph databases and read basic Cypher queries using nodes, relationships, labels, properties, `MATCH`, `WHERE`, and `RETURN`. |

## Visualization Map

These are the places where a short animation would help most.

| Guide Section | Suggested Video | Student Misunderstanding It Helps Fix |
|---|---|---|
| SQL foundations | `SELECT-FROM-WHERE` pipeline | Students often write the query in one order but forget the DBMS logically processes `FROM` and `WHERE` before `SELECT`. |
| SQL foundations | `GROUP BY` vs `HAVING` | Students confuse filtering individual rows with filtering groups. |
| SQL foundations / joins | Primary key and foreign key | Students memorize join syntax without understanding why rows connect. |
| Advanced SQL | Self join | Students struggle with using one table twice with different aliases. |
| Advanced SQL | Views as reusable queries | Students need a mental model for a view as a named query result. |
| Physical design | Sequential scan vs index lookup | Students need to see why indexes reduce search work. |
| Physical design | Denormalization tradeoff | Students need to see why faster reads can create duplicated data and update risk. |
| NoSQL | Relational rows vs document model | Students need to see the difference between normalized tables and nested documents. |
| DynamoDB | Partition key plus sort key | Students need to see composite keys and sorted item collections. |
| DynamoDB | Query vs Scan | Students need to see why a targeted key lookup is better than reading everything. |
| Neo4j | Cypher pattern matching | Students need to see that a Cypher pattern is a graph shape to match. |

---

# 1. SQL Foundations

## 1.1 What SQL Is For

SQL is the language used to define, change, and query data in relational databases.

Common categories:

| Category | Purpose | Examples |
|---|---|---|
| DDL: Data Definition Language | Define database structures | `CREATE TABLE`, `ALTER TABLE`, `CREATE VIEW` |
| DML: Data Manipulation Language | Work with stored data | `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| DCL: Data Control Language | Control access | `GRANT`, `REVOKE` |

For this guide, the most important statement is `SELECT`.

## 1.2 Basic SELECT Syntax Map

Typical written order:

```sql
SELECT column_list
FROM table_name
WHERE row_condition
GROUP BY grouping_column
HAVING group_condition
ORDER BY sort_column;
```

Logical processing order:

1. `FROM`: identify the table or joined tables.
2. `WHERE`: filter individual rows.
3. `GROUP BY`: group rows into categories.
4. `HAVING`: filter groups.
5. `SELECT`: choose and calculate output columns.
6. `ORDER BY`: sort the final result.

This is why a column alias created in `SELECT` usually cannot be used in `WHERE`: the `WHERE` step happens earlier.

## 1.3 Filtering Rows With WHERE

Use `WHERE` when you want to keep or remove individual rows before grouping.

```sql
SELECT product_id, product_name, price
FROM products
WHERE price >= 100;
```

Boolean logic:

```sql
SELECT product_id, product_name, price
FROM products
WHERE price >= 100
  AND category_id = 10;
```

Parentheses matter:

```sql
SELECT product_id, product_name, price, category_id
FROM products
WHERE category_id = 10
  AND (price >= 100 OR price IS NULL);
```

## 1.4 Calculated Columns and Aliases

Calculated columns are created in the `SELECT` clause.

```sql
SELECT
  product_id,
  product_name,
  price,
  price * 1.10 AS price_with_markup
FROM products;
```

Aliases make output columns readable. They also make complex queries easier to grade and understand.

## 1.5 Aggregate Functions

Common aggregate functions:

| Function | Meaning |
|---|---|
| `COUNT(*)` | Count rows. |
| `COUNT(column)` | Count non-null values in a column. |
| `SUM(column)` | Add values. |
| `AVG(column)` | Average values. |
| `MIN(column)` | Smallest value. |
| `MAX(column)` | Largest value. |

Example:

```sql
SELECT
  category_id,
  COUNT(*) AS product_count,
  AVG(price) AS average_price
FROM products
GROUP BY category_id;
```

Example with `SUM`:

```sql
SELECT
  category_id,
  SUM(quantity_on_hand) AS total_quantity
FROM products
GROUP BY category_id;
```

Rule of thumb: if you use aggregate functions and also select normal columns, the normal columns usually need to appear in `GROUP BY`.

For this guide, practice plain aggregate functions and plain `CASE` logic as separate skills. You are not expected to combine `SUM` and `CASE` in one expression unless a prompt explicitly asks for that.

## 1.6 WHERE vs HAVING

Use `WHERE` to filter rows before grouping.

Use `HAVING` to filter groups after aggregation.

Example:

```sql
SELECT
  category_id,
  COUNT(*) AS product_count,
  AVG(price) AS average_price
FROM products
WHERE price IS NOT NULL
GROUP BY category_id
HAVING COUNT(*) >= 3;
```

This means:

1. Remove products with missing price.
2. Group the remaining rows by category.
3. Keep only categories with at least 3 products.

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

- "Which row points to another row in the same table?"
- "Categories and their parent categories."
- "Topics and their parent topics."
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

# 3. Physical Design

Physical design is about how the database is actually stored and accessed. Logical design asks, "What tables and relationships should exist?" Physical design asks, "How can the DBMS store and retrieve this efficiently?"

## 3.1 Denormalization

Denormalization means intentionally combining or duplicating data from normalized tables to improve performance for common reads.

Normalized design example:

- `devices(device_id, device_label, room_id)`
- `rooms(room_id, room_name, building)`

Denormalized reporting design example:

- `device_room_report(device_id, device_label, room_name, building)`

Benefit:

- Reports may run faster because fewer joins are needed.

Risk:

- Room data is duplicated across many rows.
- If room name or building changes, every duplicated copy must be updated correctly.

Use denormalization carefully. First normalize to get a sound structure, then denormalize only when there is a clear performance reason.

## 3.2 Sequential Access

Sequential access means the DBMS reads through rows one by one.

This can be acceptable when:

- The table is small.
- Most rows are needed.
- There is no useful index.

It can be expensive when:

- The table is large.
- Only a few rows are needed.
- The query is frequent.

## 3.3 Indexed Access

An index is like a lookup structure that helps the DBMS find rows faster.

Indexes are often useful on:

- Primary keys.
- Foreign keys used in joins.
- Columns often used in `WHERE`.
- Columns often used in `ORDER BY`.
- Columns often used in `GROUP BY`.

Tradeoff:

- Reads can be faster.
- Inserts, updates, and deletes can be slower because the index must also be maintained.
- Indexes take extra storage.

Do not index every column. Indexes are most useful when the column has enough variety and the query returns a relatively small portion of the table.

## 3.4 Hashed Access

Hashed access uses a hashing algorithm to map a key value to a storage location.

Example idea:

```text
device_id = D100 -> hash function -> storage location
```

This can make exact-key lookup very fast. It is less useful for range questions such as "all devices between ID D100 and D200" unless the DBMS has additional structures.

# 4. Big Data and NoSQL

## 4.1 Big Data

Big Data refers to data that is too large, fast, varied, or messy for traditional tools to handle easily.

The Five V's:

| V | Meaning | Example |
|---|---|---|
| Volume | Very large amount of data | Billions of clickstream records |
| Velocity | Data arrives quickly | Sensor readings every second |
| Variety | Many formats | Tables, JSON, images, logs |
| Veracity | Data quality uncertainty | Duplicate, incomplete, or noisy data |
| Value | Data must support a useful goal | Predicting demand or detecting fraud |

## 4.2 Schema-on-Write vs Schema-on-Read

Schema-on-write:

- Structure is defined before data is stored.
- Typical relational database approach.
- Good when data is well understood and consistency is important.

Schema-on-read:

- Data is stored first.
- Structure is interpreted later when reading or analyzing it.
- Common with semi-structured formats such as JSON or XML.

## 4.3 NoSQL Overview

NoSQL means "Not Only SQL." It refers to database systems that are not based mainly on the relational model.

Common NoSQL motivations:

- Scale out across many machines.
- Handle flexible or changing data structures.
- Work naturally with cloud environments.
- Support schema-on-read or semi-structured data.

Important tradeoff:

- Traditional relational systems emphasize ACID consistency.
- Many NoSQL systems emphasize BASE: basically available, soft state, eventually consistent.

## 4.4 NoSQL Categories

| Type | Data Model | Good For | Example |
|---|---|---|---|
| Key-value | Key points to a value | Fast lookup by key | Redis, DynamoDB style access |
| Document | Key points to structured document | JSON-like records with flexible fields | MongoDB |
| Wide-column | Rows with column families | Large distributed datasets | Cassandra, HBase |
| Graph | Nodes and relationships | Relationship-heavy questions | Neo4j |

## 4.5 Relational vs NoSQL Decision Examples

Choose relational when:

- Data is highly structured.
- Relationships and constraints are stable.
- Transactions and consistency are central.
- SQL reporting is important.

Choose NoSQL when:

- The data structure changes often.
- The system needs massive horizontal scale.
- The main access pattern is key lookup or graph traversal.
- Semi-structured documents are natural for the problem.

Examples:

| Scenario | Better Fit | Why |
|---|---|---|
| Payroll system | Relational | Strong consistency and structured records matter. |
| Product catalog with flexible attributes | Document database | Products may have different fields. |
| User session cache | Key-value database | Fast lookup by session ID. |
| Friend-of-friend recommendations | Graph database | Connections are the main question. |
| IoT sensor stream | Big data / NoSQL tools | Velocity and volume may overwhelm one relational server. |

# 5. DynamoDB

## 5.1 What DynamoDB Is

DynamoDB is a fully managed AWS NoSQL database service. It is designed for fast key-based access at scale without the user managing database servers.

Core terms:

| DynamoDB Term | Meaning |
|---|---|
| Table | A collection of items. |
| Item | A single record in a table. Similar to a row, but more flexible. |
| Attribute | A data value inside an item. Similar to a column, but items do not all need the same attributes. |
| Primary key | The value or values that uniquely identify an item. |
| Partition key | The hash key used to organize and locate items. |
| Sort key | Optional second key that stores related items in sorted order under the same partition key. |

## 5.2 Simple Primary Key

A table with only a partition key has a simple primary key.

Example:

```text
Table: People
Primary key: person_id
```

Each `person_id` must be unique.

## 5.3 Composite Primary Key

A table with a partition key and sort key has a composite primary key.

Example:

```text
Table: Music
Partition key: Artist
Sort key: SongTitle
```

This allows multiple songs by the same artist:

| Artist | SongTitle | Year |
|---|---|---|
| Drake | Best I Ever Had | 2010 |
| Drake | God's Plan | 2018 |
| Beyonce | Halo | 2008 |

The pair `(Artist, SongTitle)` must be unique. The partition key alone does not need to be unique when a sort key exists.

## 5.4 Query vs Scan

`Query`:

- Finds items using the primary key.
- Requires a partition key value.
- Can optionally use the sort key.
- Efficient because it uses indexed access.

`Scan`:

- Reads every item in a table.
- Can apply filters after reading.
- Less efficient for large tables.

Rule of thumb: design your table so important application questions can use `Query`, not `Scan`.

## 5.5 Read Consistency

Eventually consistent read:

- Default for many DynamoDB read operations.
- May not immediately reflect a very recent successful write.
- Repeating the read shortly after should return the updated result.
- Uses less read capacity than strongly consistent reads.

Strongly consistent read:

- Returns the most up-to-date committed data for supported reads.
- Useful when the application cannot tolerate reading stale data.

## 5.6 DynamoDB Design Checklist

When designing a DynamoDB table, ask:

1. What questions must the application answer quickly?
2. What exact key values will be known at query time?
3. What should the partition key be?
4. Do related items need a sort key?
5. Can the main access patterns use `Query`?

# 6. Neo4j and Cypher

## 6.1 What Neo4j Is

Neo4j is a graph database. Instead of storing data primarily as tables, it stores data as nodes and relationships.

Graph databases are useful when the relationships between things are as important as the things themselves.

Examples:

- People connected to friends.
- Actors connected to movies.
- Bank accounts connected to transactions.
- Cities connected by roads.

## 6.2 Property Graph Concepts

| Concept | Meaning | Example |
|---|---|---|
| Node | An entity or object | A person, movie, company |
| Label | A category for a node | `Person`, `Movie` |
| Relationship | A directed connection between nodes | `ACTED_IN`, `DIRECTED`, `KNOWS` |
| Property | A key-value pair on a node or relationship | `name: 'Tom Hanks'`, `released: 1994` |

Example graph idea:

```text
(Person: Tom Hanks)-[:ACTED_IN]->(Movie: Forrest Gump)
(Person: Robert Zemeckis)-[:DIRECTED]->(Movie: Forrest Gump)
```

## 6.3 Reading Data With MATCH

`MATCH` describes the graph pattern Neo4j should find.

Find all people:

```cypher
MATCH (p:Person)
RETURN p;
```

Find movie titles:

```cypher
MATCH (m:Movie)
RETURN m.title;
```

Find actors and movies:

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p.name, m.title;
```

Find directors of movies after 2000:

```cypher
MATCH (p:Person)-[:DIRECTED]->(m:Movie)
WHERE m.released > 2000
RETURN p.name, m.title, m.released;
```

## 6.4 Understanding Cypher Patterns

Pattern:

```cypher
(actor:Person)-[:ACTED_IN]->(movie:Movie)<-[:DIRECTED]-(director:Person)
```

Read it as:

Find a `Person` node called `actor` who has an outgoing `ACTED_IN` relationship to a `Movie` node called `movie`, and that same movie has an incoming `DIRECTED` relationship from another `Person` node called `director`.

This is why graph databases are useful: the query is shaped like the relationship pattern you want to find.

## 6.5 Neo4j vs SQL Joins

In SQL, related data is reconnected with joins:

```sql
SELECT p.person_name, m.movie_title
FROM people p
INNER JOIN acted_in ai
  ON p.person_id = ai.person_id
INNER JOIN movies m
  ON ai.movie_id = m.movie_id;
```

In Cypher, the relationship is part of the pattern:

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p.name, m.title;
```

Both can represent relationships. The difference is that graph databases make relationship traversal the center of the model.

# 7. Final Review Checklist

Before the exam or quiz, you should be able to answer these without notes:

## SQL

- What is the difference between `WHERE` and `HAVING`?
- When do you need `GROUP BY`?
- What does an inner join exclude?
- What does a left join preserve?
- What do `RIGHT JOIN` and `FULL OUTER JOIN` preserve?
- What does `NATURAL JOIN` use as its join condition?
- Why do self joins require aliases?
- What makes a subquery correlated?
- When would you use `CASE`?
- What is a view?

## Physical Design

- Why might denormalization improve performance?
- What is the risk of denormalization?
- What does an index do?
- Why not index every column?
- What is the difference between sequential, indexed, and hashed access?

## Big Data and NoSQL

- What are the Five V's?
- What is schema-on-read?
- What are the major NoSQL categories?
- When is relational still the better choice?

## DynamoDB

- What is a table, item, and attribute?
- What is a partition key?
- What is a sort key?
- What is a composite primary key?
- What is the difference between `Query` and `Scan`?
- What is eventual consistency?

## Neo4j

- What are nodes, relationships, labels, and properties?
- What does `MATCH` do?
- What does `RETURN` do?
- How do you read a Cypher relationship pattern?

---

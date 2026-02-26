# Getting Started

## Prerequisites

- MySQL 8.0+ (or Oracle, with syntax adjustments)
- A SQL client (MySQL Workbench, DBeaver, or command line)

## Setup

### 1. Create the Schema

```sql
SOURCE schema.sql;
```

This creates 20 tables in dependency order (referenced tables first).

### 2. Load Seed Data

```sql
SOURCE seed_data.sql;
```

### 3. Verify

```sql
-- Check table counts
SELECT 'Category' AS tbl, COUNT(*) AS rows FROM Category
UNION ALL SELECT 'Product', COUNT(*) FROM Product
UNION ALL SELECT 'Product_Variant', COUNT(*) FROM Product_Variant
UNION ALL SELECT 'Customer', COUNT(*) FROM Customer
UNION ALL SELECT 'Order', COUNT(*) FROM `Order`
UNION ALL SELECT 'Review', COUNT(*) FROM Review;
```

## For Students

### Lab Setup

Use the student template to set up your own schema:

```sql
SOURCE Student_Schema_Reset_Template.sql;
```

### Reset Between Labs

If you need a clean start:

```sql
SOURCE Reset_Class_Lab_CurrentSchema.sql;
SOURCE schema.sql;
SOURCE seed_data.sql;
```

## ERD Diagrams

Open the `.drawio` files in [draw.io](https://app.diagrams.net/) (free, web-based):

- `pacodogshop_erd.drawio` — Conceptual ERD
- `pacodogshop_eerd.drawio` — Enhanced ERD (with specialization)
- `pacodogshop_relational.drawio` — Physical relational model

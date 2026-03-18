# Kimball Dimensional Modeling — Student Cheat Sheet

> Based on *The Data Warehouse Toolkit, 3rd Edition* by Ralph Kimball & Margy Ross.  
> Prepared for IS480 — Advanced Database Management, CSULB Spring 2026.

---

## The Big Idea

Organizations have **two different data needs** that require **two different systems**:

| | OLTP (Operational) | OLAP (Analytical / Data Warehouse) |
|--|--------------------|------------------------------------|
| **Purpose** | Run the business | Analyze the business |
| **Users** | Clerks, apps, front-line workers | Managers, analysts, executives |
| **Operations** | INSERT, UPDATE, DELETE (one row at a time) | SELECT with GROUP BY (millions of rows) |
| **Schema** | Normalized (3NF) — no redundancy | Denormalized (Star) — redundancy is OK |
| **Data** | Current state only | Historical snapshots over time |
| **Speed goal** | Fast writes | Fast reads & aggregations |

**Key insight:** You put data INTO operational systems. You get data OUT OF warehouses.

---

## The 4-Step Design Process

Every dimensional model follows these four steps (Kimball Ch 2-3):

### Step 1: Select the Business Process
- What **activity** generates measurable events? (e.g., sales, enrollments, appointments)
- Business processes are **verbs**: selling, ordering, registering, billing
- ❌ NOT a department ("marketing") — that's who *uses* the data, not what *generates* it

### Step 2: Declare the Grain
- **"What does one row in the fact table represent?"**
- Express in business terms: *"One row per product sold per transaction"*
- **Always start with the finest (atomic) grain** — you can always roll up, but can't drill down from summaries
- ⚠️ **Most common design mistake:** Skipping this step or mixing grains in one table

### Step 3: Identify the Dimensions
- Ask: **"Who, what, where, when, why, how?"** for each event
- Common dimensions: Date, Product, Customer, Employee, Location, Promotion
- Each dimension takes on **exactly one value** per fact row
- If a dimension would cause extra rows → wrong grain or wrong dimension

### Step 4: Identify the Facts
- Ask: **"What is the process measuring?"**
- Facts are **numeric values** you want to SUM, AVG, COUNT
- Must be **true to the grain** declared in Step 2
- If a fact belongs to a different grain → put it in a **separate fact table**

---

## Star Schema Anatomy

```
                    ┌──────────────┐
                    │  dim_date    │  ← DIMENSION: descriptive context
                    │──────────────│     (wide, denormalized, ~thousands of rows)
                    │ date_key  PK │
                    │ full_date    │
                    │ day_of_week  │
                    │ month, qtr   │
                    │ year         │
                    │ is_holiday   │
                    └──────┬───────┘
                           │
  ┌──────────────┐  ┌──────┴──────────────┐  ┌──────────────┐
  │ dim_product  │  │    fact_sales       │  │ dim_store    │
  │──────────────│  │─────────────────────│  │──────────────│
  │ product_key  ├──┤ date_key       FK   ├──┤ store_key    │
  │ product_name │  │ product_key    FK   │  │ store_name   │
  │ brand        │  │ store_key      FK   │  │ city, state  │
  │ category     │  │ promotion_key  FK   │  │ sq_footage   │
  │ department   │  │                     │  └──────────────┘
  │ package_type │  │ ── MEASURES ──      │
  └──────────────┘  │ sales_quantity      │  ← FACTS: numeric, additive
                    │ unit_price          │
                    │ extended_amount     │
                    │ extended_cost       │
                    │ gross_profit        │
                    └─────────────────────┘
                         ↑
               FACT TABLE: events/measurements
               (narrow, very tall — millions/billions of rows)
```

---

## Fact Table Rules

| Rule | Details |
|------|---------|
| **One row = one event** | Each row represents a real-world measurement event |
| **Foreign keys to dimensions** | One FK column per dimension |
| **Numeric measures** | SUM-able, AVG-able numbers |
| **Grain must be uniform** | Never mix different grains in the same table |
| **No text in fact tables** | Descriptive text belongs in dimensions |

### Three Types of Facts

| Type | Can SUM across... | Example |
|------|-------------------|---------|
| **Additive** | ALL dimensions | Revenue, quantity, cost |
| **Semi-additive** | Some dimensions (not time) | Account balance, inventory level |
| **Non-additive** | NO dimensions (compute from components) | Unit price, ratios, percentages |

> 💡 **Pro tip:** Store the additive components (numerator & denominator), then compute ratios in your query or BI tool. *"Ratio of the sums, not sum of the ratios."*

### Three Types of Fact Tables

| Type | What it captures | Row behavior |
|------|-----------------|--------------|
| **Transaction** | Individual events (a sale, a click) | Inserted once, never updated |
| **Periodic Snapshot** | State at regular intervals (daily balance) | One row per period, even if nothing happened |
| **Accumulating Snapshot** | Pipeline milestones (order → ship → deliver) | Row is updated as milestones are reached |

### Factless Fact Tables
- Fact table with **no numeric measures** — just foreign keys
- Use case 1: **Event tracking** — "Student attended class on this date" (no number, but the event matters)
- Use case 2: **Coverage analysis** — "What *didn't* happen?" (e.g., which products were on promotion but had zero sales?)

---

## Dimension Table Rules

| Rule | Details |
|------|---------|
| **Single primary key** | Use a **surrogate key** (integer, assigned sequentially) |
| **Wide and flat** | Many descriptive columns, denormalized |
| **Text-heavy** | Verbose descriptions, not codes |
| **Denormalized** | Flatten hierarchies into one table (e.g., product + category + department all in `dim_product`) |
| **Relatively small** | Thousands to low millions of rows |

### Why Surrogate Keys?

Don't use the business key (e.g., employee_id) as the dimension PK because:

1. Business keys **change** (employee rehired with new ID)
2. Multiple source systems may have **conflicting** keys
3. You need **multiple rows** for the same entity when tracking changes (SCD Type 2)
4. Surrogate keys are **simpler** and uniformly formatted

**Exception:** The `dim_date` table can use YYYYMMDD integer as its key (it's stable and meaningful).

---

## The Date Dimension

Every star schema needs one. Pre-populate it with 10-20 years of days (~7,300 rows for 20 years).

**Key columns:**

| Column | Example | Why? |
|--------|---------|------|
| date_key | 20260318 | PK (YYYYMMDD format) |
| full_date | 2026-03-18 | Actual DATE type |
| day_of_week | Wednesday | Compare Mon vs Sun sales |
| month_name | March | GROUP BY month |
| quarter | Q1 | Quarterly reports |
| year | 2026 | Year-over-year analysis |
| is_weekend | Y/N | Filter weekdays/weekends |
| is_holiday | Y/N | Holiday sales patterns |
| fiscal_quarter | FQ4 | If fiscal ≠ calendar year |

> 💡 **Why not just use a DATE column in the fact table?** Because you can't easily GROUP BY "is_holiday" or "fiscal_quarter" with a raw date. The dimension pre-computes these attributes.

---

## Slowly Changing Dimensions (SCD)

What happens when dimension data changes? (e.g., employee changes department, customer moves to new city)

| Type | Strategy | Tradeoff |
|------|----------|----------|
| **Type 0** | Retain original value forever | No history tracking |
| **Type 1** | Overwrite old value | Lose history, but always current |
| **Type 2** | Add new row with new surrogate key | **Most common.** Full history preserved. Requires effective/expiration dates. |
| **Type 3** | Add "previous" column | Only tracks one change (current + previous) |

**Type 2 example:** Doctor changes department

| doctor_key | doctor_id | name | department | effective_date | expiration_date | current |
|-----------|-----------|------|------------|---------------|----------------|---------|
| 101 | D-5432 | Dr. Park | Cardiology | 2020-01-15 | 2025-06-30 | N |
| 247 | D-5432 | Dr. Park | Neurology | 2025-07-01 | 9999-12-31 | Y |

Note: **Two surrogate keys** (101, 247) for the **same doctor**. Historical facts link to key 101 (Cardiology era), new facts link to key 247 (Neurology era).

---

## Star vs Snowflake

| | Star Schema | Snowflake Schema |
|--|-------------|-----------------|
| **Dimensions** | Denormalized (flat) | Normalized (broken into sub-tables) |
| **Joins** | Fewer (fact → dimension) | More (fact → dimension → sub-dimension) |
| **Query speed** | Faster ✅ | Slower ❌ |
| **Storage** | More (redundancy) | Less |
| **Ease of use** | Simpler ✅ | More complex |
| **Kimball's recommendation** | **Preferred** | Use sparingly |

```
STAR:                              SNOWFLAKE:
dim_product                        dim_product
├── product_name                   ├── product_name
├── brand_name      ←flattened     ├── brand_key → dim_brand
├── category_name   ←flattened     ├── category_key → dim_category
├── department_name ←flattened     └── ...
└── ...                            dim_brand
                                   ├── brand_name
                                   └── brand_country
```

> 💡 Kimball strongly recommends **star over snowflake** for analysis. Snowflaking saves storage but adds JOIN complexity and slows queries. The storage savings are negligible compared to the massive fact table.

---

## Estimating Fact Table Size

Quick estimation formula:

```
Total rows = Product of unique values across dimensions
           × Fill rate (what % of combinations actually occur)

Example (Retail):
  1,000 stores × 60,000 products × 365 days × 10% fill rate
  = ~2.2 billion rows/year

Bytes = Rows × Columns × Avg bytes/column
```

---

## ETL: Extract, Transform, Load

The process of moving data from OLTP → Star Schema:

```
EXTRACT          →    TRANSFORM           →    LOAD
Read from             Clean & reshape          Insert into
source systems        • Deduplicate            star schema
                      • Standardize formats    • Dimensions FIRST
                      • Derive new fields      • Facts SECOND
                      • Handle NULLs           • Index & optimize
                      • Map surrogate keys
```

**Key ETL principles:**
1. **Load dimensions before facts** (facts need dimension FKs)
2. **Handle data quality** — inconsistent formats, missing values, duplicates
3. **Assign surrogate keys** during dimension loads
4. **Track changes** — implement SCD logic during dimension updates
5. **Initial load vs incremental** — first load is full extract; subsequent loads capture only changes (CDC = Changed Data Capture)

---

## 10 Essential Rules for Dimensional Modeling

*(From Kimball, Chapter 2)*

1. **Use atomic facts** — Start with the finest grain
2. **Create single-process fact tables** — One business process per fact table
3. **Include a date dimension** for every fact table
4. **Enforce consistent grain** — Don't mix grains
5. **Disallow null foreign keys** in fact tables (use "Unknown" dimension row instead)
6. **Honor hierarchies** in dimensions (flatten them, don't snowflake)
7. **Decode dimension tables** — Use descriptive text, not codes
8. **Use surrogate keys** for all dimensions (except date)
9. **Conform dimensions** — Same dimension means same thing everywhere
10. **Balance requirements with actual data** — Design from both business needs AND data realities

---

## Quick Reference: Vocabulary

| Term | Definition |
|------|-----------|
| **Grain** | What one row in the fact table represents |
| **Measure / Fact** | A numeric value to aggregate (SUM, AVG) |
| **Dimension** | Descriptive context (who, what, where, when) |
| **Surrogate key** | Artificial integer PK for dimensions |
| **Natural key** | The original business key (e.g., SSN, employee_id) |
| **Degenerate dimension** | A dimension with no table — just a value in the fact table (e.g., invoice #) |
| **Conformed dimension** | A dimension shared across multiple fact tables with identical meaning |
| **Additive fact** | Can be summed across all dimensions |
| **Semi-additive fact** | Can be summed across some dimensions (not time) |
| **Non-additive fact** | Cannot be summed — must compute from components |
| **SCD** | Slowly Changing Dimension — strategy for handling dimension changes |
| **Snowflake** | Normalizing dimension tables (generally avoided) |
| **Bus matrix** | Enterprise-wide plan showing which dimensions apply to which business processes |

---

## Kimball Reading Guide for IS480

| Week | Reading | Key Takeaway |
|------|---------|-------------|
| 10 | Ch 1 (pp. 1-35) | Why warehouses exist, star schema basics, DW architecture |
| 11 | Ch 2 (pp. 37-68) | Complete technique reference — 4-step process, all fact/dimension types |
| 11 | Ch 3 (pp. 69-100) | Retail case study — see the 4 steps applied to a real business |

---

*This cheat sheet covers the essential Kimball concepts for IS480. For the complete treatment with industry case studies, refer to the full textbook.*

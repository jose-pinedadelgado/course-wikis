# 🐕 PacoDogShop

> **A dog fashion e-commerce database for teaching advanced SQL and PL/SQL.**

PacoDogShop is a realistic teaching database used in IS480 (Advanced Database Management) at California State University, Long Beach. It models an online store selling clothing, accessories, and care products for dogs.

## Why PacoDogShop?

Real-world databases are messy, interconnected, and domain-rich. PacoDogShop provides:

- **20 tables** across 5 business domains
- **Realistic seed data** with dog-themed products, breeds, and customer scenarios
- **Complex relationships** — many-to-many associations, self-referencing categories, order workflows
- **MySQL syntax** for hands-on lab exercises
- **ERD diagrams** in draw.io format for teaching data modeling

## Database at a Glance

| Domain | Tables | Key Concepts |
|---|---|---|
| **Products** | Category, Product, Product_Variant, Product_Image, Breed, Style_Tag, Product_Breed_Rec, Product_Style_Tag | Self-referencing hierarchy, many-to-many, SKU management |
| **Customers** | Customer, Customer_Address, Wishlist, Wishlist_Item | 1:M addresses, wishlist patterns |
| **Orders** | `Order`, Order_Item, Payment | Order lifecycle, payment processing |
| **Reviews** | Review | Customer-product feedback |
| **Promotions** | Promotion, Promotion_Product | Discount management, date-based validity |

## Quick Start

```sql
-- Create the database
SOURCE schema.sql;

-- Load demo data
SOURCE seed_data.sql;

-- Verify
SELECT COUNT(*) FROM Product;  -- 15 products
SELECT COUNT(*) FROM Customer; -- 10 customers
```

## Files

| File | Description |
|---|---|
| `schema.sql` | 20-table MySQL DDL |
| `seed_data.sql` | Realistic demo data |
| `pacodogshop_erd.drawio` | Entity-Relationship Diagram |
| `pacodogshop_eerd.drawio` | Enhanced ERD |
| `pacodogshop_relational.drawio` | Relational model diagram |
| `Lab_PLSQL_Solutions.sql` | PL/SQL lab solutions |
| `Reset_Class_Lab_CurrentSchema.sql` | Schema reset for labs |
| `Student_Schema_Reset_Template.sql` | Student lab setup |

## Repository

- **GitHub:** [jose-pinedadelgado/pacodogshop](https://github.com/jose-pinedadelgado/pacodogshop) (public)

---

*Jose Pineda — IS480 Advanced Database Management, CSULB*

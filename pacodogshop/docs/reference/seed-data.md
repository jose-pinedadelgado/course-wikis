# Seed Data

The `seed_data.sql` file populates all 20 tables with realistic dog fashion e-commerce data.

## Data Summary

| Table | Rows | Sample Data |
|---|---|---|
| Category | 5 | Clothing, Accessories, Care Products, Toys, Food & Treats |
| Product | 15 | Dog Raincoat, Bow Tie Collar, Paw Balm, Squeaky Bone |
| Product_Variant | 45 | Sizes S/M/L, colors Red/Blue/Green |
| Product_Image | 20 | Product photos with alt text |
| Breed | 8 | Golden Retriever, French Bulldog, Dachshund, etc. |
| Style_Tag | 6 | Casual, Formal, Sporty, Outdoor, Luxury, Seasonal |
| Product_Breed_Rec | 25 | Raincoat → Golden Retriever, Sweater → French Bulldog |
| Product_Style_Tag | 30 | Bow Tie → Formal, Raincoat → Outdoor |
| Customer | 10 | Realistic names, emails, phone numbers |
| Customer_Address | 15 | California addresses |
| Wishlist | 5 | Named wishlists per customer |
| Wishlist_Item | 12 | Products on wishlists |
| Order | 20 | Various statuses (pending, shipped, delivered) |
| Order_Item | 50 | Variant-level line items |
| Payment | 20 | Credit card, PayPal, Apple Pay |
| Review | 15 | 1-5 star ratings with text |
| Promotion | 4 | Seasonal sales with date ranges |
| Promotion_Product | 10 | Products on promotion |

## Design Choices

!!! info "Sparse Associative Tables"
    Not every product is recommended for every breed (25 of 120 possible combinations) or tagged with every style (30 of 90 possible). This is realistic — a tiny dog raincoat isn't recommended for a Great Dane.

!!! info "Realistic Merchants"
    Product names, descriptions, and prices reflect actual dog fashion products. This makes the data engaging for students while still exercising all SQL concepts.

## Querying the Seed Data

```sql
-- Products with their categories
SELECT p.name, c.name AS category, p.base_price
FROM Product p JOIN Category c ON p.category_id = c.category_id;

-- Top-rated products
SELECT p.name, AVG(r.rating) AS avg_rating, COUNT(*) AS review_count
FROM Product p JOIN Review r ON p.product_id = r.product_id
GROUP BY p.name ORDER BY avg_rating DESC;

-- Customer order history
SELECT c.first_name, c.last_name, COUNT(o.order_id) AS orders, SUM(o.total) AS total_spent
FROM Customer c JOIN `Order` o ON c.customer_id = o.customer_id
GROUP BY c.customer_id;

-- Products recommended for a breed
SELECT p.name, b.name AS breed
FROM Product p
JOIN Product_Breed_Rec pbr ON p.product_id = pbr.product_id
JOIN Breed b ON pbr.breed_id = b.breed_id
WHERE b.name = 'Golden Retriever';
```

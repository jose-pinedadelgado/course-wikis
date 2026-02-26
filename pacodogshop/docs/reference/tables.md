# Table Reference

Complete DDL reference for all 20 tables.

## Category

```sql
CREATE TABLE Category (
    category_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    parent_category_id INT NULL,
    FOREIGN KEY (parent_category_id) REFERENCES Category(category_id)
);
```

Self-referencing hierarchy. `parent_category_id` enables subcategories.

## Product

```sql
CREATE TABLE Product (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    category_id INT NOT NULL,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    base_price DECIMAL(10,2) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES Category(category_id)
);
```

## Product_Variant

```sql
CREATE TABLE Product_Variant (
    variant_id INT PRIMARY KEY AUTO_INCREMENT,
    product_id INT NOT NULL,
    sku VARCHAR(50) NOT NULL UNIQUE,
    size VARCHAR(20),
    color VARCHAR(30),
    price DECIMAL(10,2) NOT NULL,
    weight DECIMAL(8,2),
    stock_quantity INT DEFAULT 0,
    is_active BOOLEAN DEFAULT TRUE,
    FOREIGN KEY (product_id) REFERENCES Product(product_id)
);
```

Each product can have multiple variants (e.g., "Red Bandana" in S/M/L).

## Product_Image

```sql
CREATE TABLE Product_Image (
    image_id INT PRIMARY KEY AUTO_INCREMENT,
    product_id INT NOT NULL,
    variant_id INT NULL,
    image_url VARCHAR(500) NOT NULL,
    alt_text VARCHAR(200),
    sort_order INT DEFAULT 0,
    FOREIGN KEY (product_id) REFERENCES Product(product_id),
    FOREIGN KEY (variant_id) REFERENCES Product_Variant(variant_id)
);
```

## Breed

```sql
CREATE TABLE Breed (
    breed_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    size_category VARCHAR(20),
    description TEXT
);
```

## Style_Tag

```sql
CREATE TABLE Style_Tag (
    tag_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,
    description TEXT
);
```

## Product_Breed_Rec (Associative)

```sql
CREATE TABLE Product_Breed_Rec (
    product_id INT NOT NULL,
    breed_id INT NOT NULL,
    PRIMARY KEY (product_id, breed_id),
    FOREIGN KEY (product_id) REFERENCES Product(product_id),
    FOREIGN KEY (breed_id) REFERENCES Breed(breed_id)
);
```

## Product_Style_Tag (Associative)

```sql
CREATE TABLE Product_Style_Tag (
    product_id INT NOT NULL,
    tag_id INT NOT NULL,
    PRIMARY KEY (product_id, tag_id),
    FOREIGN KEY (product_id) REFERENCES Product(product_id),
    FOREIGN KEY (tag_id) REFERENCES Style_Tag(tag_id)
);
```

## Customer

```sql
CREATE TABLE Customer (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Customer_Address

```sql
CREATE TABLE Customer_Address (
    address_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    address_type VARCHAR(20) DEFAULT 'shipping',
    street VARCHAR(200) NOT NULL,
    city VARCHAR(100) NOT NULL,
    state VARCHAR(50),
    zip_code VARCHAR(20),
    country VARCHAR(50) DEFAULT 'US',
    is_default BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (customer_id) REFERENCES Customer(customer_id)
);
```

## Order

```sql
CREATE TABLE `Order` (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    shipping_address_id INT,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'pending',
    subtotal DECIMAL(10,2),
    tax DECIMAL(10,2),
    shipping_cost DECIMAL(10,2),
    total DECIMAL(10,2),
    FOREIGN KEY (customer_id) REFERENCES Customer(customer_id),
    FOREIGN KEY (shipping_address_id) REFERENCES Customer_Address(address_id)
);
```

!!! note "Reserved Word"
    `Order` is a MySQL reserved word — must be backtick-quoted in queries: `` `Order` ``.

## Order_Item, Payment, Wishlist, Wishlist_Item, Review, Promotion, Promotion_Product

See `schema.sql` for complete DDL of remaining tables. All follow the same pattern: AUTO_INCREMENT PKs, NOT NULL constraints on required fields, and FK references to parent tables.

## Overview of Comparison Operators

SQL uses six primary comparison operators to evaluate conditions in a WHERE clause. These operations typically return a boolean result: 1 (True), 0 (False) or NULL.

| Operator | Description | Example |
| --- | --- | --- |
| = | Equal to | WHERE price = 600 |
| != or <> | Not equal to | WHERE price != 800 |
| < | Less than | WHERE price < 500 |
| > | Greater than | WHERE price > 500 |
| <= | Less than or equal to | WHERE price <= 150 |
| >= | Greater than or equal to | WHERE price >= 800 |

```sql
CREATE DATABASE StoreDB;
USE StoreDB;
CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(50),
    category VARCHAR(50),
    price DECIMAL(10, 2),
    stock INT
);
INSERT INTO products(product_name, category, price, stock) VALUES
('Laptop', 'Electronics', 1200.00, 10),
('Phone', 'Electronics', 800.00, 15),
('Tablet', 'Electronics', 600.00, 20),
('Headphones', 'Accessories', 150.00, 50),
('Mouse', 'Accessories', 30.00, 100),
('Keyboard', 'Accessories', 45.00, 80);

SELECT * FROM products;
```

Examples:

```sql
-- Get all products with a price of exactly 600
SELECT * FROM products;
WHERE price = 600;

-- Get all products that are not priced at 800
SELECT * FROM products;
WHERE NOT price = 800;
OR
WHERE price != 800;
OR
WHERE price <> 800;

-- Get all products below 500
SELECT * FROM products;
WHERE products < 500;

-- Get all products priced above 700
SELECT * FROM products;
WHERE products > 700;
```

Note: Implicit Conversion: If you compare a decimal column with an integer (e.g., WHERE price > 700), MySQL automatically converts the integer to a decimal to match the data type.

```sql
-- Get all products priced at or below 150
SELECT * FROM products;
WHERE products <= 150;

-- Get all products at or above 800
SELECT * FROM products;
WHERE products >= 800;

-- Get all products where the category is exactly "Electronics"
SELECT * FROM products;
WHERE category = 'Electronics';
```

```sql
CREATE TABLE orders (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    order_date DATE,
    customer_name VARCHAR(50)
);
INSERT INTO orders (order_date, customer_name) VALUES
('2024-02-01', 'Alice'),
('2024-02-05', 'Bob'),
('2024-02-10', 'Charlie'),
('2024-02-15', 'David');
```

```sql
-- Retrieve Orders Placed Before February 10, 2024
SELECT * FROM orders;
WHERE order_date < '2024-02-10';
```

## Filtering with Strings (Lexicographical Comparison)

When comparing strings using < or >, SQL uses lexicographical order (dictionary order).

- Dictionary Logic: WHERE product_name > 'Mouse' will return products starting with 'P', 'T', etc., because they appear after 'M' in the alphabet.

- Case Sensitivity: By default, these comparisons are case-insensitive. To force a case-sensitive search, use the BINARY keyword.

## Advanced Logic: Strings vs. Numbers

- ASCII Values: Strings are compared character-by-character based on their numeric ASCII values (e.g., '1' is 49, '2' is 50). This explains why '10' < '2' might return True (1) in a string context.

- Type Casting: To force a string to be treated as a number during comparison, you can add zero (+ 0) to the expression.

```sql
SELECT '100' + 0 < '2' + 0;
```

- Partial Reading: When MySQL converts a string to a number, it reads from left to right. It stops at the first non-numeric character (e.g., '21abc' becomes 21).

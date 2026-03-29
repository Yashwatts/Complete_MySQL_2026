## Setup and Sample Data

```sql
CREATE DATABASE db13;
USE db13;
-- Create products table
CREATE TABLE products (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(100),
        price DECIMAL(10,2),
        category VARCHAR(50),
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
-- Insert sample data
INSERT INTO products (name, price, category) VALUES
('Laptop', 999.99, 'Electronics'),
('Smartphone', 499.99, 'Electronics'),
('Coffee Maker', 79.99, 'Appliances'),
('Headphones', 149.99, 'Electronics'),
('Blender', 59.99, 'Appliances'),
('Tablet', 299.99, 'Electronics'),
('Microwave', 199.99, 'Appliances'),
('Smart Watch', 249.99, 'Electronics'),
('Toaster', 39.99, 'Appliances'),
('Speaker', 89.99, 'Electronics');
```

## 1. Introduction to LIMIT Clause

The LIMIT clause is used to control the number of records returned by a query.

- **Why use it?**
    - On production servers with massive databases, running `SELECT * FROM table_name` can significantly increase server load. Using LIMIT ensures you only fetch what is necessary.

- **Basic Syntax:**

```sql
SELECT column_names FROM table_name LIMIT number;
```

- **Example:** To get only the first two records from a products table:

```sql
SELECT * FROM products LIMIT 2;
```

> **Note:** Always use ORDER BY when using LIMIT to ensure your results are consistent and meaningful.

```sql
SELECT * FROM products ORDER BY id LIMIT 2;
```

## 2. OFFSET and Pagination

Pagination is the process of dividing a large dataset into discrete pages (e.g., showing 10 records at a time).

- **OFFSET:** Specifies how many records to skip before starting to return rows.
- **Standard Syntax:**

```sql
SELECT * FROM products ORDER BY id LIMIT 2 OFFSET 2;
```

- **Alternative (Short) Syntax:**

```sql
SELECT * FROM products LIMIT 0, 3; -- Skip 0, show 3
SELECT * FROM products LIMIT 3, 3; -- Skip 3, show 3
```

### Pagination Implementation

- Page size: 3 items per page
- For page 1 (Using OFFSET syntax):
    ```sql
    SELECT * FROM products LIMIT 3 OFFSET 0;
    ```
- For page 2:
    ```sql
    SELECT * FROM products LIMIT 3 OFFSET 3;
    ```
- For page 3:
    ```sql
    SELECT * FROM products LIMIT 3 OFFSET 6;
    ```

- Alternative syntax using LIMIT offset, count
    - For page 1:
        ```sql
        SELECT * FROM products LIMIT 0, 3;
        ```
    - For page 2:
        ```sql
        SELECT * FROM products LIMIT 3, 3;
        ```
    - For page 3:
        ```sql
        SELECT * FROM products LIMIT 6, 3;
        ```

- **Pagination Formula:**
    - Offset: (Page Number - 1) * Items Per Page
    - Limit: Items Per Page
    - Example:
        ```sql
        LIMIT (page_number - 1) * items_per_page, items_per_page
        ```

## 3. Practical Use Cases

- **Top 3 most expensive products**
    ```sql
    SELECT * FROM products
    ORDER BY price DESC
    LIMIT 3;
    ```

- **Get 5 random products**
    ```sql
    SELECT * FROM products
    ORDER BY RAND()
    LIMIT 5;
    ```

## 4. Performance Considerations

- **The Inefficiency of OFFSET:** As your table grows to millions of rows, high offsets (e.g., skipping 1,000,000 rows to get the next 10) become very slow. The database still has to scan and sort all those skipped records before returning the result.

- **Best Practice:** For large-scale data, use Keyset Pagination (or "Seek Method"). Instead of using OFFSET, remember the ID or timestamp of the last record from the previous page and use a WHERE clause:
    ```sql
    WHERE id > last_seen_id LIMIT 10;
    ```

- **Example of potentially slow query with large offset**
    ```sql
    SELECT *
    FROM products  -- Note: In real scenario, this would be a much larger table
    ORDER BY created_at
    LIMIT 1000000, 10;
    ```

- **Better alternative using WHERE clause**
    ```sql
    SELECT *
    FROM products
    WHERE created_at > '2025-01-01 00:00:00'
    ORDER BY created_at
    LIMIT 10;
    ```

## Key Takeaways

- LIMIT helps in retrieving a specific number of rows.
- LIMIT offset, count is used for pagination.
- Combining ORDER BY with LIMIT is essential for meaningful result sets.
- Be cautious about performance impacts when using high offset values.
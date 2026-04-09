## SQL Indexing

Indexing is one of the most important topics in SQL performance. A well-designed index can make a query run in milliseconds instead of seconds, while a bad index strategy can slow down inserts and waste storage.


## 1. What Is an Index?

- An index is a separate data structure that helps MySQL find rows faster without scanning the whole table.
- Think of it like a book index: instead of reading every page, you jump directly to the right location.
- Without an index, MySQL often performs a full table scan.
- With the right index, MySQL can do fast lookups, range searches, sorting and joins.

Important idea:
- Indexes speed up `SELECT` (reads) but add overhead to `INSERT`, `UPDATE`, and `DELETE` (writes), because the index must also be maintained.


## 2. Why Indexes Matter

Common operations improved by indexes:
- `WHERE` filtering
- `JOIN` conditions
- `ORDER BY`
- `GROUP BY`
- `MIN()` / `MAX()` on indexed columns

Example without index:
```sql
SELECT *
FROM orders
WHERE customer_id = 1001;
```
If `customer_id` is not indexed, MySQL checks every row.

Example with index:
```sql
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
```
Now MySQL can jump directly to matching rows.


## 3. How Indexing Works Internally (B+ Trees)

In MySQL (especially InnoDB), most indexes are implemented using B+ Trees.

### B+ Tree Basics
- A B+ Tree is a balanced tree structure.
- Internal nodes store key ranges and pointers.
- Leaf nodes store ordered keys.
- Tree height is small, so lookup needs very few page reads.

### Why B+ Tree Is Fast
- Keys are sorted, so exact match and range queries are efficient.
- Balanced structure keeps performance stable as data grows.
- Leaf nodes are linked, which helps range scans like `BETWEEN`.

### InnoDB-Specific Behavior
- The table is physically organized by the Primary Key (clustered index).
- Secondary index leaf nodes store:
  - the secondary key value
  - the corresponding primary key value
- So for secondary index lookups, InnoDB may do:
  1. find matching primary key from secondary index
  2. fetch row from clustered index

This extra step is called a bookmark lookup (or back-to-table lookup).


## 4. Types of Indexes

### 4.1 Primary Index (PRIMARY KEY)

- Created automatically when you define a `PRIMARY KEY`.
- Must be unique and `NOT NULL`.
- In InnoDB, this is the clustered index.
- A table can have only one primary key.

Example:
```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(150)
);
```

Effect:
- Fast row lookup by `student_id`.
- Physical row order (InnoDB) follows primary key.


### 4.2 Unique Index

- Ensures all values in indexed column(s) are unique.
- Allows `NULL` values (MySQL can allow multiple NULLs depending on engine/rules).
- Useful for business-unique fields like email, username, phone, etc.

Example:
```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(150) NOT NULL,
    UNIQUE INDEX uq_users_username (username),
    UNIQUE INDEX uq_users_email (email)
);
```

Try duplicate insert:
```sql
INSERT INTO users (user_id, username, email)
VALUES (1, 'alice', 'alice@example.com');

INSERT INTO users (user_id, username, email)
VALUES (2, 'alice', 'alice2@example.com');
-- Fails: duplicate value for unique index uq_users_username
```


### 4.3 Composite Index (Multi-Column Index)

- Index built on multiple columns together.
- Order of columns matters.
- Follows the leftmost prefix rule.

Example:
```sql
CREATE TABLE sales (
    sale_id BIGINT PRIMARY KEY,
    store_id INT,
    sale_date DATE,
    customer_id INT,
    amount DECIMAL(10,2),
    INDEX idx_sales_store_date_customer (store_id, sale_date, customer_id)
);
```

Good query patterns for this index:
```sql
-- Uses (store_id)
SELECT * FROM sales WHERE store_id = 10;

-- Uses (store_id, sale_date)
SELECT * FROM sales WHERE store_id = 10 AND sale_date = '2026-04-01';

-- Uses all three columns
SELECT *
FROM sales
WHERE store_id = 10
  AND sale_date = '2026-04-01'
  AND customer_id = 200;
```

Poor query pattern:
```sql
-- Cannot efficiently use the composite index starting from sale_date only
SELECT * FROM sales WHERE sale_date = '2026-04-01';
```

Leftmost prefix rule summary:
- `(a, b, c)` can help with `a`, `(a,b)`, `(a,b,c)`
- Usually not effective for only `b`, only `c`, or `(b,c)`


### 4.4 Full-Text Index

- Designed for text search, not exact-value lookup.
- Supports natural language and boolean mode searches.
- Best on large text fields (`TEXT`, `VARCHAR`) where `LIKE '%word%'` is slow.

Example:
```sql
CREATE TABLE articles (
    article_id INT PRIMARY KEY,
    title VARCHAR(255),
    body TEXT,
    FULLTEXT INDEX ft_title_body (title, body)
);
```

Search examples:
```sql
-- Natural language mode
SELECT article_id, title,
       MATCH(title, body) AGAINST ('database indexing') AS relevance
FROM articles
WHERE MATCH(title, body) AGAINST ('database indexing')
ORDER BY relevance DESC;

-- Boolean mode (+ required, - excluded)
SELECT article_id, title
FROM articles
WHERE MATCH(title, body) AGAINST ('+mysql -oracle' IN BOOLEAN MODE);
```


### 4.5 Clustered vs Non-Clustered Index

In interview language, you will often hear clustered and non-clustered index.

- Clustered index:
  - Data rows are stored in index order.
  - In InnoDB, `PRIMARY KEY` is the clustered index.
  - A table has only one clustered index.

- Non-clustered index:
  - Index structure is separate from row data.
  - In InnoDB, all secondary indexes are non-clustered.
  - Leaf nodes store index key plus primary key pointer.

Quick comparison:
- Lookup by primary key is very fast (direct clustered access).
- Lookup by secondary key may need an extra hop to clustered data.

Example:
```sql
CREATE TABLE products (
    product_id BIGINT PRIMARY KEY,   -- Clustered index in InnoDB
    sku VARCHAR(40) NOT NULL,
    category_id INT,
    price DECIMAL(10,2),
    INDEX idx_products_sku (sku)     -- Non-clustered (secondary) index
);
```


### 4.6 Covering Index

- A covering index is an index that contains all columns required by a query.
- If MySQL can answer from index pages only, it avoids table row lookup.
- In `EXPLAIN`, this often appears as `Using index` in Extra.

Example:
```sql
CREATE TABLE orders (
    order_id BIGINT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10,2),
    status VARCHAR(20),
    INDEX idx_orders_customer_date_status (customer_id, order_date, status)
);
```

Query that can be covered:
```sql
EXPLAIN
SELECT customer_id, order_date, status
FROM orders
WHERE customer_id = 101
  AND order_date >= '2026-01-01';
```

Why covered:
- Selected columns: `customer_id`, `order_date`, `status`
- Filtered columns: `customer_id`, `order_date`
- All are in the same index, so no back-to-table read is needed.


### 4.7 Index Cardinality

- Cardinality means how many distinct values exist in an indexed column.
- High cardinality (many unique values) usually gives better filtering.
- Low cardinality (few distinct values) often gives weak index benefit.

Examples:
- `email` column: high cardinality -> good index candidate.
- `gender` or `is_active`: low cardinality -> often poor candidate alone.

Check cardinality:
```sql
SHOW INDEX FROM users;
```

Also estimate distinctness directly:
```sql
SELECT COUNT(DISTINCT department_id) AS distinct_departments,
       COUNT(*) AS total_rows
FROM employees;
```

Rule of thumb:
- Prefer indexing columns where distinct ratio is high:
  - distinct_ratio = COUNT(DISTINCT col) / COUNT(*)


### 4.8 Partial Index (MySQL 8 Concept)

MySQL does not support true filtered indexes like some other databases (`WHERE condition` on index definition).
But in MySQL 8, partial indexing is commonly achieved using these patterns:

1. Prefix index (index only part of a string):
```sql
CREATE TABLE customers (
    customer_id BIGINT PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    INDEX idx_customers_email_prefix (email(20))
);
```

2. Functional index with expression (MySQL 8.0.13+):
```sql
CREATE TABLE logs (
    log_id BIGINT PRIMARY KEY,
    created_at DATETIME NOT NULL,
    message TEXT
);

CREATE INDEX idx_logs_created_date
ON logs ((DATE(created_at)));
```

3. Generated column + index to emulate filtered behavior:
```sql
CREATE TABLE tasks (
    task_id BIGINT PRIMARY KEY,
    status VARCHAR(20) NOT NULL,
    due_date DATE,
    is_open TINYINT AS (status IN ('OPEN', 'IN_PROGRESS')) STORED,
    INDEX idx_tasks_is_open_due (is_open, due_date)
);
```

This gives a practical MySQL 8 approach when you want index focus on a subset pattern.


### 4.9 Invisible Index

- MySQL 8 supports invisible indexes.
- Invisible index is maintained on writes but optimizer ignores it by default.
- Very useful to test if an index can be dropped safely.

Create invisible index:
```sql
CREATE INDEX idx_orders_status ON orders(status) INVISIBLE;
```

Make existing index invisible/visible:
```sql
ALTER TABLE orders ALTER INDEX idx_orders_status INVISIBLE;
ALTER TABLE orders ALTER INDEX idx_orders_status VISIBLE;
```

Typical workflow:
- Mark index invisible.
- Monitor slow queries/regressions.
- If no regression, drop index confidently.


### 4.10 Index Hints (`USE INDEX`, `FORCE INDEX`)

Sometimes optimizer chooses a suboptimal index. Index hints let you guide it.

- `USE INDEX`: suggest preferred index(es).
- `FORCE INDEX`: strongly push optimizer to use given index.
- `IGNORE INDEX`: tell optimizer to avoid specific index.

Examples:
```sql
-- Suggest index
SELECT *
FROM orders USE INDEX (idx_orders_customer_id)
WHERE customer_id = 1001;

-- Force index
SELECT *
FROM orders FORCE INDEX (idx_orders_customer_id)
WHERE customer_id = 1001;

-- Ignore index
SELECT *
FROM orders IGNORE INDEX (idx_orders_customer_id)
WHERE customer_id = 1001;
```

Important caution:
- Hints can improve a specific query now but may become harmful after data distribution changes.
- Prefer fixing schema/statistics first and use hints only when needed.


## 5. How to Create, View, and Drop Indexes

Create index:
```sql
CREATE INDEX idx_orders_order_date ON orders(order_date);
```

Show indexes in a table:
```sql
SHOW INDEX FROM orders;
```

Drop index:
```sql
DROP INDEX idx_orders_order_date ON orders;
```

Create unique index directly:
```sql
CREATE UNIQUE INDEX uq_products_sku ON products(sku);
```


## 6. When NOT to Use Indexes

Do not index blindly. Avoid indexes in these situations:

- Very small tables:
  - Full table scan may be cheaper than index traversal.

- Columns with very low selectivity:
  - Example: `is_active` with only `0/1` values.
  - MySQL may still choose table scan.

- Heavy write workloads:
  - Many indexes increase cost of `INSERT/UPDATE/DELETE`.

- Frequently updated indexed columns:
  - Updating indexed keys causes additional index maintenance.

- Too many overlapping indexes:
  - Wastes disk and memory, can confuse optimizer.

- Columns rarely used in filtering/sorting/joining:
  - Index gives little practical value.


## 7. Performance Comparison (With and Without Index)

Use this benchmarking flow to compare query performance.

### Step 1: Create demo table
```sql
CREATE TABLE employees (
    emp_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    emp_code VARCHAR(20) NOT NULL,
    department_id INT NOT NULL,
    hire_date DATE NOT NULL,
    salary DECIMAL(10,2) NOT NULL
);
```

### Step 2: Insert large sample data
Use bulk inserts in your setup so table has many rows (for example, 100000+).

### Step 3: Query before index
```sql
EXPLAIN ANALYZE
SELECT *
FROM employees
WHERE emp_code = 'E-009876';
```

Likely behavior before index:
- access type: `ALL` (full table scan)
- high rows examined
- higher execution time

### Step 4: Add index and test again
```sql
CREATE INDEX idx_employees_emp_code ON employees(emp_code);

EXPLAIN ANALYZE
SELECT *
FROM employees
WHERE emp_code = 'E-009876';
```

Likely behavior after index:
- access type: `ref` or `const`
- very low rows examined
- much lower execution time

### Example result summary (illustrative)
- Before index: 100000 rows scanned, 180 ms
- After index: 1-2 rows scanned, 3 ms

The exact numbers depend on hardware, cache and data distribution, but the pattern is usually dramatic.


## 8. Important Best Practices

- Index columns used often in `WHERE`, `JOIN`, and `ORDER BY`.
- Prefer selective columns (many distinct values).
- For composite indexes, choose column order based on query patterns.
- Keep indexes lean; avoid duplicate and redundant indexes.
- Re-check execution plans after schema or query changes.
- Use `EXPLAIN` / `EXPLAIN ANALYZE` to verify optimizer behavior.
- Periodically audit indexes:
  - unused indexes
  - duplicate indexes
  - missing indexes for slow queries


## 9. Quick Practical Checklist

Before creating an index, ask:

- Is this column frequently used for filtering/joining/sorting?
- Is the value distribution selective enough?
- Will this hurt write performance too much?
- Can one composite index replace multiple single-column indexes?
- Did `EXPLAIN` confirm that MySQL actually uses this index?


## 10. Final Summary

- Indexes are essential for SQL performance tuning.
- Primary, Unique, Composite, and Full-text indexes solve different problems.
- MySQL mostly uses B+ Trees (except full-text internals), giving fast search and range performance.
- Indexes are not free: they consume storage and slow writes.
- The best indexing strategy is workload-driven and validated with real execution plans.

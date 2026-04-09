## SQL Views

Views are virtual tables created from SQL queries. They help simplify complex queries, control data access and improve maintainability.


## 1. What Are Views?

- A view is a named SQL query stored in the database.
- It behaves like a table when queried.
- A standard view does not store data itself (it stores the query definition).
- Data shown by the view comes from underlying base tables.

Think of a view as:
- a reusable query layer
- an abstraction layer over tables
- a security layer to expose only selected columns/rows

Basic usage:
```sql
SELECT * FROM employee_public_view;
```


## 2. Why Views Are Useful

- Reuse complex joins and calculations.
- Hide unnecessary columns from users.
- Keep application SQL cleaner.
- Provide a stable interface even if table design changes internally.
- Restrict data visibility for security/compliance.


## 3. CREATE VIEW Syntax

Basic syntax:
```sql
CREATE VIEW view_name AS
SELECT column1, column2, ...
FROM table_name
WHERE condition;
```

Practical example:
```sql
CREATE VIEW active_customers AS
SELECT customer_id, full_name, email, city
FROM customers
WHERE is_active = 1;
```

Query the view:
```sql
SELECT *
FROM active_customers
WHERE city = 'Delhi';
```


## 4. OR REPLACE and DROP VIEW

Replace existing view definition:
```sql
CREATE OR REPLACE VIEW active_customers AS
SELECT customer_id, full_name, email, city, created_at
FROM customers
WHERE is_active = 1;
```

Drop a view:
```sql
DROP VIEW active_customers;
```

Drop only if exists:
```sql
DROP VIEW IF EXISTS active_customers;
```


## 5. Simple vs Complex Views

### 5.1 Simple View

- Usually based on one table.
- No aggregates, no GROUP BY, no DISTINCT, no UNION.
- More likely to be updatable.

```sql
CREATE VIEW product_basic_view AS
SELECT product_id, product_name, price
FROM products;
```

### 5.2 Complex View

- Uses joins, aggregations, expressions, subqueries, etc.
- Mostly read-only in practice.

```sql
CREATE VIEW department_salary_summary AS
SELECT d.department_id,
       d.department_name,
       COUNT(e.emp_id) AS employee_count,
       AVG(e.salary) AS avg_salary
FROM departments d
JOIN employees e ON e.department_id = d.department_id
GROUP BY d.department_id, d.department_name;
```


## 6. Materialized View Concept

- A materialized view stores the result of a query physically, not just the query definition.
- MySQL does not support materialized views natively.
- In MySQL, you usually simulate them with:
  - a summary table
  - scheduled refresh jobs
  - triggers or ETL processes

Example idea:
```sql
-- Stored summary table acting like a materialized view
CREATE TABLE sales_summary_daily (
    sale_date DATE PRIMARY KEY,
    total_orders INT,
    total_amount DECIMAL(12,2)
);
```

This table can be refreshed periodically from base tables.


## 7. View Algorithm (MERGE vs TEMPTABLE)

MySQL can use different internal strategies when evaluating views.

### 7.1 MERGE Algorithm

- MySQL merges the view query into the outer query.
- This is usually faster because the optimizer can push filters down to base tables.
- Best for simple views.

Example:
```sql
CREATE ALGORITHM = MERGE VIEW active_customers AS
SELECT customer_id, full_name, city
FROM customers
WHERE is_active = 1;
```

### 7.2 TEMPTABLE Algorithm

- MySQL first materializes the view result into a temporary table.
- Then the outer query runs against that temporary result.
- This can be slower, but sometimes it is required for complex views.

Example:
```sql
CREATE ALGORITHM = TEMPTABLE VIEW sales_totals AS
SELECT customer_id, SUM(amount) AS total_spent
FROM sales
GROUP BY customer_id;
```

Important note:
- TEMPTABLE views are generally non-updatable.
- MERGE views are more likely to remain updatable.


## 8. When Views Become Slow

Views can become slow when they hide expensive work rather than eliminating it.

Common slow cases:
- View contains heavy joins across large tables.
- View uses aggregation (`GROUP BY`, `SUM`, `AVG`) on many rows.
- View is nested on top of another view.
- View uses `DISTINCT`, `UNION`, or derived expressions that prevent optimization.
- Underlying base tables are missing indexes.
- The optimizer cannot merge the view, so it builds a temporary table.

Example of a potentially slow pattern:
```sql
CREATE VIEW expensive_report AS
SELECT c.city,
       COUNT(*) AS orders_count,
       SUM(o.total_amount) AS revenue
FROM customers c
JOIN orders o ON o.customer_id = c.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY c.city;
```

If queried often, consider summary tables, better indexing, or precomputed reporting structures.


## 9. Difference: View vs Table

This is a common interview question.

- Table:
  - Physically stores data rows.
  - Can be inserted into directly.
  - Has its own storage footprint for row data.

- View:
  - Stores query definition, not the actual data.
  - Shows data from underlying tables.
  - Acts like a saved SELECT statement.

Comparison:
- Table is the source of data.
- View is a window into data.
- Table can be indexed directly.
- Normal view does not store its own indexes.


## 10. Read-Only Views Explicitly

- Many views are read-only by design.
- A read-only view cannot be used for `INSERT`, `UPDATE`, or `DELETE`.
- Complex views with joins, aggregates, `DISTINCT`, `GROUP BY`, or `UNION` are usually read-only.

Example read-only view:
```sql
CREATE VIEW department_salary_summary AS
SELECT d.department_id,
       d.department_name,
       COUNT(e.emp_id) AS employee_count,
       AVG(e.salary) AS avg_salary
FROM departments d
JOIN employees e ON e.department_id = d.department_id
GROUP BY d.department_id, d.department_name;
```

Trying to modify it will fail because it is a summary result, not a single-row mapping.


## 11. Updatable vs Non-Updatable Views

This is one of the most important view concepts.

### 6.1 Updatable Views

A view can be updatable if it follows rules like:
- References exactly one base table.
- No `GROUP BY`, `DISTINCT`, `HAVING`, aggregate functions (`SUM`, `AVG`, etc.).
- No `UNION`.
- No derived columns that break direct row mapping.

Example updatable view:
```sql
CREATE VIEW employee_contact_view AS
SELECT emp_id, full_name, email, city
FROM employees;
```

Update through view:
```sql
UPDATE employee_contact_view
SET city = 'Mumbai'
WHERE emp_id = 101;
```

This updates the underlying `employees` table row.

### 6.2 Non-Updatable Views

A view is non-updatable when row mapping is ambiguous or aggregated.

Common non-updatable patterns:
- `GROUP BY` / aggregates
- `DISTINCT`
- `UNION`
- many join patterns
- subqueries in select list that break direct mapping

Example non-updatable view:
```sql
CREATE VIEW sales_totals AS
SELECT customer_id, SUM(amount) AS total_spent
FROM sales
GROUP BY customer_id;
```

Trying to update this view will fail because each row represents multiple base rows.


## 12. WITH CHECK OPTION (Very Important)

`WITH CHECK OPTION` ensures inserted/updated rows still satisfy the view condition.

Example:
```sql
CREATE OR REPLACE VIEW active_customers AS
SELECT customer_id, full_name, is_active
FROM customers
WHERE is_active = 1
WITH CHECK OPTION;
```

Now this fails:
```sql
UPDATE active_customers
SET is_active = 0
WHERE customer_id = 10;
```

Why? Result would move outside view condition (`is_active = 1`), so MySQL blocks it.


## 13. Security Benefits of Views

Views are very useful for access control.

### 8.1 Column-Level Security

Expose only safe columns:
```sql
CREATE VIEW employee_public_view AS
SELECT emp_id, full_name, department_id
FROM employees;
```

Do not include sensitive fields (`salary`, `bank_account_no`, etc.).

### 8.2 Row-Level Filtering Pattern

Expose only subset rows:
```sql
CREATE VIEW open_orders_view AS
SELECT order_id, customer_id, order_date, status
FROM orders
WHERE status = 'OPEN';
```

### 8.3 Grant Permissions on View Instead of Base Table

```sql
GRANT SELECT ON open_orders_view TO 'report_user'@'localhost';
```

Now user can query allowed data through view without direct base-table access.

### 8.4 SQL SECURITY DEFINER vs INVOKER

MySQL supports security context for views:

- `SQL SECURITY DEFINER`:
  - View runs with privileges of definer user.
- `SQL SECURITY INVOKER`:
  - View runs with privileges of the caller.

Example:
```sql
CREATE SQL SECURITY DEFINER VIEW employee_public_view AS
SELECT emp_id, full_name, department_id
FROM employees;
```

Use carefully, because definers with high privileges can expose broader access if not designed properly.


## 14. Performance Notes (Important Reality)

- Views do not automatically improve performance.
- They mainly improve readability, reuse, and security.
- Performance depends on underlying query and indexes on base tables.
- Complex nested views can make optimization harder.

Best practice:
- Use views for abstraction and security.
- Tune performance with good SQL and indexing on base tables.


## 15. View Dependencies and Maintenance

- If base table columns are renamed/dropped, dependent views can break.
- Keep schema changes coordinated with view updates.
- Use `SHOW CREATE VIEW view_name;` to inspect definition.

Example:
```sql
SHOW CREATE VIEW active_customers;
```


## 16. Practical End-to-End Example

```sql
-- Base table
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL,
    department_id INT,
    salary DECIMAL(10,2),
    is_active TINYINT NOT NULL DEFAULT 1
);

-- Public view: hides salary
CREATE VIEW employee_public_view AS
SELECT emp_id, full_name, email, department_id
FROM employees;

-- Active employee view with rule protection
CREATE VIEW active_employee_view AS
SELECT emp_id, full_name, email, is_active
FROM employees
WHERE is_active = 1
WITH CHECK OPTION;

-- Query view
SELECT * FROM employee_public_view;

-- Update via updatable view
UPDATE active_employee_view
SET email = 'newmail@example.com'
WHERE emp_id = 1;
```


## 17. Limitations of Views

- No direct indexes on normal views (index base tables instead).
- Some views are not updatable.
- Overusing deeply nested views can reduce clarity.
- Changes in base schema may invalidate views.


## 18. Best Practices

- Use clear naming: `*_view` suffix is helpful.
- Keep view purpose focused (one business use per view).
- Prefer simple updatable views when DML through view is needed.
- Use `WITH CHECK OPTION` to enforce business rules.
- Grant permissions on views for safer access control.
- Document which base tables each view depends on.


## 19. Quick Interview Notes

- A view is a virtual table defined by a query.
- Most simple single-table views are updatable.
- Aggregate/join-heavy views are usually non-updatable.
- `WITH CHECK OPTION` prevents updates that violate view condition.
- Views improve security and abstraction, not guaranteed raw speed.


## 20. Final Summary

- Views simplify query reuse and hide complexity.
- `CREATE VIEW` helps build reusable logical data layers.
- Updatable vs non-updatable behavior depends on query structure.
- Views are powerful for column/row-level data exposure.
- Secure design with grants and security mode makes views production-friendly.

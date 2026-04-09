## SQL Performance Optimization

Performance optimization is about making SQL queries faster and more scalable without breaking correctness. In MySQL, the biggest performance gains usually come from good query design, proper indexing and understanding the execution plan.


## 1. What Performance Optimization Means

- Performance optimization means reducing execution time, CPU usage, I/O, memory pressure and lock contention.
- The goal is not just faster queries, but predictable performance under real data volume.
- A query that is fast on 1,000 rows may be slow on 10 million rows.

Main performance goals:
- fewer rows scanned
- fewer disk reads
- fewer unnecessary joins
- better index usage
- smaller result sets


## 2. Query Optimization Tips

### 2.1 Filter Early

Apply selective filters as early as possible.

```sql
SELECT order_id, customer_id, total_amount
FROM orders
WHERE order_status = 'PAID'
  AND order_date >= '2026-01-01';
```

This is better than fetching everything and filtering later in application code.

### 2.2 Use Sargable Conditions

A condition is sargable when MySQL can use an index efficiently.

Bad:
```sql
SELECT *
FROM orders
WHERE YEAR(order_date) = 2026;
```

Better:
```sql
SELECT *
FROM orders
WHERE order_date >= '2026-01-01'
  AND order_date < '2027-01-01';
```

### 2.3 Join Only Needed Tables

Join only when needed and only on indexed keys.

```sql
SELECT o.order_id, c.full_name
FROM orders o
JOIN customers c ON c.customer_id = o.customer_id
WHERE o.order_id = 5001;
```

### 2.4 Use LIMIT for Large Result Sets

If you only need the first few rows, request only those rows.

```sql
SELECT order_id, order_date, total_amount
FROM orders
ORDER BY order_date DESC
LIMIT 20;
```

### 2.5 Avoid Repeating Expensive Subqueries

If a subquery is reused, consider CTEs, derived tables, or summary tables.

### 2.6 Return Only Needed Columns

Selecting fewer columns reduces I/O and memory.

### 2.7 Prefer Set-Based SQL

Avoid row-by-row processing when a single set-based query can do the job.


## 3. EXPLAIN Command

`EXPLAIN` shows how MySQL plans to execute a query.

Basic use:
```sql
EXPLAIN
SELECT *
FROM orders
WHERE customer_id = 101;
```

Useful output columns:
- `id`: query block identifier
- `select_type`: type of select
- `table`: table being used
- `type`: access method
- `possible_keys`: candidate indexes
- `key`: chosen index
- `rows`: estimated rows examined
- `Extra`: additional execution details

### 3.1 Access Type Meaning

Common `type` values:
- `const`: very fast, single-row access
- `ref`: index lookup on non-unique values
- `range`: index range scan
- `index`: full index scan
- `ALL`: full table scan

Typical goal:
- avoid `ALL` when possible on large tables
- prefer `const`, `ref`, or `range`

### 3.2 EXPLAIN ANALYZE

`EXPLAIN ANALYZE` runs the query and shows actual timing.

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE customer_id = 101;
```

Use this when you want real runtime behavior, not just a plan estimate.

### 3.3 EXPLAIN FORMAT=JSON

Useful for deeper optimizer details.

```sql
EXPLAIN FORMAT=JSON
SELECT *
FROM orders
WHERE customer_id = 101;
```

### 3.4 Interpreting EXPLAIN Example

Example output idea:
- `type = ALL` means full scan
- `rows = 500000` means many rows examined
- `key = idx_orders_customer_id` means index is used
- `Extra = Using index` may indicate covering index usage


## 3.5 Query Execution Order (Important)

This is useful for understanding why some queries are slow and why some filters do not behave as expected.

Simplified logical execution order:
1. `FROM` / `JOIN`
2. `WHERE`
3. `GROUP BY`
4. `HAVING`
5. `SELECT`
6. Window functions (if any)
7. `ORDER BY`
8. `LIMIT`

Important implication:
- filtering happens before sorting
- aggregation happens before `HAVING`
- `ORDER BY` can add cost if it must sort many rows


## 4. Index Usage

Indexes are the biggest performance tool in SQL.

### 4.1 When Indexes Help

Indexes help most when columns are used in:
- `WHERE`
- `JOIN`
- `ORDER BY`
- `GROUP BY`
- `EXISTS`

### 4.2 Good Index Example

```sql
CREATE INDEX idx_orders_customer_date
ON orders(customer_id, order_date);
```

This can help queries like:
```sql
SELECT order_id, order_date, total_amount
FROM orders
WHERE customer_id = 101
ORDER BY order_date DESC;
```

### 4.3 Composite Index Order Matters

Order should match query patterns.

Good:
```sql
CREATE INDEX idx_sales_store_date
ON sales(store_id, sale_date);
```

Helpful for:
```sql
SELECT *
FROM sales
WHERE store_id = 10
  AND sale_date >= '2026-01-01';
```

### 4.4 Covering Index

A covering index contains all columns needed by the query.

Example:
```sql
CREATE INDEX idx_orders_covering
ON orders(customer_id, order_date, total_amount);
```

Query can be served from the index only:
```sql
SELECT customer_id, order_date, total_amount
FROM orders
WHERE customer_id = 101
ORDER BY order_date DESC;
```

### 4.5 Indexes Can Hurt Writes

Every extra index adds cost to `INSERT`, `UPDATE` and `DELETE`.

Best practice:
- index what you query often
- avoid duplicate and unused indexes


## 4.6 Index Selectivity

Index selectivity means how effectively an index narrows down rows.

- High selectivity:
  - many distinct values
  - good for filtering
  - example: email, order_id, phone

- Low selectivity:
  - few distinct values
  - weak filtering power
  - example: gender, is_active, status with very few values

Rule of thumb:
- higher selectivity usually means better index benefit

Example:
- indexing `email` is often better than indexing `gender`

Important nuance:
- low-selectivity columns can still be useful if combined in a composite index


## 5. Avoid `SELECT *`

`SELECT *` often causes unnecessary I/O and can slow queries.

Bad:
```sql
SELECT *
FROM orders
WHERE customer_id = 101;
```

Better:
```sql
SELECT order_id, order_date, total_amount
FROM orders
WHERE customer_id = 101;
```

Why avoid `SELECT *`:
- fetches extra columns you do not need
- can break covering index usage
- increases network payload
- can hide schema changes in application code

When `SELECT *` is acceptable:
- quick ad-hoc debugging
- small exploratory queries
- rarely in production code


## 6. Join Optimization

### 6.1 Join on Indexed Columns

```sql
SELECT o.order_id, c.full_name
FROM orders o
JOIN customers c ON c.customer_id = o.customer_id;
```

Make sure both join columns are indexed.

### 6.2 Filter Before Joining

```sql
SELECT o.order_id, c.full_name
FROM (
    SELECT order_id, customer_id
    FROM orders
    WHERE order_date >= '2026-01-01'
) o
JOIN customers c ON c.customer_id = o.customer_id;
```

This reduces rows before the join.

### 6.3 Avoid Unnecessary Join Multiplication

Be careful when joining tables that create many-to-many expansion.


## 6.4 Filesort & Temporary Table Warning

These often appear in `EXPLAIN` `Extra` column.

- `Using filesort`:
  - MySQL must sort rows in a separate step.
  - It does not necessarily mean disk sort, but it usually indicates extra sorting work.

- `Using temporary`:
  - MySQL creates a temporary table for intermediate results.
  - Common with some `GROUP BY`, `DISTINCT`, `ORDER BY`, or complex joins.

Why it matters:
- both can slow queries on large data sets

How to reduce them:
- use proper indexes
- reduce result set early
- align `ORDER BY` with index order where possible
- avoid unnecessary `DISTINCT` and wide joins


## 7. Pagination and Large Result Sets

### 7.1 Prefer Keyset Pagination for Big Tables

Offset pagination gets slower at high offsets.

Bad for large offsets:
```sql
SELECT order_id, order_date
FROM orders
ORDER BY order_id
LIMIT 20 OFFSET 50000;
```

Better:
```sql
SELECT order_id, order_date
FROM orders
WHERE order_id > 50000
ORDER BY order_id
LIMIT 20;
```

### 7.2 Use Stable Ordering

Always use a deterministic `ORDER BY` when paginating.


## 7.3 Batch Operations Tip

For large inserts, updates, or deletes:
- process in chunks instead of one huge transaction
- commit periodically if the business logic allows it
- reduce lock time and undo log growth

Example chunking idea:
```sql
DELETE FROM app_logs
WHERE created_at < '2026-01-01'
LIMIT 1000;
```

Run repeatedly in batches rather than one massive delete when safe.


## 8. Avoid Functions on Indexed Columns

Functions on indexed columns often prevent efficient index use.

Bad:
```sql
SELECT *
FROM employees
WHERE DATE(created_at) = '2026-04-01';
```

Better:
```sql
SELECT *
FROM employees
WHERE created_at >= '2026-04-01'
  AND created_at < '2026-04-02';
```


## 9. Reduce Data Early

Use filters, aggregation and subqueries to cut data size early.

Example:
```sql
SELECT customer_id, COUNT(*) AS order_count
FROM orders
WHERE order_date >= '2026-01-01'
GROUP BY customer_id;
```

This is often better than reading all rows into application code.


## 10. Use Proper Datatypes

Datatype choice affects storage and performance.

Examples:
- use `INT` for numeric IDs
- use `VARCHAR` for variable text
- use `DECIMAL` for money
- use `DATE` / `DATETIME` for dates

Avoid oversized columns when not needed.


## 11. Query Plan Symptoms to Watch For

Warning signs:
- full table scan on a large table
- filesort on a huge dataset
- too many rows examined
- missing index in join/filter column
- query returns many unnecessary columns

If a query is slow:
1. check `EXPLAIN`
2. verify indexes
3. inspect filtering logic
4. reduce result set
5. test again with `EXPLAIN ANALYZE`


## 11.1 Slow Query Log

- The slow query log records queries that exceed a configured threshold.
- It is one of the most practical ways to find real performance problems in production.

Use it to identify:
- long-running queries
- frequent offenders
- queries missing indexes

MySQL server variables often involved:
- `slow_query_log`
- `long_query_time`

High-level idea:
- enable it in non-trivial environments
- analyze the logged statements
- tune the worst ones first


## 12. Real Example: Slow Query vs Optimized Query

### Slow query

```sql
SELECT *
FROM orders
WHERE YEAR(order_date) = 2026
  AND customer_id = 101;
```

Problems:
- function on indexed column
- `SELECT *`

### Optimized query

```sql
SELECT order_id, order_date, total_amount
FROM orders
WHERE order_date >= '2026-01-01'
  AND order_date < '2027-01-01'
  AND customer_id = 101;
```

Add index:
```sql
CREATE INDEX idx_orders_customer_date
ON orders(customer_id, order_date);


## 12.1 Caching Concept (High-Level)

Caching reduces repeated work by reusing previously computed or fetched data.

High-level forms in a MySQL system:
- application cache
- query result cache in the application layer
- page cache / buffer pool effects inside the database engine

Important note:
- caching can help a lot, but it does not replace good SQL or proper indexes
- always treat cache as a performance layer, not a correctness layer


## 12.2 Partitioning (1–2 Lines)

- Partitioning splits a large table into smaller internal pieces based on a rule such as date or range.
- It can help manage very large tables, but it is not a substitute for indexing or good query design.
```


## 13. Real Example: E-commerce Order Lookup

Goal: show a customer’s recent paid orders quickly.

```sql
SELECT order_id, order_date, total_amount
FROM orders
WHERE customer_id = 101
  AND order_status = 'PAID'
ORDER BY order_date DESC
LIMIT 10;
```

Helpful index:
```sql
CREATE INDEX idx_orders_customer_status_date
ON orders(customer_id, order_status, order_date);
```


## 14. Real Example: Hospital Appointment Search

Goal: find doctor appointments for a day.

```sql
SELECT appointment_id, patient_id, appointment_date, appointment_status
FROM appointments
WHERE doctor_id = 12
  AND appointment_date >= '2026-04-09'
  AND appointment_date < '2026-04-10'
ORDER BY appointment_date;
```

Helpful index:
```sql
CREATE INDEX idx_appointments_doctor_date
ON appointments(doctor_id, appointment_date);
```


## 15. Performance Tips Checklist

- Use `EXPLAIN` before and after tuning.
- Index columns used in filters and joins.
- Avoid `SELECT *` in production code.
- Keep conditions sargable.
- Return only needed rows and columns.
- Use covering indexes where appropriate.
- Prefer keyset pagination for large offsets.
- Avoid functions on indexed columns.
- Review slow queries regularly.
- Watch for filesort and temporary tables in `EXPLAIN`.
- Use chunked batch operations for large data changes when possible.
- Use the slow query log to find real production bottlenecks.


## 16. Performance Anti-Patterns

- full table scans on large tables
- selecting unnecessary columns
- joining on unindexed columns
- using `OFFSET` for deep pagination
- functions around indexed columns
- building wide tables with unnecessary data
- too many unhelpful indexes


## 17. Quick Interview Notes

- Query optimization means reducing rows, I/O and unnecessary work.
- `EXPLAIN` shows the execution plan.
- `EXPLAIN ANALYZE` shows actual runtime behavior.
- Indexes help `WHERE`, `JOIN` and `ORDER BY`.
- Avoid `SELECT *` in production.
- Sargable predicates are key to index usage.
- High-selectivity indexes are generally more valuable than low-selectivity ones.
- `Using filesort` and `Using temporary` are common performance warnings.
- The slow query log is one of the best tools for finding slow production queries.


## 18. Final Summary

- Performance optimization starts with query shape and indexing.
- Use `EXPLAIN` to understand the plan instead of guessing.
- Avoid `SELECT *`, non-sargable filters and unnecessary joins.
- Use composite and covering indexes where they match query patterns.
- A small query rewrite can often produce a large speedup.

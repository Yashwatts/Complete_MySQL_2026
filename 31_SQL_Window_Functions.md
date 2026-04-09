## SQL Window Functions

Window functions are one of the most powerful SQL features for analytics. They let you calculate values across related rows without collapsing the result into one row per group.


## 1. What Are Window Functions?

- A window function performs a calculation across a set of rows related to the current row.
- The related set of rows is called a window.
- Unlike `GROUP BY`, window functions usually keep each original row in the result.

Key difference:
- `GROUP BY` reduces rows.
- Window functions preserve rows and add analytical columns.

Example idea:
- Show each employee with salary rank in their department.
- Show each order with running total over time.


## 2. Basic Syntax

```sql
window_function() OVER (
    PARTITION BY column1
    ORDER BY column2
)
```

Parts of `OVER`:
- `PARTITION BY`: splits rows into groups (optional).
- `ORDER BY`: defines order inside each partition (often required).
- Frame clause (`ROWS BETWEEN ...`): controls which rows are included for the calculation.


## 2.1 Execution Order (Very Important)

This is a common interview trap: window functions are not evaluated first.

Typical logical flow (simplified):
1. `FROM` / `JOIN`
2. `WHERE`
3. `GROUP BY`
4. `HAVING`
5. Window functions (`... OVER (...)`)
6. `SELECT`
7. `ORDER BY`
8. `LIMIT`

Important implication:
- You usually cannot filter window-function aliases directly in `WHERE`.
- Use a subquery/CTE, then filter in outer query.


## 3. Why Use Window Functions?

Common use cases:
- ranking rows
- top N per group
- running totals
- moving averages
- comparing current row vs previous/next row
- percentiles and distribution analysis

Benefits:
- cleaner queries
- fewer self-joins
- strong readability for analytics


## 4. Demo Table

Use this sample data for examples:

```sql
CREATE TABLE employee_sales (
    sale_id INT PRIMARY KEY,
    employee_name VARCHAR(100),
    department VARCHAR(50),
    sale_date DATE,
    amount DECIMAL(10,2)
);

INSERT INTO employee_sales (sale_id, employee_name, department, sale_date, amount)
VALUES
    (1, 'Aman',  'Electronics', '2026-01-01', 1200),
    (2, 'Aman',  'Electronics', '2026-01-03',  900),
    (3, 'Riya',  'Electronics', '2026-01-02', 1200),
    (4, 'Riya',  'Electronics', '2026-01-05', 1500),
    (5, 'Neha',  'Fashion',     '2026-01-01',  700),
    (6, 'Neha',  'Fashion',     '2026-01-04',  700),
    (7, 'Arjun', 'Fashion',     '2026-01-02',  950);
```


## 5. ROW_NUMBER()

- Assigns a unique sequential number to each row inside a partition.
- No ties: even equal values get different row numbers.

Example:
```sql
SELECT employee_name,
       department,
       amount,
       ROW_NUMBER() OVER (
           PARTITION BY department
           ORDER BY amount DESC
       ) AS row_num
FROM employee_sales;
```

Use case:
- pick top 1 row per group using `ROW_NUMBER() = 1`.

Top sale per department:
```sql
SELECT *
FROM (
    SELECT sale_id,
           employee_name,
           department,
           amount,
           ROW_NUMBER() OVER (
               PARTITION BY department
               ORDER BY amount DESC
           ) AS rn
    FROM employee_sales
) t
WHERE rn = 1;
```


## 6. RANK()

- Gives rank numbers with ties.
- If two rows tie at rank 1, next rank becomes 3 (gap appears).

Example:
```sql
SELECT employee_name,
       department,
       amount,
       RANK() OVER (
           PARTITION BY department
           ORDER BY amount DESC
       ) AS sales_rank
FROM employee_sales;
```

Behavior with tie example:
- scores: 1500, 1200, 1200, 900
- ranks: 1, 2, 2, 4


## 7. DENSE_RANK()

- Similar to `RANK()` but no gaps after ties.

Example:
```sql
SELECT employee_name,
       department,
       amount,
       DENSE_RANK() OVER (
           PARTITION BY department
           ORDER BY amount DESC
       ) AS dense_sales_rank
FROM employee_sales;
```

Behavior with same tie example:
- scores: 1500, 1200, 1200, 900
- dense ranks: 1, 2, 2, 3


## 8. ROW_NUMBER vs RANK vs DENSE_RANK (Quick Comparison)

- `ROW_NUMBER()`:
  - always unique sequence
  - ties are forced into different numbers

- `RANK()`:
  - ties share rank
  - gaps appear after ties

- `DENSE_RANK()`:
  - ties share rank
  - no gaps after ties

Practical rule:
- Need unique pick per group: `ROW_NUMBER()`
- Need competition style ranking with gaps: `RANK()`
- Need compact ranking without gaps: `DENSE_RANK()`


## 9. PARTITION BY (Very Important)

`PARTITION BY` divides data into independent groups before window calculation.

Without `PARTITION BY`:
- function runs on the entire result set.

With `PARTITION BY department`:
- each department has its own numbering/ranking/running total.

Example without partition:
```sql
SELECT employee_name,
       department,
       amount,
       ROW_NUMBER() OVER (ORDER BY amount DESC) AS overall_row_num
FROM employee_sales;
```

Example with partition:
```sql
SELECT employee_name,
       department,
       amount,
       ROW_NUMBER() OVER (
           PARTITION BY department
           ORDER BY amount DESC
       ) AS dept_row_num
FROM employee_sales;
```


## 10. Running Totals

Running total means cumulative sum up to the current row.

Example by department and date:
```sql
SELECT employee_name,
       department,
       sale_date,
       amount,
       SUM(amount) OVER (
           PARTITION BY department
           ORDER BY sale_date
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS running_total
FROM employee_sales
ORDER BY department, sale_date;
```

Explanation:
- `UNBOUNDED PRECEDING` = from first row in partition
- `CURRENT ROW` = up to current row

So each row gets cumulative amount till that date in its department.


## 11. More Useful Window Examples

### 11.1 Department Total with Row-Level Detail

```sql
SELECT employee_name,
       department,
       amount,
       SUM(amount) OVER (PARTITION BY department) AS dept_total
FROM employee_sales;
```

### 11.2 Department Average and Difference from Average

```sql
SELECT employee_name,
       department,
       amount,
       AVG(amount) OVER (PARTITION BY department) AS dept_avg,
       amount - AVG(amount) OVER (PARTITION BY department) AS diff_from_avg
FROM employee_sales;
```

### 11.3 Previous Row Comparison (LAG)

```sql
SELECT employee_name,
       department,
       sale_date,
       amount,
       LAG(amount) OVER (
           PARTITION BY employee_name
           ORDER BY sale_date
       ) AS prev_amount,
       amount - LAG(amount) OVER (
           PARTITION BY employee_name
           ORDER BY sale_date
       ) AS change_from_prev
FROM employee_sales;
```

### 11.4 Next Row Comparison (LEAD)

```sql
SELECT employee_name,
       department,
       sale_date,
       amount,
       LEAD(amount) OVER (
           PARTITION BY employee_name
           ORDER BY sale_date
       ) AS next_amount
FROM employee_sales;
```

### 11.5 NTILE()

- `NTILE(n)` splits ordered rows into `n` buckets.
- Useful for quartiles, deciles and segment-based analysis.

Example (quartiles per department):
```sql
SELECT employee_name,
       department,
       amount,
       NTILE(4) OVER (
           PARTITION BY department
           ORDER BY amount DESC
       ) AS quartile
FROM employee_sales;
```

### 11.6 FIRST_VALUE() and LAST_VALUE()

- `FIRST_VALUE()` returns the first value in the window frame.
- `LAST_VALUE()` returns the last value in the window frame.

Important detail:
- `LAST_VALUE()` often needs an explicit frame to get the true last value of the partition.

Example:
```sql
SELECT employee_name,
       department,
       sale_date,
       amount,
       FIRST_VALUE(amount) OVER (
           PARTITION BY department
           ORDER BY sale_date
           ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
       ) AS first_amount_in_dept,
       LAST_VALUE(amount) OVER (
           PARTITION BY department
           ORDER BY sale_date
           ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
       ) AS last_amount_in_dept
FROM employee_sales;
```

### 11.7 Percentiles: PERCENT_RANK() and CUME_DIST()

- `PERCENT_RANK()` gives relative rank from 0 to 1.
- `CUME_DIST()` gives cumulative distribution from 0 to 1.

Example:
```sql
SELECT employee_name,
       department,
       amount,
       PERCENT_RANK() OVER (
           PARTITION BY department
           ORDER BY amount
       ) AS pct_rank,
       CUME_DIST() OVER (
           PARTITION BY department
           ORDER BY amount
       ) AS cume_dist
FROM employee_sales;
```

Use cases:
- percentile-based grading
- top 10 percent / bottom 10 percent segmentation
- distribution analysis in reporting


## 12. Window Frame Clause (Important Detail)

Frame controls which rows are included relative to current row.

Common forms:
- `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`
- `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW`
- `ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING`

Moving 3-row sum example:
```sql
SELECT employee_name,
       sale_date,
       amount,
       SUM(amount) OVER (
           PARTITION BY employee_name
           ORDER BY sale_date
           ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
       ) AS moving_sum_3
FROM employee_sales;
```


## 13. Window Functions vs GROUP BY

`GROUP BY` example:
```sql
SELECT department, SUM(amount) AS dept_total
FROM employee_sales
GROUP BY department;
```

This returns one row per department.

Window version:
```sql
SELECT employee_name,
       department,
       amount,
       SUM(amount) OVER (PARTITION BY department) AS dept_total
FROM employee_sales;
```

This keeps all rows and adds department total per row.


## 13.1 Window Functions vs Subqueries

Both can solve analytical problems, but window functions are often cleaner.

- Subquery approach:
    - may require self-joins/correlated subqueries
    - can be verbose and harder to maintain

- Window approach:
    - keeps row-level detail and aggregate/rank in same result
    - usually easier to read for analytics

Example problem: department total per row

Subquery style:
```sql
SELECT e.employee_name,
             e.department,
             e.amount,
             d.dept_total
FROM employee_sales e
JOIN (
        SELECT department, SUM(amount) AS dept_total
        FROM employee_sales
        GROUP BY department
) d ON d.department = e.department;
```

Window style:
```sql
SELECT employee_name,
             department,
             amount,
             SUM(amount) OVER (PARTITION BY department) AS dept_total
FROM employee_sales;
```

Interview-friendly conclusion:
- Subqueries are foundational.
- Window functions are usually the preferred analytical pattern when supported.


## 14. Performance Notes

Window functions are powerful but can be heavy on large datasets.

Performance tips:
- Index partition/order columns (for example `department`, `sale_date`).
- Keep partitions reasonable.
- Avoid unnecessary wide selects.
- Test with `EXPLAIN`.

Useful index for examples:
```sql
CREATE INDEX idx_emp_sales_dept_date
ON employee_sales(department, sale_date);
```


## 15. Common Mistakes

- Forgetting `ORDER BY` for ranking or running totals.
- Confusing `RANK()` vs `DENSE_RANK()` behavior.
- Expecting window functions in `WHERE` directly.
- Using windows where simple aggregate is enough.

Important:
- In many SQL engines, window functions are computed after `WHERE`.
- Use subquery/CTE if you need to filter by window result.

Example:
```sql
SELECT *
FROM (
    SELECT employee_name,
           department,
           amount,
           ROW_NUMBER() OVER (
               PARTITION BY department
               ORDER BY amount DESC
           ) AS rn
    FROM employee_sales
) t
WHERE rn <= 2;
```


## 15.1 WHERE vs HAVING vs WINDOW (Power Clarification)

- `WHERE`:
    - filters rows before grouping and before window calculations.

- `HAVING`:
    - filters grouped results after `GROUP BY`.

- Window functions:
    - computed after `WHERE` and `HAVING`.
    - typically not directly usable in `WHERE` of the same query block.

Pattern to filter by window result:
```sql
WITH ranked_sales AS (
        SELECT employee_name,
                     department,
                     amount,
                     ROW_NUMBER() OVER (
                             PARTITION BY department
                             ORDER BY amount DESC
                     ) AS rn
        FROM employee_sales
)
SELECT *
FROM ranked_sales
WHERE rn <= 3;
```

MySQL note:
- MySQL does not support `QUALIFY`, so subquery/CTE is the standard way.


## 16. Real-World Use Cases

- Leaderboards by category/department.
- Top N products per category.
- Sales running totals over time.
- Month-over-month comparison.
- Detecting first/last transaction per customer.
- Identifying salary rank within department.


## 17. Quick Interview Notes

- Window functions analyze related rows without collapsing final row count.
- `ROW_NUMBER()` gives unique sequence.
- `RANK()` has gaps after ties.
- `DENSE_RANK()` has no gaps after ties.
- `NTILE(n)` splits rows into buckets.
- `PERCENT_RANK()` and `CUME_DIST()` are useful for percentile/distribution analysis.
- `PARTITION BY` resets calculation per group.
- Running totals are usually `SUM(...) OVER (PARTITION BY ... ORDER BY ... )`.
- Window functions are evaluated after `WHERE` and `HAVING`, so filter them using subquery/CTE.


## 18. Final Summary

- Window functions are essential for modern SQL analytics.
- They provide ranking, comparison and cumulative calculations.
- `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `PARTITION BY` and running totals are core patterns.
- They often replace complex self-joins with cleaner SQL.
- Mastering window functions makes reporting and data analysis much easier.

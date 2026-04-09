## SQL CTE (Common Table Expressions)

Common Table Expressions (CTEs) make SQL queries cleaner, more modular and easier to debug. They are especially useful for complex analytics and recursive problems such as hierarchy traversal.


## 1. What Is a CTE?

- A CTE is a temporary named result set defined within a query.
- It exists only for the duration of that query.
- It improves readability by breaking complex logic into steps.
- It can be used in `SELECT`, `INSERT`, `UPDATE` and `DELETE` statements.

Think of a CTE as:
- a temporary query block
- a readable alternative to deeply nested subqueries
- a building block for recursive queries


## 2. WITH Clause (Core Syntax)

The `WITH` clause defines one or more CTEs before the main query.

Basic syntax:
```sql
WITH cte_name AS (
    SELECT ...
)
SELECT ...
FROM cte_name;
```

Example:
```sql
WITH high_salary_employees AS (
    SELECT emp_id, full_name, salary
    FROM employees
    WHERE salary > 80000
)
SELECT *
FROM high_salary_employees
ORDER BY salary DESC;
```


## 2.1 Execution Order (Important)

This is an interview-favorite topic.

Key idea:
- `WITH` defines query blocks first (CTE definitions), then the main query uses them.
- Inside the main query, logical processing still follows SQL order (`FROM` -> `WHERE` -> `GROUP BY` -> `HAVING` -> `SELECT` -> `ORDER BY` -> `LIMIT`).

For recursive CTEs specifically:
1. Anchor member runs first.
2. Recursive member runs repeatedly using prior results.
3. Recursion stops when no new rows are produced or depth limit is hit.
4. Final combined CTE result is consumed by outer query.

Important implication:
- Filters placed in anchor/recursive parts can greatly reduce recursion cost.


## 3. Why Use CTEs?

CTEs are useful when:
- a query has multiple logical steps
- subqueries become hard to read
- you need to reuse intermediate result sets
- you need recursion (hierarchies, sequences, paths)

Benefits:
- better readability
- easier maintenance
- simpler debugging


## 4. Multiple CTEs in One Query

You can define more than one CTE in a single `WITH` block.

```sql
WITH dept_totals AS (
    SELECT department_id, SUM(salary) AS total_salary
    FROM employees
    GROUP BY department_id
),
avg_totals AS (
    SELECT AVG(total_salary) AS avg_department_salary
    FROM dept_totals
)
SELECT d.department_id,
       d.total_salary,
       a.avg_department_salary
FROM dept_totals d
CROSS JOIN avg_totals a;
```

This pattern is very useful for step-by-step analytical logic.


## 5. CTE vs Subquery vs Temporary Table

### CTE
- Best for readability and step-wise query logic.
- Exists only during query execution.

### Subquery
- Useful for small inline calculations.
- Can become messy when deeply nested.

### Temporary Table
- Useful when intermediate data is reused across multiple separate statements.
- Requires explicit create/drop lifecycle.

Rule of thumb:
- Single complex query: prefer CTE.
- Reused intermediate result in many queries: consider temporary table.


## 5.1 CTE vs View (Common Question)

- CTE:
    - Exists only for one statement.
    - Defined inline using `WITH`.
    - Great for query-specific step-by-step logic.

- View:
    - Persistent schema object.
    - Created once and reused across many queries.
    - Useful for abstraction and access control.

Quick interview summary:
- CTE is temporary and statement-scoped.
- View is permanent (until dropped) and reusable.


## 5.2 Materialization vs Inline (Advanced Insight)

MySQL optimizer may handle CTEs in different ways:

- Inline (merged):
    - CTE logic is merged into the outer query.
    - Can allow more optimizer pushdown and fewer temp structures.

- Materialized:
    - CTE is evaluated and stored as an internal temporary result first.
    - Outer query then reads from that temporary result.

Why it matters:
- Materialization can be helpful when CTE result is reused.
- But it can also add memory/disk overhead for large intermediate sets.

How to validate:
- Use `EXPLAIN` / `EXPLAIN ANALYZE` to inspect actual execution behavior.
- Do not assume CTE is always faster than subquery.


## 6. Recursive CTE (Very Important)

Recursive CTEs solve hierarchical and iterative problems.

MySQL recursive syntax:
```sql
WITH RECURSIVE cte_name AS (
    -- Anchor member (starting rows)
    SELECT ...

    UNION ALL

    -- Recursive member (refers to cte_name)
    SELECT ...
    FROM ...
    JOIN cte_name ...
)
SELECT *
FROM cte_name;
```

Two required parts:
- Anchor query: starting point.
- Recursive query: repeatedly expands result until no new rows are produced.


## 7. Recursive Query Example: Employee Hierarchy

Assume table:
```sql
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    full_name VARCHAR(100),
    manager_id INT NULL
);
```

Goal:
- Get complete reporting tree from CEO down.

```sql
WITH RECURSIVE org_chart AS (
    -- Anchor: top-level employees (no manager)
    SELECT emp_id,
           full_name,
           manager_id,
           1 AS level,
           CAST(full_name AS CHAR(500)) AS hierarchy_path
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive: fetch direct reports of previous level
    SELECT e.emp_id,
           e.full_name,
           e.manager_id,
           oc.level + 1 AS level,
           CONCAT(oc.hierarchy_path, ' -> ', e.full_name) AS hierarchy_path
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.emp_id
)
SELECT *
FROM org_chart
ORDER BY hierarchy_path;
```

This gives each employee with:
- depth level
- full reporting path


## 8. Recursive Query Example: Number Series

Generate numbers from 1 to 10:

```sql
WITH RECURSIVE numbers AS (
    SELECT 1 AS n

    UNION ALL

    SELECT n + 1
    FROM numbers
    WHERE n < 10
)
SELECT *
FROM numbers;
```

Useful for:
- calendar generation
- iterative expansion
- gap detection helper sets


## 9. Recursive Query Example: Date Calendar

```sql
WITH RECURSIVE calendar AS (
    SELECT DATE('2026-01-01') AS dt

    UNION ALL

    SELECT DATE_ADD(dt, INTERVAL 1 DAY)
    FROM calendar
    WHERE dt < '2026-01-10'
)
SELECT dt
FROM calendar;
```

This can be joined to sales tables for missing-date reports.


## 10. Preventing Infinite Recursion

Recursive CTEs must have a termination condition.

Always include conditions like:
- max depth (`level < 20`)
- end value (`n < 100`)
- path checks for cycle prevention

MySQL also has recursion depth protection:
- system variable `cte_max_recursion_depth` (default often 1000)

Check value:
```sql
SHOW VARIABLES LIKE 'cte_max_recursion_depth';
```

Set for session if needed:
```sql
SET SESSION cte_max_recursion_depth = 2000;
```

Use carefully; higher limits can increase query cost.


## 11. Cycle Handling in Hierarchies

Real hierarchies may have bad data loops (A -> B -> C -> A).

Simple protection strategy:
- keep path string
- stop when node already exists in path

Conceptual pattern:
```sql
... WHERE FIND_IN_SET(e.emp_id, oc.visited_ids) = 0
```

This avoids re-visiting same node in recursion.


## 12. CTE in INSERT, UPDATE, DELETE

### 12.1 INSERT Using CTE

```sql
WITH top_performers AS (
    SELECT emp_id, full_name, salary
    FROM employees
    WHERE salary > 100000
)
INSERT INTO bonus_candidates(emp_id, full_name, salary)
SELECT emp_id, full_name, salary
FROM top_performers;
```

### 12.2 UPDATE Using CTE

```sql
WITH low_stock AS (
    SELECT product_id
    FROM products
    WHERE stock_quantity < 10
)
UPDATE products p
JOIN low_stock ls ON p.product_id = ls.product_id
SET p.reorder_flag = 1;
```

### 12.3 DELETE Using CTE

```sql
WITH old_logs AS (
    SELECT log_id
    FROM app_logs
    WHERE created_at < DATE_SUB(CURDATE(), INTERVAL 90 DAY)
)
DELETE l
FROM app_logs l
JOIN old_logs o ON l.log_id = o.log_id;
```


## 13. Performance Considerations

CTEs improve readability, but performance depends on query logic and indexes.

Important points:
- CTEs are not automatically faster than subqueries.
- Recursive CTEs can be expensive on large trees.
- Index join keys used in recursive member (for example `manager_id`).
- Keep selected columns minimal.
- Test with `EXPLAIN`.

Helpful index for hierarchy:
```sql
CREATE INDEX idx_employees_manager_id
ON employees(manager_id);
```


## 14. Limitations of CTE

- Scope is single statement only (cannot reuse in later statements).
- You cannot create indexes directly on a CTE result.
- Large CTE intermediate results may consume memory/temp storage.
- Recursive CTEs are limited by termination logic and recursion depth settings.
- Overusing CTE layers can hurt readability instead of improving it.


## 15. Common Mistakes

- Forgetting `RECURSIVE` keyword in recursive CTE.
- Missing termination condition.
- Using `UNION` when `UNION ALL` is needed (can add unnecessary dedup overhead).
- Building recursive queries without proper indexing.
- Overusing CTEs for very simple queries.


## 16. WITH Clause Best Practices

- Use meaningful CTE names (`dept_totals`, `org_chart`, `calendar`).
- Keep each CTE focused on one logical step.
- Prefer multiple small CTEs over one giant unreadable query.
- Use comments in complex recursive logic.
- Validate recursion depth and termination behavior.


## 17. Real-World Use Cases

- Organization hierarchy and reporting chains
- Category/subcategory tree traversal
- Bill-of-material expansion
- Date calendar generation
- Running staged calculations in analytics
- Cleaning and transforming data in readable steps


## 18. Quick Interview Notes

- CTE is defined using `WITH`.
- Recursive CTE requires `WITH RECURSIVE`.
- Recursive CTE has anchor part + recursive part.
- Termination condition is mandatory to avoid infinite loops.
- CTE execution may be inline or materialized depending on optimizer decisions.
- CTE is statement-scoped; view is persistent schema object.
- CTE improves readability; performance still depends on execution plan.


## 19. Final Summary

- CTEs make SQL modular and readable.
- `WITH` clause is the core entry point for CTE usage.
- Recursive CTEs are essential for hierarchies and sequence generation.
- Proper indexing and termination logic are key for performance and safety.
- Mastering CTEs is a major step toward advanced SQL querying.

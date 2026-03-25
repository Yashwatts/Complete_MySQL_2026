## 1. Basic SELECT Syntax

The SELECT statement is used to retrieve data from a database.

- Select Specific Columns:

Example:

```sql
SELECT column1, column2, ...
FROM table_name;
```

- Select All Columns:

Example:

```sql
SELECT * FROM table_name;
```

- Column Aliases (AS):
  Use the AS keyword to rename a column in the output for better readability. This does not change the original table.

Example:

```sql
SELECT first_name AS 'First Name' FROM employees;
```


## 2. Filtering and Sorting

- WHERE Clause: Used to filter records based on specific conditions.

Example:

```sql
SELECT * FROM employees
WHERE department = 'IT';
```

- ORDER BY: Used to sort the result set in ascending (ASC, default) or descending (DESC) order.

Example:

```sql
SELECT * FROM employees
ORDER BY Salary DESC;
```

- LIMIT: Specifies the number of records (rows) to return. Useful for finding the "top" items (e.g., highest salary).

Example:

```sql
SELECT * FROM employees LIMIT 2;

SELECT * FROM employees
ORDER BY Salary DESC LIMIT 1; -- Return the person with highest salary
```


## 3. Data Refinement

- DISTINCT: Returns only unique (different) values, removing duplicates from the result.

Example:

```sql
SELECT DISTINCT department FROM employees;
```

- Mathematical Expressions: You can perform calculations directly within a SELECT statement.

Example:

```sql
SELECT first_name, last_name, salary*1.1 AS 'Salary After Raise' FROM employees;
```


## 4. Built-in Functions

MySQL provides several functions to manipulate data during retrieval.

- CONCAT(): Joins two or more strings into one.

Example:

```sql
SELECT concat(first_name, ' ', last_name) AS 'Full Name' FROM employees;
```

- YEAR(): Extracts the year part from a date field.

Example:

```sql
SELECT concat(first_name, ' ', last_name) AS 'Full Name' , YEAR(hire_date) AS 'Hiring Year' FROM employees;
```

- ROUND(): Rounds a numeric field to a specified number of decimals.

Example:

```sql
SELECT concat(first_name, ' ', last_name) AS 'Full Name' , YEAR(hire_date) AS 'Hiring Year' , ROUND(salary, 1) AS 'Rounded Salary' FROM employees
WHERE salary > 70000;
```

- NOW(): Returns the current date and time.

Example:

```sql
SELECT NOW() AS 'Time';
```


## 5. Advanced Concepts

- Subqueries: A query nested inside another query.
  Example: Finding employees whose salary is greater than average.

```sql
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

- UNION: Combines the result-set of two or more SELECT statements.

Example:

```sql
SELECT first_name, last_name FROM employees WHERE department = 'IT' UNION
SELECT first_name, last_name FROM employees WHERE department = 'HR'
```

- GROUP BY: Groups rows that have the same values into summary rows (e.g., finding the number of employees in each department).

Example:

```sql
SELECT COUNT(*), department FROM employees GROUP BY department;
```


## 6. SELECT as an Expression Evaluator

The SELECT statement can be used without a table to evaluate expressions, act as a calculator or test functions.

Example:

```sql
SELECT 5 * 2;
```

or

```sql
SELECT LENGTH('Hello');
```

or

```sql
SELECT NOW() AS 'Time';
```

or

```sql
SELECT 5 > 3;
```
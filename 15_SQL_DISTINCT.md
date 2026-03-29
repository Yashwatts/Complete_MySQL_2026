## Introduction to DISTINCT

- The `DISTINCT` keyword is used in a `SELECT` statement to eliminate duplicate rows from the result set.
- It ensures that only unique values are returned for the specified columns.
- **Key Behavior:** It treats `NULL` as a unique value. If multiple rows have `NULL` in the distinct column, only one `NULL` will be returned in the results.

**Basic Syntax:**

```sql
SELECT DISTINCT column_name FROM table_name;
```

---

## Example Setup

```sql
-- Create and use the database
CREATE DATABASE EmployeeDB;
USE EmployeeDB;

-- Create employees table
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50),
    department VARCHAR(50),
    salary DECIMAL(10,2)
);

-- Insert sample data including duplicates
INSERT INTO employees (name, department, salary) VALUES
    ('Alice', 'HR', 50000),
    ('Bob', 'Finance', 60000),
    ('Charlie', 'IT', 70000),
    ('Alice', 'HR', 50000),      -- Duplicate record
    ('David', 'Finance', 55000),
    ('Eve', 'IT', 70000),        -- Duplicate salary
    ('Frank', 'HR', 50000);      -- Duplicate department & salary

-- View all employees
SELECT * FROM employees;
```

---

## Usage Scenarios

### A. Single Column
- To find all unique values in a specific field (e.g., finding all unique departments in an employee table):

```sql
SELECT DISTINCT department
FROM employees;
```
- Instead of seeing "HR" multiple times for every employee, you will only see "HR", "Finance" and "IT" once.

### B. Multiple Columns
- When you use `DISTINCT` with multiple columns, MySQL looks for unique combinations of those columns.

```sql
SELECT DISTINCT department, salary
FROM employees;
```
- This will return unique pairs of departments and salaries. If two employees are in "HR" and both earn "50,000", only one row for that combination is shown.

### C. With Aggregate Functions
- You can combine `DISTINCT` with functions like `COUNT()` to find the number of unique entries.

```sql
SELECT COUNT(DISTINCT department) AS unique_departments
FROM employees;
```

### D. With Sorting and Filtering
- **Order By:** You can sort distinct results using `ORDER BY`.

    - Get unique salaries in descending order:
    ```sql
    SELECT DISTINCT salary
    FROM employees
    ORDER BY salary DESC;
    ```

- **WHERE Clause:** You can filter data before applying distinct logic.

    - Get unique departments where salary is greater than 50000:
    ```sql
    SELECT DISTINCT department
    FROM employees
    WHERE salary > 50000;
    ```

---

## Advanced Applications

- **Functions:** You can use `DISTINCT` with other functions like `CONCAT()` to create and identify unique strings from multiple fields.

    - Get unique name-department combinations:
    ```sql
    SELECT DISTINCT CONCAT(name, "-", department)
    FROM employees;
    ```

- **Practical Use Case:** A common use is extracting a unique list of email addresses from a large user database to avoid sending duplicate marketing emails.

---

## Handling NULL values with DISTINCT

```sql
-- Insert records with NULL departments
INSERT INTO employees (name, department, salary) VALUES
    ('Grace', NULL, 48000),
    ('Bobby', NULL, 48000);

-- Show how DISTINCT handles NULL values
SELECT DISTINCT department
FROM employees;
```

---

## Performance Considerations

- Using `DISTINCT` on very large datasets can be resource-intensive and slow down queries because the database has to compare every row.
- **Optimization:** Ensure that the columns you are applying `DISTINCT` to are indexed.
- **Production Warning:** Be cautious when running `DISTINCT` on massive production tables without indexing, as it can cause high memory usage and impact other queries.
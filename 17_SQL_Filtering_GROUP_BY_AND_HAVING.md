## 1. The GROUP BY Clause

- The GROUP BY clause is used to arrange identical data into groups. It is most commonly used with Aggregate Functions like COUNT(), SUM(), AVG(), MIN() and MAX() to summarize large datasets.
    - **Key Rules:**
        - Single Column Grouping: You can group by one column (e.g., GROUP BY department) to find metrics like "average salary per department".
        - Multiple Column Grouping: You can group by multiple columns (e.g., GROUP BY department, joining_year) to create sub-groups.
        - Restriction: You generally cannot use SELECT * with GROUP BY because MySQL won't know which specific row's data to show for non-grouped columns; you should only select the grouped columns and aggregate results.

## 2. The HAVING Clause

- The HAVING clause is used to filter data after the grouping has occurred. It is specifically designed to work with aggregate functions, which the WHERE clause cannot do.
    - **WHERE vs. HAVING:**

| Feature | WHERE | HAVING |
|---------|-------|--------|
| Execution | Filters rows before grouping. | Filters groups after grouping. |
| Aggregates | Cannot be used with functions like COUNT() or AVG(). | Can be used with aggregate functions. |

## 3. Order of Execution

- In a standard SQL query, the clauses are executed in this specific sequence:
    1. **SELECT**: Choose the columns to display.
    2. **FROM**: Identify the table.
    3. **WHERE**: Filter individual rows.
    4. **GROUP BY**: Group the filtered rows.
    5. **HAVING**: Filter the resulting groups.
    6. **ORDER BY**: Sort the final result.
    7. **LIMIT**: Restrict the number of rows returned.

## 4. Advanced Examples

- Conditional Grouping (CASE): You can use CASE statements to create custom ranges (e.g., Low, Medium, High Salary) and then group by those ranges to count employees in each bucket.
- Finding the Maximum: To find the department with the highest number of employees, you can combine GROUP BY, ORDER BY DESC and LIMIT 1.

## 5. Internal Working

- MySQL handles GROUP BY by scanning records and creating a temporary table in memory. It uses the grouping columns as a "Key" and updates the "Value" (the aggregate data, like a running count) as it iterates through the rows. 


## Database and Table Setup

- Database Setup:
    ```sql
    CREATE DATABASE db_for_group_by;
    USE db_for_group_by;
    ```
- Table Creation:
    ```sql
    CREATE TABLE employees (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(50),
        department VARCHAR(50),
        salary DECIMAL(10,2),
        joining_date DATE
    );
    ```
- Initial Data Insertion:
    ```sql
    INSERT INTO employees (name, department, salary, joining_date) VALUES
    ('Alice', 'HR', 50000, '2020-06-15'),
    ('Bob', 'HR', 55000, '2019-08-20'),
    ('Charlie', 'IT', 70000, '2018-03-25'),
    ('David', 'IT', 72000, '2017-07-10'),
    ('Eve', 'IT', 73000, '2021-02-15'),
    ('Frank', 'Finance', 60000, '2020-11-05'),
    ('Grace', 'Finance', 65000, '2019-05-30'),
    ('Hannah', 'Finance', 62000, '2021-01-12');
    ```
- Additional Data Insertion:
    ```sql
    INSERT INTO employees (name, department, salary, joining_date) VALUES
    ('Tim', 'HR', 65000, '2019-05-30'),
    ('Tom', 'IT', 62000, '2021-01-12');
    ```
- View All Employee Data:
    ```sql
    SELECT * FROM employees;
    ```

## Examples

- **Example 1: Count Employees in Each Department**
    ```sql
    SELECT department, COUNT(*) AS employee_count
    FROM employees
    GROUP BY department;
    ```
- **Example 2: Get the Average Salary Per Department**
    ```sql
    SELECT department, AVG(salary) AS average_salary
    FROM employees
    GROUP BY department;
    ```
- **Example 3: Get the Highest and Lowest Salary Per Department**
    ```sql
    SELECT department, MIN(salary) AS lowest_salary, MAX(salary) AS highest_salary
    FROM employees
    GROUP BY department;
    ```
- **Example 4: Count Employees Per Department and Joining Year**
    ```sql
    SELECT department, YEAR(joining_date) AS joining_year, COUNT(*) AS employee_count
    FROM employees
    GROUP BY joining_year, department;
    ```
- **Example 5: Order Departments by the Highest Average Salary**
    ```sql
    SELECT department, AVG(salary) as avg_salary
    FROM employees
    GROUP BY department
    ORDER BY avg_salary DESC;
    ```
- **Example 6: Group by Calculated Salary Range**
    ```sql
    SELECT
        CASE
            WHEN salary < 60000 THEN 'Low Salary'
            WHEN salary BETWEEN 60000 AND 70000 THEN 'Medium Salary'
            ELSE 'High Salary'
        END AS salary_range,
        COUNT(*) AS employee_count
    FROM employees
    GROUP BY salary_range;
    ```
- **Example 7: Find Department with the Maximum Number of Employees**
    ```sql
    SELECT department, COUNT(*) AS total_employees
    FROM employees
    GROUP BY department
    ORDER BY total_employees DESC
    LIMIT 1;
    ```
- **Example 8: Find Departments With More Than 2 Employees (With Conditions)**
    ```sql
    SELECT
        department,
        AVG(salary) AS average_salary,
        COUNT(*) AS total_employees
    FROM employees
    WHERE joining_date > '2017-07-10'
    GROUP BY department
    HAVING total_employees > 2 AND average_salary > 55000;
    ```
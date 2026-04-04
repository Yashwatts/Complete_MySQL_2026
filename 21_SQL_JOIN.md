## 1. SQL JOIN Types

- Joins are used to combine rows from two or more tables based on a related column.

| Join Type | Description |
|---|---|
| INNER JOIN | Returns records that have matching values in both tables. |
| LEFT JOIN | Returns all records from the left table and matched records from the right. |
| RIGHT JOIN | Returns all records from the right table and matched records from the left. |
| FULL JOIN | Returns all records when there is a match in either left or right table. |
| CROSS JOIN | Returns the Cartesian product (every possible combination) of both tables. |
| SELF JOIN | A regular join where a table is joined with itself. |


## 2. Deep Dive: Key Concepts

### Inner Join & Filtering

- Basic Syntax:

```sql
SELECT columns
FROM table1
JOIN_TYPE table2
ON table1.column = table2.column;
```

- Efficiency: Filtering AND withing the ON clause is often more efficient than using a WHERE clause because it filters data during the join process rather that after.


-- An INNER JOIN returns only the rows where there is a match in both tables based on the specified join condition.
-- If there's no match, the rows from both tables are excluded from the result set.

```sql

-- Create database
CREATE DATABASE db_inner_join;
USE db_inner_join;

-- Create authors table
CREATE TABLE authors (
    author_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    birth_year INT
);

-- Create books table
CREATE TABLE books (
    book_id INT PRIMARY KEY,
    title VARCHAR(100),
    author_id INT,
    publication_year INT,
    price DECIMAL(6,2)
);

-- Insert data into authors table
INSERT INTO authors (author_id, first_name, last_name, birth_year)
VALUES 
    (1, 'Jane', 'Austen', 1775),
    (2, 'George', 'Orwell', 1903),
    (3, 'Ernest', 'Hemingway', 1899),
    (4, 'Agatha', 'Christie', 1890),
    (5, 'J.K.', 'Rowling', 1965);

-- Insert data into books table
INSERT INTO books (book_id, title, author_id, publication_year, price)
VALUES 
    (101, 'Pride and Prejudice', 1, 1813, 12.99),
    (102, '1984', 2, 1949, 14.50),
    (103, 'Animal Farm', 2, 1945, 11.75),
    (104, 'The Old Man and the Sea', 3, 1952, 10.99),
    (105, 'Murder on the Orient Express', 4, 1934, 13.25),
    (106, 'Death on the Nile', 4, 1937, 12.50),
    (107, 'Emma', 1, 1815, 11.99),
    (108, 'For Whom the Bell Tolls', 3, 1940, 15.75);

-- Display table contents
SELECT * FROM authors;
SELECT * FROM books;

-- Retrieve books with their author's information with conditions and ordering
SELECT b.title, a.first_name, a.last_name, a.birth_year
FROM books AS b
INNER JOIN authors AS a
ON a.author_id = b.author_id
WHERE b.publication_year > 1940
ORDER BY birth_year;

-- Count how many books each author has written
SELECT a.first_name, a.last_name, COUNT(*) AS book_count
FROM authors AS a
JOIN books AS b
ON a.author_id = b.author_id;
GROUP BY a.author_id;


-- Create categories table for many-to-many relationship example
CREATE TABLE categories (
    category_id INT PRIMARY KEY,
    category_name VARCHAR(50)
);

INSERT INTO categories (category_id, category_name)
VALUES 
    (1, 'Fiction'),
    (2, 'Classic'),
    (3, 'Romance'),
    (4, 'Political'),
    (5, 'Mystery'),
    (6, 'Adventure');

-- Create junction table for book-category many-to-many relationship
CREATE TABLE book_categories (
    book_id INT,
    category_id INT,
    PRIMARY KEY (book_id, category_id)
);

INSERT INTO book_categories (book_id, category_id)
VALUES 
    (101, 1), (101, 2), (101, 3), -- Pride and Prejudice: Fiction, Classic, Romance
    (102, 1), (102, 2), (102, 4), -- 1984: Fiction, Classic, Political
    (103, 1), (103, 2), (103, 4), -- Animal Farm: Fiction, Classic, Political
    (104, 1), (104, 2), (104, 6), -- The Old Man and the Sea: Fiction, Classic, Adventure
    (105, 1), (105, 5), -- Murder on the Orient Express: Fiction, Mystery
    (106, 1), (106, 5), -- Death on the Nile: Fiction, Mystery
    (107, 1), (107, 2), (107, 3), -- Emma: Fiction, Classic, Romance
    (108, 1), (108, 2), (108, 6); -- For Whom the Bell Tolls: Fiction, Classic, Adventure

-- Get books with their authors and categories using GROUP_CONCAT
SELECT b.title, a.first_name, a.last_name, GROUP_CONCAT(c.category_name SEPARATOR ', ') AS categories
FROM books b
JOIN authors a
ON b.author_id = a.author_id
JOIN book_categories AS bc
ON b.book_id = bc.book_id
JOIN categories c
ON bc.category_id = c.category_id
GROUP BY b.book_id;

-- Example with join condition in ON clause (Return books published before 1950 by authors born before 1900)
SELECT b.title, a.last_name
FROM books b
INNER JOIN authors a
ON b.author_id = a.author_id AND b.publication_year < 1950 AND a.birth_year < 1900;

-- Equivalent example with join condition in WHERE clause
SELECT b.title, a.last_name
FROM books b
INNER JOIN authors a ON b.author_id = a.author_id
WHERE b.publication_year < 1950 
  AND a.birth_year < 1900;
-- But using filtering at time of Joining is more efficient

-- Example with date functions - books published more than 70 years ago
SELECT b.title, a.last_name
FROM books b
INNER JOIN authors a
ON b.author_id = a.author_id
WHERE YEAR(CURDATE()) - b.publication_year > 70;
```


- Note: INNER JOIN excludes rows with NULL values in the join columns. If you want to include rows with NULL values, you would need to use LEFT JOIN or RIGHT JOIN.


-- Find authors who have written more than one book using HAVING clause
```sql
SELECT a.first_name, a.last_name, COUNT(b.book_id) AS book_count
FROM authors a
INNER JOIN books b
ON a.author_id = b.author_id
GROUP BY a.author_id, a.first_name, a.last_name
HAVING COUNT(b.book_id) > 1;
```


### Left & Right Joins (Outer Joins)

- Used when you need to keep all records from one "primary" table, even if there's no corresponding data in the "secondary" table.

- Handling Nulls: If no match exists, the result will contain NULL for the missing side.

- Pro Tip: Use the IFNULL() function to replace NULL results with a default value (like 0) for cleaner reporting.

-- It returns ALL records from the left table and only the matching records from the right table.
-- If no match exists in the right table, NULL values will be returned for the right table's columns.

-- Basic LEFT JOIN syntax
```sql
SELECT columns
FROM table1
LEFT JOIN table2
ON table1.column = table2.column;
```


-- Create and set up database
```sql
CREATE DATABASE left_join;
USE left_join;

-- Create customers table
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    email VARCHAR(100),
    city VARCHAR(50)
);

-- Create orders table with foreign key
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE NOT NULL,
    total_amount DECIMAL(10, 2),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- Insert sample customer data
INSERT INTO customers (customer_id, customer_name, email, city)
VALUES 
    (1, 'John Smith', 'john@example.com', 'New York'),
    (2, 'Jane Doe', 'jane@example.com', 'Los Angeles'),
    (3, 'Robert Johnson', 'robert@example.com', 'Chicago'),
    (4, 'Emily Davis', 'emily@example.com', 'Houston'),
    (5, 'Michael Brown', 'michael@example.com', 'Phoenix');

-- Insert sample order data
INSERT INTO orders (order_id, customer_id, order_date, total_amount)
VALUES 
    (101, 1, '2023-01-15', 150.75),
    (102, 3, '2023-01-16', 89.50),
    (103, 1, '2023-01-20', 45.25),
    (104, 2, '2023-01-25', 210.30),
    (105, 3, '2023-02-01', 75.00);


-- Example 1: Basic LEFT JOIN
-- Get all customers and their orders (if any)
SELECT c.customer_id, c.customer_name, o.order_id, o.order_date, o.total_amount
FROM customers c
LEFT JOIN orders o
ON c.customer_id = o.customer_id;

-- Example 2: Finding customers with no orders
-- Note the use of IS NULL in the WHERE clause
SELECT c.customer_id, c.customer_name
FROM customers c
LEFT JOIN orders o
ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;

-- Example 3: Using aggregate functions with LEFT JOIN
-- Get customer order counts and total spending
SELECT c.customer_id, c.customer_name, COUNT(o.order_id) AS order_count, IFNULL(SUM(o.total_amount), 0) AS total_spent
FROM customers c
LEFT JOIN orders o
ON c.customer_id = o.customer_id
GROUP BY c.customer_id;


-- Create shipping table for multiple joins example
CREATE TABLE shipping (
    shipping_id INT PRIMARY KEY,
    order_id INT,
    shipping_date DATE,
    carrier VARCHAR(50),
    tracking_number VARCHAR(50),
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
);

-- Insert sample shipping data
INSERT INTO shipping (shipping_id, order_id, shipping_date, carrier, tracking_number)
VALUES 
    (1001, 101, '2023-01-16', 'FedEx', 'FDX123456789'),
    (1002, 104, '2023-01-26', 'UPS', 'UPS987654321'),
    (1003, 105, '2023-02-02', 'USPS', 'USPS456789123');

-- Example 4: Multiple LEFT JOINs
-- Get customers, their orders, and shipping information
SELECT c.customer_name, o.order_id, o.order_date, o.total_amount, s.carrier, s.tracking_number
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
LEFT JOIN shipping s ON o.order_id = s.order_id;

-- Example 5: Filtering with WHERE vs. ON clause
-- Method 1: Filter in WHERE clause (filters after join)
SELECT c.customer_id, c.customer_name, c.city, o.order_id, o.order_date, o.total_amount
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE c.city = 'New York';
-- Inefficient

-- Method 2: Filter in ON clause (maintains all left table rows)
SELECT c.customer_id, c.customer_name, c.city, o.order_id, o.order_date, o.total_amount
FROM customers c
LEFT JOIN orders o
ON c.customer_id = o.customer_id AND c.city = 'New York';
-- We still got all customers with NULL values as it's Left Join

-- Method 3: Using subquery to filter left table first
SELECT c.customer_id, c.customer_name, c.city, o.order_id, o.order_date, o.total_amount
FROM (SELECT * FROM customers WHERE city = 'New York') c
LEFT JOIN orders o
ON c.customer_id = o.customer_id;
-- Most Efficient

-- Example 6: Advanced filtering with aggregation
-- Find customers who haven't ordered in the past 30 days
SELECT c.customer_id, c.customer_name, MAX(o.order_date) AS last_order_date
FROM customers c
LEFT JOIN orders o
ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name
HAVING MAX(o.order_date) IS NULL OR MAX(o.order_date) < DATE_SUB(CURDATE(), INTERVAL 30 DAY);
```



#### Right Join Examples

-- Create and use the Gokuldham Society database
```sql
CREATE DATABASE gokuldham_society;
USE gokuldham_society;

-- Create apartments table to store apartment information
CREATE TABLE apartments (
    apartment_id INT PRIMARY KEY,
    apartment_number VARCHAR(10) NOT NULL,
    floor_number INT NOT NULL,
    wing_name CHAR(1) NOT NULL
);

-- Create residents table with foreign key to apartments
CREATE TABLE residents (
    resident_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    occupation VARCHAR(100),
    apartment_id INT,
    FOREIGN KEY (apartment_id) REFERENCES apartments(apartment_id)
);

-- Insert sample apartment data
INSERT INTO apartments (apartment_id, apartment_number, floor_number, wing_name) VALUES
(1, '101', 1, 'A'),
(2, '102', 1, 'A'),
(3, '201', 2, 'A'),
(4, '202', 2, 'A'),
(5, '301', 3, 'A'),
(6, '302', 3, 'A'),
(7, '401', 4, 'A'),
(8, '402', 4, 'A'),
(9, '501', 5, 'B'),
(10, '502', 5, 'B');

-- Insert sample resident data
INSERT INTO residents (resident_id, first_name, last_name, occupation, apartment_id) VALUES
(1, 'Jethalal', 'Gada', 'Electronics Shop Owner', 1),
(2, 'Daya', 'Gada', 'Housewife', 1),
(3, 'Taarak', 'Mehta', 'Writer', 2),
(4, 'Anjali', 'Mehta', 'Teacher', 2),
(5, 'Popatlal', 'Pandey', 'Reporter', 3),
(6, 'Bhide', 'Aatmaram', 'School Teacher', 4),
(7, 'Madhavi', 'Bhide', 'Housewife', 4),
(8, 'Dr', 'Hathi', 'Doctor', 5),
(9, 'Komal', 'Hathi', 'Housewife', 5);
-- Note: We've left some apartments without residents

-- Basic SELECT query to view all residents
SELECT * FROM residents;

-- DEMO: LEFT JOIN to see all apartments and their residents (if any)
SELECT a.apartment_number, a.floor_number, a.wing_name, r.first_name, r.last_name
FROM apartments a
LEFT JOIN residents r
ON r.apartment_id = a.apartment_id;

-- DEMO: RIGHT JOIN to see all apartments and their residents (if any)
SELECT a.apartment_number, a.floor_number, a.wing_name, r.first_name, r.last_name
FROM residents r
RIGHT JOIN apartments a
ON r.apartment_id = a.apartment_id;

-- Create maintenance_requests table with foreign key to apartments
CREATE TABLE maintenance_requests (
    request_id INT PRIMARY KEY,
    apartment_id INT,
    request_date DATE NOT NULL,
    description TEXT NOT NULL,
    status ENUM('Pending', 'In Progress', 'Completed') DEFAULT 'Pending',
    FOREIGN KEY (apartment_id) REFERENCES apartments(apartment_id)
);

-- Insert sample maintenance request data
INSERT INTO maintenance_requests (request_id, apartment_id, request_date, description, status) VALUES
(1, 1, '2023-01-15', 'Leaky faucet in kitchen', 'Completed'),
(2, 1, '2023-02-20', 'Broken window handle', 'Completed'),
(3, 2, '2023-03-10', 'Electricity fluctuation', 'In Progress'),
(4, 4, '2023-03-15', 'Ceiling fan not working', 'Pending'),
(5, 5, '2023-04-01', 'Bathroom door lock broken', 'Completed'),
(6, 8, '2023-04-10', 'Water seepage in wall', 'In Progress');

-- EXERCISE 1: Finding Unoccupied Apartments
SELECT a.apartment_id, a.apartment_number, a.floor_number, a.wing_name
FROM apartments a
LEFT JOIN residents r
ON a.apartment_id = r.apartment_id
WHERE r.resident_id IS NULL;

-- EXERCISE 2: Count the number of residents per apartment
SELECT a.apartment_id, a.apartment_number, COUNT(r.resident_id) AS resident_count
FROM apartments a
LEFT JOIN residents r
ON a.apartment_id = r.apartment_id
GROUP BY a.apartment_id;

-- EXERCISE 3: List all apartments with their residents and maintenance request status
SELECT a.apartment_id, a.apartment_number, a.floor_number, a.wing_name, CONCAT(r.first_name, ' ', r.last_name) AS resident_name, mr.status AS maintenance_status
FROM apartments a
LEFT JOIN residents r
ON a.apartment_id = r.apartment_id
LEFT JOIN maintenance_requests mr
ON a.apartment_id = mr.apartment_id;

-- EXERCISE 4: Write a query to find the floor with the most unoccupied apartments
SELECT floor_number, wing_name, COUNT(*) AS unoccupied_count
FROM apartments a
LEFT JOIN residents r
ON a.apartment_id = r.apartment_id
WHERE r.resident_id IS NULL
GROUP BY floor_number, wing_name
ORDER BY unoccupied_count DESC
LIMIT 1;

-- EXERCISE 5: Write a query to list all apartments along with the total number of maintenance requests
SELECT a.apartment_id, a.apartment_number, a.floor_number, a.wing_name, COUNT(mr.request_id) AS maintenance_request_count
FROM apartments a
LEFT JOIN maintenance_requests mr
ON a.apartment_id = mr.apartment_id
GROUP BY a.apartment_id;
```


### UNION and UNION ALL

- Used to combine the results of two different SELECT queries vertically (stacking rows).

- UNION: Combines result sets and removes duplicates.

- UNION ALL: Combines result sets and keeps duplicates. It is faster because it skips the duplicate-check process.

- Requirement: Both queries must have the same number of columns with compatible data types.


-- UNION allows us to combine result sets from multiple SELECT queries into a single result set
-- Key points:
-- - Combines rows from multiple queries into a single result set
-- - Appends rows vertically (stacks them on top of each other)
-- - Requires that all queries have the same number of columns
-- - Column data types must be compatible across all queries
-- - Eliminates duplicate rows by default (use UNION ALL to keep duplicates)
-- - Uses the column names from the first SELECT statement for the final result set
-- - Ignores column names from subsequent queries

-- Database Setup
-- ====================================================================
```sql
CREATE DATABASE union_demo;
USE union_demo;

-- Create tables for our demonstration
CREATE TABLE headquarters_employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    hire_date DATE,
    department VARCHAR(50),
    salary DECIMAL(10,2)
);

CREATE TABLE branch_employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    hire_date DATE,
    department VARCHAR(50),
    salary DECIMAL(10,2)
);

CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    signup_date DATE,
    status VARCHAR(20)
);

-- Sample Data
-- ====================================================================
-- Insert data into headquarters_employees
INSERT INTO headquarters_employees VALUES
(101, 'John', 'Smith', 'john.smith@company.com', '2018-03-15', 'IT', 75000.00),
(102, 'Mary', 'Johnson', 'mary.johnson@company.com', '2019-06-22', 'HR', 65000.00),
(103, 'Robert', 'Williams', 'robert.williams@company.com', '2017-11-08', 'Finance', 82000.00),
(104, 'Susan', 'Brown', 'susan.brown@company.com', '2020-01-30', 'Marketing', 68000.00),
(105, 'Michael', 'Davis', 'michael.davis@company.com', '2018-09-12', 'IT', 78000.00);

-- Insert data into branch_employees
INSERT INTO branch_employees VALUES
(201, 'James', 'Wilson', 'james.wilson@company.com', '2019-04-18', 'Sales', 62000.00),
(202, 'Patricia', 'Moore', 'patricia.moore@company.com', '2020-07-25', 'Marketing', 59000.00),
(203, 'Linda', 'Taylor', 'linda.taylor@company.com', '2018-08-15', 'HR', 61000.00),
(204, 'Robert', 'Williams', 'robert.williams@company.com', '2017-11-08', 'Finance', 82000.00), -- Duplicate employee who works at both locations
(205, 'Elizabeth', 'Anderson', 'elizabeth.anderson@company.com', '2019-12-03', 'Sales', 64000.00);

-- Insert data into customers
INSERT INTO customers VALUES
(1001, 'David', 'Miller', 'david.miller@email.com', '2019-02-14', 'Active'),
(1002, 'Sarah', 'Wilson', 'sarah.wilson@email.com', '2020-05-20', 'Active'),
(1003, 'Michael', 'Davis', 'michael.davis@email.com', '2018-11-30', 'Inactive'), -- Same name as an employee
(1004, 'Jennifer', 'Garcia', 'jennifer.garcia@email.com', '2021-01-05', 'Active'),
(1005, 'Robert', 'Martinez', 'robert.martinez@email.com', '2019-08-22', 'Active');

-- View table data
-- ====================================================================
SELECT * FROM headquarters_employees;
SELECT * FROM branch_employees;
SELECT * FROM customers;

-- Basic UNION Examples
-- ====================================================================
-- Example 1: UNION vs UNION ALL
-- Get a list of all employees from both locations (without duplicates)
SELECT first_name, last_name, email FROM headquarters_employees
UNION
SELECT first_name, last_name, email FROM branch_employees;

-- Get a list of all employees from both locations (including duplicates)
SELECT first_name, last_name, email FROM headquarters_employees
UNION ALL
SELECT first_name, last_name, email FROM branch_employees;

-- Example 2: Combining full tables
SELECT * FROM headquarters_employees
UNION ALL
SELECT * FROM branch_employees;

-- Advanced UNION Examples
-- ====================================================================
-- Example 3: Adding a descriptor column
-- Combine employee and customer contact information with a type indicator
SELECT first_name, last_name, email, 'Employee' AS contact_type
FROM headquarters_employees
UNION
SELECT first_name, last_name, email, 'Customer' AS contact_type
FROM customers;

-- Example 4: Ordering results after UNION
-- Get all employees sorted by last name
SELECT employee_id, first_name, last_name, department
FROM headquarters_employees
UNION
SELECT employee_id, first_name, last_name, department
FROM branch_employees
ORDER BY last_name;

-- Example 5: Filtering before UNION
-- Get employees with salary over 70000
SELECT employee_id, first_name, last_name, department, salary
FROM headquarters_employees
WHERE salary > 70000
UNION
SELECT employee_id, first_name, last_name, department, salary
FROM branch_employees
WHERE salary > 70000
ORDER BY salary DESC;

-- Handling Different Column Structures
-- ====================================================================
-- Example 6: Handling different table structures with NULL values
SELECT employee_id, first_name, last_name, department, salary, NULL AS status
FROM headquarters_employees
UNION
SELECT customer_id, first_name, last_name, NULL, NULL, status
FROM customers
ORDER BY first_name, last_name;

-- Practical Use Cases
-- ====================================================================
-- Example 7: Finding all unique departments across locations
SELECT department
FROM headquarters_employees
UNION
SELECT department
FROM branch_employees;

-- Example 8: Finding common departments (advanced)
-- Departments that exist in both headquarters and branch offices
SELECT department
FROM (SELECT DISTINCT department FROM headquarters_employees UNION ALL SELECT DISTINCT department FROM branch_employees) AS combined
GROUP BY department
HAVING COUNT(*) = 2;
```


### Full Join in MySQL

- MySQL does not have a native FULL JOIN command.

- Workaround: Combine a LEFT JOIN and a RIGHT JOIN using the UNION operator to simulate a full outer join.

-- FULL JOIN
-- - It returns all matching rows from both tables where the join condition is met
-- - It also returns all non-matching rows from the left table (with NULL values for columns from the right table)
-- - It also returns all non-matching rows from the right table (with NULL values for columns from the left table)
-- - It combines the results of both LEFT JOIN and RIGHT JOIN, including all records from both tables and matching records from both sides where available.

-- Join Types Comparison:
-- - INNER JOIN (only returns matching rows between tables)
-- - LEFT JOIN (returns all rows from left table and matching from right)
-- - RIGHT JOIN (returns all rows from right table and matching from left)
-- - FULL JOIN (returns all rows from both tables)


-- Database Setup - Friends Theme
-- ====================================================================
```sql
CREATE DATABASE friends_db;
USE friends_db;

-- Create tables for our demonstration
CREATE TABLE characters (
    character_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    occupation VARCHAR(100)
);

CREATE TABLE apartments (
    apartment_id INT PRIMARY KEY,
    building_address VARCHAR(100) NOT NULL,
    apartment_number VARCHAR(10) NOT NULL,
    monthly_rent DECIMAL(8, 2),
    current_tenant_id INT
);

-- Sample Data
-- ====================================================================
-- Insert data into characters
INSERT INTO characters (character_id, first_name, last_name, occupation) VALUES
(1, 'Ross', 'Geller', 'Paleontologist'),
(2, 'Rachel', 'Green', 'Fashion Executive'),
(3, 'Chandler', 'Bing', 'IT Procurement Manager'),
(4, 'Monica', 'Geller', 'Chef'),
(5, 'Joey', 'Tribbiani', 'Actor'),
(6, 'Phoebe', 'Buffay', 'Massage Therapist'),
(7, 'Gunther', 'Smith', 'Coffee Shop Manager'),
(8, 'Janice', 'Hosenstein', 'Unknown');

-- Insert data into apartments
INSERT INTO apartments (apartment_id, building_address, apartment_number, monthly_rent, current_tenant_id) VALUES
(101, '90 Bedford Street', '20', 3500.00, 3),
(102, '90 Bedford Street', '19', 3500.00, 4),
(103, '5 Morton Street', '14', 2800.00, 6),
(104, '17 Grove Street', '3B', 2200.00, NULL),
(105, '15 Yemen Road', 'Yemen', 900.00, NULL),
(106, '495 Grove Street', '7', 2400.00, 1);

-- View table data
-- ====================================================================
SELECT * FROM characters;
SELECT * FROM apartments;

-- JOIN Examples
-- ====================================================================

-- INNER JOIN Example
-- Only returns characters who have apartments and apartments that have tenants
SELECT c.character_id, c.first_name, c.last_name, c.occupation, a.apartment_id, a.building_address, a.apartment_number, a.monthly_rent
FROM characters c
INNER JOIN apartments a
ON c.character_id = a.current_tenant_id;

-- LEFT JOIN Example
-- All characters, including those without apartments
SELECT c.character_id, c.first_name, c.last_name, c.occupation, a.apartment_id, a.building_address, a.apartment_number, a.monthly_rent
FROM characters c
LEFT JOIN apartments a
ON c.character_id = a.current_tenant_id;

-- RIGHT JOIN Example
-- All apartments, including those without tenants
SELECT c.character_id, c.first_name, c.last_name, c.occupation, a.apartment_id, a.building_address, a.apartment_number, a.monthly_rent
FROM characters c
RIGHT JOIN apartments a
ON c.character_id = a.current_tenant_id;

-- FULL JOIN Example (MySQL implementation using UNION)
-- All characters and all apartments, with matches where they exist
SELECT c.character_id, c.first_name, c.last_name, c.occupation, a.apartment_id, a.building_address, a.apartment_number, a.monthly_rent
FROM characters c
LEFT JOIN apartments a
ON c.character_id = a.current_tenant_id
UNION
SELECT c.character_id, c.first_name, c.last_name, c.occupation, a.apartment_id, a.building_address, a.apartment_number, a.monthly_rent
FROM characters c
RIGHT JOIN apartments a
ON c.character_id = a.current_tenant_id


-- PostgreSQL native FULL JOIN syntax (for reference)
/*
SELECT c.character_id, c.first_name, c.last_name, c.occupation,
       a.apartment_id, a.building_address, a.apartment_number, a.monthly_rent
FROM characters c
FULL JOIN apartments a ON c.character_id = a.current_tenant_id;
*/

-- Additional examples - Employee/Department context
-- ====================================================================
-- For a typical HR database scenario, we would use:

-- Create Employee/Department tables
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL,
    location VARCHAR(100)
);
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    department_id INT,
    salary DECIMAL(10, 2),
    hire_date DATE
);

-- FULL JOIN example for employee/department context
SELECT e.employee_id, e.first_name, e.last_name, d.department_id, d.department_name, d.location
FROM employees e
LEFT JOIN departments d
ON e.department_id = d.department_id
UNION
SELECT e.employee_id, e.first_name, e.last_name, d.department_id, d.department_name, d.location
FROM employees e
RIGHT JOIN departments d
ON e.department_id = d.department_id

-- Full Join Filtering Examples
-- ====================================================================

-- Finding only characters without apartments
SELECT c.character_id, c.first_name, c.last_name
FROM characters c
LEFT JOIN apartments a
ON c.character_id = a.current_tenant_id
WHERE a.apartment_id IS NULL;

-- Finding only apartments without tenants
SELECT a.apartment_id, a.building_address, a.apartment_number
FROM apartments a
LEFT JOIN characters c
ON a.current_tenant_id = c.character_id
WHERE c.character_id IS NULL;

-- Using the FULL JOIN result to find both unmatched cases
SELECT c.character_id, c.first_name, c.last_name, a.apartment_id, a.building_address, a.apartment_number
FROM characters c
LEFT JOIN apartments a
ON c.character_id = a.current_tenant_id
WHERE a.apartment_id IS NULL;
UNION
SELECT c.character_id, c.first_name, c.last_name, a.apartment_id, a.building_address, a.apartment_number
FROM characters c
RIGHT JOIN apartments a
ON c.character_id = a.current_tenant_id
WHERE c.character_id IS NULL;
```


### Cross Join: Cartesian Product

- A cross join pairs every row from the first table with every row from the second table.

- Math: If Table A has 5 rows and Table B has 4, the result is 20 rows (5 X 4).

- Practical Example: Combining a "Products" table with a "Colors" table to see every product in every available color.

- Warning: Be carefull on large production tables! Joining two tables with 1,000 rows each results in 1,000,000 rows, which can tank performance.

-- Database Setup
```sql
CREATE DATABASE cross_join;
USE cross_join;

-- Create Table Products
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(50) NOT NULL
);

-- Create Table Colors
CREATE TABLE colors (
    color_id INT PRIMARY KEY,
    color_name VARCHAR(30) NOT NULL
);

INSERT INTO products (product_id, product_name) VALUES
(1, 'T-shirt'),
(2, 'Jeans'),
(3, 'Sweater'),
(4, 'Jacket');
INSERT INTO colors (color_id, color_name) VALUES
(1, 'Red'),
(2, 'Blue'),
(3, 'Green'),
(4, 'Black'),
(5, 'White');

-- Cross Join
SELECT p.product_name, c.color_name
FROM products p
CROSS JOIN colors c;

-- Create Table Sizes
CREATE TABLE sizes (
    size_id INT PRIMARY KEY,
    size_name VARCHAR(10) NOT NULL
);
INSERT INTO sizes (size_id, size_name) VALUES
(1, 'S'),
(2, 'M'),
(3, 'L'),
(4, 'XL');

-- 5 * 4 * 4 = 80
-- Generate all possible product variations
SELECT p.product_name, c.color_name, s.size_name, CONCAT(p.product_name, ' - ', c.color_name, ' - Size ', s.size_name) AS full_product_description
FROM products p
CROSS JOIN colors c
CROSS JOIN sizes s
WHERE p.product_name = 'T-shirt';
```


### Self Join: Internal Relationships

- A Self Join is used when a table has a relationship within itself. You must use Aliases to distinguish the two "versions" of the same table.

- Scenario: An Employees table where one column is manager_id, which refers back to an employee_id in the same table.

- Syntax Tip: SELECT * FROM Employees e1 JOIN Employees e2 ON e1.manager_id = e2.employee_id; .

- Logic Check: Used to find pairs, such as "Employees working in the same department" or "Employees earning more than their managers".

-- Create and use database
```sql
CREATE DATABASE self_join_tutorial;
USE self_join_tutorial;

-- Create employees table with manager_id referencing the same table
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    job_title VARCHAR(100),
    salary DECIMAL(10, 2),
    department VARCHAR(50),
    manager_id INT,
    hire_date DATE
);

-- Insert sample employee data with hierarchical management structure
INSERT INTO employees VALUES
(1, 'James', 'Smith', 'CEO', 150000.00, 'Executive', NULL, '2010-01-15'),
(2, 'Sarah', 'Johnson', 'CTO', 140000.00, 'Technology', 1, '2011-03-10'),
(3, 'Michael', 'Williams', 'CFO', 140000.00, 'Finance', 1, '2012-07-22'),
(4, 'Jessica', 'Brown', 'HR Director', 110000.00, 'Human Resources', 1, '2013-05-18'),
(5, 'David', 'Miller', 'Senior Developer', 95000.00, 'Technology', 2, '2014-11-05'),
(6, 'Emily', 'Davis', 'Developer', 80000.00, 'Technology', 5, '2016-08-12'),
(7, 'Robert', 'Wilson', 'Junior Developer', 65000.00, 'Technology', 5, '2019-02-28'),
(8, 'Jennifer', 'Taylor', 'Accountant', 75000.00, 'Finance', 3, '2015-09-17'),
(9, 'Thomas', 'Anderson', 'Accountant', 72000.00, 'Finance', 3, '2017-06-24'),
(10, 'Lisa', 'Martinez', 'HR Specialist', 68000.00, 'Human Resources', 4, '2018-04-30');

-- EXAMPLE 1: Basic Self JOIN to show employees with their managers
-- This joins the employees table with itself to match employees with manager information
SELECT * FROM employees emp
JOIN employees mgr
ON emp.manager_id = mgr.employee_id;

-- EXAMPLE 2: Self JOIN with LEFT JOIN to include all employees (including CEO with no manager)
-- This ensures even employees without managers (like the CEO) are included in results
SELECT * FROM employees emp
LEFT JOIN employees mgr
ON emp.manager_id = mgr.employee_id;

-- EXAMPLE 3: Group employees by department
-- Not a self join, but useful for context
SELECT department, COUNT(*) AS employee_count, GROUP_CONCAT(CONCAT(first_name, ' ', last_name) ORDER BY employee_id SEPARATOR ', ') AS employees
FROM employees
GROUP BY department;

-- EXAMPLE 4: Find employees who work in the same department
-- Self join to match employees with their colleagues in the same department
-- The condition e1.employee_id < e2.employee_id prevents duplicates and self-matches
SELECT * FROM employees e1
JOIN employees e2
ON e1.department = e2.department AND e1.employee_id < e2.employee_id;

-- EXAMPLE 5: Find employees who make less than their managers
-- Self join to compare employee salaries with their manager's salary
SELECT * FROM employees emp
JOIN employees mgr
ON emp.manager_id = mgr.employee_id
WHERE emp.salary < mgr.salary;

-- EXAMPLE 6: Calculate average salary difference between employees and managers by department
-- This shows how to use aggregate functions with self joins
SELECT 
    emp.department,
    COUNT(emp.employee_id) AS num_employees,
    ROUND(AVG(mgr.salary), 2) AS avg_manager_salary,
    ROUND(AVG(emp.salary), 2) AS avg_employee_salary,
    ROUND(AVG(mgr.salary - emp.salary), 2) AS avg_salary_difference
FROM employees emp
JOIN employees mgr
ON emp.manager_id = mgr.employee_id
GROUP BY emp.department
ORDER BY avg_salary_difference DESC;

-- IMPORTANT NOTES FOR SELF JOINS:
-- 1. Always use different aliases for each instance of the table.
-- 2. When looking for pairs, use conditions like a.id < b.id to avoid matching rows with themselves.
-- 3. Self JOINs can be resource-intensive. Consider indexing columns used in JOIN conditions.
-- 4. Consider LEFT JOINs when you need to include records without matches (like employees with no manager).
```


## 3. Performance Tips

- Indexing: Ensure join columns (Foreign Keys) are indexed to speed up lookups.

- Selectivity: Avoid SELECT *. Only request the specific columns you need.

- Limits: Use LIMIT when testing queries on large production databases to avoid system hangs.

- Explain Command: Use EXPLAIN before your query to see how MySQL intends to execute the join.
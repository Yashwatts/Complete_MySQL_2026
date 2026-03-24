## 1. Creating a Table (CREATE TABLE)

The CREATE TABLE command defines the structure of a new table, including column names, data types and constraints.

Syntax:

```sql
CREATE TABLE table_name (
    column1 datatype constraints,
    column2 datatype constraints,
    ...
    table_constraints
);
```

Key Constraints:

- NOT NULL: Ensures a column cannot have a NULL value.
- PRIMARY KEY: Uniquely identifies each record. It must be unique and cannot be null.
- UNIQUE: Ensures all values in a column are different but allows one NULL value.
- AUTO_INCREMENT: Automatically generates a unique number for new rows.
- DEFAULT: Sets a default value if none is specified (e.g., DEFAULT CURRENT_TIMESTAMP).
- CHECK: Validates that all values in a column satisfy a specific condition.

Example:

```sql
create table employees(
    employee_id int primary key auto_increment,
    first_name varchar(50) not null,
    last_name varchar(50) not null,
    hire_date date default (CURRENT_DATE()),
    email varchar(100) unique,
    phone_number varchar(100) unique,
    salary decimal(10, 2) check (salary > 0.0),
    employment_status enum('active', 'on leave', 'terminated') default 'active',
    created_at timestamp default CURRENT_TIMESTAMP,
    updated_at timestamp default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP
    // Can also write like this instead of using alter command for foreign key as a table constraint.
    // foreign key (department_id) references departments(department_id)
);

create table departments (
    department_id int primary_key auto_increment,
    department_name varchar(100) not null,
    location varchar(100),
    created_at timestamp default CURRENT_TIMESTAMP,
    updated_at timestamp default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP
)
```

> It's a professional best practice to always include created_at and updated_at columns using TIMESTAMP. Use ON UPDATE CURRENT_TIMESTAMP for the updated column so it refreshes automatically whenever a row is changed.

## 2. Insering Data (INSERT INTO)

Used to add new records to a table.

- Single Row:

Syntax:

```sql
INSERT INTO table_name (column1, column2, column3)
VALUES (value1, value2, value3);
```

- Multiple Rows: You can add multiple rows in one command by separating value sets with commas.
- Default Behavior: If a column has a DEFAULT value or is AUTO_INCREMENT, you can omit it from the insert list.

Example:

```sql
INSERT INTO employees (
    first_name,
    last_name,
    hire_date,
    email,
    phone_number,
    salary,
    employment_status
)
VALUES (
    'John',
    'Smith',
    '2026-01-15',
    'john.smith@company.com',
    '+1-555-123-4567',
    75000.00,
    'active'
);


INSERT INTO departments (department_name, location) VALUES
('IT', 'Building A'),
('HR', 'Building B'),
('Sales', 'Building C');
```

## 3. Modifying a Table (ALTER TABLE)

The ALTER commands is used to change the structure of an existing table without deleting it.

Command Actions:

- Add Column: ALTER TABLE table_name ADD COLUMN column_name datatype;
- Modify Column: Change data types or add constraints to existing columns.
- Rename Column: ALTER TABLE table_name RENAME COLUMN old_name TO new_name;
- Drop Column: Remove a column entirely.
- Add Foreign Key: Connects two tables to maintain referential integrity.

Some Examples:

```sql
// Add Column
ALTER TABLE employees
add column description text,
add column emergency_contact varchar(100);

// Modify Column
ALTER TABLE employees
modify column description tinytext,
modify column emergency_contact varchar(100) not null;

// Rename Column
ALTER TABLE employees
rename column emergency_contact to em_contact;

// Drop Column
ALTER TABLE employees
drop column emergency_contact;

// Adding Constraint
ALTER TABLE employees
add check (emergency_contact REGEXP '^[A-Za-z ]+: [0-9+-]+$' );
```

Example:

```sql
ALTER TABLE employees
ADD COLUMN department_id INT;

ALTER TABLE employees
modify COLUMN department_id INT not null;

alter table employees add foreign key (department_id) references departments(department_id);
```

## 4. Deleting a Table (DROP TABLE)

The DROP command deletes the entire table structure and all its data permanently.

- Safet Check: Use IF EXISTS to avoid errors if the table doesn't exist.
- Dependencies: SQL will prevent you from dropping a table if another table depends on it via a Foreign key.

Example:

```sql
drop table departments; // will fail because it has dependencies
drop table employees;
drop table ok; // will give error because ok table doesn't exist.
drop table if exists ok; // Now instead of giving error it will give warning.
```
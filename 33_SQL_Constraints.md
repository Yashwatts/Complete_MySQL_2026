## SQL Constraints

Constraints are rules applied to table columns to protect data quality and enforce business logic at the database level.

They are critical because they prevent invalid or inconsistent data from entering your tables.


## 1. What Are Constraints?

- Constraints are integrity rules defined on table columns.
- They are checked automatically by MySQL during `INSERT` and `UPDATE`.
- If data violates a constraint, MySQL throws an error.

Why constraints matter:
- ensure accuracy
- avoid duplicate or invalid values
- enforce required fields
- reduce data bugs in applications


## 2. Types of Constraints (Overview)

Common MySQL constraints:
- `NOT NULL`
- `UNIQUE`
- `CHECK`
- `DEFAULT`
- `PRIMARY KEY`
- `FOREIGN KEY`

This chapter focuses deeply on:
- `NOT NULL`
- `UNIQUE`
- `CHECK`
- `DEFAULT`


## 2.1 PRIMARY KEY vs UNIQUE (Difference)

This is a very common interview question.

- `PRIMARY KEY`:
  - uniquely identifies each row
  - cannot be `NULL`
  - only one primary key per table
  - automatically creates a unique index in MySQL

- `UNIQUE`:
  - prevents duplicate values
  - can allow `NULL` values depending on the column definition and MySQL behavior
  - a table can have multiple unique constraints

Practical difference:
- Use `PRIMARY KEY` for the main row identifier.
- Use `UNIQUE` for business-unique fields like email, username, SKU.

Example:
```sql
CREATE TABLE members (
    member_id INT PRIMARY KEY,
    email VARCHAR(150) NOT NULL UNIQUE,
    username VARCHAR(50) NOT NULL UNIQUE
);
```


## 3. NOT NULL Constraint

`NOT NULL` ensures a column must always have a value.

If a row is inserted without that value, insertion fails.

Example:
```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL
);
```

Invalid insert:
```sql
INSERT INTO customers (customer_id, full_name, email)
VALUES (1, NULL, 'test@example.com');
-- Error: full_name cannot be NULL
```

Valid insert:
```sql
INSERT INTO customers (customer_id, full_name, email)
VALUES (1, 'Aman Sharma', 'aman@example.com');
```

Use `NOT NULL` for required fields like:
- names
- email
- status
- created timestamps


## 4. UNIQUE Constraint

`UNIQUE` ensures all values in a column (or combination of columns) are distinct.

### 4.1 Single-Column UNIQUE

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(150) NOT NULL,
    UNIQUE (username),
    UNIQUE (email)
);
```

Duplicate insert fails:
```sql
INSERT INTO users (user_id, username, email)
VALUES (1, 'riya', 'riya@example.com');

INSERT INTO users (user_id, username, email)
VALUES (2, 'riya', 'riya2@example.com');
-- Error: duplicate value for username
```

### 4.2 Composite UNIQUE

Use when uniqueness depends on multiple columns together.

```sql
CREATE TABLE enrollments (
    enrollment_id INT PRIMARY KEY,
    student_id INT NOT NULL,
    course_id INT NOT NULL,
    semester VARCHAR(20) NOT NULL,
    CONSTRAINT uq_student_course_sem UNIQUE (student_id, course_id, semester)
);
```

This allows:
- same student in same course in different semester

But blocks:
- exact duplicate student-course-semester row

### 4.3 UNIQUE and NULL Behavior

MySQL can allow multiple `NULL` values in a `UNIQUE` column because `NULL` is treated as unknown, not equal.

So if strict required uniqueness is needed, combine:
- `UNIQUE`
- `NOT NULL`


## 5. CHECK Constraint

`CHECK` enforces a Boolean condition on column values.

Rows violating the condition are rejected.

Example:
```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    stock_quantity INT NOT NULL,
    CHECK (price > 0),
    CHECK (stock_quantity >= 0)
);
```

Invalid rows:
```sql
INSERT INTO products (product_id, product_name, price, stock_quantity)
VALUES (1, 'Mouse', -500, 10);
-- Error: CHECK (price > 0) violated
```

### 5.1 Named CHECK Constraints

```sql
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    age INT,
    salary DECIMAL(10,2),
    CONSTRAINT chk_employee_age CHECK (age >= 18),
    CONSTRAINT chk_employee_salary CHECK (salary >= 0)
);
```

### 5.2 MySQL Version Note

In modern MySQL versions (8.0.16+), `CHECK` constraints are enforced.
In older versions, they were parsed but often ignored.

For interview answer:
- mention version awareness when discussing `CHECK`.


## 6. DEFAULT Constraint

`DEFAULT` provides an automatic value when no value is supplied.

Example:
```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_status VARCHAR(20) DEFAULT 'PENDING',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Insert without status:
```sql
INSERT INTO orders (order_id, customer_id)
VALUES (101, 5);
```

Result:
- `order_status` becomes `PENDING`
- `created_at` gets current timestamp

### 6.1 DEFAULT with NOT NULL

Very common pattern:
- `NOT NULL` ensures value cannot be null
- `DEFAULT` ensures missing input gets a safe value

```sql
is_active TINYINT NOT NULL DEFAULT 1
```


## 7. Column-Level vs Table-Level Constraint Syntax

### 7.1 Column-Level

Defined directly with column:
```sql
email VARCHAR(150) NOT NULL UNIQUE
```

### 7.2 Table-Level

Defined after columns:
```sql
CONSTRAINT uq_email UNIQUE (email)
```

Use table-level when:
- naming constraints
- composite constraints
- better readability for large schemas


## 8. Add Constraints with ALTER TABLE

### 8.1 Add NOT NULL

```sql
ALTER TABLE customers
MODIFY full_name VARCHAR(100) NOT NULL;
```

### 8.2 Add UNIQUE

```sql
ALTER TABLE users
ADD CONSTRAINT uq_users_phone UNIQUE (phone);
```

### 8.3 Add CHECK

```sql
ALTER TABLE products
ADD CONSTRAINT chk_price_positive CHECK (price > 0);
```

### 8.4 Add DEFAULT

```sql
ALTER TABLE orders
ALTER order_status SET DEFAULT 'PENDING';
```


## 9. Drop or Change Constraints

Drop unique constraint (often through index name):
```sql
ALTER TABLE users
DROP INDEX uq_users_phone;
```

Drop check constraint:
```sql
ALTER TABLE products
DROP CHECK chk_price_positive;
```

Remove default:
```sql
ALTER TABLE orders
ALTER order_status DROP DEFAULT;
```


## 9.1 FOREIGN KEY Quick Recap

Foreign keys are a related integrity concept worth remembering.

- A foreign key links child rows to a parent table.
- It prevents orphan records.
- It enforces referential integrity.

Quick example:
```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    CONSTRAINT fk_orders_customer
        FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
);
```


## 9.2 ON UPDATE / ON DELETE (Foreign Key Behavior)

Even though this chapter focuses on constraints, foreign key actions are a common reference point.

- `ON DELETE CASCADE`:
  - deleting a parent row deletes matching child rows

- `ON DELETE RESTRICT` or `NO ACTION`:
  - parent delete is blocked if child rows exist

- `ON UPDATE CASCADE`:
  - updating parent key updates child foreign key values

- `SET NULL`:
  - child foreign key is set to `NULL` when parent changes or is deleted (if column allows `NULL`)

Example:
```sql
CREATE TABLE order_items (
    item_id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    CONSTRAINT fk_order_items_order
        FOREIGN KEY (order_id)
        REFERENCES orders(order_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);
```


## 10. Practical End-to-End Example

```sql
CREATE TABLE account_users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(150) NOT NULL,
    age INT NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'ACTIVE',
    balance DECIMAL(12,2) NOT NULL DEFAULT 0,

    CONSTRAINT uq_account_users_username UNIQUE (username),
    CONSTRAINT uq_account_users_email UNIQUE (email),
    CONSTRAINT chk_account_users_age CHECK (age >= 18),
    CONSTRAINT chk_account_users_balance CHECK (balance >= 0)
);
```

Valid insert:
```sql
INSERT INTO account_users (user_id, username, email, age)
VALUES (1, 'neha01', 'neha@example.com', 22);
```

Invalid insert examples:
```sql
-- Duplicate email
INSERT INTO account_users (user_id, username, email, age)
VALUES (2, 'neha02', 'neha@example.com', 21);

-- Age below 18
INSERT INTO account_users (user_id, username, email, age)
VALUES (3, 'kid01', 'kid@example.com', 15);
```


## 11. Constraint Naming Importance

- Named constraints make schema maintenance easier.
- Clear names help debugging when MySQL returns an error.
- Good names improve readability in migrations and schema reviews.

Recommended pattern:
- `chk_<table>_<column>` for check constraints
- `uq_<table>_<column>` for unique constraints
- `fk_<childtable>_<parenttable>` for foreign keys

Examples:
```sql
CONSTRAINT uq_account_users_email UNIQUE (email)
CONSTRAINT chk_account_users_age CHECK (age >= 18)
CONSTRAINT fk_orders_customer FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
```


## 12. Constraint vs Application Validation

- Application validation:
  - happens before data reaches the database
  - gives user-friendly messages
  - can be bypassed if not enforced at DB layer

- Database constraints:
  - final enforcement layer
  - protect data even if the application has bugs
  - ensure integrity across all clients and scripts

Best practice:
- use both
- application validation for user experience
- database constraints for correctness


## 13. Constraint Violation Error Handling Strategy

In application code:
- catch database error codes/messages
- return user-friendly validation error
- avoid relying only on front-end validation

Best practice:
- use both application validation and database constraints
- database constraints are final protection layer


## 14. Performance and Design Notes

- Constraints add validation overhead, but usually worth it for integrity.
- `UNIQUE` often creates indexes, which can improve reads but add write cost.
- Too many constraints can make heavy bulk loads slower.
- During migration or bulk import, validate strategy carefully.


## 15. When to Use Which Constraint

- `NOT NULL`:
  - required fields

- `UNIQUE`:
  - business-unique values (email, username, sku)

- `CHECK`:
  - range/rule enforcement (age >= 18, price > 0)

- `DEFAULT`:
  - fallback values (status, timestamps, flags)


## 16. Common Mistakes

- Assuming `UNIQUE` prevents multiple NULLs in all scenarios.
- Forgetting `NOT NULL` with `UNIQUE` for required unique fields.
- Using defaults that hide bad data entry silently.
- Expecting `CHECK` behavior without considering MySQL version.
- Adding constraints without cleaning existing dirty data first.


## 17. Quick Interview Notes

- Constraints enforce data integrity at database level.
- `NOT NULL` = value required.
- `UNIQUE` = no duplicate values.
- `CHECK` = condition must be true.
- `DEFAULT` = auto value if none provided.
- Best practice: combine constraints to enforce strong data rules.
- `PRIMARY KEY` uniquely identifies the row; `UNIQUE` enforces business uniqueness.
- Foreign keys protect relationships and can cascade updates/deletes.
- Naming constraints clearly makes maintenance easier.
- Application validation is helpful, but database constraints are the final guardrail.


## 18. Final Summary

- Constraints are essential for reliable SQL schema design.
- `NOT NULL`, `UNIQUE`, `CHECK` and `DEFAULT` solve different integrity problems.
- Strong schema constraints reduce application bugs and bad data.
- Well-designed constraints improve long-term maintainability and trust in data.

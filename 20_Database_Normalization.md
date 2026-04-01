# Introduction to Normalization

- Normalization is the process of organizing data in a database to reduce redundancy and improve data integrity.
- Without it, databases suffer from:
    - Storage Inefficiency: Repeating the same customer details (name, email, address) for every order.
    - Update Anomalies: If a customer changes their address, you must update multiple rows, risking inconsistency.
    - Deletion/Insertion Issues: You might accidentally delete book details when removing a customer, or be unable to add a book without assigning it to a customer.

```sql
-- Create and use bookstore database
CREATE DATABASE bookstore;
USE bookstore;

-- Original denormalized table
CREATE TABLE book_orders (
    order_id INT,
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    customer_address VARCHAR(255),
    book_isbn VARCHAR(20),
    book_title VARCHAR(200),
    book_author VARCHAR(100),
    book_price DECIMAL(10, 2),
    order_date DATE,
    quantity INT,
    total_price DECIMAL(10, 2)
);

-- Sample data for denormalized table
INSERT INTO book_orders VALUES
(1, 'John Smith', 'john@example.com', '123 Main St, Anytown', '978-0141439518', 'Pride and Prejudice', 'Jane Austen', 9.99, '2023-01-15', 1, 9.99),
(2, 'John Smith', 'john@example.com', '123 Main St, Anytown', '978-0451524935', '1984', 'George Orwell', 12.99, '2023-01-15', 2, 25.98),
(3, 'Mary Johnson', 'mary@example.com', '456 Oak Ave, Somewhere', '978-0061120084', 'To Kill a Mockingbird', 'Harper Lee', 14.99, '2023-01-20', 1, 14.99),
(4, 'Robert Brown', 'robert@example.com', '789 Pine Rd, Nowhere', '978-0141439518', 'Pride and Prejudice', 'Jane Austen', 9.99, '2023-01-25', 1, 9.99);

-- View the denormalized data
SELECT * FROM book_orders;
```

## First Normal Form (1NF)

- A table is in 1NF if it follows these rules:
    - Atomic Values: Each column must contain indivisible values (e.g., no comma-separated phone numbers in one field).
    - Uniform Data Types: Columns should only hold one type of data.
    - Unique Rows: Every row must be uniquely identifiable, typically through a Primary Key.
    - No Repeating Groups: Instead of having columns like Phone1, Phone2, and Phone3, create a separate table for phone numbers linked by a Foreign Key.

```sql
CREATE TABLE book_orders_1nf (
    order_id INT,
    book_isbn VARCHAR(20),
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    customer_address VARCHAR(255),
    book_title VARCHAR(200),
    book_author VARCHAR(100),
    book_price DECIMAL(10, 2),
    order_date DATE,
    quantity INT,
    total_price DECIMAL(10, 2),
    PRIMARY KEY (order_id, book_isbn)
);
```

## Second Normal Form (2NF)

- To reach 2NF, a table must first be in 1NF and then meet the following:
    - No Partial Dependencies: Every "non-key" column must depend on the entire primary key, not just a part of it.
    - Example: In a table where the Primary Key is a combination of OrderID and BookID, the CustomerName only depends on the OrderID. This is a partial dependency and must be moved to a separate Orders table.

```sql
CREATE TABLE orders_2nf (
    order_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    customer_address VARCHAR(255),
    order_date DATE
);

CREATE TABLE books_2nf (
    isbn VARCHAR(20) PRIMARY KEY,
    title VARCHAR(200),
    author VARCHAR(100),
    price DECIMAL(10, 2)
);

CREATE TABLE order_items_2nf (
    order_id INT,
    book_isbn VARCHAR(20),
    quantity INT,
    total_price DECIMAL(10, 2),
    PRIMARY KEY (order_id, book_isbn),
    FOREIGN KEY (order_id) REFERENCES orders_2nf(order_id),
    FOREIGN KEY (book_isbn) REFERENCES books_2nf(isbn)
);

-- Sample data for 2NF tables
INSERT INTO orders_2nf VALUES
(1, 'John Smith', 'john@example.com', '123 Main St, Anytown', '2023-01-15'),
(2, 'Mary Johnson', 'mary@example.com', '456 Oak Ave, Somewhere', '2023-01-20'),
(3, 'Robert Brown', 'robert@example.com', '789 Pine Rd, Nowhere', '2023-01-25');

INSERT INTO books_2nf VALUES
('978-0141439518', 'Pride and Prejudice', 'Jane Austen', 9.99),
('978-0451524935', '1984', 'George Orwell', 12.99),
('978-0061120084', 'To Kill a Mockingbird', 'Harper Lee', 14.99);

INSERT INTO order_items_2nf VALUES
(1, '978-0141439518', 1, 9.99),
(1, '978-0451524935', 2, 25.98),
(2, '978-0061120084', 1, 14.99),
(3, '978-0141439518', 1, 9.99);
```

## Third Normal Form (3NF)

- A table is in 3NF if it is in 2NF and removes:
    - Transitive Dependencies: A non-key attribute should not depend on another non-key attribute.
    - Example: In an Orders table, CustomerEmail depends on CustomerName, which then depends on OrderID. Since Email depends on Name (both non-keys), you should create a dedicated Customers table to store that info.
    - Calculated Fields: Remove fields like TotalPrice if they can be calculated from Quantity and UnitPrice to ensure maximum normalization.

```sql
CREATE TABLE customers_3nf (
    customer_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    address VARCHAR(255)
);

CREATE TABLE orders_3nf (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers_3nf(customer_id)
);

CREATE TABLE books_3nf (
    isbn VARCHAR(20) PRIMARY KEY,
    title VARCHAR(200),
    author VARCHAR(100),
    price DECIMAL(10, 2)
);

CREATE TABLE order_items_3nf (
    order_id INT,
    book_isbn VARCHAR(20),
    quantity INT,
    PRIMARY KEY (order_id, book_isbn),
    FOREIGN KEY (order_id) REFERENCES orders_3nf(order_id),
    FOREIGN KEY (book_isbn) REFERENCES books_3nf(isbn)
);
```

- Note: The 3NF design removes the derived column total_price from order_items as it can be calculated from quantity * price.

## Practical Trade-offs

- While 3NF is the industry standard, higher normalization can lead to:
    - Complexity: More tables mean more complex queries
    - Performance Issues: Frequent use of JOINS can slow down the system
    - Denormalization: Sometimes, developers intentionally move back to 2NF or a lower form to improve read performance in large-scale systems.
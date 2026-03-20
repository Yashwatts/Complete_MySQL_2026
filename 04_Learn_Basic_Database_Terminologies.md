# What is Schema?

A schema is the logical blueprint or structure that defines how data is organized within a database

It decides:

- The tables in the database
- The relationships between those tables
- The attributes (columns) within each table
- The constraints and rules that govern the data
- The data types for each field

## Types of Schemas

- Database Schema: The complete structure of the entire database => Higher Level View
    E.g., Student, Department and Courses tables in a college database.
- Table Schema: The structure of a specific table including column names, data types and constraints.
    E.g., Roll No, Name column in a table and Number, Text as their data types.
- Subschema / View Schema: A portion of the database visible to specific users/applications.
    E.g., In a hospital, a Doctor sees patient history but might not see billing or insurance details.

## Index

A technique used to speed up data retrieval by sorting data (like a dictionary) so you don't have to scan the entire table.

Example: Finding a word in an alphabetical dictionary is much faster than searching through a book with random words.

## Query

A specific request made to the database to retrieve or manipulate data.

Example:

```sql
SELECT FirstName, LastName
FROM Employees
WHERE Department = 'IT';
```

## Result Set

The actual data returned by the database after a query is executed, usually displayed in a table format.

## Example

```sql
CREATE DATABASE SchoolDB;

USE SchoolDB;

CREATE TABLE Students (
        StudentID INT PRIMARY KEY,
        FirstName VARCHAR(50),
        LastName VARCHAR(50),
        EnrollmentDate DATE
);

INSERT INTO Students (StudentID, FirstName, LastName, EnrollmentDate)
VALUES
(1, 'John', 'Doe', '2023-01-15'),
(2, 'Jane', 'Smith', '2023-01-20');
```

## Query

```sql
SELECT * FROM Students;
```

## Result Set

| StudentID | FirstName | LastName | EnrollmentDate |
|-----------|-----------|----------|----------------|
| 1         | John      | Doe      | 2023-01-15     |
| 2         | Jane      | Smith    | 2023-01-20     |
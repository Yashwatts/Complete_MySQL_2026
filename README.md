# MySQL Complete Guide

This repository is a beginner-friendly, step-by-step set of notes to help you learn MySQL and core database concepts in a practical and structured way.

## Why this repo exists

I created this repository to:
- Learn MySQL in a structured way
- Build strong database fundamentals
- Document everything I learn (like a personal knowledge base)
- Help others who are starting with SQL

## What You Will Learn

- Database fundamentals and the relational model
- SQL basics and command categories (DDL, DML, DCL, TCL)
- MySQL installation and setup on Windows
- Essential database terminologies
- MySQL data types: Numeric, String, Date & Time, and Binary
- Database-level SQL commands (CREATE, SHOW, USE, SELECT DATABASE, DROP)
- Table-level SQL commands (CREATE TABLE, INSERT, ALTER, DROP)
- SELECT queries with WHERE, ORDER BY, LIMIT, DISTINCT, aggregate functions, subqueries, and UNION
- WHERE clause filtering with logical operators, NULL handling, LIKE patterns, ranges, lists, and subqueries
- Logical operators (AND, OR, NOT) to combine multiple conditions in queries
- Comparison operators (=, !=, <>, <, >, <=, >=) with numeric, date, and string filtering
- ORDER BY sorting with ASC/DESC, multi-column ordering, custom sorting, and performance tips
- Writing and understanding basic SQL queries
- LIMIT and OFFSET for pagination, efficient data retrieval, and performance best practices
- Aliases for columns, tables, expressions, and subqueries to improve readability
- Using DISTINCT to eliminate duplicate rows and return unique values
- SQL functions: string, numeric, date/time, aggregate, and more with practical examples
- Advanced filtering with GROUP BY, HAVING, and conditional logic
- Primary keys, composite keys, clustered indexes, pages, and buffer pool internals
- Foreign keys, referential integrity, relationship types, and junction tables
- Database normalization (1NF, 2NF, 3NF), anomalies, and practical trade-offs
- SQL JOINs (INNER, LEFT, RIGHT, FULL workaround, CROSS, SELF) and UNION operations
- Subqueries, correlated subqueries, EXISTS, NOT EXISTS, and subquery filtering patterns
- UPDATE statement basics, safe update mode, timestamp tracking, and update controls
- DELETE vs TRUNCATE, foreign key delete behaviors, and data removal best practices
- REPLACE statement behavior (insert-or-overwrite), conflict handling, and bulk replace with SELECT
- SQL indexing end-to-end: index types, B+ Tree internals, cardinality, covering indexes, invisible indexes, partial indexing patterns and index hints

## Repository Structure

- `01_Database_Explained.md` - Introduction to databases and relational concepts
- `02_SQL_Basics.md` - SQL fundamentals and command categories
- `03_Install_MySQL.md` - MySQL installation guide for Windows
- `04_Learn_Basic_Database_Terminologies.md` - Core database terms with examples
- `05_MySQL_DataTypes_Mastery.md` - Deep dive into MySQL data types (Numeric, String, Date & Time, Binary) with examples
- `06_SQL_Database_Commands.md` - Database-level SQL commands with syntax, tips, and examples
- `07_SQL_Table_Commands.md` - Table-level SQL commands with constraints, inserts, alterations, and drops
- `08_SQL_SELECT.md` - SELECT queries with syntax, filtering, sorting, functions, and advanced techniques
- `09_SQL_WHERE_Clause.md` - WHERE clause queries with filtering, operators, pattern matching, NULL checks, and subqueries
- `10_SQL_Logical_Operators.md` - Logical operators (AND, OR, NOT) for combining conditions with practical examples
- `11_SQL_Comparison_Operators.md` - Comparison operators with practical examples across numbers, dates, and strings
- `12_SQL_ORDER_BY.md` - ORDER BY clause with basic, multi-column, custom, and performance-focused sorting examples
- `13_SQL_LIMIT_AND_OFFSET.md` - LIMIT and OFFSET for pagination, efficient data retrieval, and performance best practices
- `14_SQL_Alias.md` - Aliases for columns, tables, expressions, and subqueries to improve readability
- `15_SQL_DISTINCT.md` - Using DISTINCT to eliminate duplicates and return unique values in result sets
- `16_SQL_Functions.md` - Comprehensive guide to SQL functions: string, numeric, date/time, aggregate, and more with practical examples
- `17_SQL_Filtering_GROUP_BY_AND_HAVING.md` - GROUP BY and HAVING clauses for grouping data, filtering groups, and advanced aggregation techniques
- `18_SQL_Primary_Keys.md` - Primary key concepts, syntax, composite keys, clustered index internals, pages, and buffer pool overview
- `19_SQL_Foreign_Keys.md` - Foreign key concepts, relationship types, constraints, and practical implementation examples
- `20_Database_Normalization.md` - Normalization fundamentals, 1NF/2NF/3NF design, anomaly reduction, and denormalization trade-offs
- `21_SQL_JOIN.md` - JOIN types, UNION/UNION ALL, full join workaround in MySQL, and performance tips
- `22_MySQL_Subqueries.md` - Subquery fundamentals, correlated subqueries, EXISTS/NOT EXISTS, and advanced filtering examples
- `23_MySQL_UPDATE.md` - UPDATE syntax, safe update mode, multi-column updates, timestamps, and update constraints
- `24_Delete_Vs_Truncate.md` - DELETE and TRUNCATE behavior, foreign key handling, and practical data removal scenarios
- `25_SQL_REPLACE.md` - REPLACE statement logic, conflict resolution, and bulk data replacement examples
- `26_Indexing_SQL.md` - primary/unique/composite/full-text indexes, clustered vs non-clustered, covering indexes, cardinality, partial indexing patterns, invisible indexes, index hints and performance benchmarking

## Who is this for?

- Beginners starting with MySQL and databases
- Students preparing for interviews or exams
- Developers who want a quick SQL and DBMS refresher

## Contribution / Support

- Found an issue or typo? Open an issue or suggest improvements.
- Want to contribute? Share better examples, clearer explanations or additional learning notes.
- Feel free to fork and improve this repo!
- If you like this repository, give it a ⭐
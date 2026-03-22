# DATA TYPES

## 1. Numeric Data Types

Numeric data is broadly categorized into Integers, Fixed-Point and Floating-Point numbers.

### A. Integer Types

| Data Type     | Storage (Bytes) | Range (Signed)                                     | Range (Unsigned)                         |
|---------------|------------------|----------------------------------------------------|------------------------------------------|
| TINYINT       | 1                | -128 to 127                                        | 0 to 255                                 |
| SMALLINT      | 2                | -32,768 to 32,767                                  | 0 to 65,535                              |
| MEDIUMINT     | 3                | -8,388,608 to 8,388,607                            | 0 to 16,777,215                          |
| INT / INTEGER | 4                | -2,147,483,648 to 2,147,483,647                   | 0 to 4,294,967,295                       |
| BIGINT        | 8                | -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 | 0 to 18,446,744,073,709,551,615 |

> No need to memorize these values. They are only for reference. You can always check the documentation or your notes.

For Example:
You can drink water from a glass, not from a bucket. This means every container size has a different role based on the need. Similarly, we choose number data types based on the requirement.

Suppose there are columns in a table like: `id`, `Mobile_Number`.
If there are 1000 students in a school, we can choose `SMALLINT` because it can easily cover all student IDs in this range.
If the user does not want to store `Mobile_Number` in string/text format, then `BIGINT` is a better choice.
(There are 10 digits in `INT/INTEGER` too, but if a mobile number starts from `9xxxxxxxxx`, it can go out of range for the `INTEGER` data type. Hence, we have to choose `BIGINT`.)

### Signed vs Unsigned

Signed allows negative and positive numbers.
Unsigned allows only positive numbers and increases the positive range.

### B. Floating-Point Types

| Data Type | Storage (Bytes) | Range |
|-----------|------------------|-------|
| FLOAT     | 4                | -3.402823466E+38 to -1.175494351E-38, 0 and 1.175494351E-38 to 3.402823466E+38 |
| DOUBLE    | 8                | -1.7976931348623157E+308 to -2.2250738585072014E-308, 0 and 2.2250738585072014E-308 to 1.7976931348623157E+308 |

For Example:

```sql
CREATE TABLE test1(num FLOAT);
INSERT INTO test1 VALUES (1234567891234567);
SELECT * FROM test1;

-- Output: 1.23457e15
```

### C. Fixed-Point Types

| Data Type    | Storage (Bytes) | Range (Signed)                                  | Range (Unsigned) |
|--------------|------------------|-------------------------------------------------|------------------|
| DECIMAL(p, s) | Varies          | Defined by precision (p) and scale (s)         | Same as signed   |

For Example:

```sql
CREATE TABLE test2 (num DECIMAL(5, 4));
INSERT INTO test2 VALUES (1);
SELECT * FROM test2;

-- Output: 1.0000

INSERT INTO test2 VALUES(1.23456885);

-- Output: 1.2346
```

- Note 1: Precision (p) max value is 65 and Scale (s) max value is 30.
- Note 2: If nothing is stored then by default it takes DECIMAL(10, 0).

## 2. String (Text) Data Types

| Data Type                 | Max Size                  | Storage                     | Best For                                           | Notes                                                                 | Example Usage                                |
|--------------------------|---------------------------|-----------------------------|---------------------------------------------------|-----------------------------------------------------------------------|----------------------------------------------|
| CHAR(n)                  | 255 characters            | Fixed n bytes               | Fixed-length strings (e.g., state codes, ISO codes) | Right-padded with spaces. Use sparingly - VARCHAR is usually better   | CHAR(2) for US state codes                   |
| VARCHAR(n)               | 65,535 bytes              | String length + 1-2 bytes   | Variable-length strings, most text fields         | Most commonly used string type. Length is in bytes, not characters    | VARCHAR(255) for product names               |
| TINYTEXT                 | 255 bytes                 | String length + 1 byte      | Short strings without need for index              | Cannot have default value. Rarely used - VARCHAR preferred            | Short comments or notes                      |
| TEXT                     | 65,535 bytes              | String length + 2 bytes     | Long text like articles or descriptions           | Cannot be part of an index without length specification               | Product descriptions, short articles         |
| MEDIUMTEXT               | 16,777,215 bytes (~16MB)  | String length + 3 bytes     | Large text documents                              | Consider performance impact on queries                                 | Blog posts, long articles                    |
| LONGTEXT                 | 4GB                       | String length + 4 bytes     | Very large text documents                         | Use with caution - can impact performance significantly                | Document storage, large text dumps           |
| ENUM('val1','val2',...)  | 65,535 members            | 1 or 2 bytes                | Single selection from a fixed list                | Consider VARCHAR if list might change often                            | ENUM('active','inactive','pending')          |
| SET('val1','val2',...)   | 64 members                | 1, 2, 3, 4, or 8 bytes      | Multiple selections from a fixed list             | Can store multiple values. Use with caution                            | SET('monday','tuesday','wednesday',...)      |

For Example:

```sql
CREATE TABLE product_details(
    product_id INT AUTO_INCREMENT PRIMARY KEY, -- Unique identifier for each product
    product_code CHAR(10), -- Fixed-length product code
    product_name VARCHAR(100), -- Product name
    short_description TINYTEXT, -- A brief description of the product
    detailed_description TEXT, -- Detailed product description
    additional_info MEDIUMTEXT, -- Additional product info like usage or warranty details
    full_manual LONGTEXT, -- Full product manual or documentation
    size ENUM('Small', 'Medium', 'Large'), -- Size options for the product
    available_colors SET('Red', 'Green', 'Blue', 'Black', 'White') -- Available color options
);
```

### A. CHAR(n)

- Fixed-length string: stores strings with a fixed length of n characters.
- If the data is shorter than n, it is padded with spaces on the right.
- Maximum length: 255 characters.
- Suitable for storing uniform-length data, such as codes or identifiers.

### B. VARCHAR(n)

- Variable-length string: stores strings up to n characters.
- Actual storage depends on string length, plus 1 or 2 bytes for length information.
- Maximum length: 65,535 bytes (depends on row size and character set, because each character can use different bytes).
- Commonly used for fields like names and addresses.
- Queries are generally fast because data is stored inside the table row.
- During indexing, VARCHAR can be fully indexed if size is less than or equal to 767 bytes (in newer versions, this limit was increased to 3072 bytes). If greater, we use prefix indexing.

### C. TINYTEXT

- Stores a small text string.
- Maximum length: 255 characters (1 byte overhead).
- Suitable for short descriptions or small text values.

### D. TEXT

- For MySQL TEXT types, the inline storage threshold is 768 bytes. After this size, data moves to separate pages and only a pointer stays in the table row.
- This can cause performance issues or slower queries because data is stored outside the main table row.
- During indexing, TEXT cannot be fully indexed, so we use prefix indexing.

### E. ENUM('val1', 'val2',...)

- Enumeration type: a string object with a predefined set of possible values.
- Each value is stored as a numeric index for efficiency.
- Useful for fields with a limited set of options, such as gender ('Male','Female').

## 3. Date & Time Data Types

| Data Type  | Storage (Bytes) | Description                                                                 | Example               |
|------------|------------------|-----------------------------------------------------------------------------|-----------------------|
| DATE       | 3                | Date values from '1000-01-01' to '9999-12-31'.                            | '2026-01-01'          |
| DATETIME   | 8                | Date and time values from '1000-01-01 00:00:00' to '9999-12-31 23:59:59'. | '2026-01-04 12:00:00' |
| TIMESTAMP  | 4                | UTC-based timestamp from '1970-01-01 00:00:01' UTC to '2038-01-19 03:14:07' UTC. | '2026-01-04 12:00:00' |
| TIME       | 3                | Time values from '-838:59:59' to '838:59:59'.                              | '12:34:56'            |
| YEAR       | 1                | Year values from '1901' to '2155'.                                          | '2026'                |

> Note: UTC helps represent time in a standard format across regions. MySQL can then convert and show time according to timezone settings, which makes time conversion easier and more reliable.

For Example:

```sql
CREATE TABLE EventSchedule (
    event_id INT AUTO_INCREMENT PRIMARY KEY,
    event_date DATE, -- To store the date of the event (e.g., '2026-01-16')
    event_datetime DATETIME, -- To store the exact date and time of the event (e.g., '2026-01-16 14:30:00')
    event_timestamp TIMESTAMP, -- To store the exact date and time in UTC (e.g., '2026-01-16 14:30:00')
    event_time TIME, -- To store the time of the event (e.g., '14:30:00')
    event_year YEAR -- To store the year of the event (e.g., '2026')
);
```
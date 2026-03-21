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



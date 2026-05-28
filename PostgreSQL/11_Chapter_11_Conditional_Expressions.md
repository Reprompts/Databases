
# PostgreSQL Hands-On Tutorial
## Chapter 11: Conditional Expressions

In PostgreSQL, conditional expressions allow queries to make decisions and dynamically compute values based on conditions. They work similarly to if–else logic in programming languages.

Conditional expressions are widely used in:

- data transformation
- reporting queries
- handling missing values
- conditional calculations
- data cleaning
- analytics queries

For example:

- categorizing users by age group
- replacing NULL values with defaults
- avoiding division-by-zero errors
- conditionally displaying values

PostgreSQL provides several powerful conditional expressions:

- CASE expressions
- COALESCE function
- NULLIF function
- NULL value handling mechanisms

This guide explains these concepts in extreme detail with practical SQL examples.

---

# 1. Understanding Conditional Expressions

A conditional expression evaluates a condition and returns a result based on that condition.

Conceptually:

```sql
IF condition THEN result
ELSE other_result
````

SQL implements this through:

* CASE
* COALESCE
* NULLIF

These expressions can be used in:

* SELECT queries
* UPDATE statements
* ORDER BY clauses
* WHERE conditions
* calculated columns

---

# 2. CASE Expressions

The CASE expression is the most powerful conditional expression in SQL.

It works like if–else or switch statements in programming languages.

## Types of CASE Expressions

PostgreSQL supports two types:

* Simple CASE
* Searched CASE

---

## Simple CASE Expression

Simple CASE compares a value against multiple possible values.

### Syntax

```sql
CASE expression
    WHEN value1 THEN result1
    WHEN value2 THEN result2
    ELSE result3
END
```

### Example Table

```sql
CREATE TABLE employees (
    id SERIAL,
    name TEXT,
    department TEXT
);
```

### Insert Data

```sql
INSERT INTO employees (name,department) VALUES
('Alice','IT'),
('Bob','HR'),
('Charlie','Finance'),
('David','IT');
```

### Example Query

```sql
SELECT name,
CASE department
    WHEN 'IT' THEN 'Technology'
    WHEN 'HR' THEN 'Human Resources'
    ELSE 'Other Department'
END AS department_name
FROM employees;
```

### Result

| name    | department_name  |
| ------- | ---------------- |
| Alice   | Technology       |
| Bob     | Human Resources  |
| Charlie | Other Department |
| David   | Technology       |

---

## Searched CASE Expression

The searched CASE version uses full conditions.

### Syntax

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ELSE result3
END
```

### Example: Categorizing Ages

```sql
SELECT name,
CASE
    WHEN age < 18 THEN 'Minor'
    WHEN age BETWEEN 18 AND 30 THEN 'Young Adult'
    WHEN age BETWEEN 31 AND 50 THEN 'Adult'
    ELSE 'Senior'
END AS age_group
FROM users;
```

### Result

| name | age_group   |
| ---- | ----------- |
| Emma | Young Adult |
| Bob  | Adult       |

---

## CASE in ORDER BY

CASE can control sorting behavior.

### Example

```sql
SELECT name, department
FROM employees
ORDER BY
CASE
    WHEN department = 'IT' THEN 1
    WHEN department = 'HR' THEN 2
    ELSE 3
END;
```

This prioritizes departments.

---

## CASE in UPDATE

CASE can modify data conditionally.

### Example

```sql
UPDATE employees
SET salary =
CASE
    WHEN department = 'IT' THEN salary * 1.10
    WHEN department = 'HR' THEN salary * 1.05
    ELSE salary
END;
```

### Meaning

* IT → 10% raise
* HR → 5% raise
* Others → unchanged

---

## CASE in Aggregations

### Example

Count users by category.

```sql
SELECT
SUM(CASE WHEN age < 30 THEN 1 ELSE 0 END) AS young_users,
SUM(CASE WHEN age >= 30 THEN 1 ELSE 0 END) AS older_users
FROM users;
```

This technique is widely used in analytics queries.

---

# 3. COALESCE Function

The COALESCE function returns the first non-null value from a list.

## Syntax

```sql
COALESCE(value1, value2, value3)
```

### Execution

Return first value that is NOT NULL.

---

## Example

Table:

```text
users
name | phone | mobile
```

Some users may have missing phone numbers.

### Query

```sql
SELECT name,
COALESCE(phone, mobile) AS contact_number
FROM users;
```

### Meaning

* use phone
* if phone is NULL → use mobile

---

## Example with Default Values

```sql
SELECT name,
COALESCE(email, 'No Email Provided') AS email
FROM users;
```

### Result

| name  | email             |
| ----- | ----------------- |
| Alice | alice@email       |
| Bob   | No Email Provided |

---

## COALESCE with Multiple Values

### Example

```sql
SELECT
COALESCE(phone, mobile, office_phone, 'No contact')
FROM users;
```

### Priority Order

```text
phone → mobile → office_phone → fallback text
```

---

## Practical Use Case

Handling missing numeric values.

### Example

```sql
SELECT price * COALESCE(discount,1)
FROM products;
```

If discount is NULL:

```text
default multiplier = 1
```

---

# 4. NULLIF Function

The NULLIF function returns NULL if two values are equal.

## Syntax

```sql
NULLIF(value1, value2)
```

### Behavior

* if value1 = value2 → return NULL
* else → return value1

---

## Example

```sql
SELECT NULLIF(10,10);
```

### Result

```text
NULL
```

---

## Another Example

```sql
SELECT NULLIF(10,5);
```

### Result

```text
10
```

---

## Practical Example - Prevent Division by Zero

Division by zero causes an error.

### Problem

```sql
SELECT sales / quantity
FROM orders;
```

If quantity = 0 → error.

### Solution

```sql
SELECT sales / NULLIF(quantity,0)
FROM orders;
```

If quantity = 0:

* NULLIF returns NULL
* division becomes NULL instead of error

---

## Another Example

Handling empty strings.

```sql
SELECT NULLIF(username,'')
FROM users;
```

Empty strings become NULL.

---

# 5. Understanding NULL Values

NULL represents missing or unknown data.

Important concept:

```text
NULL ≠ 0
NULL ≠ empty string
NULL ≠ false
```

NULL means:

```text
unknown value
```

---

## Example Table

```text
users
name | phone
----------------
Alice | NULL
Bob   | 999999
```

---

## Checking NULL Values

You cannot use:

```sql
= NULL
```

Instead use:

```sql
IS NULL
IS NOT NULL
```

### Example

Find users without phone numbers:

```sql
SELECT *
FROM users
WHERE phone IS NULL;
```

---

## Example - IS NOT NULL

```sql
SELECT *
FROM users
WHERE phone IS NOT NULL;
```

---

## NULL Behavior in Comparisons

### Example

```sql
SELECT 5 = NULL;
```

### Result

```text
NULL
```

Not true or false.

---

## NULL in Aggregation

Functions like:

* AVG()
* SUM()
* COUNT()

ignore NULL values.

### Example

Values:

```text
10, 20, NULL
```

Average:

```text
(10 + 20) / 2 = 15
```

---

## COUNT Behavior

### Example

```sql
SELECT COUNT(phone)
FROM users;
```

Counts only non-null values.

But:

```sql
SELECT COUNT(*)
FROM users;
```

Counts all rows.

---

# 6. Combining CASE, COALESCE, and NULLIF

These functions are often used together.

### Example

```sql
SELECT name,
CASE
    WHEN age IS NULL THEN 'Unknown Age'
    WHEN age < 18 THEN 'Minor'
    ELSE 'Adult'
END
FROM users;
```

---

## Another Example

```sql
SELECT
COALESCE(NULLIF(phone,''),'No Phone')
FROM users;
```

### Logic

```text
empty string → NULL
NULL → replace with 'No Phone'
```

---

# 7. Real-World Example - E-Commerce System

## Products Table

```sql
CREATE TABLE products (
    id SERIAL,
    name TEXT,
    price NUMERIC,
    discount NUMERIC
);
```

---

## Calculate Final Price

```sql
SELECT name,
price - COALESCE(discount,0) AS final_price
FROM products;
```

If discount is NULL:

```text
assume discount = 0
```

---

## Categorize Product Price

```sql
SELECT name,
CASE
    WHEN price < 100 THEN 'Cheap'
    WHEN price BETWEEN 100 AND 1000 THEN 'Moderate'
    ELSE 'Expensive'
END
FROM products;
```

---

## Avoid Division Errors

```sql
SELECT revenue / NULLIF(quantity,0)
FROM sales;
```

---

# 8. Execution Behavior

Conditional expressions are evaluated during query execution.

## Query Pipeline

```text
Parser
Planner
Executor
```

CASE and functions execute row-by-row during the executor stage.

---

# 9. Best Practices

* Use CASE for complex conditional logic.
* Use COALESCE to replace NULL values.
* Use NULLIF to prevent errors (especially division by zero).
* Always check NULL using:

  * IS NULL
  * IS NOT NULL

Avoid:

```sql
= NULL
```

---

# 10. Summary

Conditional expressions allow PostgreSQL queries to behave intelligently based on data conditions.

## Key Tools

| Expression | Purpose                      |
| ---------- | ---------------------------- |
| CASE       | Conditional logic            |
| COALESCE   | Replace NULL values          |
| NULLIF     | Convert equal values to NULL |
| IS NULL    | Check missing values         |

These expressions are essential for:

* analytics queries
* reporting systems
* data cleaning
* application logic

They allow SQL queries to become dynamic, safe, and flexible.

```

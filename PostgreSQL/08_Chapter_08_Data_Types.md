# PostgreSQL Hands-On Tutorial
## Chapter 8: Data Types

In PostgreSQL, data types define the kind of data that can be stored in a column. Every column in a table must have a defined data type, which determines:

* how the data is stored in memory
* how much space it uses
* what operations can be performed on it
* how PostgreSQL validates and processes the data

Choosing the correct data type is critical for:

* data integrity
* query performance
* storage efficiency
* correct calculations

PostgreSQL provides a rich and powerful set of built-in data types, far more advanced than many other relational databases.

This guide explains PostgreSQL data types in extreme practical detail, including:

* Numeric types
* Character types
* Date and time types
* Boolean
* UUID
* JSON / JSONB
* Array types
* Composite types

---

# 1. Numeric Types in PostgreSQL

Numeric types store numbers used for calculations, counters, measurements, and financial data.

PostgreSQL numeric types include:

| Type              | Description            |
| ----------------- | ---------------------- |
| SMALLINT          | Small integer          |
| INTEGER           | Standard integer       |
| BIGINT            | Large integer          |
| REAL              | Single precision float |
| DOUBLE PRECISION  | Double precision float |
| NUMERIC / DECIMAL | Exact precision number |

---

# 2. Integer Types

Integer types store whole numbers without decimals.

## PostgreSQL Integer Types

| Type     | Size    | Range                    |
| -------- | ------- | ------------------------ |
| SMALLINT | 2 bytes | -32,768 to 32,767        |
| INTEGER  | 4 bytes | Approximately -2B to +2B |
| BIGINT   | 8 bytes | Very large numbers       |

## SMALLINT

Used for small numbers.

Example:

```sql
CREATE TABLE ratings (
    score SMALLINT
);
```

Insert data:

```sql
INSERT INTO ratings VALUES (5);
```

Good use cases:

* rating systems
* small counters
* status codes

---

## INTEGER

Most commonly used numeric type.

Example:

```sql
CREATE TABLE users (
    id INTEGER
);
```

Typical use cases:

* IDs
* counters
* quantities
* indexes

---

## BIGINT

Used for very large numbers.

Example:

```sql
CREATE TABLE video_views (
    views BIGINT
);
```

Use cases:

* social media counters
* analytics
* financial systems

---

# 3. Floating Point Types

Floating-point types store approximate decimal numbers.

These are used for:

* scientific data
* measurements
* statistics

---

## REAL

Single precision floating point.

Example:

```sql
CREATE TABLE sensors (
    temperature REAL
);
```

Insert:

```sql
INSERT INTO sensors VALUES (23.56);
```

---

## DOUBLE PRECISION

Higher precision floating point.

Example:

```sql
CREATE TABLE physics_data (
    velocity DOUBLE PRECISION
);
```

More accurate than `REAL`.

⚠ Important:

Floating numbers may produce rounding errors.

Example:

```text
0.1 + 0.2 ≠ exactly 0.3
```

Because floats use binary approximation.

---

# 4. Decimal / Numeric Types

For exact numeric values, PostgreSQL provides:

* NUMERIC
* DECIMAL

These store numbers with precise decimal places.

## Syntax

```sql
NUMERIC(precision, scale)
```

Where:

* precision = total digits
* scale = digits after decimal

Example:

```sql
CREATE TABLE products (
    price NUMERIC(10,2)
);
```

Meaning:

* maximum digits = 10
* decimal digits = 2

Example values:

* 12.99
* 1000.50

## Use Cases

Use `NUMERIC` for:

* financial systems
* banking
* accounting
* currency calculations

Example:

```sql
CREATE TABLE invoices (
    amount NUMERIC(12,2)
);
```

---

# 5. Character Types

Character types store text and string data.

PostgreSQL supports:

| Type       | Description            |
| ---------- | ---------------------- |
| CHAR(n)    | Fixed-length string    |
| VARCHAR(n) | Variable-length string |
| TEXT       | Unlimited text         |

---

# 6. Character (CHAR)

Stores fixed-length strings.

Example:

```sql
CHAR(10)
```

If shorter strings are inserted, PostgreSQL pads spaces.

Example:

```text
'abc'
```

Stored as:

```text
'abc       '
```

Example table:

```sql
CREATE TABLE country_codes (
    code CHAR(2)
);
```

Use cases:

* ISO codes
* fixed identifiers

---

# 7. VARCHAR

Variable-length character string.

Example:

```sql
VARCHAR(50)
```

Stores up to 50 characters.

Example:

```sql
CREATE TABLE users (
    username VARCHAR(50)
);
```

Insert:

```sql
INSERT INTO users VALUES ('ganesh');
```

Advantages:

* no wasted storage
* flexible length

---

# 8. TEXT

Stores unlimited length text.

Example:

```sql
CREATE TABLE articles (
    content TEXT
);
```

Good for:

* blog posts
* descriptions
* logs
* documents

Important fact:

In PostgreSQL:

`TEXT` and `VARCHAR` have nearly identical performance.

Therefore `TEXT` is widely used.

---

# 9. Date and Time Types

PostgreSQL provides powerful time handling.

Types include:

| Type      | Purpose         |
| --------- | --------------- |
| DATE      | Calendar date   |
| TIME      | Time of day     |
| TIMESTAMP | Date + time     |
| INTERVAL  | Time difference |

---

# 10. DATE

Stores calendar date.

Example:

```sql
CREATE TABLE events (
    event_date DATE
);
```

Insert:

```sql
INSERT INTO events VALUES ('2025-01-10');
```

Format:

```text
YYYY-MM-DD
```

Use cases:

* birthdays
* appointments
* deadlines

---

# 11. TIME

Stores time of day.

Example:

```sql
CREATE TABLE meetings (
    start_time TIME
);
```

Insert:

```sql
INSERT INTO meetings VALUES ('14:30:00');
```

Format:

```text
HH:MM:SS
```

---

# 12. TIMESTAMP

Stores date and time together.

Example:

```sql
CREATE TABLE logs (
    created_at TIMESTAMP
);
```

Insert:

```sql
INSERT INTO logs VALUES ('2025-01-10 14:30:00');
```

Common use:

```sql
DEFAULT CURRENT_TIMESTAMP
```

Example:

```sql
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
```

---

# 13. INTERVAL

Stores time durations.

Example:

```sql
CREATE TABLE tasks (
    duration INTERVAL
);
```

Insert:

```sql
INSERT INTO tasks VALUES ('2 hours');
```

Examples:

* '3 days'
* '5 hours'
* '30 minutes'

Example query:

```sql
SELECT NOW() + INTERVAL '2 days';
```

---

# 14. Boolean Type

Stores true or false values.

Possible values:

* TRUE
* FALSE
* NULL

Example:

```sql
CREATE TABLE users (
    is_active BOOLEAN
);
```

Insert:

```sql
INSERT INTO users VALUES (TRUE);
```

Accepted inputs:

* true
* false
* t
* f
* 1
* 0
* yes
* no

---

# 15. UUID Type

UUID = Universally Unique Identifier.

Example value:

```text
550e8400-e29b-41d4-a716-446655440000
```

Used for globally unique IDs.

Example table:

```sql
CREATE TABLE api_keys (
    id UUID
);
```

Generate automatically:

```sql
CREATE EXTENSION "uuid-ossp";
```

Then:

```sql
SELECT uuid_generate_v4();
```

Use cases:

* distributed systems
* microservices
* API identifiers

---

# 16. JSON Type

PostgreSQL supports storing JSON documents.

Example:

```sql
CREATE TABLE products (
    data JSON
);
```

Insert:

```sql
INSERT INTO products VALUES
('{"name":"Laptop","price":1200}');
```

Advantages:

* flexible schema
* nested data
* document-like storage

---

# 17. JSONB Type

JSONB = Binary JSON

It is the recommended JSON type.

## Differences

| JSON  | JSONB            | Description          |
| ----- | ---------------- | -------------------- |
| JSON  | Stored as text   | Preserves formatting |
| JSONB | Stored as binary | Optimized structure  |
| JSON  | Slower queries   |                      |
| JSONB | Faster queries   |                      |

Example:

```sql
CREATE TABLE orders (
    details JSONB
);
```

Insert:

```sql
INSERT INTO orders VALUES
('{"items":3,"total":150}');
```

Query JSONB:

```sql
SELECT details->>'items'
FROM orders;
```

---

# 18. Array Types

PostgreSQL allows arrays inside columns.

Example:

```sql
CREATE TABLE students (
    marks INTEGER[]
);
```

Insert:

```sql
INSERT INTO students VALUES
('{80,85,90}');
```

Query array elements:

```sql
SELECT marks[1]
FROM students;
```

Output:

```text
80
```

Use cases:

* tags
* multiple values
* category lists

---

# 19. Composite Types

Composite types allow custom structured types.

Example:

```sql
CREATE TYPE address AS (
    street TEXT,
    city TEXT,
    zip TEXT
);
```

Now use in tables:

```sql
CREATE TABLE customers (
    id SERIAL,
    home_address address
);
```

Insert:

```sql
INSERT INTO customers VALUES
(1, ('Main Street','Mumbai','400001'));
```

Composite types are similar to objects or structs.

---

# 20. Practical Example — Real Database Table

Example combining multiple data types:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email TEXT,
    age INTEGER,
    salary NUMERIC(10,2),
    is_active BOOLEAN,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    preferences JSONB,
    tags TEXT[]
);
```

Example insert:

```sql
INSERT INTO users
(name,email,age,salary,is_active,preferences,tags)
VALUES
(
 'Ganesh',
 'ganesh@email.com',
 25,
 50000.00,
 TRUE,
 '{"theme":"dark"}',
 '{developer,postgres}'
);
```

---

# 21. PostgreSQL Data Type Architecture

Internally PostgreSQL stores data types in system catalogs:

* pg_type
* pg_attribute
* pg_class

Example query:

```sql
SELECT typname
FROM pg_type;
```

This lists all PostgreSQL types.

---

# 22. Best Practices for Data Types

Professional PostgreSQL systems follow these rules:

## Use INTEGER for IDs

```sql
id SERIAL
```

---

## Use NUMERIC for Money

```sql
price NUMERIC(10,2)
```

---

## Use TEXT for Long Content

```sql
description TEXT
```

---

## Use TIMESTAMP for Logs

```sql
created_at TIMESTAMP
```

---

## Use JSONB for Flexible Data

```sql
metadata JSONB
```

---

# Conclusion

PostgreSQL provides one of the most powerful and flexible data type systems among relational databases.

These data types allow PostgreSQL to support:

* traditional relational data
* financial calculations
* time-series data
* document storage
* semi-structured data
* complex structures

Key categories include:

* numeric types
* character types
* date/time types
* boolean
* UUID
* JSON / JSONB
* arrays
* composite types

Understanding these types is essential for designing efficient, scalable, and reliable PostgreSQL databases.

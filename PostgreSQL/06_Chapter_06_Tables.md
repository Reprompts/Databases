# PostgreSQL Hands-On Tutorial
## Chapter 6: Tables

Tables are the primary structure used to store data in PostgreSQL. Almost all persistent data inside a PostgreSQL database is stored in tables.

A table consists of:

* **Rows (records)** — individual entries of data
* **Columns (fields)** — attributes describing the data

Tables can represent real-world entities such as:

* users
* orders
* products
* transactions
* logs

This article provides an extremely detailed and practical explanation of tables in PostgreSQL, including operations such as:

* Creating tables
* Listing tables
* Altering tables
* Renaming tables
* Dropping tables
* Temporary tables
* Unlogged tables

---

# 1. Understanding Tables in PostgreSQL

A table stores structured data in rows and columns.

Example table:

```text
users
------------------------------------------------
id | name       | email               | created
------------------------------------------------
1  | Alice      | alice@email.com     | 2024-01-01
2  | Bob        | bob@email.com       | 2024-01-03
```

## Table Components

A table contains several important elements.

## Columns

Columns define the structure of the table.

Example:

* id
* name
* email
* created_at

Each column has:

* data type
* constraints
* default values

## Rows

Rows represent actual data.

Example row:

```text
1 | Alice | alice@email.com | 2024-01-01
```

## Constraints

Constraints enforce rules.

Examples:

* PRIMARY KEY
* UNIQUE
* NOT NULL
* CHECK
* FOREIGN KEY

## Table Metadata

PostgreSQL stores table information in system catalogs.

Important catalogs:

* `pg_class`
* `pg_attribute`
* `pg_tables`
* `pg_constraint`

---

# 2. Creating Tables

Tables are created using the `CREATE TABLE` command.

## Basic Syntax

```sql
CREATE TABLE table_name (
    column_name data_type,
    column_name data_type
);
```

## Example

```sql
CREATE TABLE users (
    id SERIAL,
    name TEXT,
    email TEXT
);
```

This creates a table with three columns.

## Table with Constraints

Example:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Explanation

| Element     | Meaning                |
| ----------- | ---------------------- |
| SERIAL      | Auto-increment integer |
| PRIMARY KEY | Unique identifier      |
| NOT NULL    | Prevents empty values  |
| UNIQUE      | Prevents duplicates    |
| DEFAULT     | Automatic value        |

## Example Table for E-commerce

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    price NUMERIC(10,2),
    stock INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Creating Tables in Specific Schema

Example:

```sql
CREATE TABLE inventory.products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    price NUMERIC
);
```

Now the table belongs to the `inventory` schema.

---

# 3. Listing Tables

PostgreSQL provides multiple ways to list tables.

## Method 1: psql Command

Inside psql:

```sql
\dt
```

Example output:

```text
Schema | Name     | Type  | Owner
----------------------------------
public | users    | table | postgres
public | products | table | postgres
```

## List Tables in Specific Schema

```sql
\dt inventory.*
```

## Method 2: SQL Query

Tables are stored in:

```text
pg_tables
```

Query:

```sql
SELECT tablename
FROM pg_tables
WHERE schemaname = 'public';
```

Example output:

```text
tablename
---------
users
products
orders
```

## Method 3: information_schema

```sql
SELECT table_name
FROM information_schema.tables
WHERE table_schema='public';
```

---

# 4. Altering Tables

Tables often need modifications after creation.

PostgreSQL uses the `ALTER TABLE` command.

## Adding a Column

### Syntax

```sql
ALTER TABLE table_name
ADD COLUMN column_name data_type;
```

### Example

```sql
ALTER TABLE users
ADD COLUMN age INTEGER;
```

Now the table contains a new column.

## Adding Column with Default Value

Example:

```sql
ALTER TABLE users
ADD COLUMN status TEXT DEFAULT 'active';
```

## Removing a Column

### Syntax

```sql
ALTER TABLE table_name
DROP COLUMN column_name;
```

### Example

```sql
ALTER TABLE users
DROP COLUMN age;
```

## Changing Column Type

Example:

```sql
ALTER TABLE users
ALTER COLUMN name TYPE VARCHAR(100);
```

## Setting NOT NULL

```sql
ALTER TABLE users
ALTER COLUMN name SET NOT NULL;
```

## Removing NOT NULL

```sql
ALTER TABLE users
ALTER COLUMN name DROP NOT NULL;
```

## Adding Constraints

Example:

```sql
ALTER TABLE users
ADD CONSTRAINT email_unique UNIQUE(email);
```

## Dropping Constraints

Example:

```sql
ALTER TABLE users
DROP CONSTRAINT email_unique;
```

---

# 5. Renaming Tables

Tables can be renamed using `ALTER TABLE`.

## Syntax

```sql
ALTER TABLE old_name
RENAME TO new_name;
```

## Example

```sql
ALTER TABLE users
RENAME TO customers;
```

Now the table name becomes:

```text
customers
```

## Renaming Columns

Example:

```sql
ALTER TABLE customers
RENAME COLUMN name TO full_name;
```

---

# 6. Dropping Tables

Dropping a table removes it permanently.

## Syntax

```sql
DROP TABLE table_name;
```

## Example

```sql
DROP TABLE users;
```

This deletes:

* table structure
* all rows
* indexes
* constraints

## Safe Drop

```sql
DROP TABLE IF EXISTS users;
```

## Drop Multiple Tables

```sql
DROP TABLE users, products;
```

## Drop with Dependencies

If other objects depend on the table:

```text
ERROR: cannot drop because other objects depend on it
```

Use `CASCADE`:

```sql
DROP TABLE users CASCADE;
```

---

# 7. Temporary Tables

Temporary tables exist only during a database session.

When the session ends, the table is automatically removed.

## Creating Temporary Table

```sql
CREATE TEMP TABLE temp_users (
    id INT,
    name TEXT
);
```

## Characteristics

Temporary tables:

* visible only to current session
* automatically dropped
* stored in temporary schema

## Example Workflow

```text
Session Start
    |
Create TEMP table
    |
Insert data
    |
Session End
    |
Table Deleted
```

## Example

```sql
CREATE TEMP TABLE session_data (
    key TEXT,
    value TEXT
);
```

Use case:

* caching results
* intermediate calculations
* ETL processing

---

# 8. Unlogged Tables

Unlogged tables improve performance by skipping WAL logging.

However, they are not crash-safe.

## Creating Unlogged Table

```sql
CREATE UNLOGGED TABLE logs (
    id SERIAL,
    message TEXT
);
```

## Characteristics

| Feature        | Behavior       |
| -------------- | -------------- |
| WAL logging    | Disabled       |
| Performance    | Faster         |
| Crash recovery | Data lost      |
| Replication    | Not replicated |

## Use Cases

Unlogged tables are useful for:

* temporary analytics data
* logs
* staging tables
* caching

## Example

```sql
CREATE UNLOGGED TABLE analytics_cache (
    query TEXT,
    result JSON
);
```

---

# 9. Table Storage Internals

Each PostgreSQL table is stored as a heap file.

Location:

```text
data/base/<database_oid>/<table_oid>
```

Example:

```text
data/base/16384/24576
```

Internally:

* tables are divided into pages
* each page is 8KB

Structure:

```text
Table File
 │
 ├── Page 1
 ├── Page 2
 ├── Page 3
 └── Page N
```

Each page stores multiple rows.

---

# 10. Practical Example — Complete Workflow

Let's create and manage a table step by step.

## Step 1: Create Table

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name TEXT,
    department TEXT,
    salary NUMERIC
);
```

## Step 2: Insert Data

```sql
INSERT INTO employees (name, department, salary)
VALUES
('Alice','Engineering',70000),
('Bob','Marketing',50000);
```

## Step 3: List Tables

```sql
\dt
```

## Step 4: Modify Table

```sql
ALTER TABLE employees
ADD COLUMN hired_date DATE;
```

## Step 5: Rename Table

```sql
ALTER TABLE employees
RENAME TO staff;
```

## Step 6: Drop Table

```sql
DROP TABLE staff;
```

---

# 11. Best Practices for Table Design

Professional database systems follow these practices.

## Use Meaningful Table Names

Good:

* users
* orders
* products

Bad:

* table1
* data123

## Use Primary Keys

Every table should have:

```text
PRIMARY KEY
```

Example:

```sql
id SERIAL PRIMARY KEY
```

## Avoid Too Many Columns

Large tables reduce performance.

## Use Proper Data Types

Example:

* `INTEGER` for numbers
* `TIMESTAMP` for dates
* `TEXT` for strings

## Normalize Data

Example:

Instead of:

```text
orders
name
address
product
```

Split into:

```text
users
orders
products
```

---

# PostgreSQL Table Architecture Overview

```text
Database
   │
   ▼
Schema
   │
   ▼
Table
   │
 ┌─┴──────────────┐
 │ Columns        │
 │ Constraints    │
 │ Indexes        │
 └───────────────┘
   │
   ▼
Rows
   │
   ▼
Disk Storage (Pages)
```

---

# Conclusion

Tables are the foundation of data storage in PostgreSQL. They provide a structured way to store, organize, and manage information.

Key table operations include:

* creating tables
* listing tables
* altering table structures
* renaming tables
* dropping tables
* using temporary tables
* using unlogged tables

Understanding these operations allows developers and database administrators to build robust and scalable database systems.

Tables combined with:

* schemas
* roles
* indexes
* constraints

form the core structure of PostgreSQL data management.

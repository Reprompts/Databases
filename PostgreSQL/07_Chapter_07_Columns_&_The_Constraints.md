# PostgreSQL Hands-On Tutorial
## Chapter 7: Columns & the Constraints 

In PostgreSQL, tables are defined by columns, and constraints enforce rules on the data stored in those columns. Together, columns and constraints ensure that the data stored in the database is structured, valid, and consistent.

This article explains table columns and constraints in PostgreSQL in extreme detail with practical examples, including:

* Column operations
* Adding columns
* Modifying columns
* Dropping columns
* Primary key constraint
* Foreign key constraint
* Unique constraint
* Check constraint
* Not null constraint
* Default values

---

# 1. Understanding Table Columns

A column represents a specific attribute of an entity stored in a table.

Example table:

```text
users
------------------------------------------------
id | name | email | age | created_at
------------------------------------------------
1  | Alice | alice@email.com | 25 | 2024-01-01
```

Each column has:

* a name
* a data type
* optional constraints
* optional default values

## Column Structure

Example definition:

```sql
name TEXT NOT NULL
```

Components:

| Element  | Meaning                     |
| -------- | --------------------------- |
| name     | Column name                 |
| TEXT     | Data type                   |
| NOT NULL | Constraint (cannot be NULL) |

---

# 2. Column Operations

Columns can be modified after a table is created.

Common column operations include:

* adding columns
* modifying columns
* dropping columns

These operations are performed using the `ALTER TABLE` command.

---

# 3. Adding Columns

Columns can be added to existing tables.

## Basic Syntax

```sql
ALTER TABLE table_name
ADD COLUMN column_name data_type;
```

## Example

Suppose we have a table:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT
);
```

Add an email column:

```sql
ALTER TABLE users
ADD COLUMN email TEXT;
```

Now the table becomes:

```text
users
--------------------------------
id | name | email
--------------------------------
```

## Adding Column with Default Value

```sql
ALTER TABLE users
ADD COLUMN status TEXT DEFAULT 'active';
```

Now new rows automatically receive:

```text
status = 'active'
```

## Adding Column with Constraint

Example:

```sql
ALTER TABLE users
ADD COLUMN age INTEGER CHECK (age >= 18);
```

Now only values 18 or above are allowed.

## Adding Multiple Columns

```sql
ALTER TABLE users
ADD COLUMN phone TEXT,
ADD COLUMN address TEXT;
```

---

# 4. Modifying Columns

Columns may need to be changed after creation.

Operations include:

* changing data type
* adding constraints
* removing constraints
* renaming columns

## Changing Column Data Type

### Syntax

```sql
ALTER TABLE table_name
ALTER COLUMN column_name TYPE new_data_type;
```

### Example

```sql
ALTER TABLE users
ALTER COLUMN name TYPE VARCHAR(100);
```

Now the column stores up to 100 characters.

## Changing Column Default Value

Example:

```sql
ALTER TABLE users
ALTER COLUMN status SET DEFAULT 'pending';
```

Remove default:

```sql
ALTER TABLE users
ALTER COLUMN status DROP DEFAULT;
```

## Renaming Columns

Example:

```sql
ALTER TABLE users
RENAME COLUMN name TO full_name;
```

## Adding NOT NULL Constraint

```sql
ALTER TABLE users
ALTER COLUMN email SET NOT NULL;
```

## Removing NOT NULL Constraint

```sql
ALTER TABLE users
ALTER COLUMN email DROP NOT NULL;
```

---

# 5. Dropping Columns

Columns can be removed from tables.

## Syntax

```sql
ALTER TABLE table_name
DROP COLUMN column_name;
```

## Example

```sql
ALTER TABLE users
DROP COLUMN age;
```

Now the column disappears from the table.

## Drop with CASCADE

If objects depend on the column:

```sql
ALTER TABLE users
DROP COLUMN age CASCADE;
```

This removes dependent objects like indexes.

---

# 6. Constraints

Constraints enforce rules on data.

They ensure that invalid data cannot be inserted or updated.

Common constraints include:

* Primary key
* Foreign key
* Unique
* Check
* Not null
* Default values

---

# 7. Primary Key Constraint

A primary key uniquely identifies each row in a table.

## Properties

* unique
* not null
* indexed automatically

## Syntax

```sql
PRIMARY KEY (column_name)
```

## Example

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT
);
```

Table:

```text
users
-----------------
id | name
-----------------
1  | Alice
2  | Bob
```

Duplicate IDs are not allowed.

## Composite Primary Key

Sometimes multiple columns form the key.

Example:

```sql
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    PRIMARY KEY (order_id, product_id)
);
```

This ensures the same product cannot appear twice in one order.

---

# 8. Foreign Key Constraint

A foreign key links one table to another.

It ensures referential integrity.

## Example Scenario

Two tables:

* users
* orders

Orders belong to users.

## Example Table

Users table:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT
);
```

Orders table:

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT,
    FOREIGN KEY (user_id)
    REFERENCES users(id)
);
```

## Referential Integrity

Allowed:

```text
user_id = existing user
```

Not allowed:

```text
user_id = non-existing user
```

Example error:

```text
ERROR: insert or update violates foreign key constraint
```

## Foreign Key Actions

```text
+-----------+----------------------------------------------+
| Action    | Meaning                                      |
+-----------+----------------------------------------------+
| CASCADE   | Delete dependent rows                        |
| SET NULL  | Set foreign key column value to NULL         |
| RESTRICT  | Prevent deletion if dependent rows exist     |
| NO ACTION | Default behavior (similar to RESTRICT)       |
+-----------+----------------------------------------------+
```

## Example with CASCADE

```sql
FOREIGN KEY (user_id)
REFERENCES users(id)
ON DELETE CASCADE
```

Now deleting a user automatically deletes their orders.

---

# 9. Unique Constraint

A unique constraint ensures that values in a column are unique.

## Example

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email TEXT UNIQUE
);
```

Allowed:

```text
alice@email.com
bob@email.com
```

Not allowed:

```text
alice@email.com
alice@email.com
```

## Multiple Column Unique

Example:

```sql
CREATE TABLE students (
    first_name TEXT,
    last_name TEXT,
    UNIQUE(first_name, last_name)
);
```

Now duplicate name combinations are prevented.

---

# 10. Check Constraint

A check constraint enforces custom conditions.

## Example

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    salary NUMERIC CHECK (salary > 0)
);
```

Allowed:

```text
salary = 5000
```

Not allowed:

```text
salary = -100
```

## Complex Check Constraint

Example:

```sql
CHECK (age >= 18 AND age <= 60)
```

## Adding Check Constraint Later

```sql
ALTER TABLE employees
ADD CONSTRAINT salary_check
CHECK (salary > 0);
```

---

# 11. Not Null Constraint

The `NOT NULL` constraint ensures that a column cannot contain `NULL` values.

## Example

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);
```

Allowed:

```text
name = Alice
```

Not allowed:

```text
name = NULL
```

Error:

```text
ERROR: null value violates not-null constraint
```

---

# 12. Default Values

Default values automatically populate columns when no value is provided.

## Example

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    status TEXT DEFAULT 'pending'
);
```

Insert:

```sql
INSERT INTO orders DEFAULT VALUES;
```

Result:

```text
status = pending
```

## Default with Functions

Example:

```sql
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
```

Now each row records creation time.

---

# 13. Practical Example — Complete Table Design

Example table:

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE,
    age INT CHECK (age >= 18),
    salary NUMERIC CHECK (salary > 0),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Insert Example

```sql
INSERT INTO employees (name,email,age,salary)
VALUES ('Alice','alice@email.com',30,50000);
```

## Invalid Insert

```sql
INSERT INTO employees (name,age,salary)
VALUES ('Bob',15,20000);
```

Error:

```text
CHECK constraint failed
```

---

# 14. Constraint Architecture

Internally PostgreSQL stores constraints in system catalogs.

Important tables:

* `pg_constraint`
* `pg_class`
* `pg_attribute`

Example query:

```sql
SELECT conname
FROM pg_constraint;
```

---

# 15. Best Practices for Columns and Constraints

Professional database systems follow strict design rules.

## Always Use Primary Keys

Example:

```sql
id SERIAL PRIMARY KEY
```

## Use NOT NULL When Required

Example:

```sql
name TEXT NOT NULL
```

## Enforce Data Rules with CHECK

Example:

```sql
CHECK (price > 0)
```

## Use Foreign Keys for Relationships

Example:

```text
orders.user_id → users.id
```

## Use Default Values for Automatic Data

Example:

```sql
created_at DEFAULT CURRENT_TIMESTAMP
```

---

# PostgreSQL Column and Constraint Architecture

```text
Table
  │
  ▼
Columns
  │
  ├── Data Types
  ├── Default Values
  ├── Constraints
  │
  ▼
Constraints
  │
  ├── Primary Key
  ├── Foreign Key
  ├── Unique
  ├── Check
  └── Not Null
```

Constraints guarantee data integrity.

---

# Conclusion

Columns and constraints form the structural and logical foundation of PostgreSQL tables.

Columns define how data is stored, while constraints define rules that ensure data validity.

Through operations such as:

* adding columns
* modifying columns
* dropping columns

and constraints such as:

* primary key
* foreign key
* unique
* check
* not null
* default values

PostgreSQL ensures that databases remain structured, consistent, and reliable.

These mechanisms are critical for building robust applications and scalable data systems.

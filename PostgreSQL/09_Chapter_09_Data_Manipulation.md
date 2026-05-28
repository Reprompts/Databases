# PostgreSQL Hands-On Tutorial
## Chapter 9: Data Manipulation

In PostgreSQL, **Data Manipulation Language (DML)** refers to SQL commands used to insert, modify, and delete data stored in tables.

While DDL (Data Definition Language) creates database structures (tables, schemas, etc.), DML works with the actual data inside those structures.

The main DML operations are:

* `INSERT` — add new rows
* `UPDATE` — modify existing rows
* `DELETE` — remove rows
* `UPSERT` — insert or update depending on existence

These operations are used constantly in real-world applications, such as:

* registering users
* updating product prices
* recording transactions
* deleting expired data

This article explains DML in extreme practical detail, including syntax, behavior, best practices, and real examples.

---

# 1. INSERT — Adding Data to Tables

The `INSERT` command is used to add new rows into a table.

## Basic Syntax

```sql
INSERT INTO table_name (column1, column2, column3)
VALUES (value1, value2, value3);
```

## Example Table

First create a sample table:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT,
    email TEXT,
    age INT
);
```

## Basic Insert

```sql
INSERT INTO users (name, email, age)
VALUES ('Alice', 'alice@email.com', 25);
```

After insertion:

```text
id | name  | email            | age
---------------------------------------
1  | Alice | alice@email.com  | 25
```

Notice:

* `id` is auto-generated because of `SERIAL`.

---

# 2. Insert Without Specifying Columns

You can insert values in the same order as table columns.

```sql
INSERT INTO users
VALUES (DEFAULT, 'Bob', 'bob@email.com', 30);
```

But this is not recommended, because:

* column order may change
* code becomes harder to maintain

## Best Practice

Always specify column names.

---

# 3. Insert with Default Values

If columns have default values, they can be omitted.

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
id | status
-----------
1  | pending
```

---

# 4. Insert Multiple Rows

PostgreSQL supports inserting multiple rows in a single query.

## Syntax

```sql
INSERT INTO table_name (column1, column2)
VALUES
(value1, value2),
(value3, value4),
(value5, value6);
```

## Example

```sql
INSERT INTO users (name, email, age)
VALUES
('Charlie', 'charlie@email.com', 28),
('David', 'david@email.com', 35),
('Emma', 'emma@email.com', 22);
```

Now table:

```text
id | name    | age
---------------------
1  | Alice   | 25
2  | Bob     | 30
3  | Charlie | 28
4  | David   | 35
5  | Emma    | 22
```

## Advantages

* faster than individual inserts
* reduces network overhead

---

# 5. Insert from Another Table

You can insert query results into a table.

## Example Tables

* `customers`
* `vip_customers`

## Query

```sql
INSERT INTO vip_customers (name, email)
SELECT name, email
FROM customers
WHERE total_orders > 100;
```

This copies rows from one table to another.

---

# 6. Returning Inserted Data

PostgreSQL supports the `RETURNING` clause.

This returns inserted rows immediately.

## Example

```sql
INSERT INTO users (name, email, age)
VALUES ('Frank','frank@email.com',40)
RETURNING *;
```

Output:

```text
id | name  | email            | age
---------------------------------------
6  | Frank | frank@email.com  | 40
```

Very useful in applications.

---

# 7. UPDATE — Modifying Existing Data

The `UPDATE` command modifies existing rows.

## Basic Syntax

```sql
UPDATE table_name
SET column1 = value1,
    column2 = value2
WHERE condition;
```

## Example Table

```text
users
--------------------------------
id | name | email | age
--------------------------------
1 | Alice | alice@email | 25
2 | Bob   | bob@email   | 30
```

## Updating a Row

```sql
UPDATE users
SET age = 26
WHERE name = 'Alice';
```

Result:

* Alice age updated to 26

## Updating Multiple Columns

```sql
UPDATE users
SET name = 'Robert',
    age = 31
WHERE id = 2;
```

## Updating Multiple Rows

```sql
UPDATE users
SET age = age + 1;
```

This increases age for all users.

> ⚠ Warning:
> Without `WHERE`, all rows are updated.

## UPDATE with RETURNING

```sql
UPDATE users
SET age = 27
WHERE name = 'Alice'
RETURNING *;
```

Returns updated row.

---

# 8. DELETE — Removing Data

The `DELETE` command removes rows from a table.

## Basic Syntax

```sql
DELETE FROM table_name
WHERE condition;
```

## Example

Delete one user:

```sql
DELETE FROM users
WHERE name = 'Bob';
```

Result:

* Bob is removed.

## Delete Multiple Rows

```sql
DELETE FROM users
WHERE age < 25;
```

Removes all users younger than 25.

> ⚠ Important Warning:
>
> ```sql
> DELETE FROM users;
> ```
>
> This deletes all rows.
>
> But the table structure remains.

## DELETE with RETURNING

```sql
DELETE FROM users
WHERE age < 20
RETURNING *;
```

This shows which rows were deleted.

---

# 9. UPSERT — Insert or Update

UPSERT means:

* Insert if row does not exist
* Update if row already exists

PostgreSQL implements this using:

```sql
INSERT ... ON CONFLICT
```

## Why UPSERT is Important

Consider a table:

* `users`
* `email UNIQUE`

If you insert a duplicate email, an error occurs.

UPSERT solves this problem.

## Example Table

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email TEXT UNIQUE,
    name TEXT
);
```

## Insert with Conflict Handling

```sql
INSERT INTO users (email, name)
VALUES ('alice@email.com','Alice')
ON CONFLICT (email)
DO NOTHING;
```

If email already exists → query ignored.

## UPSERT with Update

```sql
INSERT INTO users (email, name)
VALUES ('alice@email.com','Alice Updated')
ON CONFLICT (email)
DO UPDATE SET name = EXCLUDED.name;
```

## Explanation

`EXCLUDED` = attempted inserted row

Now:

* existing row gets updated

## Example Workflow

Initial table:

```text
email            | name
-----------------------
alice@email.com  | Alice
```

Insert:

```text
Alice Updated
```

Result:

```text
email            | name
-----------------------
alice@email.com  | Alice Updated
```

---

# 10. Practical Real-World Example

Consider an e-commerce system.

## Products Table

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    price NUMERIC(10,2),
    stock INT
);
```

## Insert Product

```sql
INSERT INTO products (name, price, stock)
VALUES ('Laptop', 1000.00, 10);
```

## Update Product Stock

```sql
UPDATE products
SET stock = stock - 1
WHERE name = 'Laptop';
```

## Delete Discontinued Product

```sql
DELETE FROM products
WHERE name = 'Old Laptop';
```

## Upsert Product

```sql
INSERT INTO products (id,name,price,stock)
VALUES (1,'Laptop',950,20)
ON CONFLICT (id)
DO UPDATE
SET price = EXCLUDED.price,
    stock = EXCLUDED.stock;
```

---

# 11. DML and Transactions

DML operations are typically executed inside transactions.

## Example

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

COMMIT;
```

If something fails:

```sql
ROLLBACK;
```

This ensures data consistency.

---

# 12. Performance Considerations

Large DML operations may affect performance.

## Example Problems

* mass updates
* bulk deletes
* heavy insert loads

## Solutions

* batch inserts
* indexes
* vacuum maintenance

---

# 13. Best Practices

## Use WHERE Clauses Carefully

Example:

```sql
UPDATE users SET age = 30;
```

Bad practice.

## Use Transactions

```sql
BEGIN;
COMMIT;
ROLLBACK;
```

## Use Bulk Inserts for Performance

```sql
INSERT INTO table VALUES (...), (...), (...);
```

## Use UPSERT for Idempotent Operations

Important in APIs.

---

# 14. Internal PostgreSQL Behavior

When DML operations occur:

* PostgreSQL does not overwrite rows
* It creates new row versions

This is due to:

## MVCC — Multi Version Concurrency Control

Old rows remain until `VACUUM` cleans them.

This allows:

* concurrent reads
* consistent transactions

---

# PostgreSQL DML Architecture

```text
Application
     │
     ▼
SQL Query
     │
     ▼
Query Parser
     │
     ▼
Planner / Optimizer
     │
     ▼
Executor
     │
     ▼
Table Storage
```

DML commands modify rows stored in heap tables.

---

# Conclusion

Data Manipulation Language (DML) is the core mechanism used to work with actual data inside PostgreSQL tables.

The most important operations are:

```text
+-----------+----------------------+
| Operation | Purpose              |
+-----------+----------------------+
| INSERT    | Add new rows         |
| UPDATE    | Modify existing rows |
| DELETE    | Remove rows          |
| UPSERT    | Insert or update     |
+-----------+----------------------+
```

These commands power nearly every database-driven application, from websites and APIs to enterprise systems.

Mastering PostgreSQL DML allows developers to efficiently create, update, and maintain data in relational databases.

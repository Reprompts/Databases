# PostgreSQL Hands-On Tutorial
## Chapter 10: Querying Data

The `SELECT` statement is the most frequently used SQL command in PostgreSQL. It is used to retrieve data from database tables.

In real-world systems, almost every application feature relies on `SELECT` queries:

* displaying users in an admin dashboard
* searching products in an e-commerce store
* showing posts in a social network
* generating reports and analytics

The `SELECT` command allows developers to:

* retrieve all or specific columns
* filter rows
* sort results
* remove duplicates
* limit the number of results

This guide explains the `SELECT` query system in PostgreSQL in extreme practical detail, including:

* basic `SELECT` queries
* selecting specific columns
* filtering with `WHERE`
* sorting with `ORDER BY`
* limiting results with `LIMIT`
* offsetting results with `OFFSET`
* using `DISTINCT`

---

# 1. Basic SELECT Queries

The `SELECT` statement retrieves data from tables.

## Basic Syntax

```sql
SELECT column1, column2
FROM table_name;
```

---

## Example Table

Create a table for examples:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT,
    email TEXT,
    age INT,
    city TEXT
);
```

Insert sample data:

```sql
INSERT INTO users (name,email,age,city) VALUES
('Alice','alice@email.com',25,'Mumbai'),
('Bob','bob@email.com',30,'Delhi'),
('Charlie','charlie@email.com',28,'Mumbai'),
('David','david@email.com',35,'Pune'),
('Emma','emma@email.com',22,'Delhi');
```

---

## Retrieve All Data

```sql
SELECT * FROM users;
```

Result:

```text
id | name    | email             | age | city
------------------------------------------------
1  | Alice   | alice@email.com   | 25  | Mumbai
2  | Bob     | bob@email.com     | 30  | Delhi
3  | Charlie | charlie@email.com | 28  | Mumbai
4  | David   | david@email.com   | 35  | Pune
5  | Emma    | emma@email.com    | 22  | Delhi
```

`*` means:

* select all columns

> ⚠ Best Practice
>
> Avoid `SELECT *` in production systems because:
>
> * unnecessary data transfer
> * slower queries
> * schema changes can break code
>
> Instead select only required columns.

---

# 2. Selecting Specific Columns

You can retrieve only specific columns.

## Syntax

```sql
SELECT column1, column2
FROM table_name;
```

---

## Example

```sql
SELECT name, age
FROM users;
```

Result:

```text
name    | age
---------------
Alice   | 25
Bob     | 30
Charlie | 28
David   | 35
Emma    | 22
```

## Advantages

* faster queries
* reduced memory usage
* better performance

---

## Column Aliases

Aliases rename columns in results.

## Syntax

```sql
SELECT column AS alias
FROM table;
```

Example:

```sql
SELECT name AS username,
       age AS user_age
FROM users;
```

Result:

```text
username | user_age
-------------------
Alice    | 25
Bob      | 30
```

Aliases improve readability and reporting.

---

# 3. Filtering Data with WHERE

The `WHERE` clause filters rows based on conditions.

## Syntax

```sql
SELECT columns
FROM table
WHERE condition;
```

---

## Example

```sql
SELECT *
FROM users
WHERE city = 'Mumbai';
```

Result:

```text
Alice
Charlie
```

---

## Comparison Operators

`WHERE` supports comparison operators.

```text
+-----------+--------------------+
| Operator  | Meaning            |
+-----------+--------------------+
| =         | Equal              |
| !=        | Not equal          |
| >         | Greater than       |
| <         | Less than          |
| >=        | Greater or equal   |
| <=        | Less or equal      |
+-----------+--------------------+
```

Example:

```sql
SELECT name, age
FROM users
WHERE age > 30;
```

Result:

```text
David
```

---

## Multiple Conditions

Conditions can be combined.

### AND Operator

```sql
SELECT *
FROM users
WHERE city = 'Delhi' AND age > 25;
```

Result:

```text
Bob
```

---

### OR Operator

```sql
SELECT *
FROM users
WHERE city = 'Delhi' OR city = 'Mumbai';
```

Result:

```text
Alice
Bob
Charlie
Emma
```

---

### NOT Operator

```sql
SELECT *
FROM users
WHERE NOT city = 'Mumbai';
```

---

## IN Operator

Checks multiple values.

```sql
SELECT *
FROM users
WHERE city IN ('Delhi','Pune');
```

---

## BETWEEN Operator

Example:

```sql
SELECT *
FROM users
WHERE age BETWEEN 25 AND 30;
```

Result:

```text
Alice
Bob
Charlie
```

---

## LIKE Operator

Used for pattern matching.

Example:

```sql
SELECT *
FROM users
WHERE name LIKE 'A%';
```

Result:

```text
Alice
```

### Pattern Symbols

```text
+--------+--------------------+
| Symbol | Meaning            |
+--------+--------------------+
| %      | any characters     |
| _      | single character   |
+--------+--------------------+
```

Examples:

```text
'A%'  → names starting with A
'%a'  → names ending with a
```

---

# 4. Sorting Results with ORDER BY

`ORDER BY` sorts query results.

## Syntax

```sql
SELECT columns
FROM table
ORDER BY column;
```

---

## Example

```sql
SELECT name, age
FROM users
ORDER BY age;
```

Result:

```text
Emma     22
Alice    25
Charlie  28
Bob      30
David    35
```

Default sorting:

* ascending

---

## DESCENDING ORDER

```sql
SELECT name, age
FROM users
ORDER BY age DESC;
```

Result:

```text
David
Bob
Charlie
Alice
Emma
```

---

## Sorting by Multiple Columns

Example:

```sql
SELECT name, city, age
FROM users
ORDER BY city, age;
```

Sorting order:

1. city
2. age

Example output:

```text
Delhi   Emma     22
Delhi   Bob      30
Mumbai  Alice    25
Mumbai  Charlie  28
Pune    David    35
```

---

# 5. Limiting Results with LIMIT

`LIMIT` restricts the number of rows returned.

## Syntax

```sql
SELECT columns
FROM table
LIMIT number;
```

---

## Example

```sql
SELECT *
FROM users
LIMIT 3;
```

Result:

```text
first 3 rows only
```

Useful for:

* pagination
* preview results
* dashboards

---

## LIMIT with ORDER BY

Example:

Top 2 oldest users:

```sql
SELECT name, age
FROM users
ORDER BY age DESC
LIMIT 2;
```

Result:

```text
David
Bob
```

---

# 6. Offset Results

`OFFSET` skips rows before returning results.

## Syntax

```sql
SELECT columns
FROM table
LIMIT n
OFFSET m;
```

Meaning:

* skip `m` rows
* return next `n` rows

---

## Example

```sql
SELECT *
FROM users
LIMIT 2 OFFSET 2;
```

Explanation:

* skip first 2 rows
* return next 2 rows

Result:

```text
Charlie
David
```

---

## Pagination Example

Page size:

```text
10 records
```

Page 1:

```sql
LIMIT 10 OFFSET 0
```

Page 2:

```sql
LIMIT 10 OFFSET 10
```

Page 3:

```sql
LIMIT 10 OFFSET 20
```

This is how most websites implement pagination.

---

# 7. DISTINCT — Removing Duplicate Results

`DISTINCT` removes duplicate rows.

## Syntax

```sql
SELECT DISTINCT column
FROM table;
```

---

## Example

```sql
SELECT DISTINCT city
FROM users;
```

Result:

```text
Mumbai
Delhi
Pune
```

Duplicates removed.

---

## DISTINCT on Multiple Columns

Example:

```sql
SELECT DISTINCT city, age
FROM users;
```

Removes duplicate combinations.

---

## DISTINCT ON (PostgreSQL Feature)

PostgreSQL supports `DISTINCT ON`.

Example:

```sql
SELECT DISTINCT ON (city) *
FROM users
ORDER BY city, age DESC;
```

Meaning:

* one row per city
* choose highest age

Result:

```text
Delhi   Bob
Mumbai  Charlie
Pune    David
```

Very powerful PostgreSQL feature.

---

# 8. SELECT Query Execution Order

Although written differently, PostgreSQL executes queries in this logical order:

1. `FROM`
2. `WHERE`
3. `SELECT`
4. `DISTINCT`
5. `ORDER BY`
6. `LIMIT`
7. `OFFSET`

---

## Example Query

```sql
SELECT name
FROM users
WHERE age > 25
ORDER BY age
LIMIT 3;
```

Execution:

1. scan table
2. filter rows
3. select columns
4. sort results
5. limit output

---

# 9. Practical Real-World Example

## Example Table

```sql
CREATE TABLE products (
    id SERIAL,
    name TEXT,
    price NUMERIC,
    category TEXT
);
```

Insert data:

```sql
INSERT INTO products (name,price,category) VALUES
('Laptop',1000,'Electronics'),
('Phone',700,'Electronics'),
('Shirt',50,'Clothing'),
('Shoes',120,'Clothing'),
('Tablet',500,'Electronics');
```

---

## Query Electronics Products

```sql
SELECT name, price
FROM products
WHERE category = 'Electronics';
```

---

## Top 2 Most Expensive Products

```sql
SELECT name, price
FROM products
ORDER BY price DESC
LIMIT 2;
```

---

## Unique Categories

```sql
SELECT DISTINCT category
FROM products;
```

---

# 10. Performance Considerations

Large datasets require optimization.

## Problems Include

* full table scans
* slow filtering
* expensive sorting

---

## Solutions

* indexes
* proper `WHERE` conditions
* limiting result size

Example index:

```sql
CREATE INDEX idx_users_city
ON users(city);
```

Now queries filtering by city become faster.

---

# SELECT Query Architecture in PostgreSQL

Internally the query system works like this:

```text
Client Application
       │
       ▼
SQL Query
       │
       ▼
Parser
       │
       ▼
Query Planner
       │
       ▼
Query Optimizer
       │
       ▼
Executor
       │
       ▼
Table Storage
```

The planner decides the fastest way to execute the query.

---

# Best Practices

## Always Filter Data

Use:

```sql
WHERE conditions
```

---

## Avoid

```sql
SELECT *
```

---

## Use Indexes for Filtering Columns

Example:

```sql
CREATE INDEX idx_users_city
ON users(city);
```

---

## Use LIMIT for Large Queries

---

## Use DISTINCT Only When Necessary

`DISTINCT` can be expensive on large datasets.

---

# Conclusion

The `SELECT` statement is the foundation of data retrieval in PostgreSQL.

It allows developers to:

* retrieve specific data
* filter results
* sort output
* remove duplicates
* limit query results

Key operations include:

```text
+-----------+----------------------------+
| Operation | Purpose                    |
+-----------+----------------------------+
| SELECT    | Retrieve columns           |
| WHERE     | Filter rows                |
| ORDER BY  | Sort data                  |
| LIMIT     | Restrict number of rows    |
| OFFSET    | Skip rows                  |
| DISTINCT  | Remove duplicates          |
+-----------+----------------------------+
```

Understanding `SELECT` queries deeply is essential for building efficient and scalable database applications.

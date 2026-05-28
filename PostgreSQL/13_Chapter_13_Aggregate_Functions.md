# PostgreSQL Hands-On Tutorial
## Chapter 13: Aggregate Functions

Aggregate functions are used in PostgreSQL to perform calculations on multiple rows and return a single summarized value.

Instead of retrieving individual rows, aggregate functions help you analyze and summarize data.

They are essential for:

* analytics and reporting
* statistics
* dashboards
* business intelligence queries
* financial calculations
* data summarization

For example:

* How many users registered today?
* What is the total revenue?
* What is the average product price?
* Which order has the highest value?

All of these questions are answered using aggregate functions.

PostgreSQL provides several built-in aggregate functions, including:

* `COUNT`
* `SUM`
* `AVG`
* `MIN`
* `MAX`

These functions are commonly used with:

* `GROUP BY`
* `HAVING`

to analyze grouped data.

---

# 1. What Are Aggregate Functions?

An aggregate function processes a set of rows and produces one result value.

Example:

Table: `orders`

```sql
id | price
-----------
1  | 100
2  | 200
3  | 300
```

Query:

```sql
SELECT SUM(price) FROM orders;
```

Result:

```text
600
```

The function aggregates multiple rows into a single value.

---

# 2. COUNT Function

The `COUNT()` function returns the number of rows.

## Basic COUNT

```sql
SELECT COUNT(*) FROM customers;
```

Example result:

```text
120
```

Meaning:

* There are 120 rows in the table.
* `COUNT(*)` counts all rows.

## COUNT Specific Column

```sql
SELECT COUNT(email) FROM users;
```

Important behavior:

`COUNT(column)` ignores `NULL` values.

Example:

```text
+----+------------+
| id | email      |
+----+------------+
| 1  | a@mail.com |
| 2  | NULL       |
| 3  | b@mail.com |
+----+------------+
```

Query:

```sql
SELECT COUNT(email) FROM users;
```

Result:

```text
2
```

## COUNT DISTINCT

Used to count unique values.

Example table:

```text
+---------+
| product |
+---------+
| Phone   |
| Laptop  |
| Phone   |
+---------+
```

Query:

```sql
SELECT COUNT(DISTINCT product)
FROM products;
```

Result:

```text
2
```

---

# 3. SUM Function

The `SUM()` function calculates the total of numeric values.

Example table: `orders`

```text
+----+--------+
| id | amount |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
```

Query:

```sql
SELECT SUM(amount) FROM orders;
```

Result:

```text
600
```

## SUM With Condition

```sql
SELECT SUM(amount)
FROM orders
WHERE amount > 100;
```

Result:

```text
500
```

## SUM With Expressions

```sql
SELECT SUM(price * quantity)
FROM order_items;
```

This calculates total revenue.

---

# 4. AVG Function

The `AVG()` function calculates the average value.

Example:

```text
+-------+
| price |
+-------+
| 100   |
| 200   |
| 300   |
+-------+
```

Query:

```sql
SELECT AVG(price)
FROM products;
```

Result:

```text
200
```

Formula internally:

```text
SUM(values) / COUNT(values)
```

## AVG Ignoring NULL

`AVG` also ignores `NULL` values.

Example:

```text
price
-----
100
NULL
300
```

Average:

```text
(100 + 300) / 2 = 200
```

---

# 5. MIN Function

The `MIN()` function returns the smallest value.

Example table:

```text
price
-----
100
200
50
```

Query:

```sql
SELECT MIN(price)
FROM products;
```

Result:

```text
50
```

`MIN` works with:

* numbers
* dates
* text (alphabetical)

Example:

```sql
SELECT MIN(created_at)
FROM users;
```

Returns the earliest date.

---

# 6. MAX Function

The `MAX()` function returns the largest value.

Example:

```sql
SELECT MAX(price)
FROM products;
```

Result:

```text
200
```

Example with timestamps:

```sql
SELECT MAX(created_at)
FROM users;
```

Returns latest date.

---

# 7. Using Multiple Aggregate Functions

You can use many aggregates in one query.

Example:

```sql
SELECT
COUNT(*) AS total_orders,
SUM(amount) AS revenue,
AVG(amount) AS avg_order,
MIN(amount) AS smallest_order,
MAX(amount) AS largest_order
FROM orders;
```

Example output:

```text
+--------------+---------+-----------+-----+------+
| total_orders | revenue | avg_order | min | max  |
+--------------+---------+-----------+-----+------+
| 100          | 50000   | 500       | 50  | 2000 |
+--------------+---------+-----------+-----+------+
```

---

# 8. Grouping Results

Often we want aggregates per category.

Example question:

```text
How many orders each customer made?
```

For this we use `GROUP BY`.

---

# 9. GROUP BY Clause

The `GROUP BY` clause divides rows into groups.

Aggregate functions are then applied to each group separately.

Example table:

```text
+----------+--------+
| customer | amount |
+----------+--------+
| Alice    | 100    |
| Alice    | 200    |
| Bob      | 300    |
+----------+--------+
```

Query:

```sql
SELECT customer, SUM(amount)
FROM orders
GROUP BY customer;
```

Result:

```text
+----------+-----+
| customer | sum |
+----------+-----+
| Alice    | 300 |
| Bob      | 300 |
+----------+-----+
```

Each customer becomes a group.

---

# 10. GROUP BY With Multiple Columns

You can group by multiple columns.

Example table:

```text
+----------+---------+--------+
| customer | product | amount |
+----------+---------+--------+
| Alice    | Phone   | 100    |
| Alice    | Laptop  | 500    |
| Bob      | Phone   | 200    |
+----------+---------+--------+
```

Query:

```sql
SELECT customer, product, SUM(amount)
FROM sales
GROUP BY customer, product;
```

Result:

```text
+----------+---------+-----+
| customer | product | sum |
+----------+---------+-----+
| Alice    | Phone   | 100 |
| Alice    | Laptop  | 500 |
| Bob      | Phone   | 200 |
+----------+---------+-----+
```

---

# 11. GROUP BY Rules

Important rule:

Every column in `SELECT` must either be:

* inside an aggregate function

or

* listed in `GROUP BY`

Example invalid query:

```sql
SELECT customer, amount
FROM orders
GROUP BY customer;
```

This fails because `amount` is neither grouped nor aggregated.

Correct version:

```sql
SELECT customer, SUM(amount)
FROM orders
GROUP BY customer;
```

---

# 12. Filtering Groups with HAVING

The `HAVING` clause filters groups, not rows.

Difference:

* `WHERE` → filters rows
* `HAVING` → filters groups

Example:

Find customers whose total orders exceed `500`.

```sql
SELECT customer, SUM(amount)
FROM orders
GROUP BY customer
HAVING SUM(amount) > 500;
```

Result:

```text
+----------+-----+
| customer | sum |
+----------+-----+
| Alice    | 700 |
+----------+-----+
```

---

# 13. WHERE vs HAVING

Example query:

```sql
SELECT customer, SUM(amount)
FROM orders
WHERE amount > 100
GROUP BY customer
HAVING SUM(amount) > 500;
```

Execution steps:

1. `WHERE` filters rows
2. `GROUP BY` creates groups
3. aggregates are calculated
4. `HAVING` filters groups

---

# 14. Practical Example — Sales Analysis

Table: `sales`

```text
+----+---------+----------+-------+
| id | product | quantity | price |
+----+---------+----------+-------+
| 1  | Laptop  | 2        | 1000  |
| 2  | Phone   | 5        | 500   |
| 3  | Laptop  | 1        | 1000  |
+----+---------+----------+-------+
```

## Total Revenue

```sql
SELECT SUM(quantity * price)
FROM sales;
```

## Revenue Per Product

```sql
SELECT
product,
SUM(quantity * price) AS revenue
FROM sales
GROUP BY product;
```

Result:

```text
+---------+---------+
| product | revenue |
+---------+---------+
| Laptop  | 3000    |
| Phone   | 2500    |
+---------+---------+
```

## Products With Revenue > 2000

```sql
SELECT
product,
SUM(quantity * price) AS revenue
FROM sales
GROUP BY product
HAVING SUM(quantity * price) > 2000;
```

---

# 15. Combining Joins With Aggregates

Aggregates often combine with joins.

Example:

Tables:

* `customers`
* `orders`

Query:

```sql
SELECT
c.name,
COUNT(o.id) AS total_orders
FROM customers c
LEFT JOIN orders o
ON c.id = o.customer_id
GROUP BY c.name;
```

Result:

```text
+----------+--------+
| customer | orders |
+----------+--------+
| Alice    | 5      |
| Bob      | 2      |
+----------+--------+
```

---

# 16. Aggregate Performance Tips

## Use indexes on grouping columns

Example:

```sql
CREATE INDEX idx_orders_customer
ON orders(customer_id);
```

This improves grouping performance.

## Avoid grouping huge datasets without filtering

Use `WHERE` first.

Example:

```sql
WHERE created_at > '2024-01-01'
```

---

# 17. Real-World Use Cases

Aggregate functions are used for:

## Business Analytics

* daily revenue
* monthly sales

## Monitoring Systems

* avg response time
* max latency

## Finance

* total income
* avg transaction

## Social Networks

* number of likes
* avg engagement

---

# 18. Summary

Aggregate functions summarize data across rows.

## Core PostgreSQL Aggregates

```text
+----------+------------------+
| Function | Purpose          |
+----------+------------------+
| COUNT    | Number of rows   |
| SUM      | Total value      |
| AVG      | Average value    |
| MIN      | Smallest value   |
| MAX      | Largest value    |
+----------+------------------+
```

## Key Concepts

```text
+----------+----------------------------------+
| Concept  | Description                      |
+----------+----------------------------------+
| GROUP BY | Groups rows                      |
| HAVING   | Filters groups                   |
| WHERE    | Filters rows before grouping     |
+----------+----------------------------------+
```

Aggregate functions form the foundation of SQL analytics and reporting.

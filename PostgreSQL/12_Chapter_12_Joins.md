
# PostgreSQL Hands-On Tutorial
## Chapter 12: Joins

In relational databases like PostgreSQL, data is usually distributed across multiple tables. Joins allow us to combine data from these tables into a single result set based on relationships between them.

Joins are one of the most important and powerful features in SQL, enabling complex data retrieval from normalized database structures.

In real-world applications, joins are used constantly:

- displaying orders with customer details
- showing employees with department names
- retrieving products with category information
- generating reports from multiple tables

Without joins, relational databases would not be able to connect related pieces of data stored in different tables.

This article explains PostgreSQL joins in extreme detail, including:

- INNER JOIN
- LEFT JOIN
- RIGHT JOIN
- FULL JOIN
- CROSS JOIN
- joining multiple tables
- join conditions

---

# 1. Why Joins Are Needed

Relational databases follow normalization principles, meaning data is split into separate tables to avoid redundancy.

## Example Schema

- customers
- orders
- products
- order_items

Instead of storing all data in one large table, information is separated.

## Example Tables

### customers

| id | name |
|----|-------|
| 1  | Alice |
| 2  | Bob   |

### orders

| id | customer_id | total |
|----|-------------|-------|
| 1  | 1           | 200   |
| 2  | 2           | 150   |

To retrieve orders with customer names, we must join tables.

## Example Query

```sql
SELECT customers.name, orders.total
FROM customers
JOIN orders
ON customers.id = orders.customer_id;
````

This combines data from two tables.

---

# 2. Join Syntax

Basic join syntax:

```sql
SELECT columns
FROM table1
JOIN table2
ON table1.column = table2.column;
```

## Components

| Part   | Description    |
| ------ | -------------- |
| table1 | First table    |
| table2 | Second table   |
| JOIN   | Join operation |
| ON     | Join condition |

---

# 3. INNER JOIN

The INNER JOIN returns rows that have matching values in both tables.

Rows without matches are excluded.

## Example Tables

### customers

| id | name    |
| -- | ------- |
| 1  | Alice   |
| 2  | Bob     |
| 3  | Charlie |

### orders

| id | customer_id | total |
| -- | ----------- | ----- |
| 1  | 1           | 200   |
| 2  | 2           | 150   |

> Note: Charlie has no orders.

## Query

```sql
SELECT customers.name, orders.total
FROM customers
INNER JOIN orders
ON customers.id = orders.customer_id;
```

## Result

| name  | total |
| ----- | ----- |
| Alice | 200   |
| Bob   | 150   |

Charlie is excluded because no matching order exists.

## INNER JOIN Characteristics

* returns only matching rows
* most commonly used join
* fastest in many cases

---

# 4. LEFT JOIN

The LEFT JOIN returns:

* all rows from left table
* matching rows from right table

If no match exists, NULL values are returned.

## Query

```sql
SELECT customers.name, orders.total
FROM customers
LEFT JOIN orders
ON customers.id = orders.customer_id;
```

## Result

| name    | total |
| ------- | ----- |
| Alice   | 200   |
| Bob     | 150   |
| Charlie | NULL  |

Charlie appears because LEFT JOIN keeps all rows from left table.

## LEFT JOIN Use Cases

LEFT JOIN is useful for:

* finding customers without orders
* identifying missing relationships
* optional relationships

## Example - Customers Without Orders

```sql
SELECT customers.name
FROM customers
LEFT JOIN orders
ON customers.id = orders.customer_id
WHERE orders.id IS NULL;
```

## Result

```text
Charlie
```

This finds customers with no orders.

---

# 5. RIGHT JOIN

The RIGHT JOIN is the opposite of LEFT JOIN.

It returns:

* all rows from right table
* matching rows from left table

## Query

```sql
SELECT customers.name, orders.total
FROM customers
RIGHT JOIN orders
ON customers.id = orders.customer_id;
```

## Result

| name  | total |
| ----- | ----- |
| Alice | 200   |
| Bob   | 150   |

If an order had no matching customer, the customer columns would be NULL.

## Important Note

RIGHT JOIN is rarely used, because the same result can usually be achieved using LEFT JOIN by swapping table positions.

---

# 6. FULL JOIN

The FULL JOIN (FULL OUTER JOIN) returns:

* all rows from both tables

Matched rows combine normally.

Unmatched rows show NULL values.

## Example Tables

### customers

| id | name    |
| -- | ------- |
| 1  | Alice   |
| 2  | Bob     |
| 3  | Charlie |

### orders

| id | customer_id | total |
| -- | ----------- | ----- |
| 1  | 1           | 200   |
| 2  | 2           | 150   |
| 3  | 4           | 300   |

> Note: Order 3 belongs to a non-existing customer.

## Query

```sql
SELECT customers.name, orders.total
FROM customers
FULL JOIN orders
ON customers.id = orders.customer_id;
```

## Result

| name    | total |
| ------- | ----- |
| Alice   | 200   |
| Bob     | 150   |
| Charlie | NULL  |
| NULL    | 300   |

## FULL JOIN Use Cases

* data reconciliation
* comparing two datasets
* identifying unmatched records

---

# 7. CROSS JOIN

A CROSS JOIN creates a Cartesian product.

This means every row from table1 is combined with every row from table2.

## Example Tables

### colors

| color |
| ----- |
| red   |
| blue  |

### sizes

| size  |
| ----- |
| small |
| large |

## Query

```sql
SELECT colors.color, sizes.size
FROM colors
CROSS JOIN sizes;
```

## Result

| color | size  |
| ----- | ----- |
| red   | small |
| red   | large |
| blue  | small |
| blue  | large |

## Result Size

If:

* table1 = 10 rows
* table2 = 5 rows

Result:

```text
10 × 5 = 50 rows
```

## CROSS JOIN Use Cases

* generating combinations
* testing datasets
* matrix-style outputs

---

# 8. Joining Multiple Tables

PostgreSQL allows joining multiple tables in one query.

## Example Schema

* customers
* orders
* products
* order_items

## Example Query

```sql
SELECT
customers.name,
products.name,
order_items.quantity
FROM customers
JOIN orders
ON customers.id = orders.customer_id
JOIN order_items
ON orders.id = order_items.order_id
JOIN products
ON order_items.product_id = products.id;
```

## Result

| customer | product | quantity |
| -------- | ------- | -------- |
| Alice    | Laptop  | 2        |
| Alice    | Mouse   | 1        |
| Bob      | Phone   | 1        |

This query joins four tables together.

---

# 9. Join Conditions

Join conditions define how tables relate to each other.

They are written using the ON clause.

## Example

```sql
ON customers.id = orders.customer_id
```

This means:

```text
match rows where IDs are equal
```

## Multiple Conditions

### Example

```sql
JOIN orders
ON customers.id = orders.customer_id
AND orders.total > 100;
```

---

## Using WHERE vs ON

### JOIN condition

Defines relationship between tables.

### WHERE clause

Filters rows after joining.

## Example

```sql
SELECT *
FROM customers
JOIN orders
ON customers.id = orders.customer_id
WHERE orders.total > 100;
```

---

# 10. Table Aliases

Aliases make join queries easier to read.

## Example

```sql
SELECT c.name, o.total
FROM customers c
JOIN orders o
ON c.id = o.customer_id;
```

Instead of writing:

```text
customers.name
orders.total
```

---

# 11. Join Execution Internals

PostgreSQL uses several algorithms to execute joins.

---

## Nested Loop Join

Best for small datasets.

### Logic

```text
for each row in table1
   search matching rows in table2
```

---

## Hash Join

Used when join columns are not indexed.

### Steps

* build hash table
* match rows using hash lookup

---

## Merge Join

Requires sorted datasets.

Used when indexes exist.

Very efficient for large joins.

---

## Query Planner Role

The PostgreSQL query planner automatically chooses the fastest join method.

---

# 12. Real-World Example - E-commerce System

## Tables

* customers
* orders
* products
* order_items

### customers

```text
id | name
```

### orders

```text
id | customer_id
```

### products

```text
id | name | price
```

### order_items

```text
order_id | product_id | quantity
```

---

## Retrieve Complete Order Information

```sql
SELECT
c.name AS customer,
p.name AS product,
p.price,
oi.quantity,
(p.price * oi.quantity) AS total
FROM customers c
JOIN orders o ON c.id = o.customer_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id;
```

This query produces a complete purchase report.

---

# 13. Join Best Practices

* Always define proper join conditions.
* Avoid joins without ON clause (except CROSS JOIN).
* Use indexes on join columns.

## Example

```sql
CREATE INDEX idx_orders_customer
ON orders(customer_id);
```

This greatly improves join performance.

* Use table aliases for readability.

### Example

```text
customers → c
orders → o
```

---

# 14. Visual Join Concept

Imagine two tables as circles.

## INNER JOIN

```text
only intersection
```

## LEFT JOIN

```text
left circle + intersection
```

## RIGHT JOIN

```text
right circle + intersection
```

## FULL JOIN

```text
both circles
```

## CROSS JOIN

```text
every combination
```

---

# 15. Summary

Joins allow PostgreSQL to combine related data stored across multiple tables.

## Join Types

| Join Type  | Description               |
| ---------- | ------------------------- |
| INNER JOIN | Only matching rows        |
| LEFT JOIN  | All rows from left table  |
| RIGHT JOIN | All rows from right table |
| FULL JOIN  | All rows from both tables |
| CROSS JOIN | Cartesian product         |

Joins are essential for:

* relational data modeling
* reporting queries
* analytics
* application backends

Understanding joins deeply is critical for building efficient SQL queries and scalable database systems.

```

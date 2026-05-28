# PostgreSQL Hands-On Tutorial
## Chapter 14: Subqueries

A subquery is a query written inside another SQL query.

It allows you to use the result of one query as input to another query. Subqueries make SQL very powerful because they allow step-by-step logical data retrieval inside a single statement.

In PostgreSQL, subqueries are used for:

* filtering rows based on results of another query
* calculating derived values
* building temporary result sets
* performing advanced analytics
* comparing values across tables

Subqueries are commonly used in:

* `SELECT`
* `WHERE`
* `FROM`
* `INSERT`
* `UPDATE`
* `DELETE`

But in this article we will focus on the most important use cases:

* Scalar subqueries
* Correlated subqueries
* Subqueries in `SELECT`
* Subqueries in `WHERE`
* Subqueries in `FROM`

---

# 1. What Is a Subquery?

A subquery is a query nested inside another SQL query.

General structure:

```sql
SELECT column
FROM table
WHERE column = (
    SELECT column
    FROM another_table
);
```

The inner query executes first, and its result is used by the outer query.

## Example Tables

### employees

```text
+----+---------+---------------+--------+
| id | name    | department_id | salary |
+----+---------+---------------+--------+
| 1  | Alice   | 1             | 5000   |
| 2  | Bob     | 2             | 6000   |
| 3  | Charlie | 1             | 4500   |
+----+---------+---------------+--------+
```

### departments

```text
+----+-------------+
| id | name        |
+----+-------------+
| 1  | Engineering |
| 2  | HR          |
+----+-------------+
```

## Example Query

```sql
SELECT name
FROM employees
WHERE department_id = (
    SELECT id
    FROM departments
    WHERE name = 'Engineering'
);
```

## Result

```text
Alice
Charlie
```

The inner query returns the department ID, and the outer query uses it to filter employees.

---

# 2. Types of Subqueries

Subqueries are categorized into different types depending on their behavior:

```text
+---------------------+----------------------------------+
| Type                | Description                      |
+---------------------+----------------------------------+
| Scalar Subquery     | Returns a single value           |
| Row Subquery        | Returns a single row             |
| Table Subquery      | Returns multiple rows            |
| Correlated Subquery | Depends on outer query           |
+---------------------+----------------------------------+
```

In PostgreSQL, scalar and correlated subqueries are especially common.

---

# 3. Scalar Subqueries

A scalar subquery returns exactly one value.

This value can be used anywhere a normal value is used.

## Example

```sql
SELECT
    name,
    salary,
    (SELECT AVG(salary) FROM employees) AS average_salary
FROM employees;
```

## Result

```text
+---------+--------+----------------+
| name    | salary | average_salary |
+---------+--------+----------------+
| Alice   | 5000   | 5166           |
| Bob     | 6000   | 5166           |
| Charlie | 4500   | 5166           |
+---------+--------+----------------+
```

The subquery calculates average salary once, and the value is shown for each row.

## Example — Compare Salary With Average

```sql
SELECT name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

## Result

```text
Bob
```

Bob earns more than the company average.

---

# 4. Subqueries in SELECT Clause

A subquery can appear in the `SELECT` list to compute additional values.

## Example

```sql
SELECT
    name,
    salary,
    (SELECT MAX(salary) FROM employees) AS highest_salary
FROM employees;
```

## Result

```text
+---------+--------+----------------+
| name    | salary | highest_salary |
+---------+--------+----------------+
| Alice   | 5000   | 6000           |
| Bob     | 6000   | 6000           |
| Charlie | 4500   | 6000           |
+---------+--------+----------------+
```

---

## Practical Example — Order Count Per Customer

### customers

```text
+----+-------+
| id | name  |
+----+-------+
| 1  | Alice |
| 2  | Bob   |
+----+-------+
```

### orders

```text
+----+-------------+
| id | customer_id |
+----+-------------+
| 1  | 1           |
| 2  | 1           |
| 3  | 2           |
+----+-------------+
```

## Query

```sql
SELECT
    name,
    (
        SELECT COUNT(*)
        FROM orders
        WHERE orders.customer_id = customers.id
    ) AS order_count
FROM customers;
```

## Result

```text
+-------+-------------+
| name  | order_count |
+-------+-------------+
| Alice | 2           |
| Bob   | 1           |
+-------+-------------+
```

This subquery calculates order count per customer.

---

# 5. Subqueries in WHERE Clause

Subqueries are most frequently used inside `WHERE` conditions.

They help filter rows based on results from another query.

---

## Using `=` With Subqueries

```sql
SELECT name
FROM employees
WHERE department_id = (
    SELECT id
    FROM departments
    WHERE name = 'HR'
);
```

---

## Using `IN`

When the subquery returns multiple values, use `IN`.

```sql
SELECT name
FROM employees
WHERE department_id IN (
    SELECT id
    FROM departments
);
```

---

## Using `NOT IN`

```sql
SELECT name
FROM employees
WHERE department_id NOT IN (
    SELECT id
    FROM departments
);
```

Finds employees belonging to unknown departments.

---

## Using `EXISTS`

`EXISTS` checks whether a subquery returns any rows.

```sql
SELECT name
FROM customers
WHERE EXISTS (
    SELECT 1
    FROM orders
    WHERE orders.customer_id = customers.id
);
```

This returns customers who have at least one order.

---

# 6. Correlated Subqueries

A correlated subquery depends on values from the outer query.

Unlike normal subqueries, it executes once for every row of the outer query.

## Structure

```sql
SELECT column
FROM table1
WHERE column = (
    SELECT column
    FROM table2
    WHERE table2.col = table1.col
);
```

---

## Example — Employees With Above Department Average Salary

```sql
SELECT name, salary
FROM employees e
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE department_id = e.department_id
);
```

## Explanation

For each employee:

* PostgreSQL calculates average salary in that employee's department
* Compares employee salary with department average

## Example Result

```text
Bob
```

---

## Correlated Subquery Execution Concept

For each row in outer query:

1. run inner query
2. compare result
3. return row if condition satisfied

Because of this repeated execution, correlated subqueries can sometimes be slower than joins.

---

# 7. Subqueries in FROM Clause

Subqueries can appear in the `FROM` clause as temporary tables.

These are called **derived tables**.

## Example

```sql
SELECT *
FROM (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
) AS dept_avg;
```

## Result

```text
+---------------+------------+
| department_id | avg_salary |
+---------------+------------+
| 1             | 4750       |
| 2             | 6000       |
+---------------+------------+
```

The subquery produces a temporary table, which the outer query uses.

---

## Practical Example — Employees Above Department Average

```sql
SELECT e.name, e.salary
FROM employees e
JOIN (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
) AS dept_avg
ON e.department_id = dept_avg.department_id
WHERE e.salary > dept_avg.avg_salary;
```

This approach is often more efficient than correlated subqueries.

---

# 8. Nested Subqueries

Subqueries can be nested multiple levels.

## Example

```sql
SELECT name
FROM employees
WHERE department_id = (
    SELECT id
    FROM departments
    WHERE id = (
        SELECT MIN(department_id)
        FROM employees
    )
);
```

Here there are three levels of queries.

---

# 9. Subqueries vs Joins

Many subqueries can also be written using joins.

## Example Using Subquery

```sql
SELECT name
FROM employees
WHERE department_id = (
    SELECT id
    FROM departments
    WHERE name = 'Engineering'
);
```

## Equivalent Join

```sql
SELECT e.name
FROM employees e
JOIN departments d
ON e.department_id = d.id
WHERE d.name = 'Engineering';
```

---

## Comparison

### Subqueries

Advantages:

* often more readable
* easier for nested logic
* useful for scalar calculations

Disadvantages:

* correlated subqueries may be slower
* sometimes harder to optimize

### Joins

Advantages:

* often faster for large datasets
* easier for optimizer
* better for relational retrieval

Disadvantages:

* can become complex with many tables

---

# 10. Performance Considerations

## Prefer JOIN Over Correlated Subqueries for Large Datasets

Example slow pattern:

```sql
SELECT (subquery executed per row)
```

Better approach:

```sql
JOIN aggregated data
```

---

## Use Indexes for Subquery Columns

Example:

```sql
CREATE INDEX idx_orders_customer
ON orders(customer_id);
```

---

## Avoid Returning Large Result Sets Unnecessarily

Bad:

```sql
SELECT *
FROM huge_table
```

inside subqueries when only one column is needed.

Better:

```sql
SELECT id
FROM huge_table
```

---

# 11. Real-World Examples

## Example — Highest Paid Employee

```sql
SELECT name
FROM employees
WHERE salary = (
    SELECT MAX(salary)
    FROM employees
);
```

---

## Example — Customers With No Orders

```sql
SELECT name
FROM customers
WHERE id NOT IN (
    SELECT customer_id
    FROM orders
);
```

---

## Example — Average Salary per Department

```sql
SELECT *
FROM (
    SELECT department_id, AVG(salary)
    FROM employees
    GROUP BY department_id
) AS dept_avg;
```

---

# 12. Internal Execution Behavior

When PostgreSQL executes a query with subqueries, the process typically looks like this:

```text
SQL Query
   │
   ▼
Parser
   │
   ▼
Planner / Optimizer
   │
   ▼
Subquery Execution
   │
   ▼
Outer Query Execution
   │
   ▼
Final Result Set
```

The PostgreSQL planner may sometimes optimize subqueries into joins internally.

---

# 13. Best Practices

## Use Scalar Subqueries for Single-Value Calculations

Example:

```sql
SELECT AVG(price)
```

---

## Use EXISTS for Existence Checks

Efficient for checking related rows.

```sql
WHERE EXISTS (...)
```

---

## Prefer JOINs for Large Relational Queries

Especially when dealing with many rows.

---

## Avoid Deeply Nested Queries When Simpler Alternatives Exist

Overly nested SQL becomes difficult to debug and maintain.

---

## Use Aliases for Readability

Example:

```sql
employees e
departments d
```

---

# 14. Summary

Subqueries allow queries to use results from other queries dynamically.

## Key Types

```text
+------------------------+----------------------------------------------+
| Subquery Type          | Description                                  |
+------------------------+----------------------------------------------+
| Scalar subquery        | Returns one value                            |
| Correlated subquery    | Depends on outer query                       |
| Subquery in SELECT     | Computes values                              |
| Subquery in WHERE      | Filters rows                                 |
| Subquery in FROM       | Creates temporary tables                     |
+------------------------+----------------------------------------------+
```

Subqueries are extremely useful for:

* data filtering
* dynamic comparisons
* nested analytics
* building complex queries

They are a core feature of SQL query design.

---

# Conclusion

Subqueries are one of PostgreSQL’s most powerful query-building features.

They allow developers to:

* break complex logic into smaller steps
* dynamically compare data
* create temporary datasets
* perform advanced filtering and analytics

Key concepts include:

* scalar subqueries
* correlated subqueries
* subqueries in `SELECT`
* subqueries in `WHERE`
* subqueries in `FROM`

Understanding subqueries deeply is essential for writing advanced SQL queries and building scalable PostgreSQL applications.

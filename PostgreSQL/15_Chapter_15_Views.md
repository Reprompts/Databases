# PostgreSQL Hands-On Tutorial
## Chapter 15: Views

In PostgreSQL, a view is a powerful database feature that allows you to store a SQL query as a virtual table.

Instead of repeatedly writing complex queries, you can save them as a view and use them like a normal table.

Views are widely used in real-world systems for:

* simplifying complex queries
* improving security (restricting data access)
* creating reusable query logic
* reporting and analytics
* building clean database APIs

PostgreSQL also supports **materialized views**, which store actual data for performance optimization.

This guide explains views in extreme practical detail, including:

* what views are
* advantages of views
* creating views
* using views
* updating views
* dropping views
* materialized views

---

# 1. What Is a View?

A view is a virtual table based on the result of a `SELECT` query.

It does **NOT** store data itself (in normal views). Instead, it stores the query definition.

## Basic Idea

Think of a view as:

```text id="m2a9cx"
Saved SELECT query that behaves like a table
```

---

## Example

Instead of writing:

```sql id="6ghxqp"
SELECT name, age
FROM users
WHERE age > 18;
```

Every time, we create a view:

```sql id="r1f8zb"
CREATE VIEW adult_users AS
SELECT name, age
FROM users
WHERE age > 18;
```

Now you can simply run:

```sql id="e0md5v"
SELECT * FROM adult_users;
```

---

# 2. Why Views Are Important

Views are extremely useful in real-world database systems.

---

## 1. Simplify Complex Queries

### Without View

```sql id="jq4snp"
SELECT c.name, SUM(o.amount)
FROM customers c
JOIN orders o ON c.id = o.customer_id
GROUP BY c.name;
```

### With View

```sql id="7av1gu"
SELECT * FROM customer_order_summary;
```

---

## 2. Improve Security

You can restrict sensitive columns.

Example:

```sql id="3rj7ko"
CREATE VIEW safe_users AS
SELECT id, name
FROM users;
```

No access to email/password.

---

## 3. Code Reusability

The same logic can be reused across multiple queries and applications.

---

## 4. Cleaner Applications

Backend systems can query views instead of writing complex joins repeatedly.

---

## 5. Logical Data Abstraction

Views hide underlying table complexity from developers and applications.

---

# 3. Creating Views

## Basic Syntax

```sql id="x0d3nh"
CREATE VIEW view_name AS
SELECT columns
FROM table
WHERE condition;
```

---

## Example

```sql id="6m9cbr"
CREATE VIEW high_salary_employees AS
SELECT name, salary
FROM employees
WHERE salary > 5000;
```

---

## Using the View

```sql id="9cl1fx"
SELECT * FROM high_salary_employees;
```

## Output

```text id="pn8tcy"
Alice   6000
Bob     7000
```

---

# 4. Views with Joins

Views can include complex joins.

## Example

```sql id="4d8sqp"
CREATE VIEW employee_departments AS
SELECT
    e.name,
    d.name AS department
FROM employees e
JOIN departments d
ON e.department_id = d.id;
```

Now:

```sql id="6fh4tm"
SELECT * FROM employee_departments;
```

## Result

```text id="o2lg6w"
Alice   Engineering
Bob     HR
```

---

# 5. Updating Views

In PostgreSQL, updating a view depends on its complexity.

---

## Simple Views (Updatable)

If a view is based on a single table, it can often be updated.

### Example

```sql id="mw4kde"
CREATE VIEW user_names AS
SELECT id, name
FROM users;
```

Now update:

```sql id="y0cn2j"
UPDATE user_names
SET name = 'John'
WHERE id = 1;
```

This updates the underlying `users` table.

---

## Complex Views (Not Always Updatable)

Views with:

* joins
* aggregations
* `DISTINCT`
* `GROUP BY`

may not be directly updatable.

### Example (Non-updatable View)

```sql id="6ynazd"
CREATE VIEW sales_summary AS
SELECT product, SUM(amount)
FROM sales
GROUP BY product;
```

You cannot directly `UPDATE` this view.

---

# 6. Dropping Views

To remove a view:

```sql id="4rjvzl"
DROP VIEW view_name;
```

---

## Example

```sql id="p4nlt5"
DROP VIEW high_salary_employees;
```

---

## Safe Drop

Avoid errors if the view does not exist:

```sql id="4rwewi"
DROP VIEW IF EXISTS high_salary_employees;
```

---

# 7. Renaming Views

You can rename a view.

## Syntax

```sql id="0tmzba"
ALTER VIEW old_name RENAME TO new_name;
```

---

## Example

```sql id="t1hzlo"
ALTER VIEW high_salary_employees
RENAME TO top_earners;
```

---

# 8. Materialized Views

A materialized view is different from a normal view.

## Key Difference

```text id="9j0g7o"
+--------------------+------------------+----------------------+
| Feature            | View             | Materialized View    |
+--------------------+------------------+----------------------+
| Stores data        | No               | Yes                  |
| Query speed        | Slower           | Faster               |
| Always updated     | Yes              | No (needs refresh)   |
+--------------------+------------------+----------------------+
```

---

## Why Materialized Views Exist

### Normal Views

* run query every time

### Materialized Views

* store precomputed results

---

# 9. Creating Materialized Views

## Syntax

```sql id="b8j1ys"
CREATE MATERIALIZED VIEW view_name AS
SELECT columns
FROM table;
```

---

## Example

```sql id="zxl7ql"
CREATE MATERIALIZED VIEW sales_summary AS
SELECT product, SUM(amount) AS total_sales
FROM sales
GROUP BY product;
```

Now query:

```sql id="nltwsk"
SELECT * FROM sales_summary;
```

Very fast because data is precomputed.

---

# 10. Refreshing Materialized Views

Materialized views are **not automatically updated**.

To refresh:

```sql id="z00kci"
REFRESH MATERIALIZED VIEW sales_summary;
```

---

## Concurrent Refresh (Non-blocking Reads)

```sql id="hvh40o"
REFRESH MATERIALIZED VIEW CONCURRENTLY sales_summary;
```

Requires a unique index.

---

# 11. When to Use Materialized Views

Use materialized views when:

* queries are expensive
* data does not change frequently
* performance is critical
* reporting dashboards are used

---

## Examples

* monthly reports
* analytics dashboards
* aggregated sales data

---

# 12. View vs Table vs Materialized View

```text id="4a2yxm"
+----------------------+-----------+--------------------+-------------------+
| Feature              | Table     | View               | Materialized View |
+----------------------+-----------+--------------------+-------------------+
| Stores data          | Yes       | No                 | Yes               |
| Automatically updated| Yes       | Yes                | No                |
| Performance          | Fast      | Slower             | Fast              |
| Storage usage        | High      | None               | Medium            |
+----------------------+-----------+--------------------+-------------------+
```

---

# 13. Real-World Example

## Problem

An e-commerce system has:

* customers
* orders
* order_items
* products

Complex query:

```sql id="j0s7im"
SELECT c.name, SUM(oi.quantity * p.price)
FROM customers c
JOIN orders o ON c.id = o.customer_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
GROUP BY c.name;
```

---

## Solution — Create View

```sql id="dbbg2p"
CREATE VIEW customer_revenue AS
SELECT
    c.name,
    SUM(oi.quantity * p.price) AS total_revenue
FROM customers c
JOIN orders o ON c.id = o.customer_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
GROUP BY c.name;
```

Now usage becomes simple:

```sql id="mjlwmk"
SELECT * FROM customer_revenue;
```

---

## Better Performance — Materialized View

```sql id="gnf91x"
CREATE MATERIALIZED VIEW customer_revenue_mv AS
SELECT
    c.name,
    SUM(oi.quantity * p.price) AS total_revenue
FROM customers c
JOIN orders o ON c.id = o.customer_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
GROUP BY c.name;
```

Refresh periodically:

```sql id="8i6n8v"
REFRESH MATERIALIZED VIEW customer_revenue_mv;
```

---

# 14. Views and Security

Views are commonly used for access control.

## Example

```sql id="vrckmy"
CREATE VIEW public_users AS
SELECT id, name
FROM users;
```

Then:

```sql id="gdz71t"
GRANT SELECT ON public_users TO app_user;
```

Users cannot access sensitive fields directly.

---

# 15. Performance Considerations

## Normal Views

* no storage cost
* query executed every time

---

## Materialized Views

* faster reads
* require storage
* must be refreshed

---

## Indexes on Materialized Views

Indexes can be added to materialized views.

Example:

```sql id="2bzgzy"
CREATE INDEX idx_sales_summary
ON sales_summary(product);
```

---

# 16. Common Mistakes

## 1. Thinking Views Store Data

Normal views do **NOT** store data.

Only materialized views store data physically.

---

## 2. Overusing Views

Too many nested views can reduce performance and make debugging difficult.

---

## 3. Not Refreshing Materialized Views

Data becomes stale if refresh operations are ignored.

---

# 17. Internal Architecture of Views

Internally PostgreSQL handles views like this:

```text id="mibgka"
Application Query
       │
       ▼
View Definition
       │
       ▼
Underlying SQL Query
       │
       ▼
Query Planner
       │
       ▼
Table Access
       │
       ▼
Final Result
```

For materialized views:

```text id="utvzw4"
Materialized View
       │
       ▼
Stored Physical Data
       │
       ▼
Fast Query Access
```

---

# 18. Best Practices

## Use Views for Reusable Query Logic

Avoid duplicating complex SQL in multiple places.

---

## Use Materialized Views for Heavy Analytics

Especially useful for dashboards and reports.

---

## Keep View Definitions Simple

Complex nested views may hurt readability and performance.

---

## Refresh Materialized Views Strategically

Examples:

* nightly refresh
* hourly refresh
* scheduled refresh jobs

---

## Use Security-Focused Views

Expose only necessary columns to applications/users.

---

# 19. Summary

Views in PostgreSQL are powerful tools for:

* simplifying SQL
* improving security
* reusing queries
* organizing database logic

## Types

```text id="3e5vab"
+--------------------+---------------------------+
| Type               | Description               |
+--------------------+---------------------------+
| View               | Virtual table             |
| Materialized View  | Stored result table       |
+--------------------+---------------------------+
```

---

## Key Operations

```text id="mtl1fe"
CREATE VIEW
SELECT from view
DROP VIEW
CREATE MATERIALIZED VIEW
REFRESH MATERIALIZED VIEW
```

Views are essential for building clean, maintainable, and scalable database systems.

---

# Conclusion

Views provide an abstraction layer over database tables, allowing developers to simplify complex queries and improve maintainability.

Key concepts include:

* normal views
* materialized views
* reusable query logic
* security-focused data access
* performance optimization

Understanding views deeply is important for building professional PostgreSQL systems, especially in analytics platforms, enterprise applications, dashboards, and large-scale backend architectures.

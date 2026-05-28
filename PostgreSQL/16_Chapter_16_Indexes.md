# PostgreSQL Hands-On Tutorial
## Chapter 16: Indexes

Indexes are one of the most important performance features in PostgreSQL. They are used to make data retrieval much faster, especially on large tables.

Without indexes, PostgreSQL must perform a full table scan (checking every row). With indexes, it can directly locate rows using an optimized data structure.

Indexes are essential for:

* fast `SELECT` queries
* efficient filtering (`WHERE`)
* quick joins
* sorting (`ORDER BY`) optimization
* enforcing uniqueness
* large-scale analytics systems

However, indexes come with trade-offs:

* extra storage
* slower `INSERT` / `UPDATE` / `DELETE`
* maintenance overhead

This guide explains indexes in extreme practical detail, including types, usage, and internal behavior.

---

# 1. What Is an Index?

An index is a special lookup structure that allows PostgreSQL to find rows quickly without scanning the entire table.

## Real-Life Analogy

Think of a table as a book:

* Without index → reading every page to find a word
* With index → using the book index section to jump directly

## Example Table

```sql
CREATE TABLE users (
    id SERIAL,
    name TEXT,
    email TEXT,
    age INT
);
```

Without index:

```sql
SELECT * FROM users WHERE email = 'a@mail.com';
```

→ scans entire table

With index:

→ directly jumps to matching row

---

# 2. Why Indexes Are Important

Indexes improve performance for:

## 1. WHERE Conditions

```sql
SELECT * FROM users WHERE email = 'a@mail.com';
```

## 2. JOIN Operations

```sql
ON users.id = orders.user_id
```

## 3. Sorting

```sql
ORDER BY age
```

## 4. Uniqueness Enforcement

```sql
email UNIQUE
```

---

# 3. How Indexes Work Internally

PostgreSQL commonly uses:

* B-Tree structure

An index stores:

```text
key → pointer to row location
```

Instead of scanning:

```text
row1 → row2 → row3 → rowN
```

It performs:

```text
binary-like search → direct location
```

---

# 4. Creating Indexes

## Basic Syntax

```sql
CREATE INDEX index_name
ON table_name(column_name);
```

## Example

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Now queries become faster:

```sql
SELECT * FROM users WHERE email = 'test@mail.com';
```

---

# 5. Unique Indexes

A unique index ensures all values are unique.

## Syntax

```sql
CREATE UNIQUE INDEX index_name
ON table(column);
```

## Example

```sql
CREATE UNIQUE INDEX idx_users_email_unique
ON users(email);
```

Now duplicates are not allowed:

```sql
INSERT INTO users(email) VALUES ('a@mail.com');

INSERT INTO users(email) VALUES ('a@mail.com'); -- ERROR
```

## Important Note

A `PRIMARY KEY` automatically creates a unique index.

---

# 6. Composite Indexes

A composite index uses multiple columns together.

## Syntax

```sql
CREATE INDEX index_name
ON table(column1, column2);
```

## Example

```sql
CREATE INDEX idx_users_name_age
ON users(name, age);
```

## How It Works

Optimized for queries like:

```sql
SELECT * FROM users
WHERE name = 'Alice' AND age = 25;
```

## Index Order Matters

Index:

```text
(name, age)
```

Efficient for:

```sql
WHERE name = ?
WHERE name = ? AND age = ?
```

Not efficient for:

```sql
WHERE age = ?
```

---

# 7. Dropping Indexes

To remove an index:

```sql
DROP INDEX index_name;
```

## Example

```sql
DROP INDEX idx_users_email;
```

## Safe Drop

```sql
DROP INDEX IF EXISTS idx_users_email;
```

---

# 8. Rebuilding Indexes

Indexes can become inefficient over time due to updates and deletions.

## Rebuild Specific Index

```sql
REINDEX INDEX index_name;
```

## Rebuild All Indexes on Table

```sql
REINDEX TABLE users;
```

## Why Rebuild?

* fragmented index
* performance degradation
* heavy update/delete workloads

---

# 9. Types of Indexes in PostgreSQL

PostgreSQL supports multiple index types optimized for different workloads.

---

# 9.1 B-Tree Index (Default)

Most common index type.

## Used For

* equality searches (`=`)
* range queries (`>`, `<`, `BETWEEN`)
* sorting (`ORDER BY`)

## Example

```sql
CREATE INDEX idx_users_age
ON users USING BTREE(age);
```

Works for:

```sql
WHERE age = 25
WHERE age > 20
ORDER BY age
```

## Why B-Tree?

* balanced tree structure
* logarithmic search time
* efficient general-purpose index

---

# 9.2 Hash Index

Used for:

* equality comparisons only

## Example

```sql
CREATE INDEX idx_users_email_hash
ON users USING HASH(email);
```

Works for:

```sql
WHERE email = 'a@mail.com'
```

## Limitations

* not used for range queries
* less commonly used than B-Tree
* historically less stable (improved in modern PostgreSQL)

---

# 9.3 GIN Index (Generalized Inverted Index)

Used for:

* arrays
* `JSONB`
* full-text search

## Example (JSONB)

```sql
CREATE INDEX idx_data_json
ON documents USING GIN (data);
```

## Query Example

```sql
SELECT *
FROM documents
WHERE data @> '{"status":"active"}';
```

## Use Cases

* search engines
* JSON querying
* tag systems
* text search

---

# 9.4 GiST Index (Generalized Search Tree)

Used for:

* geometric data
* range types
* spatial queries

## Example

```sql
CREATE INDEX idx_location
ON places USING GIST(location);
```

## Use Cases

* maps
* geolocation systems
* distance calculations

---

# 9.5 BRIN Index (Block Range Index)

Designed for very large tables.

Stores summaries of data ranges.

## Example

```sql
CREATE INDEX idx_logs_time
ON logs USING BRIN(created_at);
```

## Best For

* logs
* time-series data
* huge datasets

## Why BRIN?

Instead of indexing every row:

* stores range of blocks
* very small and efficient

---

# 10. Partial Indexes

A partial index indexes only a subset of rows.

## Example

```sql
CREATE INDEX idx_active_users
ON users(email)
WHERE status = 'active';
```

## Benefits

* smaller index
* faster updates
* optimized queries

## Query Benefit

```sql
SELECT * FROM users
WHERE status = 'active'
AND email = 'a@mail.com';
```

---

# 11. Expression Indexes

An expression index is built on computed values.

## Example

```sql
CREATE INDEX idx_lower_email
ON users (LOWER(email));
```

## Query

```sql
SELECT * FROM users
WHERE LOWER(email) = 'test@mail.com';
```

## Use Cases

* case-insensitive search
* computed columns
* functions in `WHERE` clause

---

# 12. Index Usage in Joins

Indexes are critical for joins.

## Example

```sql
SELECT *
FROM orders o
JOIN users u
ON o.user_id = u.id;
```

If `user_id` is indexed:

* join becomes much faster

---

# 13. When NOT to Use Indexes

Indexes are not always beneficial.

Avoid indexing when:

* table is very small
* workload has heavy writes
* column is rarely queried

---

# 14. Index Performance Trade-Off

| Operation | Effect |
| --------- | ------ |
| SELECT    | Faster |
| INSERT    | Slower |
| UPDATE    | Slower |
| DELETE    | Slower |

## Reason

Indexes must also be updated when data changes.

---

# 15. Checking Index Usage

PostgreSQL provides:

```sql
EXPLAIN ANALYZE
```

## Example

```sql
EXPLAIN ANALYZE
SELECT * FROM users
WHERE email = 'a@mail.com';
```

Shows:

* index scan
* sequential scan
* execution cost
* timing information

---

# 16. Real-World Example

## E-Commerce System

```sql
CREATE INDEX idx_orders_user
ON orders(user_id);

CREATE INDEX idx_orders_date
ON orders(created_at);

CREATE INDEX idx_products_category
ON products(category);
```

## Result

* faster order history lookup
* faster filtering
* faster analytics queries

---

# 17. Summary

Indexes are critical performance structures in PostgreSQL.

## Key Concepts

| Concept | Meaning               |
| ------- | --------------------- |
| Index   | Fast lookup structure |
| B-Tree  | Default index         |
| Hash    | Equality only         |
| GIN     | JSON/text/arrays      |
| GiST    | Spatial data          |
| BRIN    | Huge datasets         |

---

## Advanced Features

* composite indexes
* partial indexes
* expression indexes
* index rebuilding

---

# Core Rule

Indexes speed up reads but slow down writes.

---

# Conclusion

Indexes are essential for building high-performance PostgreSQL systems. Proper index design can improve query speed from seconds to milliseconds.

However, incorrect indexing can degrade performance, so understanding index types and use cases is critical.

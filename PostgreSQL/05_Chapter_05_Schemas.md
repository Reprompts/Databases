# PostgreSQL Hands-On Tutorial

## Chapter 5: Schemas

In PostgreSQL, schemas are logical containers used to organize database objects such as tables, views, functions, indexes, and sequences. They act as namespaces that help structure and manage complex databases efficiently.

Schemas provide:

* Logical organization of objects
* Isolation between different applications or modules
* Namespace management to avoid naming conflicts
* Security boundaries for access control

This article explains PostgreSQL schemas in extremely detailed and practical depth, including their concepts, architecture, and real-world operations.

---

# 1. What Schemas Are

A schema is a namespace within a PostgreSQL database that groups related database objects.

Schemas help structure large databases by organizing objects into logical groups.

Think of schemas as folders inside a database.

---

## Hierarchy in PostgreSQL

The PostgreSQL data hierarchy looks like this:

```text
PostgreSQL Cluster
        │
        ▼
     Database
        │
        ▼
      Schema
        │
        ▼
 Database Objects
 (tables, views, indexes, functions)
```

### Example

```text
Database: shop
│
├── Schema: public
│      ├── users table
│      ├── orders table
│
├── Schema: sales
│      ├── invoices table
│      ├── payments table
│
└── Schema: analytics
       ├── reports table
       └── dashboards table
```

Each schema contains related objects.

---

# 2. Default Schema

Every PostgreSQL database automatically contains a schema called:

```text
public
```

This schema is created during database initialization.

Most simple applications place tables inside this schema.

---

## Example

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT
);
```

The table actually exists as:

```text
public.users
```

---

# 3. Schema Organization

Schemas help organize large systems by grouping objects logically.

This is essential for enterprise systems and large applications.

---

## Example Organization

Consider an e-commerce application.

```text
Database: ecommerce
│
├── Schema: auth
│     ├── users
│     ├── roles
│
├── Schema: products
│     ├── items
│     ├── categories
│
├── Schema: orders
│     ├── orders
│     ├── order_items
│
└── Schema: analytics
      ├── sales_reports
      └── customer_metrics
```

### Benefits

* Clean structure
* Easier maintenance
* Clear separation of modules

---

# 4. Schema Isolation

Schemas isolate objects within a database.

This prevents naming conflicts.

---

## Naming Conflict Example

Without schemas:

```text
users table
users table
```

Conflict occurs.

With schemas:

```text
auth.users
analytics.users
```

Both tables can exist simultaneously.

---

## Fully Qualified Names

Schema objects are accessed using:

```text
schema_name.object_name
```

### Example

```sql
SELECT * FROM auth.users;
```

This ensures PostgreSQL knows exactly which object to access.

---

# 5. Schema Search Path

PostgreSQL uses a search path to locate objects when schema is not specified.

Default search path:

```text
"$user", public
```

### Example Query

```sql
SELECT * FROM users;
```

PostgreSQL searches:

1. Schema named after user
2. `public` schema

---

## Viewing Search Path

```sql
SHOW search_path;
```

### Example Output

```text
"$user", public
```

---

## Changing Search Path

### Example

```sql
SET search_path TO sales, public;
```

Now PostgreSQL searches:

1. `sales` schema
2. `public` schema

---

# 6. Creating Schemas

Schemas are created using the `CREATE SCHEMA` command.

---

## Basic Syntax

```sql
CREATE SCHEMA schema_name;
```

### Example

```sql
CREATE SCHEMA sales;
```

Now the schema exists inside the database.

---

## Creating Schema with Owner

You can assign ownership during creation.

```sql
CREATE SCHEMA analytics
AUTHORIZATION alice;
```

Now Alice owns the schema.

---

## Creating Objects in Schema

Tables can be created directly inside schemas.

### Example

```sql
CREATE TABLE sales.orders (
    id SERIAL PRIMARY KEY,
    total NUMERIC
);
```

This creates the table inside the `sales` schema.

---

## Alternative Method

Switch schema using search path:

```sql
SET search_path TO sales;

CREATE TABLE orders (
    id SERIAL PRIMARY KEY
);
```

Now the table is created inside the `sales` schema.

---

# 7. Listing Schemas

PostgreSQL allows listing schemas using `psql` commands or SQL queries.

---

## Method 1: psql Command

Inside `psql`:

```sql
\dn
```

### Example Output

```text
List of schemas
Name      | Owner
----------+--------
public    | postgres
sales     | postgres
analytics | postgres
```

---

## Method 2: SQL Query

Schemas are stored in:

```text
pg_namespace
```

### Query

```sql
SELECT nspname
FROM pg_namespace;
```

### Example Output

```text
nspname
---------
public
sales
analytics
pg_catalog
information_schema
```

---

# 8. System Schemas

PostgreSQL includes several built-in schemas.

---

## pg_catalog

Contains PostgreSQL system tables and functions.

### Example Objects

* `pg_class`
* `pg_database`
* `pg_roles`

### Query Example

```sql
SELECT * FROM pg_catalog.pg_tables;
```

---

## information_schema

A standard SQL schema that provides metadata.

### Example

```sql
SELECT table_name
FROM information_schema.tables;
```

---

## pg_toast

Stores large object data internally.

Users rarely interact with it directly.

---

# 9. Altering Schemas

Schemas can be modified using `ALTER SCHEMA`.

---

## Renaming Schema

### Syntax

```sql
ALTER SCHEMA old_name
RENAME TO new_name;
```

### Example

```sql
ALTER SCHEMA sales
RENAME TO store_sales;
```

Now the schema name becomes:

```text
store_sales
```

---

## Moving Objects Between Schemas

Tables can be moved to another schema.

### Example

```sql
ALTER TABLE public.orders
SET SCHEMA sales;
```

Now the table exists as:

```text
sales.orders
```

---

# 10. Setting Schema Ownership

Schemas have owners.

The owner can:

* Modify schema
* Create objects
* Manage permissions

---

## Change Owner

### Syntax

```sql
ALTER SCHEMA schema_name
OWNER TO new_owner;
```

### Example

```sql
ALTER SCHEMA sales
OWNER TO alice;
```

Now Alice owns the schema.

---

## Checking Schema Owner

### Query

```sql
SELECT nspname, pg_get_userbyid(nspowner)
FROM pg_namespace;
```

### Example Output

```text
schema  | owner
--------+-------
public  | postgres
sales   | alice
```

---

# 11. Dropping Schemas

Schemas can be removed using `DROP SCHEMA`.

---

## Basic Syntax

```sql
DROP SCHEMA schema_name;
```

### Example

```sql
DROP SCHEMA sales;
```

---

## Problem

If the schema contains objects:

```text
ERROR: schema contains objects
```

---

## Drop with CASCADE

`CASCADE` removes all objects inside the schema.

### Example

```sql
DROP SCHEMA sales CASCADE;
```

This deletes:

* Tables
* Views
* Indexes
* Functions

---

## Safe Drop

Use `IF EXISTS`:

```sql
DROP SCHEMA IF EXISTS sales;
```

---

# 12. Practical Example

Let's build a real schema structure.

---

## Step 1: Create Schemas

```sql
CREATE SCHEMA auth;
CREATE SCHEMA inventory;
CREATE SCHEMA orders;
```

---

## Step 2: Create Tables

### Auth Schema

```sql
CREATE TABLE auth.users (
    id SERIAL PRIMARY KEY,
    username TEXT
);
```

### Inventory Schema

```sql
CREATE TABLE inventory.products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    price NUMERIC
);
```

### Orders Schema

```sql
CREATE TABLE orders.orders (
    id SERIAL PRIMARY KEY,
    user_id INT
);
```

---

## Step 3: Query Data

### Example

```sql
SELECT * FROM inventory.products;
```

---

# 13. Multi-Tenant Architecture Using Schemas

Schemas are commonly used in multi-tenant applications.

### Example

```text
Database: SaaS_platform

tenant1 schema
tenant2 schema
tenant3 schema
```

Each tenant has isolated tables.

```text
tenant1.users
tenant2.users
tenant3.users
```

### Benefits

* Strong isolation
* Easier backups
* Tenant-level control

---

# 14. Best Practices for Schema Design

Professional PostgreSQL systems follow schema design principles.

---

## Use Schemas for Modules

### Example

* `auth`
* `billing`
* `inventory`
* `analytics`

---

## Avoid Too Many Schemas

Too many schemas increase complexity.

---

## Use Explicit Schema References

### Good Practice

```sql
SELECT * FROM inventory.products;
```

Instead of:

```sql
SELECT * FROM products;
```

---

## Manage Permissions at Schema Level

### Example

```sql
GRANT USAGE ON SCHEMA sales TO analyst_role;
```

---

# PostgreSQL Schema Architecture Overview

Complete architecture:

```text
PostgreSQL Cluster
        │
        ▼
     Database
        │
        ▼
     Schemas
        │
 ┌──────┼────────┐
 │      │        │
 ▼      ▼        ▼
Tables  Views  Functions
Indexes Sequences Triggers
```

Each schema acts as a namespace boundary.

---

# Conclusion

Schemas are a fundamental organizational feature in PostgreSQL that allow developers and administrators to structure databases cleanly and securely.

They provide:

* Logical grouping of database objects
* Namespace isolation
* Modular architecture
* Improved security management

Through operations such as:

* Creating schemas
* Listing schemas
* Altering schemas
* Assigning ownership
* Dropping schemas

PostgreSQL allows administrators to maintain well-organized and scalable database systems.

Schemas are especially important in:

* Enterprise applications
* Microservices architectures
* Multi-tenant SaaS systems

---

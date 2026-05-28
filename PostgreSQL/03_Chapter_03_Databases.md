# PostgreSQL Hands-On Tutorial
## Databases 


In PostgreSQL, a database is a logical container that stores structured data such as tables, indexes, views, functions, and other objects. A PostgreSQL server can manage multiple databases simultaneously, each isolated from the others but operating within the same database cluster.

This article provides a deep, practical, and extremely detailed explanation of PostgreSQL database operations, including:

* Creating a database
* Listing databases
* Connecting to a database
* Renaming a database
* Dropping a database
* Database templates
* Cloning databases

The explanations include conceptual understanding, internal behavior, and practical SQL examples.

---

# 1. Understanding PostgreSQL Databases

Before learning operations, it is important to understand how PostgreSQL organizes databases.

A PostgreSQL cluster contains multiple databases.

```text id="v7a3pd"
PostgreSQL Cluster
│
├── Database: postgres
├── Database: template0
├── Database: template1
├── Database: shop
├── Database: analytics
```

Each database contains its own:

* Schemas
* Tables
* Indexes
* Functions
* Extensions
* Views

Databases cannot directly access tables in another database.

To share data between databases, systems typically use:

* Foreign Data Wrappers
* APIs
* Data replication

---

# 2. Creating a Database

Creating a database is one of the first operations performed when setting up an application.

A database acts as the main container for all application data.

---

## Basic Syntax

```sql id="b4d9qn"
CREATE DATABASE database_name;
```

### Example

```sql id="g5u1cx"
CREATE DATABASE shop;
```

This creates a new database called `shop`.

---

## What Happens Internally

When PostgreSQL creates a database:

1. It copies the template database (usually `template1`)
2. It creates a new directory in the data folder
3. System catalog entries are created

Internally:

```text id="s8w2rf"
pg_database
```

This catalog stores metadata about databases.

---

## Creating Database with Options

PostgreSQL allows specifying parameters.

### Syntax

```sql id="n2x7lm"
CREATE DATABASE name
WITH
OWNER owner_name
ENCODING encoding
LC_COLLATE locale
LC_CTYPE locale
TEMPLATE template
TABLESPACE tablespace;
```

---

### Example

```sql id="u9c4yk"
CREATE DATABASE shop
WITH
OWNER postgres
ENCODING 'UTF8'
TEMPLATE template1;
```

---

## Explanation

| Parameter  | Meaning            |
| ---------- | ------------------ |
| OWNER      | Database owner     |
| ENCODING   | Character encoding |
| TEMPLATE   | Template used      |
| TABLESPACE | Storage location   |

---

## Character Encoding

Encoding defines how text is stored.

### Common Encodings

| Encoding  | Description        |
| --------- | ------------------ |
| UTF8      | Most common        |
| LATIN1    | Older encoding     |
| SQL_ASCII | Minimal validation |

### Example

```sql id="x3f7va"
CREATE DATABASE mydb
ENCODING 'UTF8';
```

UTF8 supports all languages.

---

## Tablespace Option

A tablespace controls where data is stored on disk.

### Example

```sql id="m1p6oz"
CREATE DATABASE sales
TABLESPACE fast_storage;
```

This allows storing databases on specific disks.

---

# 3. Listing Databases

PostgreSQL provides several ways to list databases.

---

## Method 1: psql Command

Inside the `psql` shell:

```sql id="q6d4bw"
\l
```

or

```sql id="k7n2eh"
\list
```

### Example Output

```text id="r4v8sy"
Name       | Owner    | Encoding | Collate | Ctype
-----------+----------+----------+---------+---------
postgres   | postgres | UTF8     | en_US   | en_US
template0  | postgres | UTF8     | en_US   | en_US
template1  | postgres | UTF8     | en_US   | en_US
shop       | postgres | UTF8     | en_US   | en_US
```

---

## Method 2: SQL Query

You can query the system catalog.

```sql id="z5x1uc"
SELECT datname
FROM pg_database;
```

### Output

```text id="j8m0tr"
datname
---------
postgres
template0
template1
shop
```

---

## Database Metadata

More detailed information:

```sql id="o3n9lf"
SELECT *
FROM pg_database;
```

### Important Columns

| Column       | Description        |
| ------------ | ------------------ |
| datname      | Database name      |
| datowner     | Owner              |
| encoding     | Character encoding |
| datcollate   | Collation          |
| datctype     | Character type     |
| datallowconn | Connection allowed |

---

# 4. Connecting to a Database

To perform operations inside a database, you must connect to it.

Connections can be made using:

* `psql` CLI
* Application drivers
* GUI tools

---

## Using psql

### Syntax

```bash id="h2v8ke"
psql -U username -d database_name
```

### Example

```bash id="y4m7sa"
psql -U postgres -d shop
```

---

## Connecting Inside psql

If already inside `psql`:

```sql id="c9p2xd"
\c shop
```

### Example Output

```text id="l7f1wr"
You are now connected to database "shop".
```

---

## What Happens Internally

When a connection occurs:

1. PostgreSQL authenticates the user
2. A backend process is created
3. Database catalogs are loaded
4. Session variables are initialized

### Architecture

```text id="d8s6nm"
Client
  |
  v
PostgreSQL Server
  |
  v
Backend Process
  |
  v
Database
```

---

## Checking Current Database

Inside `psql`:

```sql id="f5x8qo"
SELECT current_database();
```

### Example

```text id="n0t4zc"
current_database
----------------
shop
```

---

# 5. Renaming a Database

Databases can be renamed using `ALTER DATABASE`.

---

## Syntax

```sql id="w7c9ye"
ALTER DATABASE old_name
RENAME TO new_name;
```

---

## Example

```sql id="p4u1dr"
ALTER DATABASE shop
RENAME TO ecommerce;
```

Now the database name becomes:

```text id="a2x5gh"
ecommerce
```

---

## Important Restrictions

Renaming a database has limitations.

You cannot rename a database if it has active connections.

### Example Error

```text id="t3k8ms"
ERROR: database "shop" is being accessed by other users
```

---

## How to Rename Safely

### Step 1: Disconnect Users

```sql id="r9y2fj"
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE datname = 'shop';
```

---

### Step 2: Rename Database

```sql id="v8d5pl"
ALTER DATABASE shop RENAME TO ecommerce;
```

---

# 6. Dropping a Database

Dropping a database permanently deletes it.

All tables, data, and objects are removed.

---

## Syntax

```sql id="u1r6zx"
DROP DATABASE database_name;
```

### Example

```sql id="e7m4ba"
DROP DATABASE shop;
```

---

## Safety Rule

You cannot drop the database you are connected to.

### Example Error

```text id="s4n7hy"
ERROR: cannot drop the currently open database
```

---

## Proper Method

Switch to another database:

```sql id="o6w2pq"
\c postgres
```

Then drop:

```sql id="b1t8xn"
DROP DATABASE shop;
```

---

## Drop with Condition

Avoid errors if database does not exist:

```sql id="m7k5rv"
DROP DATABASE IF EXISTS shop;
```

---

## Internal Behavior

Dropping a database:

* Removes catalog entries
* Deletes database directory
* Removes files

---

# 7. Database Templates

PostgreSQL uses template databases to create new databases.

Templates act as blueprints.

---

## Default Templates

Every PostgreSQL installation contains:

* `template0`
* `template1`

---

## template1

Default template used when creating databases.

```sql id="f8x1cm"
CREATE DATABASE newdb;
```

Equivalent to:

```sql id="g3u9vl"
CREATE DATABASE newdb TEMPLATE template1;
```

---

## template0

`template0` is a clean template.

Used when a fresh database is required.

### Example

```sql id="j2n4we"
CREATE DATABASE newdb TEMPLATE template0;
```

### Use Cases

* Changing encoding
* Restoring backups
* Creating clean databases

---

## Viewing Templates

### Query

```sql id="c5v7qz"
SELECT datname, datistemplate
FROM pg_database;
```

### Example Output

```text id="x8l2ro"
datname     | datistemplate
------------+--------------
template0   | true
template1   | true
postgres    | false
shop        | false
```

---

# 8. Cloning Databases

PostgreSQL allows cloning databases using templates.

Cloning copies:

* Schema
* Tables
* Data
* Indexes
* Functions

---

## Basic Cloning

### Example

```sql id="y7m3dp"
CREATE DATABASE shop_test
TEMPLATE shop;
```

This creates a copy of the `shop` database.

---

## Practical Use Cases

Cloning is useful for:

* Development environments
* Testing
* Backups
* Staging systems

---

## Example Workflow

```text id="z9r5xf"
Production Database
       |
       v
Clone Database
       |
       v
Testing Environment
```

---

## Example

### Create Production Database

```sql id="n4w8kb"
CREATE DATABASE production;
```

### Clone for Testing

```sql id="q1v6et"
CREATE DATABASE testing
TEMPLATE production;
```

Now `testing` contains identical data.

---

## Limitations

Database cloning requires:

* No active connections
* Sufficient disk space

If users are connected, cloning fails.

---

# 9. Database Storage on Disk

Each database is stored in its own directory.

### Location

```text id="l6k3vo"
data/base/
```

### Example Structure

```text id="r2x7qp"
data/
 └── base/
      ├── 1
      ├── 16384
      ├── 24576
```

These numbers represent database OIDs.

---

## Checking Database OID

```sql id="t5m9zx"
SELECT oid, datname
FROM pg_database;
```

### Example Output

```text id="w3f8ly"
oid   | datname
------+---------
16384 | shop
24576 | analytics
```

---

# 10. Best Practices for Database Management

Experienced PostgreSQL administrators follow best practices.

---

## Use Meaningful Names

### Good Examples

* `inventory_db`
* `user_service_db`
* `analytics_db`

### Avoid

* `db1`
* `test123`

---

## Separate Databases for Services

### Example Microservices Architecture

* `user_service_db`
* `payment_service_db`
* `order_service_db`

---

## Limit Permissions

Control access using roles.

### Example

```sql id="u8r2df"
GRANT CONNECT ON DATABASE shop TO app_user;
```

---

## Avoid Frequent Database Creation

Most applications use one database with multiple schemas instead of many databases.

---

# Complete PostgreSQL Database Operations Flow

```text id="m5q9xn"
PostgreSQL Cluster
        │
        ▼
Create Database
        │
        ▼
List Databases
        │
        ▼
Connect to Database
        │
        ▼
Create Tables
        │
        ▼
Use Database
        │
        ▼
Clone Database
        │
        ▼
Rename Database
        │
        ▼
Drop Database
```

---

# Conclusion

PostgreSQL databases form the logical foundation of data organization inside a PostgreSQL cluster.

Understanding database operations is essential for:

* Application development
* System administration
* Database management

Key operations include:

* Creating databases
* Listing databases
* Connecting to databases
* Renaming databases
* Deleting databases
* Using database templates
* Cloning databases

These operations allow developers and administrators to organize data, manage environments, and maintain systems efficiently.

---

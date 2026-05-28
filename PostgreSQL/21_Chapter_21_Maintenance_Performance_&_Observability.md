
# PostgreSQL Hands-On Tutorial
## Chapter 21: Maintenance, Performance & Observability 

In production environments, a PostgreSQL database must do more than just store data. It must be maintained, monitored, and optimized continuously to ensure reliability and performance.

Three core operational pillars help achieve this:

- Backup and Restore → Protecting data from loss
- Vacuum and Analyze → Maintaining storage health and query performance
- Logging and Monitoring → Observing system behavior and diagnosing problems
- Extensions → Enhancing PostgreSQL capabilities

Together these ensure that a PostgreSQL system remains stable, efficient, and recoverable.

---

# 1. Backup and Restore

Backups are essential for data protection and disaster recovery. PostgreSQL provides powerful tools to create backups and restore databases.

Main tools include:

- `pg_dump`
- `pg_restore`

---

# 1.1 Using `pg_dump`

`pg_dump` is a command-line utility used to create logical backups of PostgreSQL databases.

Logical backups contain:

- table definitions
- indexes
- data
- constraints
- functions

## Basic Backup Command

```bash
pg_dump -U postgres mydatabase > backup.sql
````

### Explanation

| Parameter      | Meaning                                        |
| -------------- | ---------------------------------------------- |
| `-U`           | Specifies the database user for authentication |
| `postgres`     | The username used to connect to the database   |
| `mydatabase`   | The name of the database to be backed up       |
| `> backup.sql` | Redirects the output into a backup file        |

This produces a SQL script that can recreate the database.

## Example Output

The backup file contains commands like:

```sql
CREATE TABLE users (
    id integer,
    name text
);

INSERT INTO users VALUES (1, 'Alice');
```

---

# 1.2 Full Database Backup

To back up the entire database with compression:

```bash
pg_dump -U postgres -F c -f backup.dump mydatabase
```

## Options Explained

| Option        | Meaning                       |
| ------------- | ----------------------------- |
| `-F c`        | Uses custom compressed format |
| `-f`          | Specifies output file         |
| `backup.dump` | Name of backup file           |

Custom format backups are more flexible for restoration.

---

# 1.3 Schema Backup

Sometimes only database structure is required.

Example:

```bash
pg_dump -U postgres --schema-only mydatabase > schema.sql
```

This includes:

* table structures
* indexes
* constraints
* functions

But no data.

---

# 1.4 Data-Only Backup

To backup only the data:

```bash
pg_dump -U postgres --data-only mydatabase > data.sql
```

---

# 1.5 Using `pg_restore`

`pg_restore` restores backups created in custom format.

## Example Restore Command

```bash
pg_restore -U postgres -d mydatabase backup.dump
```

### Explanation

| Option       | Meaning                                |
| ------------ | -------------------------------------- |
| `-U`         | Username used for authentication       |
| `postgres`   | Database user                          |
| `-d`         | Target database                        |
| `mydatabase` | Database where backup will be restored |

## Restore into New Database

First create the database:

```sql
CREATE DATABASE restored_db;
```

Then restore:

```bash
pg_restore -U postgres -d restored_db backup.dump
```

---

# 1.6 Restoring SQL Backups

If the backup file is `.sql`:

```bash
psql -U postgres -d mydatabase -f backup.sql
```

---

# 1.7 Backup Best Practices

In production environments:

* automate backups using cron jobs
* store backups offsite
* test restore procedures regularly
* maintain multiple backup versions

## Example Cron Job

```bash
0 2 * * * pg_dump mydatabase > daily_backup.sql
```

This runs backup every night at 2 AM.

---

# 2. Vacuum and Analyze (Operational Perspective)

PostgreSQL uses MVCC, meaning updates create new row versions.

Over time this causes:

* dead tuples
* storage bloat
* performance degradation

Maintenance tools clean this up.

---

# 2.1 Running `VACUUM`

`VACUUM` removes dead tuples created by:

* `UPDATE`
* `DELETE`

## Basic Command

```sql
VACUUM;
```

## Vacuum Specific Table

```sql
VACUUM users;
```

## Vacuum with Statistics Update

```sql
VACUUM ANALYZE users;
```

This performs:

* cleanup
* statistics update

---

# 2.2 `VACUUM FULL`

A more aggressive cleanup:

```sql
VACUUM FULL users;
```

This:

* rewrites the table
* reduces disk space usage
* locks the table temporarily

Used when database bloat is severe.

---

# 2.3 Running `ANALYZE`

`ANALYZE` collects statistics about:

* data distribution
* table sizes
* column values

These statistics help the query planner choose optimal execution strategies.

Example:

```sql
ANALYZE users;
```

---

# 2.4 Autovacuum

Manual maintenance is impractical in large systems.

PostgreSQL automatically runs autovacuum workers.

Autovacuum performs:

* automatic `VACUUM`
* automatic `ANALYZE`

---

# 2.5 Autovacuum Configuration

Important parameters in `postgresql.conf`:

```conf
autovacuum = on
autovacuum_max_workers = 3
autovacuum_vacuum_scale_factor = 0.2
autovacuum_analyze_scale_factor = 0.1
```

## Explanation

| Parameter                         | Meaning                          |
| --------------------------------- | -------------------------------- |
| `autovacuum_max_workers`          | Number of vacuum workers         |
| `autovacuum_vacuum_scale_factor`  | Threshold for vacuum             |
| `autovacuum_analyze_scale_factor` | Threshold for statistics updates |

---

# 2.6 Monitoring Autovacuum

You can monitor activity using:

```sql
SELECT * FROM pg_stat_activity;
```

Or:

```sql
SELECT * FROM pg_stat_user_tables;
```

These views show:

* vacuum statistics
* table usage patterns

---

# 3. Logging and Monitoring

Monitoring PostgreSQL allows administrators to detect:

* slow queries
* performance bottlenecks
* connection issues
* system errors

PostgreSQL provides a built-in logging framework.

---

# 3.1 PostgreSQL Logging System

Logging captures:

* errors
* warnings
* connections
* queries
* system events

Logs are configured in:

```text
postgresql.conf
```

---

# 3.2 Configuring Logs

Important logging settings:

```conf
logging_collector = on
log_directory = 'pg_log'
log_filename = 'postgresql.log'
log_statement = 'all'
```

## Explanation

| Parameter           | Meaning                         |
| ------------------- | ------------------------------- |
| `logging_collector` | Enables log collection          |
| `log_directory`     | Directory where logs are stored |
| `log_filename`      | Log file name                   |
| `log_statement`     | Queries to log                  |

---

# 3.3 Logging Slow Queries

Slow queries often cause performance problems.

Configuration:

```conf
log_min_duration_statement = 1000
```

Meaning:

* log queries taking longer than 1 second

---

# 3.4 Analyzing Logs

Logs help identify:

* slow queries
* deadlocks
* connection spikes
* authentication failures

## Example Log Entry

```text
duration: 1534 ms  statement: SELECT * FROM orders;
```

This query took 1.5 seconds.

---

# 3.5 Monitoring Queries

PostgreSQL provides system views to monitor activity.

## Active Queries

```sql
SELECT * FROM pg_stat_activity;
```

Shows:

* running queries
* user sessions
* connection status

## Query Statistics

```sql
SELECT * FROM pg_stat_statements;
```

Provides:

* query execution counts
* total execution time
* average execution time

This is extremely useful for performance tuning.

---

# 4. PostgreSQL Extensions

Extensions allow PostgreSQL to be extended with additional functionality without modifying the core system.

They provide features such as:

* geospatial queries
* advanced indexing
* monitoring tools
* performance improvements

---

# 4.1 Installing Extensions

First enable extension support:

```sql
CREATE EXTENSION extension_name;
```

Example:

```sql
CREATE EXTENSION pg_stat_statements;
```

---

# 4.2 Listing Installed Extensions

```sql
SELECT * FROM pg_extension;
```

---

# 4.3 Removing Extensions

```sql
DROP EXTENSION extension_name;
```

---

# 4.4 Managing Extensions

You can upgrade extensions:

```sql
ALTER EXTENSION extension_name UPDATE;
```

---

# 4.5 Popular PostgreSQL Extensions

## `pg_stat_statements`

Tracks query performance.

Example query:

```sql
SELECT query, total_exec_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC;
```

---

## `PostGIS`

Adds geospatial capabilities.

Used for:

* GIS systems
* maps
* location tracking

---

## `pg_trgm`

Supports fuzzy string matching.

Used for:

* search suggestions
* spell correction

---

## `uuid-ossp`

Generates UUID identifiers.

Example:

```sql
SELECT uuid_generate_v4();
```

---

# Production Monitoring Workflow

In real systems:

1. Logs detect slow queries
2. `pg_stat_statements` identifies expensive queries
3. `EXPLAIN ANALYZE` diagnoses query plans
4. Indexes or schema changes improve performance

---

# Maintenance Strategy for Production Databases

A typical PostgreSQL maintenance workflow includes:

## Daily Tasks

* monitor logs
* check query performance
* verify backups

## Weekly Tasks

* analyze slow queries
* inspect table bloat

## Monthly Tasks

* test restore procedures
* review storage growth
* update extensions

---

# Final Conceptual Architecture

PostgreSQL operations can be viewed as four operational layers:

## Data Protection Layer

```text
Backups and Restore
```

## Storage Maintenance Layer

```text
Vacuum and Analyze
```

## Observability Layer

```text
Logging and Monitoring
```

## Extensibility Layer

```text
PostgreSQL Extensions
```

These layers ensure that PostgreSQL systems remain:

* reliable
* performant
* scalable
* observable

---

```

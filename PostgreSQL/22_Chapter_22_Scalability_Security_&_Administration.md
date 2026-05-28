
# PostgreSQL Hands-On Tutorial
## Chapter 22: Scalability, Secutiry & Observability 

When PostgreSQL moves from development to production environments, several additional responsibilities become critical:

- Scaling databases to handle large workloads
- Protecting data through strong security
- Managing users, resources, and storage
- Ensuring high availability and fault tolerance

This article explains the production architecture of PostgreSQL systems with practical commands and examples.

---

# 1. Replication in PostgreSQL

Replication allows PostgreSQL to copy data from one server to another.

It is essential for:

- high availability
- load balancing
- disaster recovery
- read scaling

In replication, we typically have:

```text
Primary Server (Master)
       ↓
Replica Servers (Standby)
````

The primary server handles writes, while replicas usually handle reads.

---

# 1.1 Replication Types

PostgreSQL supports two main replication approaches:

| Replication Type      | Description                          |
| --------------------- | ------------------------------------ |
| Streaming Replication | Physical block-level replication     |
| Logical Replication   | Replicates database objects and rows |

---

# 1.2 Streaming Replication

Streaming replication is the most common replication method.

It works by continuously sending Write-Ahead Log (WAL) changes from the primary server to replicas.

## WAL Concept

Every change in PostgreSQL is recorded in:

```text
Write Ahead Log (WAL)
```

Replicas replay this log to stay synchronized.

## Streaming Replication Architecture

```text
Application
    |
Primary PostgreSQL
    |
WAL Stream
    |
Standby Server
```

## Setting Up Streaming Replication (Basic Steps)

### Step 1: Configure Primary Server

Edit `postgresql.conf`:

```conf
wal_level = replica
max_wal_senders = 5
wal_keep_size = 1GB
```

### Step 2: Configure Replication Access

Edit `pg_hba.conf`:

```conf
host replication replicator 192.168.1.100/32 md5
```

This allows the replica server to connect.

### Step 3: Create Replication User

```sql
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'password';
```

### Step 4: Create Base Backup

Run on standby server:

```bash
pg_basebackup -h primary_host -D /var/lib/postgresql/data -U replicator -P
```

### Step 5: Start Replica Server

The standby server now starts and begins receiving WAL updates.

---

# 1.3 Logical Replication

Logical replication replicates individual tables and data changes rather than entire database blocks.

It allows:

* partial replication
* cross-version replication
* selective table replication

## Logical Replication Components

| Component    | Purpose                    |
| ------------ | -------------------------- |
| Publisher    | Source database            |
| Subscriber   | Target database            |
| Publication  | Set of tables to replicate |
| Subscription | Connection to publisher    |

## Example Setup

### Create Publication

```sql
CREATE PUBLICATION my_publication
FOR TABLE users, orders;
```

### Create Subscription

```sql
CREATE SUBSCRIPTION my_subscription
CONNECTION 'host=primary dbname=mydb user=replicator password=pass'
PUBLICATION my_publication;
```

Now data changes replicate automatically.

---

# 2. Security in PostgreSQL

Database security ensures:

* only authorized users access data
* sensitive data remains protected
* communication is encrypted

---

# 2.1 Authentication Methods

PostgreSQL supports multiple authentication methods.

| Method        | Description                        | Trust Level                 |
| ------------- | ---------------------------------- | --------------------------- |
| Trust         | Allows connection without password | No authentication required  |
| Password      | Plain password authentication      | Low security                |
| MD5           | Encrypted password authentication  | Moderate security           |
| SCRAM-SHA-256 | Strong authentication              | High security               |
| Peer          | OS user authentication             | Depends on OS user identity |

Recommended for production:

```text
SCRAM-SHA-256
```

---

# 2.2 Host-Based Authentication

Connection permissions are defined in:

```text
pg_hba.conf
```

## Example Entry

```conf
host all all 192.168.1.0/24 md5
```

Meaning:

* allow connections
* from local network
* using password authentication

---

# 2.3 Encryption Support

PostgreSQL supports encrypted connections via:

```text
SSL/TLS
```

Encryption protects:

* credentials
* data in transit

---

# 2.4 Configuring SSL

Edit `postgresql.conf`:

```conf
ssl = on
ssl_cert_file = 'server.crt'
ssl_key_file = 'server.key'
```

Restart PostgreSQL afterward.

---

# 2.5 Managing SSL Connections

Verify SSL usage:

```sql
SELECT * FROM pg_stat_ssl;
```

This shows:

* encrypted sessions
* SSL versions
* cipher usage

---

# 3. Partitioning

Large tables can slow down queries.

Partitioning splits large tables into smaller logical pieces.

---

# 3.1 What is Table Partitioning?

Instead of storing all rows in one table:

```text
orders
```

We split into:

```text
orders_2023
orders_2024
orders_2025
```

PostgreSQL automatically routes data to the correct partition.

---

# 3.2 Partitioning Strategies

| Strategy           | Description              |
| ------------------ | ------------------------ |
| Range Partitioning | Based on value ranges    |
| List Partitioning  | Based on specific values |
| Hash Partitioning  | Based on hash algorithm  |

---

# 3.3 Range Partition Example

## Create Parent Table

```sql
CREATE TABLE orders (
    id INT,
    order_date DATE
) PARTITION BY RANGE (order_date);
```

## Create Partitions

```sql
CREATE TABLE orders_2024
PARTITION OF orders
FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');
```

Now PostgreSQL automatically routes inserts.

---

# 3.4 Managing Partitions

## List Partitions

```sql
SELECT * FROM pg_partitions;
```

## Add New Partition

```sql
CREATE TABLE orders_2026
PARTITION OF orders
FOR VALUES FROM ('2026-01-01') TO ('2027-01-01');
```

---

# 4. PostgreSQL Administration

Database administrators manage system health and resources.

Core tasks include:

* user management
* storage management
* connection control
* performance monitoring

---

# 4.1 Database Maintenance

Routine tasks include:

* running `VACUUM`
* analyzing statistics
* managing indexes
* checking disk usage

Example:

```sql
VACUUM ANALYZE;
```

---

# 4.2 User Management

## Create User

```sql
CREATE USER analyst WITH PASSWORD 'password';
```

## Grant Privileges

```sql
GRANT SELECT ON orders TO analyst;
```

## Remove User

```sql
DROP USER analyst;
```

---

# 4.3 Resource Monitoring

## Monitor Database Size

```sql
SELECT pg_database_size('mydb');
```

## Monitor Table Size

```sql
SELECT pg_total_relation_size('orders');
```

---

# 4.4 Managing Connections

## View Active Connections

```sql
SELECT * FROM pg_stat_activity;
```

## Terminate Connection

```sql
SELECT pg_terminate_backend(pid);
```

---

# 4.5 Managing Storage

Check table storage:

```sql
SELECT relname, pg_size_pretty(pg_total_relation_size(relid))
FROM pg_catalog.pg_statio_user_tables;
```

This helps detect large tables or storage issues.

---

# 5. High Availability and Scaling

Production systems must remain available even if servers fail.

High availability ensures minimal downtime.

---

# 5.1 Failover Systems

Failover automatically promotes a standby server if the primary fails.

## Workflow

```text
Primary fails
     ↓
Standby promoted
     ↓
Application reconnects
```

## Tools Used

* Patroni
* Pgpool-II
* repmgr

---

# 5.2 Load Balancing

Large systems distribute read traffic across replicas.

## Architecture

```text
Application
   |
Load Balancer
   |
Primary (writes)
Replicas (reads)
```

This improves scalability.

---

# 5.3 Database Clustering

Clustering uses multiple nodes to distribute workload.

## Common Approaches

* Citus (distributed PostgreSQL)
* sharding systems
* multi-node clusters

This enables massive horizontal scaling.

---

# 6. Best Practices in PostgreSQL

Production PostgreSQL systems follow several best practices.

---

# 6.1 Designing Efficient Schemas

## Guidelines

* normalize data structures
* avoid unnecessary duplication
* choose appropriate data types

Example:

```text
Use INTEGER instead of TEXT when possible
```

---

# 6.2 Managing Indexes Properly

Indexes improve query speed but consume storage.

## Best Practices

* index frequently queried columns
* avoid too many indexes
* monitor index usage

## Check Unused Indexes

```sql
SELECT * FROM pg_stat_user_indexes;
```

---

# 6.3 Writing Optimized Queries

Always analyze queries using:

```sql
EXPLAIN ANALYZE
```

Avoid:

```sql
SELECT *
```

Prefer:

```sql
SELECT specific_columns
```

---

# 6.4 Securing Database Systems

## Security Recommendations

* enforce strong passwords
* disable unused accounts
* enable SSL encryption
* restrict network access

---

# 6.5 Monitoring Database Health

Regularly monitor:

* CPU usage
* disk space
* query performance
* replication lag

## Important Views

* `pg_stat_activity`
* `pg_stat_replication`
* `pg_stat_database`

---

# Production PostgreSQL Architecture

A modern PostgreSQL deployment may look like:

```text
Clients / Applications
        |
Load Balancer
        |
Primary Database
        |
Streaming Replication
        |
Replica Servers
        |
Backup Storage
```

Monitoring systems track:

* logs
* query performance
* resource usage

---

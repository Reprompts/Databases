# PostgreSQL Hands-On Tutorial
## Chapter 2: Architecture


Understanding PostgreSQL architecture is essential for understanding how data is stored, processed, optimized, and maintained internally. PostgreSQL is designed with a robust process-based architecture, strong memory management, and reliable storage mechanisms that ensure data integrity, concurrency, and high performance.

This article explains PostgreSQL architecture in extremely detailed and practical terms, covering:

* Database cluster
* Database instance
* Server processes
* Client-server communication
* Memory architecture
* Shared buffers
* Write-Ahead Logging (WAL)
* Background processes
* Checkpointer
* Autovacuum

---

# 1. Database Cluster

In PostgreSQL terminology, a database cluster does **not** mean a distributed cluster (like in Hadoop). Instead, it refers to a collection of databases managed by a single PostgreSQL server instance.

A PostgreSQL cluster contains:

* Multiple databases
* Global configuration
* System catalogs
* Shared system tables

All databases inside the cluster share the same:

* Configuration files
* System users
* Server processes
* Storage directory

---

## Structure of a Database Cluster

```text id="g5y8lb"
PostgreSQL Cluster
│
├── Database 1
│     ├── Tables
│     ├── Indexes
│     └── Views
│
├── Database 2
│     ├── Tables
│     ├── Indexes
│     └── Views
│
├── Global Metadata
│
├── WAL Logs
│
└── Configuration Files
```

Each database is logically isolated but stored within the same cluster.

---

## Example

Create two databases:

```sql id="h0rjgm"
CREATE DATABASE shop;
CREATE DATABASE analytics;
```

Both databases belong to the same PostgreSQL cluster.

---

## Cluster Directory

When PostgreSQL is installed, it creates a data directory.

### Typical Location (Linux)

```text id="k1m2cs"
/var/lib/postgresql/data
```

Inside this directory:

```text id="l7fxon"
data/
 ├── base/
 ├── global/
 ├── pg_wal/
 ├── pg_xact/
 ├── pg_stat/
 ├── pg_multixact/
 ├── pg_subtrans/
 ├── pg_tblspc/
 └── pg_commit_ts/
```

These directories contain the internal files for the cluster.

---

# 2. Database Instance

A PostgreSQL instance refers to the running PostgreSQL server process that manages a database cluster.

In simple terms:

```text id="9w2mzd"
Database Cluster = Data on Disk
Database Instance = Running PostgreSQL Server
```

The instance:

* Loads the database cluster
* Manages memory
* Starts background processes
* Accepts client connections
* Executes queries

---

## PostgreSQL Instance Components

```text id="v3q7oe"
PostgreSQL Instance
│
├── Postmaster Process
│
├── Backend Processes
│
├── Background Processes
│
└── Shared Memory
```

The Postmaster is the main controlling process.

---

# 3. PostgreSQL Server Processes

PostgreSQL uses a process-based architecture instead of threads.

This means:

* Each client connection creates a separate backend process
* Processes communicate using shared memory

---

## Major PostgreSQL Processes

```text id="y4lb7x"
Postmaster
   │
   ├── Backend Process (Client 1)
   ├── Backend Process (Client 2)
   ├── Backend Process (Client 3)
   │
   ├── Checkpointer
   ├── WAL Writer
   ├── Background Writer
   ├── Autovacuum Launcher
   └── Stats Collector
```

---

## Postmaster Process

The Postmaster is the main PostgreSQL process.

### Responsibilities

* Start server
* Listen for client connections
* Spawn backend processes
* Manage background workers
* Handle crashes

### Process Example

```text id="0m2rki"
postgres
```

If the Postmaster crashes, the entire instance stops.

---

## Backend Processes

Each client connection creates a backend process.

### Example

User connects with `psql`.

```text id="rn8yxu"
psql → PostgreSQL
```

The server spawns a process:

```text id="o8rjlwm"
backend process
```

### Responsibilities

* Parse SQL queries
* Plan execution
* Execute queries
* Return results

---

## Process Model Example

If 50 users connect:

```text id="9bg7qf"
Postmaster
│
├── Backend 1
├── Backend 2
├── Backend 3
...
├── Backend 50
```

Each backend handles one client.

---

# 4. Client-Server Communication

PostgreSQL follows a client-server architecture.

Applications communicate with PostgreSQL using:

* TCP/IP
* Unix sockets
* Database drivers

---

## Connection Flow

```text id="m0s7pu"
Application
     │
     ▼
Database Driver
     │
     ▼
Network Protocol
     │
     ▼
PostgreSQL Server
     │
     ▼
Backend Process
     │
     ▼
Database Storage
```

---

## Example Application Flow

### Python Example

```python id="l2z8xh"
import psycopg2

conn = psycopg2.connect(
    dbname="shop",
    user="postgres",
    password="password"
)

cur = conn.cursor()

cur.execute("SELECT * FROM products")

rows = cur.fetchall()
```

### Communication Steps

1. Application sends SQL query
2. PostgreSQL parser validates query
3. Query planner generates execution plan
4. Executor retrieves data
5. Results returned to client

---

## PostgreSQL Protocol

PostgreSQL uses a custom wire protocol.

### Communication Steps

* Connection request
* Authentication
* Query messages
* Result messages

---

# 5. PostgreSQL Memory Architecture

PostgreSQL uses shared memory and local memory.

Memory is divided into:

```text id="kw7jvl"
Memory Architecture
│
├── Shared Memory
│     ├── Shared Buffers
│     ├── WAL Buffers
│     ├── Lock Tables
│     └── Background Worker Data
│
└── Local Memory (per backend)
      ├── Work Memory
      ├── Maintenance Memory
      └── Temporary Buffers
```

---

## Shared Memory

Shared memory is used by all server processes.

It stores:

* Cached data
* WAL buffers
* Lock information
* Shared statistics

Shared memory is allocated when PostgreSQL starts.

---

## Local Memory

Each backend process has private memory.

### Examples

* Query execution workspace
* Sorting buffers
* Hash join memory

These are controlled by parameters like:

* `work_mem`
* `maintenance_work_mem`

---

# 6. Shared Buffers

Shared buffers are the most important memory component.

They store cached database pages.

Instead of reading from disk every time, PostgreSQL reads from shared buffers.

---

## How Shared Buffers Work

```text id="s4e7xt"
Client Query
      │
      ▼
Check Shared Buffers
      │
 ┌────┴────┐
 │         │
Hit       Miss
 │         │
Return   Read From Disk
Data      │
          ▼
     Store In Buffer
```

---

## Example

Query:

```sql id="e6zcnr"
SELECT * FROM users WHERE id = 5;
```

### Steps

1. PostgreSQL checks shared buffer
2. If page exists → return quickly
3. If not → read from disk
4. Store page in buffer

---

## Shared Buffer Size

Configuration parameter:

```text id="bnv1tk"
shared_buffers
```

### Example

```text id="gj8qwr"
shared_buffers = 2GB
```

### Typical Recommendation

25–40% of system RAM.

---

# 7. Write Ahead Log (WAL)

Write Ahead Logging (WAL) is PostgreSQL's crash recovery mechanism.

### Rule

Changes must be written to the WAL before writing to disk.

This ensures durability and consistency.

---

## WAL Workflow

```text id="az6vot"
Transaction Begins
       │
       ▼
Data Modified
       │
       ▼
Write Change To WAL
       │
       ▼
WAL Written To Disk
       │
       ▼
Data Page Written Later
```

This allows PostgreSQL to recover from crashes.

---

## WAL Storage

WAL files are stored in:

```text id="3ruo6y"
pg_wal/
```

### Example

```text id="0t9nqc"
pg_wal/
000000010000000000000001
000000010000000000000002
```

---

## Crash Recovery

If PostgreSQL crashes:

1. Restart server
2. Read WAL logs
3. Replay transactions
4. Restore database consistency

---

## WAL and Replication

WAL is also used for:

* Streaming replication
* Point-in-time recovery
* Backups

---

# 8. Background Processes

PostgreSQL uses several background processes to maintain database health.

---

## Key Background Processes

```text id="dw3xqa"
Background Processes
│
├── Checkpointer
├── Background Writer
├── WAL Writer
├── Autovacuum Launcher
├── Autovacuum Workers
└── Stats Collector
```

These processes run automatically.

---

# 9. Checkpointer Process

The Checkpointer is responsible for writing dirty memory pages to disk.

---

## What Is a Dirty Page?

A dirty page is a memory page that:

* Has been modified
* But not yet written to disk

---

## Checkpoint Operation

During a checkpoint:

* Dirty pages flushed to disk
* WAL position recorded
* Checkpoint record written

---

## Checkpoint Flow

```text id="k2x9ws"
Shared Buffers
     │
Dirty Pages
     │
     ▼
Checkpointer
     │
     ▼
Write To Disk
```

---

## Why Checkpoints Are Important

Checkpoints help:

* Reduce crash recovery time
* Maintain data consistency
* Control WAL growth

---

## Configuration

Important settings:

* `checkpoint_timeout`
* `checkpoint_completion_target`

### Example

```text id="f4ev0d"
checkpoint_timeout = 5min
```

---

# 10. Autovacuum Process

PostgreSQL uses MVCC (Multi-Version Concurrency Control).

This creates dead tuples when rows are updated or deleted.

---

## Example

```sql id="c9q2nh"
UPDATE users SET name='Bob';
```

Instead of overwriting the row:

* PostgreSQL creates a new row version
* Old row becomes a dead tuple

---

## Why Dead Tuples Are a Problem

Dead tuples:

* Waste storage
* Slow queries
* Increase table size

---

## Autovacuum Solution

Autovacuum automatically:

* Removes dead tuples
* Frees space
* Updates statistics

---

## Autovacuum Components

```text id="r5w1pl"
Autovacuum System
│
├── Autovacuum Launcher
│
└── Autovacuum Workers
      ├── Vacuum Tables
      ├── Remove Dead Tuples
      └── Update Statistics
```

---

## Vacuum Operation

```text id="z8x3mf"
Table
 │
Dead Rows
 │
 ▼
Vacuum Process
 │
 ▼
Remove Dead Tuples
 │
 ▼
Free Space Reused
```

---

## Example Manual Vacuum

```sql id="7gm9cw"
VACUUM users;
```

Or full cleanup:

```sql id="vc7q4s"
VACUUM FULL users;
```

---

## Autovacuum Configuration

Important parameters:

* `autovacuum = on`
* `autovacuum_max_workers`
* `autovacuum_naptime`

### Example

```text id="3xk2ev"
autovacuum_max_workers = 3
```

---

# Complete PostgreSQL Architecture Overview

Putting everything together:

```text id="5kq7ye"
Client Application
        │
        ▼
   PostgreSQL Server
        │
        ▼
      Postmaster
        │
 ┌──────┼──────────────┐
 │      │              │
 ▼      ▼              ▼
Backend Processes  Background Processes
 │      │              │
 ▼      ▼              ▼
Query Parser      Checkpointer
Query Planner     WAL Writer
Query Executor    Autovacuum
 │
 ▼
Shared Memory
 │
 ├── Shared Buffers
 ├── WAL Buffers
 └── Lock Tables
 │
 ▼
Storage Manager
 │
 ▼
Disk Files
 │
 ├── Tables
 ├── Indexes
 └── WAL Logs
```

---

# Conclusion

PostgreSQL architecture is carefully designed to provide:

* High reliability
* High concurrency
* Efficient memory usage
* Strong crash recovery
* Scalable query execution

Key architectural pillars include:

* Process-based server model
* Shared memory buffers
* Write-Ahead Logging (WAL)
* Background maintenance processes
* Automatic cleanup with Autovacuum

These mechanisms make PostgreSQL one of the most robust database systems in the world.

---


# PostgreSQL Hands-On Tutorial
## Chapter 18: Database Internals & Concurrency 


PostgreSQL is designed for multi-user, high-concurrency environments where many reads and writes happen at the same time. To keep data consistent, fast, and safe, it uses a combination of:

- MVCC (Multi-Version Concurrency Control)
- Locking mechanisms
- Transaction isolation levels
- Deadlock detection
- Maintenance systems like `VACUUM`
- Query planner and optimizer

This article explains how these systems work internally and practically, not just theoretically.

---

# 1. Concurrency Control in PostgreSQL

Concurrency control ensures:

- Multiple users can access the database simultaneously
- No data corruption happens
- Transactions remain consistent

PostgreSQL achieves this mainly through MVCC.

---

# 1.1 Multiversion Concurrency Control (MVCC)

## What is MVCC?

MVCC allows PostgreSQL to keep multiple versions of a row instead of overwriting it immediately.

So instead of:

```sql
UPDATE → overwrite old data
````

PostgreSQL does:

```sql
UPDATE → create new version of row
           keep old version for other transactions
```

## Why MVCC is Powerful

It enables:

* Readers never block writers
* Writers never block readers
* High concurrency performance
* Consistent snapshots of data

## Practical Example

Imagine a bank account:

| Transaction A        | Transaction B         |
| -------------------- | --------------------- |
| Reads balance = 1000 | Updates balance = 500 |
| Still sees 1000      |                       |
| Commit               |                       |

Even after update, Transaction A sees old snapshot, not dirty data.

## Internal Mechanism

Each row contains hidden system columns:

* `xmin` → transaction that created row
* `xmax` → transaction that deleted/updated row

Old rows are not immediately removed.

---

# 1.2 Isolation Levels

Isolation levels define how transactions interact with each other.

PostgreSQL supports:

## 1. READ COMMITTED (Default)

Each query sees:

* only committed data at that moment

### Behavior

Different queries in same transaction may see different results.

### Example

```sql
BEGIN;

SELECT balance FROM accounts;

-- later

SELECT balance FROM accounts;
```

If another transaction commits in between → second query sees updated data.

---

## 2. REPEATABLE READ

Transaction sees a fixed snapshot.

Same result every time inside transaction.

### Example

```sql
BEGIN ISOLATION LEVEL REPEATABLE READ;
```

Even if data changes externally → you won't see it.

---

## 3. SERIALIZABLE (Strictest)

Ensures transactions behave like they ran one after another.

Prevents anomalies completely.

### But

* Higher chance of rollback due to conflicts

---

# 1.3 Locking Mechanisms

Even with MVCC, PostgreSQL still uses locks for coordination.

## Types of Locks

### 1. Row-Level Locks

Lock specific rows:

```sql
SELECT * FROM accounts
WHERE id = 1
FOR UPDATE;
```

Used when:

* updating rows
* preventing concurrent modifications

---

### 2. Table-Level Locks

Lock entire table for operations like:

* `ALTER TABLE`
* bulk updates

## Lock Modes (Important Concept)

* `SHARE`
* `ROW EXCLUSIVE`
* `ACCESS EXCLUSIVE` (strongest)

## Practical Scenario

Two users try:

```sql
UPDATE accounts
SET balance = balance - 100
WHERE id = 1;
```

PostgreSQL:

* locks row
* prevents conflicting updates
* queues second transaction

---

# 1.4 Deadlock Detection

## What is a Deadlock?

A deadlock happens when:

* Transaction A waits for B
* Transaction B waits for A

Neither can proceed.

## Example

| T1              | T2              |
| --------------- | --------------- |
| locks row 1     | locks row 2     |
| waits for row 2 | waits for row 1 |

## PostgreSQL Solution

PostgreSQL:

* automatically detects deadlock
* aborts one transaction
* frees resources

You may see:

```sql
ERROR: deadlock detected
```

---

# 2. VACUUM and ANALYZE (Database Maintenance)

Because of MVCC, PostgreSQL does NOT delete old rows immediately.

This creates bloat.

---

# 2.1 Database Bloat

## What is Bloat?

When old row versions accumulate:

* disk usage increases
* queries slow down
* index efficiency drops

## Why It Happens

Because:

```sql
UPDATE = new row created
DELETE = row marked dead (not removed instantly)
```

---

# 2.2 VACUUM

## What VACUUM Does

* removes dead tuples
* frees storage space internally
* improves performance

## Basic Command

```sql
VACUUM;
```

## Full Cleanup

```sql
VACUUM FULL;
```

> ⚠️ Note:
>
> * locks table
> * slower but compacts disk fully

## When to Use

* large updates/deletes
* performance degradation
* scheduled maintenance

---

# 2.3 ANALYZE

## What ANALYZE Does

* collects statistics about tables
* helps query planner choose best strategy

## Command

```sql
ANALYZE;
```

## Why It Matters

Without statistics:

* PostgreSQL may choose bad query plans
* joins become slow
* indexes may not be used

## Example

If table has:

* 1,000 rows → `ANALYZE` sees full scan OK
* 10 million rows → `ANALYZE` suggests index scan

---

# 2.4 AUTOVACUUM

## What is It?

Background process that automatically runs:

* `VACUUM`
* `ANALYZE`

## Why It Is Important

Without autovacuum:

* database bloats quickly
* manual maintenance required

## Configuration Idea

In `postgresql.conf`:

```conf
autovacuum = on
autovacuum_vacuum_scale_factor
autovacuum_analyze_scale_factor
```

## Practical Insight

High-write systems like:

* e-commerce
* banking
* logging systems

rely heavily on autovacuum tuning.

---

# 3. Performance Optimization (Core Concepts)

This is where PostgreSQL decides:

> "How should I execute this query efficiently?"

---

# 3.1 Query Planning

Before executing SQL, PostgreSQL:

1. Parses query
2. Rewrites it
3. Creates execution plan
4. Picks best strategy

## Example

```sql
SELECT * FROM users
WHERE age > 30;
```

Planner decides:

* sequential scan
  OR
* index scan

---

# 3.2 Execution Plans

Execution plan = step-by-step breakdown of query execution.

## Example

```text
Seq Scan on users
  Filter: age > 30
```

OR

```text
Index Scan using idx_users_age
```

## Why It Matters

Execution plan tells:

* why query is slow
* whether index is used
* how joins are performed

---

# 3.3 Using EXPLAIN

## Basic Usage

```sql
EXPLAIN
SELECT * FROM users
WHERE age > 30;
```

## Detailed Analysis

```sql
EXPLAIN ANALYZE
SELECT * FROM users
WHERE age > 30;
```

This shows:

* actual execution time
* rows processed
* plan accuracy

## Practical Interpretation

| Output      | Meaning                                 |
| ----------- | --------------------------------------- |
| Seq Scan    | Full table scan (slow for large tables) |
| Index Scan  | Optimized lookup                        |
| Nested Loop | Simple join                             |
| Hash Join   | Large dataset join                      |

---

# 3.4 Query Optimization Strategies

## 1. Use Indexes Properly

```sql
CREATE INDEX idx_users_age
ON users(age);
```

---

## 2. Avoid `SELECT *`

### Bad

```sql
SELECT * FROM users;
```

### Better

```sql
SELECT id, name FROM users;
```

---

## 3. Filter Early

```sql
SELECT *
FROM orders
WHERE status = 'paid';
```

---

## 4. Optimize Joins

* ensure indexed foreign keys
* avoid unnecessary joins

---

## 5. Keep Statistics Updated

```sql
ANALYZE;
```

---

## 6. Avoid Unnecessary Full Table Scans

If `EXPLAIN` shows:

```text
Seq Scan
```

→ consider indexing.

---

# Final Mental Model (Very Important)

Think of PostgreSQL like this:

## 1. MVCC

> "I never overwrite data, I create versions"

## 2. Locks

> "I coordinate conflicting operations safely"

## 3. VACUUM

> "I clean up old invisible data"

## 4. ANALYZE

> "I study my data to choose better plans"

## 5. Query Planner

> "I decide the fastest way to run your query"

---

# Summary

This system works together:

* MVCC → concurrency without blocking
* Locks → safety for conflicting operations
* Deadlocks → automatic recovery
* VACUUM → cleanup engine
* ANALYZE → intelligence system
* EXPLAIN → debugging tool
* Query planner → execution decision maker

```

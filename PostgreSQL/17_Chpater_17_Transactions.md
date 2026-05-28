# PostgreSQL Hands-On Tutorial
## Chapter 17: Transactions

Transactions are one of the most important concepts in PostgreSQL and all relational databases. They ensure that multiple database operations behave as a single, reliable unit of work.

In real-world systems, transactions are used everywhere:

* banking transfers
* online payments
* order processing
* inventory updates
* booking systems

Without transactions, systems would suffer from:

* partial updates
* inconsistent data
* corrupted records
* race conditions

PostgreSQL transactions are built on ACID properties, which guarantee reliability.

This guide explains transactions in extreme practical detail, including:

* ACID properties
* transaction lifecycle
* `BEGIN`, `COMMIT`, `ROLLBACK`
* savepoints
* real-world examples

---

# 1. What Is a Transaction?

A transaction is a group of SQL operations that execute as a single unit.

Either:

* ALL operations succeed → `COMMIT`

OR

* ANY operation fails → `ROLLBACK`

## Example Problem Without Transactions

Imagine a bank transfer:

### Step 1

Deduct ₹1000 from Account A

### Step 2

Add ₹1000 to Account B

If Step 1 succeeds but Step 2 fails:

```text
Money disappears → system inconsistency
```

## With Transactions

Both steps are treated as one unit:

```text
Either both succeed or both fail
```

---

# 2. ACID Properties

PostgreSQL transactions follow ACID principles.

---

# A — Atomicity

A transaction is all or nothing.

## Example

Transfer money:

* debit account A
* credit account B

If one fails:

```text
entire transaction is cancelled
```

---

# C — Consistency

Database always moves from one valid state to another.

Rules such as constraints and triggers are never violated.

## Example

```text
account balance cannot become negative
```

---

# I — Isolation

Transactions execute independently.

Even if multiple users run transactions:

```text
each transaction behaves as if it is alone
```

---

# D — Durability

Once committed:

```text
data is permanently saved
```

Even during system crashes.

---

# 3. Transaction Lifecycle

A PostgreSQL transaction goes through these stages.

## 1. Begin

Transaction starts.

```sql id="5i0u5g"
BEGIN;
```

---

## 2. Execution Phase

SQL operations run:

* `INSERT`
* `UPDATE`
* `DELETE`

---

## 3. Commit or Rollback Decision

* `COMMIT` → save changes
* `ROLLBACK` → undo changes

---

## 4. End State

Transaction completes.

---

## Lifecycle Diagram

```text id="m8h1jw"
BEGIN
  ↓
SQL Operations
  ↓
SUCCESS / FAILURE
  ↓
COMMIT OR ROLLBACK
  ↓
END
```

---

# 4. Beginning a Transaction

To start a transaction:

```sql id="jwq4x8"
BEGIN;
```

or:

```sql id="wx20ny"
START TRANSACTION;
```

Both are equivalent.

## Example

```sql id="wwy6pq"
BEGIN;

UPDATE accounts
SET balance = balance - 500
WHERE id = 1;
```

At this point:

```text
changes are NOT permanent yet
```

---

# 5. Committing a Transaction

Commit means:

```text
save all changes permanently
```

## Syntax

```sql id="9y2lq4"
COMMIT;
```

## Example

```sql id="o9kpqa"
BEGIN;

UPDATE accounts
SET balance = balance - 500
WHERE id = 1;

UPDATE accounts
SET balance = balance + 500
WHERE id = 2;

COMMIT;
```

After `COMMIT`:

```text
changes are permanently stored
```

---

# 6. Rolling Back a Transaction

Rollback means:

```text
undo all changes in the transaction
```

## Syntax

```sql id="y3zodl"
ROLLBACK;
```

## Example

```sql id="3prf7f"
BEGIN;

UPDATE accounts
SET balance = balance - 500
WHERE id = 1;

-- error occurs

ROLLBACK;
```

## Result

```text
no changes are saved
```

---

# 7. Savepoints

A savepoint allows partial rollback inside a transaction.

It creates checkpoints within a transaction.

## Syntax

```sql id="mwj4x2"
SAVEPOINT savepoint_name;
```

## Example

```sql id="ncljvu"
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

SAVEPOINT step1;

UPDATE accounts
SET balance = balance - 200
WHERE id = 2;

ROLLBACK TO step1;

COMMIT;
```

## What Happens Here

* First update succeeds
* Savepoint created
* Second update executed
* Rollback only second update
* First update remains
* Transaction committed

---

## Savepoint Diagram

```text id="3fx49k"
BEGIN
  ↓
Step 1 (OK)
  ↓
SAVEPOINT
  ↓
Step 2 (FAIL)
  ↓
ROLLBACK TO SAVEPOINT
  ↓
COMMIT
```

---

# 8. Real-World Example — Bank Transfer

## Table

```sql id="t3u7ak"
CREATE TABLE accounts (
    id INT,
    balance NUMERIC
);
```

## Transaction Example

```sql id="9i2xhz"
BEGIN;

-- Deduct from sender
UPDATE accounts
SET balance = balance - 1000
WHERE id = 1;

-- Add to receiver
UPDATE accounts
SET balance = balance + 1000
WHERE id = 2;

COMMIT;
```

## If Something Fails

```sql id="g13q4r"
BEGIN;

UPDATE accounts
SET balance = balance - 1000
WHERE id = 1;

-- system crash or error

ROLLBACK;
```

## Result

```text
no money lost
```

---

# 9. Transactions with Savepoints (Advanced Banking Example)

```sql id="w2m9sj"
BEGIN;

UPDATE accounts
SET balance = balance - 500
WHERE id = 1;

SAVEPOINT after_debit;

UPDATE accounts
SET balance = balance + 500
WHERE id = 2;

-- error occurs here

ROLLBACK TO after_debit;

UPDATE accounts
SET balance = balance + 500
WHERE id = 3;

COMMIT;
```

---

# 10. Transaction Isolation Levels

PostgreSQL supports different isolation levels:

* `READ COMMITTED` (default)
* `REPEATABLE READ`
* `SERIALIZABLE`

These control how transactions see each other's changes.

## Example Problem

Two users updating the same data simultaneously.

Without isolation:

```text
data corruption may occur
```

With isolation:

```text
transactions behave safely
```

---

# 11. Transaction Behavior in PostgreSQL (MVCC)

PostgreSQL uses MVCC (Multi-Version Concurrency Control).

Instead of overwriting data:

```text
it creates new row versions
```

This allows:

* concurrent reads
* non-blocking reads
* consistent snapshots

---

# 12. Common Transaction Patterns

---

# Pattern 1 — Safe Update

```sql id="2m7slx"
BEGIN;

UPDATE inventory
SET stock = stock - 1
WHERE product_id = 10;

COMMIT;
```

---

# Pattern 2 — Conditional Rollback

```sql id="59e6nf"
BEGIN;

UPDATE accounts
SET balance = balance - 100;

IF balance < 0 THEN
    ROLLBACK;
END IF;

COMMIT;
```

*(Usually handled in application logic.)*

---

# Pattern 3 — Multi-Step Workflow

```sql id="slj8dl"
BEGIN;

INSERT INTO orders VALUES (...);

INSERT INTO order_items VALUES (...);

UPDATE inventory
SET stock = stock - 1;

COMMIT;
```

---

# 13. Common Mistakes

## 1. Forgetting COMMIT

Changes are not saved.

---

## 2. Not Using ROLLBACK

Partial failures can cause inconsistency.

---

## 3. Long-Running Transactions

Can cause:

* locking issues
* performance degradation

---

## 4. Mixing Transaction Logic Incorrectly

Each transaction should represent one logical unit of work.

---

# 14. Performance Considerations

* keep transactions short
* avoid unnecessary operations inside transactions
* reduce lock time
* batch operations when possible

---

# 15. Summary

Transactions ensure data reliability, consistency, and safety in PostgreSQL.

---

# Key Concepts

| Concept   | Meaning           |
| --------- | ----------------- |
| BEGIN     | Start transaction |
| COMMIT    | Save changes      |
| ROLLBACK  | Undo changes      |
| SAVEPOINT | Partial rollback  |

---

# ACID Properties

| Property    | Meaning               |
| ----------- | --------------------- |
| Atomicity   | All or nothing        |
| Consistency | Valid state always    |
| Isolation   | Independent execution |
| Durability  | Permanent storage     |

---

# Core Idea

```text
Transactions protect your data from partial failure.
```

---

# Conclusion

Transactions are the foundation of reliable database systems. They ensure that even in complex operations like banking, e-commerce, or logistics, data remains consistent and safe.


# PostgreSQL Hands-On Tutorial
## Chapter 19: Server-Side Programming 

PostgreSQL is not just a database, it is also a server-side programming platform. This means you can write business logic directly inside the database, reducing application complexity and improving performance.

This layer is built using:

- Functions
- Stored Procedures
- PL/pgSQL language
- Triggers

These allow PostgreSQL to behave like a mini backend engine inside the database itself.

---

# 1. Why Server-Side Programming Matters

Instead of writing logic in your application (Python/Java/Node.js), you can push logic closer to the data.

## Benefits

- Faster execution (no network round trips)
- Centralized business logic
- Data integrity enforcement
- Reusable database logic
- Automatic execution via triggers

---

# 2. PL/pgSQL (PostgreSQL Procedural Language)

## What is PL/pgSQL?

It is PostgreSQL's built-in procedural language used to write:

- Functions
- Stored procedures
- Trigger logic

It supports:

- Variables
- Loops
- Conditions
- Exceptions

## Basic Structure

```sql
CREATE FUNCTION function_name()
RETURNS return_type AS $$
BEGIN
    -- logic here
END;
$$ LANGUAGE plpgsql;
````

---

# 3. Stored Functions in PostgreSQL

---

# 3.1 What is a Function?

A function:

* Returns a value
* Can be used inside SQL queries
* Is immutable or stable depending on use case

---

# 3.2 Creating Functions

## Example: Simple Function

```sql
CREATE FUNCTION add_numbers(a INT, b INT)
RETURNS INT AS $$
BEGIN
    RETURN a + b;
END;
$$ LANGUAGE plpgsql;
```

---

# 3.3 Calling Functions

## Method 1: Direct Call

```sql
SELECT add_numbers(10, 20);
```

### Output

```text
30
```

---

## Method 2: Inside Query

```sql
SELECT name, add_numbers(score1, score2)
FROM students;
```

---

# 3.4 Function with Variables

```sql
CREATE FUNCTION get_discount(price NUMERIC)
RETURNS NUMERIC AS $$
DECLARE
    discount NUMERIC;
BEGIN
    discount := price * 0.10;
    RETURN discount;
END;
$$ LANGUAGE plpgsql;
```

---

# 3.5 Function with Conditions

```sql
CREATE FUNCTION grade(marks INT)
RETURNS TEXT AS $$
BEGIN
    IF marks >= 90 THEN
        RETURN 'A';
    ELSIF marks >= 75 THEN
        RETURN 'B';
    ELSE
        RETURN 'C';
    END IF;
END;
$$ LANGUAGE plpgsql;
```

---

# 3.6 Dropping Functions

```sql
DROP FUNCTION add_numbers(INT, INT);
```

> ⚠️ Must match parameter types exactly.

---

# 4. Stored Procedures (Advanced Server-Side Logic)

---

# 4.1 What is a Stored Procedure?

A stored procedure:

* Does NOT necessarily return a value
* Can perform transactions
* Can commit or rollback internally
* Is executed using `CALL`

---

# 4.2 Creating Stored Procedures

```sql
CREATE PROCEDURE transfer_money(
    sender_id INT,
    receiver_id INT,
    amount NUMERIC
)
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE accounts
    SET balance = balance - amount
    WHERE id = sender_id;

    UPDATE accounts
    SET balance = balance + amount
    WHERE id = receiver_id;

    COMMIT;
END;
$$;
```

---

# 4.3 Executing Procedures

```sql
CALL transfer_money(1, 2, 500);
```

---

# 4.4 Function vs Procedure

| Feature             | Function | Procedure |
| ------------------- | -------- | --------- |
| Returns value       | Yes      | Optional  |
| Used in SELECT      | Yes      | No        |
| Transaction control | No       | Yes       |
| CALL support        | No       | Yes       |

---

# 5. Triggers in PostgreSQL

---

# 5.1 What is a Trigger?

A trigger is an automatic function that runs when an event occurs.

## Events Include

* `INSERT`
* `UPDATE`
* `DELETE`

---

# 5.2 Why Triggers are Powerful

They enforce:

* auditing
* validation
* automatic updates
* logging
* business rules

---

# 6. Trigger Events

Triggers fire on:

| Event  | Meaning       |
| ------ | ------------- |
| INSERT | New row added |
| UPDATE | Row modified  |
| DELETE | Row removed   |

---

# 7. Row-Level vs Statement-Level Triggers

---

# 7.1 Row-Level Trigger

Runs once per row.

### Example

If 100 rows updated → trigger runs 100 times.

---

# 7.2 Statement-Level Trigger

Runs once per SQL statement.

### Example

```sql
UPDATE employees
SET salary = salary + 10;
```

Trigger runs only once, even if 1000 rows affected.

---

# 8. Creating Triggers (Step-by-Step)

Triggers require 2 parts:

---

## Step 1: Trigger Function

```sql
CREATE FUNCTION log_salary_change()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO audit_log(emp_id, old_salary, new_salary)
    VALUES (OLD.id, OLD.salary, NEW.salary);

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

---

## Step 2: Create Trigger

```sql
CREATE TRIGGER salary_update_trigger
AFTER UPDATE ON employees
FOR EACH ROW
EXECUTE FUNCTION log_salary_change();
```

## How It Works Internally

1. UPDATE happens
2. Trigger fires
3. `OLD` and `NEW` rows available
4. Logic executes
5. Audit inserted

---

# 9. Trigger Types

---

# 9.1 BEFORE Trigger

Runs before operation.

## Used For

* validation
* modifying data before insert/update

---

# 9.2 AFTER Trigger

Runs after operation.

## Used For

* logging
* auditing
* syncing data

---

# 9.3 INSTEAD OF Trigger

Used with views:

* replaces operation completely

---

# 10. Updating Triggers

You cannot directly modify a trigger.

Instead:

## Step 1: Drop

```sql
DROP TRIGGER salary_update_trigger ON employees;
```

---

## Step 2: Recreate

```sql
CREATE TRIGGER salary_update_trigger
AFTER UPDATE ON employees
FOR EACH ROW
EXECUTE FUNCTION log_salary_change();
```

---

# 11. Dropping Triggers

```sql
DROP TRIGGER trigger_name ON table_name;
```

---

# 12. Advanced Practical Use Cases

---

# 12.1 Audit System (Most Common)

Track changes automatically:

* salary changes
* user updates
* order modifications

---

# 12.2 Data Validation

Prevent invalid data:

```sql
IF NEW.salary < 0 THEN
    RAISE EXCEPTION 'Invalid salary';
END IF;
```

---

# 12.3 Automatic Timestamping

```sql
NEW.updated_at := NOW();

RETURN NEW;
```

---

# 12.4 Business Rules Enforcement

## Example

* prevent deleting VIP customers
* enforce stock limits
* auto-calculate discounts

---

# 13. Real-World Architecture View

When using server-side programming:

## Application Layer

* sends request
* minimal logic

## Database Layer

* functions handle computation
* procedures handle workflows
* triggers enforce rules automatically

---

# 14. Mental Model (Very Important)

Think of PostgreSQL as:

## Functions → "Calculators"

Return values when called.

---

## Procedures → "Workers"

Perform tasks step-by-step.

---

## Triggers → "Autopilots"

React automatically to events.

---

# 15. Best Practices

## 1. Keep Functions Small

Avoid complex logic explosion.

---

## 2. Use Triggers Carefully

Too many triggers = hidden complexity.

---

## 3. Avoid Heavy Computation in Triggers

Triggers slow down write operations.

---

## 4. Prefer Functions for Reusable Logic

Procedures for workflows only.

---

## 5. Always Test with EXPLAIN + Logging

Understand impact on performance.

---

# Final Summary

PostgreSQL server-side programming allows you to:

* Run business logic inside the database
* Automate actions using triggers
* Build reusable functions
* Execute complex workflows using procedures

## Core Flow

```text
SQL Query
    ↓
Function
    ↓
Procedure
    ↓
Trigger
    ↓
Data Change
    ↓
Audit/Action
```

If you want next, I can turn this into:

* a real project (banking system using procedures + triggers)
* a visual flow diagram of triggers and execution lifecycle
* a hands-on SQL lab with exercises and outputs


```

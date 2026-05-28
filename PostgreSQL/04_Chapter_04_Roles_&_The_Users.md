# PostgreSQL Hands-On Tutorial 

## Chapter 4: Roles & the Users 

PostgreSQL provides a powerful role-based access control (RBAC) system for managing authentication, authorization, and permissions. Instead of managing users and permissions separately, PostgreSQL uses a unified concept called **roles**.

Roles control:

* Who can access the database
* What actions they can perform
* What objects they can access
* What administrative privileges they have

This article explains PostgreSQL users and roles in extremely detailed practical depth, including:

* User vs role concepts
* Role attributes
* Login roles
* Superuser roles
* Creating users and roles
* Listing roles
* Managing passwords
* Granting login privileges
* Assigning roles to users
* Dropping users and roles

---

# 1. User vs Role in PostgreSQL

In PostgreSQL, the terms **user** and **role** are closely related.

Historically, PostgreSQL had two separate entities:

* Users
* Groups

Later versions unified them into a single concept:

```text id="m8k2xa"
Roles
```

---

## Role Concept

A role is an entity that can:

* Own database objects
* Have privileges
* Connect to databases
* Inherit permissions from other roles

A role may act as:

* A user
* A group
* Both

---

## User vs Role

In PostgreSQL:

| Concept | Meaning                      |
| ------- | ---------------------------- |
| User    | Role that can login          |
| Role    | Generic permission container |
| Group   | Role used to group users     |

---

## Example

```text id="v7d1qo"
Role: developers
Role: analysts

User: alice
User: bob
```

Users can inherit permissions:

```text id="n3w6rf"
alice → developers
bob → analysts
```

---

## Visual Model

```text id="q2z8kt"
Roles System
        Roles
       /     \
   Users     Groups
   alice     developers
   bob       analysts
```

Users are simply roles with `LOGIN` capability.

---

# 2. Role Attributes

Roles can have attributes that control their behavior and privileges.

These attributes determine what the role can do.

---

## Common Role Attributes

| Attribute   | Meaning                   |
| ----------- | ------------------------- |
| LOGIN       | Allows login              |
| SUPERUSER   | Full privileges           |
| CREATEDB    | Can create databases      |
| CREATEROLE  | Can create roles          |
| INHERIT     | Inherits privileges       |
| REPLICATION | Used for replication      |
| BYPASSRLS   | Bypass row-level security |

---

## Example Role with Attributes

```sql id="w4r7ny"
CREATE ROLE admin
WITH
LOGIN
CREATEDB
CREATEROLE;
```

This role can:

* Login
* Create databases
* Create roles

---

## Viewing Role Attributes

### Query

```sql id="j6m9pc"
SELECT rolname, rolsuper, rolcreatedb, rolcreaterole
FROM pg_roles;
```

### Example Output

```text id="e1t4vb"
rolname   | rolsuper | rolcreatedb | rolcreaterole
----------+----------+-------------+---------------
postgres  | true     | true        | true
app_user  | false    | false       | false
admin     | false    | true        | true
```

---

# 3. Login Roles

A login role is a role that can authenticate and connect to PostgreSQL.

Without `LOGIN` privilege, a role cannot connect.

---

## Creating Login Role

```sql id="x5u8lh"
CREATE ROLE alice LOGIN;
```

Now Alice can login to PostgreSQL.

---

## Example

```text id="d7p3zw"
Role: alice
LOGIN: yes
```

### Connection Example

```bash id="c4n2yr"
psql -U alice -d shop
```

---

## Role Without Login

Roles can exist without login capability.

### Example

```sql id="o9v6fa"
CREATE ROLE developers;
```

This role acts as a group role.

---

## Example Structure

```text id="f3k7mu"
developers (role)
   |
   ├── alice
   └── bob
```

---

# 4. Superuser Roles

A superuser has unrestricted access to PostgreSQL.

Superusers can:

* Access all databases
* Modify system catalogs
* Bypass permissions
* Manage users
* Execute administrative commands

---

## Default Superuser

When PostgreSQL is installed, a superuser is created:

```text id="g8x5cn"
postgres
```

---

## Creating Superuser

```sql id="s2m9we"
CREATE ROLE admin
WITH
LOGIN
SUPERUSER;
```

---

## Checking Superusers

```sql id="h7r1dq"
SELECT rolname, rolsuper
FROM pg_roles;
```

### Example Output

```text id="t5z8kv"
rolname  | rolsuper
---------+----------
postgres | true
admin    | true
alice    | false
```

---

## Security Recommendation

Superuser access should be limited.

### Best Practice

* Application users → normal roles
* Administrators → superusers

---

# 5. Creating Users

PostgreSQL provides a convenient command:

```text id="k2u4xp"
CREATE USER
```

Internally, it is equivalent to:

```text id="m9q7lh"
CREATE ROLE WITH LOGIN
```

---

## Basic Syntax

```sql id="z1v8oc"
CREATE USER username;
```

### Example

```sql id="b6n3yd"
CREATE USER alice;
```

---

## Creating User with Password

```sql id="r4m7ke"
CREATE USER alice
WITH PASSWORD 'secure_password';
```

---

## Creating User with Privileges

### Example

```sql id="w8f2tp"
CREATE USER manager
WITH
PASSWORD 'pass123'
CREATEDB
CREATEROLE;
```

This user can:

* Create databases
* Create roles

---

# 6. Creating Roles

Roles are created using `CREATE ROLE`.

---

## Syntax

```sql id="p3x9wr"
CREATE ROLE role_name;
```

### Example

```sql id="n5k1vh"
CREATE ROLE developers;
```

This role cannot login.

It acts as a group role.

---

## Role Example

```text id="u7r4cz"
Role: developers
Purpose: group developers
```

Later users can inherit this role.

---

# 7. Listing Users and Roles

PostgreSQL stores role information in system catalogs.

---

## Method 1: psql Command

Inside `psql`:

```sql id="d2w6mo"
\du
```

### Example Output

```text id="a8q5ls"
Role name | Attributes
----------+-------------------------
postgres  | Superuser, Create role
alice     | Login
developers| No login
```

---

## Method 2: SQL Query

### Query

```sql id="v1m8ye"
SELECT rolname
FROM pg_roles;
```

### Output

```text id="q7k3ru"
rolname
---------
postgres
alice
developers
```

---

## Detailed Role Information

```sql id="f9x2ld"
SELECT *
FROM pg_roles;
```

### Important Fields

| Column        | Meaning           |
| ------------- | ----------------- |
| rolname       | Role name         |
| rolsuper      | Superuser         |
| rolcreatedb   | Database creation |
| rolcreaterole | Role creation     |
| rolcanlogin   | Login ability     |

---

# 8. Altering User Attributes

User attributes can be modified using `ALTER ROLE`.

---

## Syntax

```sql id="g4w9pn"
ALTER ROLE role_name
WITH attribute;
```

---

## Example: Grant Database Creation

```sql id="s8k2yv"
ALTER ROLE alice
CREATEDB;
```

Now Alice can create databases.

---

## Remove Privilege

```sql id="m6q7tx"
ALTER ROLE alice
NOCREATEDB;
```

---

## Multiple Attributes

```sql id="r1v4fe"
ALTER ROLE alice
WITH
CREATEDB
CREATEROLE;
```

---

# 9. Setting Passwords

Passwords are required for authentication.

---

## Setting Password During Creation

```sql id="p8n3cz"
CREATE USER alice
WITH PASSWORD 'mypassword';
```

---

## Changing Password

```sql id="k5r9xb"
ALTER USER alice
WITH PASSWORD 'newpassword';
```

---

## Password Encryption

PostgreSQL stores passwords using encryption methods.

Modern versions use:

```text id="u2m7qo"
SCRAM-SHA-256
```

This improves security.

---

## Example

### Check Password Method

```sql id="x4w1ve"
SHOW password_encryption;
```

### Output

```text id="n8k6ts"
scram-sha-256
```

---

# 10. Granting Login Privileges

Roles without `LOGIN` cannot connect.

You can enable login using `ALTER ROLE`.

---

## Grant Login

```sql id="b7f2pr"
ALTER ROLE developers
LOGIN;
```

Now `developers` can login.

---

## Remove Login

```sql id="d9m4kx"
ALTER ROLE alice
NOLOGIN;
```

This disables login ability.

---

# 11. Assigning Roles to Users

Roles can be assigned to other roles.

This enables role inheritance.

---

## Syntax

```sql id="q3v8yn"
GRANT role_name TO user_name;
```

---

## Example

### Create Role

```sql id="w1k5cf"
CREATE ROLE developers;
```

### Create User

```sql id="m8t2zl"
CREATE USER alice;
```

### Assign Role

```sql id="x6r4po"
GRANT developers TO alice;
```

---

## Result

`alice` inherits privileges from `developers`.

---

## Visual Representation

```text id="h2n7vd"
developers (role)
     |
     ▼
   alice (user)
```

---

## Removing Role Assignment

```sql id="c9p5xa"
REVOKE developers FROM alice;
```

Now Alice no longer inherits permissions.

---

# 12. Dropping Users

Users can be removed using `DROP USER`.

---

## Syntax

```sql id="j7r2mk"
DROP USER username;
```

### Example

```sql id="v4x8te"
DROP USER alice;
```

---

## Important Restriction

A user cannot be dropped if they:

* Own objects
* Own databases
* Have active connections

---

## Check Owned Objects

### Query

```sql id="u6n1yp"
SELECT *
FROM pg_tables
WHERE tableowner = 'alice';
```

---

## Remove Ownership

### Example

```sql id="f3q9rw"
REASSIGN OWNED BY alice TO postgres;
```

Then drop the user.

---

# 13. Dropping Roles

Roles can also be removed.

---

## Syntax

```sql id="r8w4kv"
DROP ROLE role_name;
```

### Example

```sql id="t1m6zo"
DROP ROLE developers;
```

---

## Safe Drop

Use:

```sql id="k9v2py"
DROP ROLE IF EXISTS developers;
```

---

## Internal Behavior

Dropping a role:

* Removes catalog entry
* Revokes privileges
* Deletes role metadata

---

# 14. Practical Example Workflow

Real-world PostgreSQL role setup.

---

## Step 1: Create Roles

```sql id="n2x5wr"
CREATE ROLE developers;
CREATE ROLE analysts;
```

---

## Step 2: Create Users

```sql id="p7m1dv"
CREATE USER alice WITH PASSWORD 'pass1';
CREATE USER bob WITH PASSWORD 'pass2';
```

---

## Step 3: Assign Roles

```sql id="s4q8yt"
GRANT developers TO alice;
GRANT analysts TO bob;
```

---

## Step 4: Grant Database Access

```sql id="e5w3xn"
GRANT CONNECT ON DATABASE shop TO developers;
```

---

## Final Structure

```text id="z8v6lc"
Roles System

developers
   |
   └── alice

analysts
   |
   └── bob
```

---

# 15. Best Practices for Roles and Users

Professional PostgreSQL systems follow these principles.

---

## Use Role-Based Permissions

Instead of granting permissions directly to users:

* Grant → role
* Assign → user

---

## Example

### Bad

```sql id="m1r7qv"
GRANT SELECT TO alice;
GRANT SELECT TO bob;
```

### Good

```sql id="f8v2pk"
CREATE ROLE readers;

GRANT SELECT TO readers;

GRANT readers TO alice;
GRANT readers TO bob;
```

---

## Limit Superusers

Only administrators should be superusers.

---

## Use Strong Passwords

### Example

```text id="x5n8tc"
SCRAM-SHA-256
```

---

## Separate Application Roles

### Example

* `app_reader`
* `app_writer`
* `app_admin`

---

# PostgreSQL Roles System Architecture

Complete overview:

```text id="q4m9wy"
PostgreSQL Role System
         Roles
           │
           ▼
   ┌───────────────┐
   │   Attributes  │
   │ LOGIN         │
   │ SUPERUSER     │
   │ CREATEDB      │
   │ CREATEROLE    │
   └───────────────┘
           │
           ▼
   Role Relationships
           │
           ▼
   ┌───────────────┐
   │ Role Inherits │
   │ Privileges    │
   └───────────────┘
           │
           ▼
        Database
           │
           ▼
      Tables / Objects
```

---

# Conclusion

PostgreSQL's role-based access control system provides a flexible and powerful way to manage authentication and permissions.

Key concepts include:

* Roles as unified entities
* Login roles for authentication
* Superuser roles for administration
* Role inheritance for permission management
* Secure password handling

Operations such as:

* Creating users and roles
* Assigning roles
* Altering privileges
* Managing login access
* Dropping roles

allow administrators to securely manage database access in large-scale systems.

---

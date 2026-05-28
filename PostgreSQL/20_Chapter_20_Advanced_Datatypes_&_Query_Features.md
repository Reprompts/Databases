
# PostgreSQL Hands-On Tutorial
## Chapter 20: Advanced Datatypes & Query Features

PostgreSQL is known for being an advanced object-relational database, not just a traditional relational database. It supports several powerful data types and query capabilities that allow you to store and process complex data structures directly inside the database.

These include:

- Sequences and auto-increment columns
- Full-text search
- JSON and JSONB data handling
- Arrays

These features allow PostgreSQL to handle use cases like:

- document storage
- search engines
- analytics
- flexible schemas
- complex structured data

This article explains how these features work internally and how to use them in real applications.

---

# 1. Sequences and Auto Increment

## What is a Sequence?

A sequence is a special PostgreSQL object that generates unique numeric values automatically.

It is commonly used for:

- Primary keys
- Invoice numbers
- Order IDs
- Ticket numbers

Sequences guarantee that each generated number is unique and sequential.

---

## 1.1 Auto Increment Columns

PostgreSQL supports automatic numbering using:

- `SERIAL`
- `BIGSERIAL`
- `SMALLSERIAL`

These are shortcuts that automatically create a sequence behind the scenes.

### Example Table with Auto Increment

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT,
    email TEXT
);
````

When you insert rows:

```sql
INSERT INTO users(name, email)
VALUES ('Ganesh', 'ganesh@example.com');
```

PostgreSQL automatically assigns:

```text
id = 1
```

Next insert:

```text
id = 2
```

---

## 1.2 How SERIAL Works Internally

When you write:

```sql
SERIAL
```

PostgreSQL actually performs:

1. Create sequence
2. Set default value using `nextval()`

Equivalent SQL:

```sql
CREATE SEQUENCE users_id_seq;

CREATE TABLE users (
    id INT DEFAULT nextval('users_id_seq'),
    name TEXT
);
```

---

## 1.3 Creating Sequences Manually

You can create sequences directly.

```sql
CREATE SEQUENCE order_seq
START 1000
INCREMENT 1;
```

Meaning:

* First value = 1000
* Next values = 1001, 1002, 1003...

---

## 1.4 Using Sequences

### Generate Next Value

```sql
SELECT nextval('order_seq');
```

Example output:

```text
1000
1001
1002
```

### Get Current Value

```sql
SELECT currval('order_seq');
```

### Use Sequence in Table

```sql
CREATE TABLE orders (
    id INT DEFAULT nextval('order_seq'),
    product TEXT
);
```

---

## 1.5 Resetting Sequences

Sometimes sequences need resetting after data import.

Example:

```sql
ALTER SEQUENCE order_seq RESTART WITH 1;
```

Or:

```sql
SELECT setval('order_seq', 500);
```

Next value becomes:

```text
501
```

---

## 1.6 Dropping Sequences

```sql
DROP SEQUENCE order_seq;
```

---

# 2. Full Text Search

Traditional SQL search:

```sql
WHERE description LIKE '%phone%'
```

Problems:

* slow
* no ranking
* no linguistic understanding

PostgreSQL provides a full-text search engine built into the database.

---

## What Full-Text Search Does

It supports:

* word stemming
* ranking results
* language processing
* dictionary support
* efficient indexing

---

## 2.1 Text Search Configuration

Configuration defines:

* language rules
* stemming behavior
* stop words

Example:

```sql
english
simple
french
```

Example conversion:

```sql
SELECT to_tsvector('english', 'PostgreSQL is a powerful database');
```

Output:

```text
'databas':5 'postgresql':1 'power':4
```

Words are converted into lexemes.

---

## 2.2 Search Dictionaries

Dictionaries process words by:

* removing stop words
* reducing words to root forms

Example:

```text
running → run
runs → run
ran → run
```

This improves search accuracy.

---

## 2.3 Performing Text Search Queries

Search is performed using:

* `tsvector`
* `tsquery`

### Example Table

```sql
CREATE TABLE articles (
    id SERIAL,
    title TEXT,
    content TEXT
);
```

### Insert Data

```sql
INSERT INTO articles(title, content)
VALUES
('PostgreSQL Guide', 'PostgreSQL is a powerful open source database'),
('Python Tutorial', 'Python is popular for data science');
```

### Search Query

```sql
SELECT *
FROM articles
WHERE to_tsvector(content) @@ to_tsquery('database');
```

This returns articles containing database-related words.

---

# 3. JSON and JSONB Data Handling

Modern applications often store semi-structured data like:

* API responses
* user preferences
* product attributes

PostgreSQL supports this using:

* `JSON`
* `JSONB`

---

## 3.1 JSON vs JSONB

| Feature    | JSON      | JSONB      |
| ---------- | --------- | ---------- |
| Storage    | text      | binary     |
| Speed      | slower    | faster     |
| Indexing   | limited   | powerful   |
| Formatting | preserved | normalized |

In most cases:

```text
JSONB is recommended
```

---

## 3.2 Storing JSON Data

### Example Table

```sql
CREATE TABLE products (
    id SERIAL,
    data JSONB
);
```

### Insert JSON

```sql
INSERT INTO products(data)
VALUES
('{"name":"Laptop","price":700,"brand":"Dell"}');
```

---

## 3.3 Querying JSON Data

Access fields using `->` and `->>`.

### Extract JSON Field

```sql
SELECT data->'name'
FROM products;
```

Output:

```text
"Laptop"
```

### Extract as Text

```sql
SELECT data->>'name'
FROM products;
```

Output:

```text
Laptop
```

### Filter Using JSON Field

```sql
SELECT *
FROM products
WHERE data->>'brand' = 'Dell';
```

---

## 3.4 Updating JSON Fields

Use `jsonb_set`.

Example:

```sql
UPDATE products
SET data = jsonb_set(data, '{price}', '800')
WHERE id = 1;
```

Result:

```text
price updated to 800
```

---

## 3.5 Indexing JSON Data

Without indexing, JSON queries can be slow.

PostgreSQL supports GIN indexes.

Example:

```sql
CREATE INDEX idx_products_json
ON products
USING GIN (data);
```

Now JSON queries become much faster.

---

# 4. Arrays in PostgreSQL

Unlike many databases, PostgreSQL supports native array data types.

Arrays allow storing multiple values in a single column.

Example use cases:

* tags
* categories
* product sizes
* multiple phone numbers

---

## 4.1 Array Data Types

Example types:

* `INT[]`
* `TEXT[]`
* `VARCHAR[]`

---

## 4.2 Creating Arrays

### Example Table

```sql
CREATE TABLE students (
    id SERIAL,
    name TEXT,
    marks INT[]
);
```

### Insert Array Data

```sql
INSERT INTO students(name, marks)
VALUES
('Rahul', ARRAY[80, 85, 90]);
```

---

## 4.3 Querying Arrays

### Access Specific Element

```sql
SELECT marks[1]
FROM students;
```

Output:

```text
80
```

### Find Students with Specific Value

```sql
SELECT *
FROM students
WHERE 90 = ANY(marks);
```

### Check Array Length

```sql
SELECT array_length(marks, 1)
FROM students;
```

---

## 4.4 Updating Arrays

### Add New Element

```sql
UPDATE students
SET marks = array_append(marks, 95)
WHERE id = 1;
```

### Remove Element

```sql
UPDATE students
SET marks = array_remove(marks, 80)
WHERE id = 1;
```

### Replace Element

```sql
UPDATE students
SET marks[2] = 88
WHERE id = 1;
```

---

# Practical Use Cases

## Sequences

Used in:

* primary keys
* order numbers
* invoice numbers

---

## Full Text Search

Used in:

* search engines
* document search
* blog search
* knowledge bases

---

## JSONB

Used in:

* storing flexible metadata
* microservices
* API storage
* dynamic product attributes

---

## Arrays

Used in:

* tags
* categories
* preferences
* analytics data

---

# Final Architecture Perspective

These features make PostgreSQL capable of supporting multiple data paradigms:

| Paradigm         | PostgreSQL Feature |
| ---------------- | ------------------ |
| Relational Data  | Tables             |
| Document Data    | JSONB              |
| Search Data      | Full Text Search   |
| Multi-value Data | Arrays             |
| Sequential IDs   | Sequences          |

This is why PostgreSQL is often called:

```text
"The most advanced open-source relational database."
```



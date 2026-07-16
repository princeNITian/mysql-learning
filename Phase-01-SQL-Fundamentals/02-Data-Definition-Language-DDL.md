# Chapter 2 — Data Definition Language (DDL)

> **Estimated Time:** 3–4 Hours  
> **Prerequisites:** Chapter 1 — Introduction to Relational Databases

---

# Learning Objectives

After completing this chapter, you will be able to:

- Understand what DDL is.
- Create databases.
- Create tables.
- Modify table structures.
- Rename tables.
- Remove tables.
- Understand the difference between DELETE, TRUNCATE, and DROP.
- Use DDL from both MySQL Workbench and Node.js.

---

# Table of Contents

1. What is DDL?
2. CREATE DATABASE
3. DROP DATABASE
4. CREATE TABLE
5. ALTER TABLE
6. RENAME TABLE
7. TRUNCATE TABLE
8. DROP TABLE
9. DDL in Node.js
10. Best Practices
11. Common Mistakes
12. Hands-on Lab
13. Interview Questions
14. Summary

---

# 1. What is DDL?

DDL stands for **Data Definition Language**.

DDL commands define the structure of a database.

Think of DDL as the commands used to build the skeleton of an application.

You are **not manipulating data** here—you are manipulating the structure that stores data.

Example:

```
Database

↓

Tables

↓

Columns

↓

Constraints
```

---

# DDL Commands

| Command | Purpose |
|----------|---------|
| CREATE | Create a new object |
| ALTER | Modify an existing object |
| DROP | Remove an object permanently |
| TRUNCATE | Remove all rows but keep the table |
| RENAME | Rename an object |

---

# 2. CREATE DATABASE

Creates a new database.

Syntax

```sql
CREATE DATABASE database_name;
```

Example

```sql
CREATE DATABASE ecommerce;
```

Verify

```sql
SHOW DATABASES;
```

Output

```
information_schema
mysql
performance_schema
sys
ecommerce
```

---

# Using a Database

Before creating tables, tell MySQL which database to use.

```sql
USE ecommerce;
```

Verify

```sql
SELECT DATABASE();
```

Output

```
ecommerce
```

---

# 3. DROP DATABASE

Deletes an entire database.

Syntax

```sql
DROP DATABASE database_name;
```

Example

```sql
DROP DATABASE ecommerce;
```

⚠️ Warning

This permanently removes:

- Tables
- Data
- Indexes
- Views
- Procedures

Everything inside the database is lost.

---

# 4. CREATE TABLE

A table stores information about a single entity.

Example

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE,
    age INT,
    city VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Verify

```sql
SHOW TABLES;
```

Describe the table

```sql
DESCRIBE users;
```

or

```sql
DESC users;
```

---

# Internal Working

When you create a table, MySQL stores:

- Table metadata
- Column definitions
- Constraints
- Storage engine information

No actual user data exists yet.

---

# 5. ALTER TABLE

Used to modify an existing table.

---

## Add a Column

```sql
ALTER TABLE users
ADD phone VARCHAR(20);
```

---

## Add Multiple Columns

```sql
ALTER TABLE users
ADD country VARCHAR(50),
ADD state VARCHAR(50);
```

---

## Modify a Column

```sql
ALTER TABLE users
MODIFY age SMALLINT;
```

---

## Rename a Column

```sql
ALTER TABLE users
RENAME COLUMN phone TO mobile;
```

---

## Drop a Column

```sql
ALTER TABLE users
DROP COLUMN state;
```

---

# 6. RENAME TABLE

Rename an existing table.

```sql
RENAME TABLE users TO customers;
```

Rename it back.

```sql
RENAME TABLE customers TO users;
```

---

# 7. TRUNCATE TABLE

Removes every row.

```sql
TRUNCATE TABLE users;
```

Table structure remains.

Rows become:

```
0
```

Auto Increment resets.

---

# 8. DROP TABLE

Deletes the table permanently.

```sql
DROP TABLE users;
```

Everything is removed:

- Data
- Structure
- Indexes
- Constraints

---

# DELETE vs TRUNCATE vs DROP

| Feature | DELETE | TRUNCATE | DROP |
|----------|---------|-----------|------|
| Removes Data | ✅ | ✅ | ✅ |
| Removes Table | ❌ | ❌ | ✅ |
| WHERE Allowed | ✅ | ❌ | ❌ |
| Resets AUTO_INCREMENT | ❌ | ✅ | N/A |
| Faster | ❌ | ✅ | ✅ |

---

# DDL from Node.js

Creating a table.

```javascript
const sql = `
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100)
);
`;

await connection.query(sql);
```

Creating a database.

```javascript
await connection.query(
    "CREATE DATABASE ecommerce"
);
```

Changing database.

```javascript
await connection.query(
    "USE ecommerce"
);
```

---

# Best Practices

✅ Always use meaningful table names.

Good

```
users
products
orders
payments
```

Bad

```
t1
abc
table_new
```

---

Use singular or plural consistently.

Example

```
users
orders
payments
```

Avoid mixing

```
user
orders
payment
```

---

Always define a Primary Key.

---

Choose appropriate data types.

Instead of

```sql
VARCHAR(500)
```

Use

```sql
VARCHAR(100)
```

if the value will never exceed 100 characters.

---

Avoid dropping production databases.

---

# Common Mistakes

Creating tables without Primary Keys.

Using VARCHAR for numeric values.

Not specifying NOT NULL where appropriate.

Dropping tables accidentally.

Ignoring AUTO_INCREMENT.

---

# Hands-on Lab

Complete the following tasks in your AWS RDS instance.

## Task 1

Create a database called

```
school
```

---

## Task 2

Create a table

```
students
```

Columns

```
id
name
email
age
city
created_at
```

---

## Task 3

Add a new column

```
phone
```

---

## Task 4

Rename

```
phone

↓

mobile
```

---

## Task 5

Drop

```
mobile
```

---

## Task 6

Rename

```
students

↓

learners
```

---

## Task 7

Rename it back.

---

## Task 8

Describe the table.

---

## Task 9

Show all databases.

---

## Task 10

Show all tables.

---

# Interview Questions

### What is DDL?

---

### Difference between DELETE, TRUNCATE, and DROP?

---

### Why should every table have a Primary Key?

---

### Can we recover a dropped table?

---

### What happens internally when CREATE TABLE is executed?

---

### Why is TRUNCATE faster than DELETE?

---

# Summary

In this chapter you learned:

- CREATE DATABASE
- DROP DATABASE
- CREATE TABLE
- ALTER TABLE
- RENAME TABLE
- TRUNCATE TABLE
- DROP TABLE
- DDL using Node.js
- Best practices
- Common interview questions

---

# Next Chapter

➡️ **03-CRUD-Operations.md**
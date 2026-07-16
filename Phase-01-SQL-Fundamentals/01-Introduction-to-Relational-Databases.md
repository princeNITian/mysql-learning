# Chapter 1 — Introduction to Relational Databases

> **Estimated Time:** 2–3 Hours  
> **Prerequisites:** None

---

# Learning Objectives

After completing this chapter, you will be able to:

- Understand what a database is.
- Explain why databases exist.
- Differentiate between SQL and MySQL.
- Understand what an RDBMS is.
- Explain Database, Schema, Table, Row, and Column.
- Understand Primary Keys and Foreign Keys at a high level.
- Recognize common SQL data types.
- Relate database concepts to backend applications.

---

# Table of Contents

1. Why Do We Need Databases?
2. What is a Database?
3. Why Not Store Everything in JSON?
4. What is an RDBMS?
5. SQL vs MySQL
6. Database vs Schema
7. Tables
8. Rows
9. Columns
10. Data Types
11. Primary Keys
12. Foreign Keys (Introduction)
13. NULL Values
14. How a Backend Uses a Database
15. Summary
16. Exercises

---

# 1. Why Do We Need Databases?

Imagine you're building an e-commerce application.

Users can:

- Register
- Login
- Add products
- Place orders
- Make payments

Where should all this information be stored?

Possible options:

- Excel Sheet
- JSON File
- Text File
- Database

As applications grow, thousands or even millions of users may access the system simultaneously. Managing this data reliably requires much more than simply writing it to files.

Databases are designed to solve these problems.

---

# Problems with Files

Suppose you store users in a JSON file.

```json
[
  {
    "id": 1,
    "name": "Prince",
    "email": "prince@gmail.com"
  }
]
```

Initially this looks simple.

As your application grows, challenges appear:

- Searching becomes slower.
- Multiple users may overwrite each other's changes.
- Data duplication increases.
- Relationships between data become difficult.
- Crash recovery is difficult.
- Security becomes harder.

---

# Databases Solve These Problems

A database provides:

- Fast searching
- Efficient updates
- Concurrent access
- Data consistency
- Relationships
- Backup & recovery
- Security
- Transactions

---

# 2. What is a Database?

A database is an organized collection of related data that allows efficient storage, retrieval, modification, and deletion.

Example:

| id | name | email |
|----|------|-------|
| 1 | Prince | prince@gmail.com |
| 2 | Rahul | rahul@gmail.com |

Although this looks like a spreadsheet, databases provide querying, indexing, transactions, constraints, and concurrency on top of storing the data.

---

# 3. What is an RDBMS?

RDBMS stands for **Relational Database Management System**.

It stores data in the form of related tables.

Example:

Users

| user_id | name |
|----------|------|
| 1 | Prince |
| 2 | Rahul |

Orders

| order_id | user_id | amount |
|----------|----------|--------|
| 101 | 1 | 499 |
| 102 | 2 | 799 |

Notice that `Orders.user_id` refers to `Users.user_id`.

This relationship is why these systems are called **Relational** databases.

---

# Popular RDBMS

- MySQL
- PostgreSQL
- Oracle Database
- Microsoft SQL Server
- MariaDB

---

# 4. SQL vs MySQL

These two terms are often confused.

## SQL

SQL stands for **Structured Query Language**.

It is a language used to communicate with relational databases.

Example:

```sql
SELECT * FROM users;
```

SQL itself is **not** a database.

---

## MySQL

MySQL is a relational database management system.

It understands SQL and executes SQL statements.

Think of it this way:

```
SQL       → Language

MySQL     → Software

PostgreSQL → Software

Oracle    → Software
```

---

# 5. Database vs Schema

A database is a logical container that holds related data.

Inside a database are tables.

Example:

```
ecommerce
│
├── users
├── products
├── orders
└── payments
```

For now, you can think of **database** as the top-level container for your application data. (Some database systems distinguish between databases and schemas more explicitly; we'll revisit that later.)

---

# 6. Tables

A table stores data about a single entity.

Examples:

- Users
- Products
- Orders
- Employees

Example:

| id | name | age |
|----|------|-----|
| 1 | Prince | 25 |
| 2 | Rahul | 28 |

A table consists of rows and columns.

---

# 7. Rows

A row represents a single record.

Example:

| id | name | age |
|----|------|-----|
| 1 | Prince | 25 |

This row represents one user.

---

# 8. Columns

Columns describe the attributes of a record.

Example:

| Column | Meaning |
|---------|---------|
| id | Unique identifier |
| name | User's name |
| age | User's age |

Every row has values for these columns.

---

# 9. Data Types

Each column stores a specific type of data.

Common MySQL data types:

| Type | Example |
|------|---------|
| INT | 25 |
| BIGINT | 1000000000 |
| VARCHAR(100) | Prince |
| TEXT | Long description |
| BOOLEAN | TRUE |
| DATE | 2026-01-01 |
| DATETIME | 2026-01-01 10:30:00 |
| TIMESTAMP | Auto-generated timestamp |
| DECIMAL(10,2) | 1999.99 |

Choosing appropriate data types improves storage efficiency and validation.

---

# 10. Primary Key

A Primary Key uniquely identifies each row.

Example:

| id | name |
|----|------|
| 1 | Prince |
| 2 | Rahul |

The `id` column is unique.

Rules:

- Cannot be NULL
- Must be unique
- One primary key per table

---

# 11. Foreign Key (Introduction)

A Foreign Key connects two tables.

Users

| id | name |
|----|------|
| 1 | Prince |

Orders

| order_id | user_id |
|----------|----------|
| 101 | 1 |

Here, `user_id` in the `orders` table refers to the `id` column in the `users` table.

We'll explore foreign keys in detail in Phase 2.

---

# 12. NULL Values

`NULL` means **no value is stored**.

It does **not** mean:

- 0
- Empty string
- False

Example:

| id | phone |
|----|--------|
| 1 | NULL |

This means the phone number is unknown or hasn't been provided.

---

# 13. How a Backend Uses a Database

A typical request flow looks like this:

```
Client
   │
   ▼
Node.js API
   │
   ▼
MySQL
   │
   ▼
Data Returned
   │
   ▼
Client
```

Example:

A client requests:

```
GET /users/1
```

Node.js executes:

```sql
SELECT * FROM users WHERE id = 1;
```

MySQL returns the matching row, and the API sends it back to the client.

---

# Summary

In this chapter you learned:

- Why databases exist.
- What an RDBMS is.
- Difference between SQL and MySQL.
- Database vs table.
- Rows and columns.
- Data types.
- Primary keys.
- Foreign keys (basic idea).
- NULL values.
- How Node.js communicates with MySQL.

---

# Exercises

1. Explain the difference between SQL and MySQL.
2. What is an RDBMS?
3. Why are JSON files not suitable for large applications?
4. What is a Primary Key?
5. Can a Primary Key contain NULL?
6. What is a Foreign Key?
7. Explain the difference between a row and a column.
8. List five common MySQL data types.
9. What does NULL mean?
10. Describe how a Node.js application retrieves data from MySQL.

---

# Next Chapter

➡️ **02-Data-Definition-Language-DDL.md**
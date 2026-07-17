# Chapter 3 — CRUD Operations

> **Estimated Time:** 2–3 Days  
> **Prerequisites:**  
> - Chapter 1 — Introduction to Relational Databases
> - Chapter 2 — Data Definition Language (DDL)

---

# Learning Objectives

After completing this chapter, you will be able to:

- Understand the four fundamental database operations.
- Insert single and multiple records.
- Retrieve data efficiently.
- Modify existing records.
- Delete records safely.
- Execute CRUD operations from Node.js.
- Understand how MySQL processes CRUD queries internally.
- Write secure queries using parameterized statements.
- Avoid common mistakes made in production systems.
- Explain CRUD concepts confidently in backend interviews.

---

# Table of Contents

1. What is CRUD?
2. CRUD in Real Applications
3. Understanding the CRUD Lifecycle
4. INSERT
5. SELECT
6. UPDATE
7. DELETE
8. Parameterized Queries
9. Prepared Statements
10. SQL Injection
11. Understanding mysql2 Result Objects
12. CRUD Flow Inside MySQL
13. CRUD Flow Inside Node.js
14. Best Practices
15. Performance Notes
16. Hands-on Lab
17. Interview Questions
18. Summary

---

# 1. What is CRUD?

CRUD represents the four basic operations performed on data stored inside a relational database.

| Operation | SQL Command | Purpose |
|------------|-------------|---------|
| Create | INSERT | Add new data |
| Read | SELECT | Retrieve existing data |
| Update | UPDATE | Modify existing data |
| Delete | DELETE | Remove existing data |

Almost every backend application revolves around these four operations.

Whether you're building a social media platform, banking application, e-commerce website, hospital management system, or food delivery app, every user interaction eventually translates into one or more CRUD operations.

For example, consider an e-commerce application.

| User Action | Database Operation |
|--------------|-------------------|
| Register | INSERT |
| Login | SELECT |
| View Products | SELECT |
| Edit Profile | UPDATE |
| Add Address | INSERT |
| Change Password | UPDATE |
| Place Order | INSERT |
| Cancel Order | UPDATE |
| Delete Account | DELETE |

Although the user interacts with buttons, forms, and APIs, the backend ultimately communicates with the database using CRUD operations.

Understanding CRUD thoroughly is therefore one of the most important skills for any backend engineer.

---

# 2. CRUD in Real Applications

Let's examine how CRUD appears in applications you use every day.

## Example 1 — Instagram

Creating a new account

```
INSERT INTO users (...)
```

Logging in

```
SELECT * FROM users
WHERE email = ?;
```

Updating your profile picture

```
UPDATE users
SET profile_picture = ?
WHERE id = ?;
```

Deleting your account

```
DELETE FROM users
WHERE id = ?;
```

---

## Example 2 — Amazon

Adding a product

```
INSERT INTO products
```

Viewing products

```
SELECT * FROM products
```

Updating inventory

```
UPDATE products
SET stock = stock - 1
```

Removing a discontinued product

```
DELETE FROM products
```

---

## Example 3 — Banking System

Creating an account

```
INSERT
```

Viewing account balance

```
SELECT
```

Updating account information

```
UPDATE
```

Closing an account

```
DELETE
```

Notice a pattern?

Regardless of the domain, CRUD remains the foundation of data management.

---

# 3. Understanding the CRUD Lifecycle

Consider a simple user registration flow.

```
Client
   │
   ▼
Node.js API
   │
   ▼
MySQL Server
```

Suppose a user signs up.

The frontend sends

```
POST /users
```

The backend receives

```json
{
    "name": "Prince Raj",
    "email": "prince@gmail.com",
    "age": 25,
    "city": "Bengaluru"
}
```

Node.js executes

```sql
INSERT INTO users(name, email, age, city)
VALUES('Prince Raj', 'prince@gmail.com', 25, 'Bengaluru');
```

MySQL stores the row.

Later,

the user opens the profile page.

Node.js executes

```sql
SELECT *
FROM users
WHERE id = 1;
```

The user changes the city.

Node.js executes

```sql
UPDATE users
SET city = 'Hyderabad'
WHERE id = 1;
```

Finally,

if the user deletes the account,

Node.js executes

```sql
DELETE
FROM users
WHERE id = 1;
```

This entire lifecycle demonstrates the four CRUD operations.

---

# Before We Begin

Throughout this chapter, we'll use the following table.

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    age INT,
    city VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

We'll continue using this schema in every SQL and Node.js example.

---

# 4. INSERT

## What is INSERT?

The INSERT statement adds one or more new rows into a table.

Whenever new information enters your application, an INSERT operation is executed.

Examples include:

- User registration
- Creating an order
- Adding a product
- Posting a comment
- Uploading a document
- Sending a message

Without INSERT, your database would remain empty.

---

## Syntax

```sql
INSERT INTO table_name(column1, column2, ...)
VALUES(value1, value2, ...);
```

Example

```sql
INSERT INTO users(name, email, age, city)
VALUES(
    'Prince Raj',
    'prince@gmail.com',
    25,
    'Bengaluru'
);
```

Here,

`users` is the table,

while

`name`,
`email`,
`age`,
and
`city`

are the columns receiving the new values.

---

# Understanding What Happens

Let's visualize what happens internally.

Initially,

```
users
```

| id | name |
|----|------|
| 1 | Rahul |
| 2 | Amit |

After executing

```sql
INSERT INTO users(name)
VALUES('Prince');
```

the table becomes

| id | name |
|----|------|
| 1 | Rahul |
| 2 | Amit |
| 3 | Prince |

A completely new row has been appended.

Notice that we never supplied the `id`.

MySQL generated it automatically because the column was defined using

```sql
AUTO_INCREMENT
```

---

# Internal Working of INSERT

When MySQL receives an INSERT query, much more happens than simply writing data to disk.

```
Client
    │
    ▼
MySQL Server
    │
    ▼
SQL Parser
    │
    ▼
Permission Check
    │
    ▼
Syntax Validation
    │
    ▼
Constraint Validation
    │
    ▼
AUTO_INCREMENT Generator
    │
    ▼
Storage Engine (InnoDB)
    │
    ▼
Buffer Pool
    │
    ▼
Redo Log
    │
    ▼
Disk
```

Let's briefly understand each stage.

### SQL Parser

Checks whether the SQL syntax is valid.

Example:

```sql
INSERT users VALUES(...)
```

is invalid because `INTO` is missing.

The parser rejects the query before any data changes occur.

---

### Permission Check

MySQL verifies whether the current user has permission to insert rows into the table.

Without sufficient privileges, the query is rejected.

---

### Constraint Validation

MySQL validates every constraint.

For example,

if

```sql
email
```

is marked

```sql
UNIQUE
```

and another user already has the same email,

the INSERT fails.

Similarly,

NOT NULL columns cannot receive NULL values unless explicitly allowed.

---

### AUTO_INCREMENT

If the primary key uses AUTO_INCREMENT,

MySQL generates the next available identifier automatically.

For example,

Current IDs

```
1
2
3
```

Next inserted row receives

```
4
```

without your application calculating it.

This reduces complexity and prevents duplicate identifiers under normal usage.

---

### Storage Engine

Once validation succeeds,

the query reaches the storage engine (InnoDB by default).

The storage engine is responsible for physically storing the row, maintaining indexes, writing redo logs, and ensuring durability.

Understanding InnoDB in depth will become important later when we study indexes, transactions, and performance tuning.

---
### Writing to Disk

Before data is permanently stored, InnoDB first writes the change to the **Redo Log**.

This ensures that even if the database crashes immediately after the INSERT, MySQL can recover the committed transaction during restart.

Only after the necessary logging and memory updates does the data eventually get flushed to disk.

This process is one of the reasons why relational databases are reliable even during unexpected failures.

---

# Single Row INSERT

The most common INSERT operation adds a single record.

Example:

```sql
INSERT INTO users(name, email, age, city)
VALUES(
    'Rahul Sharma',
    'rahul@gmail.com',
    28,
    'Delhi'
);
```

Let's verify the inserted record.

```sql
SELECT *
FROM users;
```

Output

| id | name | email | age | city |
|----|------|-------|-----|------|
| 1 | Rahul Sharma | rahul@gmail.com | 28 | Delhi |

---

# Inserting into Selected Columns

You are not required to provide values for every column.

Example

```sql
INSERT INTO users(name, email)
VALUES(
    'Ankit Verma',
    'ankit@gmail.com'
);
```

Since

- `age`
- `city`

were not provided,

their values become

```
NULL
```

unless a DEFAULT value has been defined.

Result

| id | name | email | age | city |
|----|------|-------|-----|------|
| 2 | Ankit Verma | ankit@gmail.com | NULL | NULL |

---

# Omitting AUTO_INCREMENT Columns

Since

```sql
id
```

is defined as

```sql
AUTO_INCREMENT
```

we should never manually calculate the next ID.

Incorrect

```sql
INSERT INTO users
VALUES(
    3,
    'Prince',
    'prince@gmail.com',
    25,
    'Bengaluru',
    NOW()
);
```

Although valid, manually supplying IDs can lead to duplicate key errors and maintenance issues.

Preferred

```sql
INSERT INTO users(
    name,
    email,
    age,
    city
)
VALUES(
    'Prince Raj',
    'prince@gmail.com',
    25,
    'Bengaluru'
);
```

MySQL automatically generates

```
id = 3
```

---

# DEFAULT Values

Suppose our table contains

```sql
created_at TIMESTAMP
DEFAULT CURRENT_TIMESTAMP
```

When we execute

```sql
INSERT INTO users(
    name,
    email
)
VALUES(
    'Neha',
    'neha@gmail.com'
);
```

MySQL automatically stores the current timestamp.

Result

| id | name | created_at |
|----|------|---------------------|
| 4 | Neha | 2026-07-17 11:45:32 |

No application code is required.

This reduces duplication and ensures consistency.

---

# INSERT Multiple Rows

One INSERT statement can create multiple records.

```sql
INSERT INTO users(
    name,
    email,
    age,
    city
)
VALUES
(
    'Amit Kumar',
    'amit@gmail.com',
    30,
    'Mumbai'
),
(
    'Priya Singh',
    'priya@gmail.com',
    27,
    'Hyderabad'
),
(
    'Rohit Jain',
    'rohit@gmail.com',
    29,
    'Ahmedabad'
);
```

Instead of sending three SQL statements,

only one statement is sent.

This reduces

- Network latency
- Parsing overhead
- Transaction overhead

and significantly improves performance.

---

# Why Batch INSERT is Faster

Consider inserting 1000 users.

### Approach 1

```text
1000 INSERT statements
```

Every statement requires

```
Network

↓

SQL Parser

↓

Optimizer

↓

Execution

↓

Disk Write
```

This process repeats one thousand times.

---

### Approach 2

```text
One INSERT statement

↓

1000 VALUES(...)
```

Now MySQL parses only one statement and executes it in a single operation.

This is substantially faster.

Production systems frequently batch inserts whenever possible.

---

# INSERT ... SET

MySQL also supports an alternate syntax.

```sql
INSERT INTO users
SET
name = 'Karan Mehta',
email = 'karan@gmail.com',
age = 31,
city = 'Jaipur';
```

Although functionally equivalent,

most teams prefer the

```sql
INSERT ... VALUES
```

syntax because

- it follows the SQL standard
- it is portable across different databases
- it is easier to generate dynamically

---

# Common INSERT Errors

## Duplicate Primary Key

```sql
INSERT INTO users(
id,
name
)
VALUES(
1,
'Rahul'
);
```

Error

```
Duplicate entry '1'
for key 'PRIMARY'
```

---

## Duplicate UNIQUE Value

Suppose

```sql
email
```

is UNIQUE.

Executing

```sql
INSERT INTO users(
name,
email
)
VALUES(
'Someone',
'rahul@gmail.com'
);
```

produces

```
Duplicate entry
'rahul@gmail.com'
for key 'email'
```

---

## NOT NULL Violation

Suppose

```sql
name
```

is

```sql
NOT NULL
```

Executing

```sql
INSERT INTO users(email)
VALUES(
'test@gmail.com'
);
```

fails because

```
name
```

has no value.

---

# INSERT from Node.js

Using our existing project,

```javascript
import { db } from "./connection.js";

const connection = await db();

const sql = `
INSERT INTO users
(name, email, age, city)
VALUES (?, ?, ?, ?)
`;

const [result] = await connection.execute(
    sql,
    [
        "Prince Raj",
        "prince@gmail.com",
        25,
        "Bengaluru"
    ]
);

console.log(result);
```

Notice two important things.

First,

we are using

```javascript
execute()
```

instead of

```javascript
query()
```

We'll discuss the differences later in this chapter.

Second,

instead of concatenating values into the SQL string,

we use

```
?
```

placeholders.

This is known as a **parameterized query** and is the recommended approach for production applications.

---

# Understanding INSERT Result

After executing

```javascript
const [result] =
await connection.execute(...);
```

`result` is an object.

Example

```javascript
console.log(result);
```

Output

```javascript
ResultSetHeader {
    fieldCount: 0,
    affectedRows: 1,
    insertId: 7,
    info: '',
    serverStatus: 2,
    warningStatus: 0
}
```

Let's understand each property.

| Property | Meaning |
|----------|---------|
| affectedRows | Number of rows inserted |
| insertId | AUTO_INCREMENT value generated by MySQL |
| warningStatus | Number of warnings |
| serverStatus | Internal server status |
| fieldCount | Number of columns returned (usually 0 for INSERT) |

The two properties you'll use most frequently are

- `affectedRows`
- `insertId`

Example

```javascript
console.log(
    `Inserted ${result.affectedRows} row`
);

console.log(
    `Generated ID: ${result.insertId}`
);
```

---

# Best Practices for INSERT

✅ Specify column names explicitly.

Good

```sql
INSERT INTO users(name, email)
VALUES('Prince', 'prince@gmail.com');
```

Avoid

```sql
INSERT INTO users
VALUES(...);
```

because changes to the table structure can break the query.

---

✅ Use parameterized queries.

---

✅ Insert multiple rows together whenever possible.

---

✅ Define sensible DEFAULT values.

---

✅ Let AUTO_INCREMENT generate primary keys.

---

✅ Validate data before sending it to MySQL.

Database constraints should be your last line of defense, not the first.

---

# Common Mistakes

❌ Manually calculating primary keys.

❌ Forgetting UNIQUE constraints.

❌ Ignoring NULL values.

❌ Concatenating user input into SQL.

❌ Inserting rows one by one inside loops when batch insertion is possible.

---

# Performance Notes

- Batch INSERT operations are significantly faster than multiple single-row INSERT statements.
- Avoid unnecessary indexes on tables with frequent inserts, as every index must also be updated.
- Large transactions consume more memory and can hold locks for longer durations.
- Using prepared statements reduces parsing overhead for repeated INSERT operations.

---

# Quick Revision

- `INSERT` adds new rows to a table.
- Use `AUTO_INCREMENT` for surrogate primary keys.
- Prefer specifying column names explicitly.
- Batch inserts improve performance.
- Use parameterized queries to prevent SQL injection.
- `affectedRows` tells you how many rows were inserted.
- `insertId` returns the generated primary key.

---

The next section introduces the most frequently used SQL command in backend development: **SELECT**, where we'll explore data retrieval, result sets, aliases, expressions, and how MySQL efficiently finds rows.

# 5. SELECT

## What is SELECT?

The `SELECT` statement is used to retrieve data from one or more tables.

Unlike `INSERT`, `UPDATE`, and `DELETE`, which modify data, `SELECT` is a **read-only** operation.

Almost every API in a backend application performs one or more `SELECT` queries.

Examples include:

- User Login
- Profile Page
- Product Listing
- Search
- Dashboard
- Analytics
- Reports
- Notifications

Whenever an application needs to display information, it uses `SELECT`.

---

# Basic Syntax

```sql
SELECT column1, column2
FROM table_name;
```

To retrieve every column,

```sql
SELECT *
FROM users;
```

Example Output

| id | name | email | age | city | created_at |
|----|------|-------|-----|------|------------|
| 1 | Rahul | rahul@gmail.com | 28 | Delhi | ... |
| 2 | Prince | prince@gmail.com | 25 | Bengaluru | ... |

---

# Understanding SELECT

Suppose our table contains

| id | name | city |
|----|------|------|
| 1 | Rahul | Delhi |
| 2 | Prince | Bengaluru |
| 3 | Amit | Mumbai |

Executing

```sql
SELECT *
FROM users;
```

returns

| id | name | city |
|----|------|------|
| 1 | Rahul | Delhi |
| 2 | Prince | Bengaluru |
| 3 | Amit | Mumbai |

Notice something important.

The database itself **does not change**.

SELECT only reads data.

---

# Internal Working of SELECT

Although SELECT looks simple,

```sql
SELECT *
FROM users;
```

MySQL performs several operations internally.

```
Client

↓

MySQL Server

↓

SQL Parser

↓

Permission Check

↓

Optimizer

↓

Execution Plan

↓

Storage Engine (InnoDB)

↓

Buffer Pool

↓

Index Lookup / Table Scan

↓
124.123.151.33
Rows Returned
```

Let's understand each step.

---

## SQL Parser

The parser verifies that the SQL statement is valid.

Example

```sql
SELECT FROM users;
```

This fails because no columns were specified.

---

## Permission Check

MySQL verifies whether the current user has

```
SELECT
```

permission on the table.

Without permission,

the query fails.

---

## Optimizer

The optimizer decides the most efficient way to retrieve the requested rows.

Questions it considers include:

- Can an index be used?
- Is a full table scan required?
- Which index is the most selective?
- Which join order is cheapest?

We'll study the optimizer in detail during the Query Optimization phase.

---

## Storage Engine

The optimizer passes the execution plan to InnoDB.

InnoDB retrieves the required rows.

If the data already exists in memory (Buffer Pool),

disk access is avoided,

making the query much faster.

---

# SELECT *

The easiest way to retrieve data is

```sql
SELECT *
FROM users;
```

The asterisk (`*`) means

> Return every column.

Result

| id | name | email | age | city | created_at |
|----|------|-------|-----|------|------------|

---

# Why SELECT * is Discouraged

Although convenient,124.123.151.33

`SELECT *` is generally discouraged in production code.

Imagine a table

```text
users
```

contains

```
id

name

email

password_hash

phone

address

profile_picture

bio

created_at

updated_at
```

Now consider an API that only needs

```
name

email
```

Using

```sql
SELECT *
FROM users;
```

retrieves every column,

including unnecessary data.

This increases

- Network traffic
- Memory usage
- Serialization cost
- Query execution time

Instead,

retrieve only the columns you actually need.

Good

```sql
SELECT
name,
email
FROM users;
```

---

# Selecting Specific Columns

Instead of requesting every column,

you can request only the required ones.

Example

```sql
SELECT
name,
city
FROM users;
```

Output

| name | city |
|------|------|
| Rahul | Delhi |
| Prince | Bengaluru |
| Amit | Mumbai |

This is both faster and easier to understand.

---

# Column Order

The order of columns in the SELECT statement determines the order of the output.

Example

```sql
SELECT
city,
name
FROM users;
```

Output

| city | name |
|------|------|
| Delhi | Rahul |
| Bengaluru | Prince |

Notice that the table structure hasn't changed.

Only the result set changed.

---

# Aliases

Aliases rename columns in the result.

Syntax

```sql
SELECT column AS alias
FROM table;
```

Example

```sql
SELECT
name AS FullName,
email AS EmailAddress
FROM users;
```

Output

| FullName | EmailAddress |
|-----------|--------------|
| Rahul | rahul@gmail.com |

Aliases improve readability,

especially when generating reports.

---

# Aliases Without AS

The `AS` keyword is optional.

These two queries are identical.

```sql
SELECT
name AS FullName
FROM users;
```

```sql
SELECT
name FullName
FROM users;
```

However,

using `AS` is generally recommended because it makes the query easier to read.

---

# DISTINCT

Sometimes duplicate values exist.

Suppose

| id | city |
|----|------|
| 1 | Delhi |
| 2 | Delhi |
| 3 | Mumbai |
| 4 | Bengaluru |
| 5 | Mumbai |

Executing

```sql
SELECT city
FROM users;
```

returns

```
Delhi
Delhi
Mumbai
Bengaluru
Mumbai
```

Using

```sql
SELECT DISTINCT city
FROM users;
```

returns

```
Delhi

Mumbai

Bengaluru
```

Duplicate values are removed.

---

# Expressions

SELECT can evaluate expressions.

Example

```sql
SELECT
10 + 20;
```

Output

```
30
```

Another example

```sql
SELECT
name,
age + 5 AS AgeAfterFiveYears
FROM users;
```

Output

| name | AgeAfterFiveYears |
|------|-------------------|
| Rahul | 33 |
| Prince | 30 |

Expressions are evaluated for every row.

---

# Using Functions

SELECT can execute built-in functions.

Example

```sql
SELECT
UPPER(name)
FROM users;
```

Output

```
RAHUL

PRINCE

AMIT
```

Another example

```sql
SELECT
NOW();
```

returns

```
Current Date and Time
```

We'll cover SQL functions in detail later.

---

# LIMIT

Sometimes you don't want every row.

Example

```sql
SELECT *
FROM users
LIMIT 5;
```

Only five rows are returned.

This is especially useful for

- Pagination
- Testing
- Dashboards
- Preview screens

---

# Result Set

The output returned by SELECT is called a **Result Set**.

Example

```sql
SELECT
name,
email
FROM users;
```

Result Set

| name | email |
|------|-------|
| Rahul | rahul@gmail.com |
| Prince | prince@gmail.com |

This result exists only for the duration of the query.

Nothing is permanently stored.

---

# SELECT in Node.js

Using mysql2,

```javascript
const sql = `
SELECT
id,
name,
email,
city
FROM users
`;

const [rows] =
await connection.execute(sql);

console.log(rows);
```

Output

```javascript
[
    {
        id: 1,
        name: "Rahul",
        email: "rahul@gmail.com",
        city: "Delhi"
    },
    {
        id: 2,
        name: "Prince",
        email: "prince@gmail.com",
        city: "Bengaluru"
    }
]
```

Notice that

```javascript
rows
```

is an **array of objects**.

Each object represents one row returned by MySQL.

This is why we often write

```javascript
rows[0]
```

to access the first record.

---

# Understanding rows

Suppose

```sql
SELECT *
FROM users;
```

returns

three records.

Internally,

mysql2 creates

```javascript
[
    { id: 1, name: "Rahul" },
    { id: 2, name: "Prince" },
    { id: 3, name: "Amit" }
]
```

Therefore,

```javascript
rows.length
```

returns

```
3
```

The first row

```javascript
rows[0]
```

The second row

```javascript
rows[1]
```

The third row

```javascript
rows[2]
```

This matches standard JavaScript array indexing.

---

# Best Practices for SELECT

✅ Retrieve only the columns you need.

✅ Avoid `SELECT *` in production APIs.

✅ Use meaningful aliases.

✅ Use `LIMIT` when appropriate.

✅ Prefer indexed columns for filtering (covered later).

---

# Common Mistakes

❌ Using `SELECT *` everywhere.

❌ Fetching thousands of rows when only a few are needed.

❌ Forgetting to limit results in APIs.

❌ Returning sensitive columns such as passwords or tokens.

❌ Assuming the result is always non-empty.

Always check

```javascript
if (rows.length === 0) {
    // Handle no records found
}
```

---

# Performance Notes

- Reading fewer columns reduces I/O.
- Reading fewer rows improves response time.
- Proper indexes significantly speed up SELECT queries.
- Frequently accessed data may be served directly from the InnoDB Buffer Pool without reading from disk.

---

# Quick Revision

- `SELECT` retrieves data without modifying it.
- `SELECT *` returns every column.
- Retrieve only the required columns in production.
- `DISTINCT` removes duplicate values.
- Aliases improve readability.
- `LIMIT` restricts the number of rows returned.
- The result of a SELECT query is called a **Result Set**.
- In `mysql2`, rows are returned as an array of JavaScript objects.

---
# 6. UPDATE

## What is UPDATE?

The `UPDATE` statement is used to modify existing records in a table.

Unlike `INSERT`, which creates new rows, UPDATE changes the values of rows that already exist.

Real-world examples include:

- Updating a user's profile
- Changing a password
- Updating product inventory
- Changing an order status
- Updating employee salary
- Modifying shipping address
- Changing account preferences

Whenever existing information changes, an UPDATE operation is performed.

---

# Basic Syntax

```sql
UPDATE table_name
SET column1 = value1,
    column2 = value2
WHERE condition;
```

Example

```sql
UPDATE users
SET city = 'Hyderabad'
WHERE id = 1;
```

Result

Before

| id | name | city |
|----|------|------|
| 1 | Rahul | Delhi |

After

| id | name | city |
|----|------|------|
| 1 | Rahul | Hyderabad |

Notice that

- No new row is created.
- No row is deleted.
- Only the specified column changes.

---

# How UPDATE Works

Imagine the following table.

| id | name | city |
|----|------|------|
| 1 | Rahul | Delhi |
| 2 | Prince | Bengaluru |
| 3 | Amit | Mumbai |

Executing

```sql
UPDATE users
SET city = 'Pune'
WHERE id = 2;
```

Results in

| id | name | city |
|----|------|------|
| 1 | Rahul | Delhi |
| 2 | Prince | Pune |
| 3 | Amit | Mumbai |

Only the matching row changes.

---

# Internal Working of UPDATE

Although UPDATE appears simple, MySQL performs several operations internally.

```
Client
    │
    ▼
MySQL Server
    │
    ▼
SQL Parser
    │
    ▼
Permission Check
    │
    ▼
Optimizer
    │
    ▼
Locate Matching Rows
    │
    ▼
Acquire Required Locks
    │
    ▼
Modify Row
    │
    ▼
Update Indexes
    │
    ▼
Redo Log
    │
    ▼
Commit Transaction
```

Let's understand each stage.

---

## SQL Parser

Checks whether the SQL statement is valid.

---

## Permission Check

Verifies whether the user has UPDATE permission.

Without permission,

the statement is rejected.

---

## Optimizer

Determines the fastest way to locate the rows.

If an index exists,

the optimizer usually uses it.

Otherwise,

MySQL performs a full table scan.

---

## Row Lookup

MySQL finds every row matching the WHERE clause.

Example

```sql
WHERE id = 10
```

If `id` is indexed,

the lookup is extremely fast.

---

## Row Lock

Before modifying the row,

InnoDB acquires an exclusive lock.

This prevents another transaction from updating the same row simultaneously.

We'll study locking in detail during the Transactions chapter.

---

## Modify Row

The required columns are updated in memory.

---

## Update Indexes

If an indexed column changes,

MySQL updates the corresponding index entries.

Example

Suppose

```sql
email
```

is indexed.

Changing

```sql
UPDATE users
SET email = 'new@gmail.com'
WHERE id = 5;
```

requires both

- the table
- the email index

to be updated.

---

## Redo Log

Before the transaction completes,

the change is written to the redo log.

This guarantees durability even if the server crashes.

---

# Updating Multiple Columns

UPDATE can modify multiple columns simultaneously.

Example

```sql
UPDATE users
SET
name = 'Prince Raj',
city = 'Mumbai',
age = 26
WHERE id = 2;
```

Result

| id | name | age | city |
|----|------|-----|------|
| 2 | Prince Raj | 26 | Mumbai |

---

# Updating Every Row

If the WHERE clause is omitted,

every row in the table is updated.

Example

```sql
UPDATE users
SET city = 'India';
```

Suppose

Before

| id | city |
|----|------|
| 1 | Delhi |
| 2 | Mumbai |
| 3 | Pune |

After

| id | city |
|----|------|
| 1 | India |
| 2 | India |
| 3 | India |

Every row has changed.

This is one of the most common production mistakes.

---

# ⚠️ Always Use WHERE

Consider

```sql
UPDATE users
SET age = 30;
```

If your table contains one million users,

all one million records are modified.

Recovery may require restoring from backups.

A safer query is

```sql
UPDATE users
SET age = 30
WHERE id = 1;
```

Always verify the WHERE clause before executing an UPDATE.

---

# Using Expressions

UPDATE can use expressions.

Increase everyone's age by one year.

```sql
UPDATE users
SET age = age + 1;
```

This is different from

```sql
SET age = 1;
```

The first increments the existing value.

The second replaces it.

---

# Using Built-in Functions

Functions can also be used.

```sql
UPDATE users
SET name = UPPER(name)
WHERE id = 1;
```

Result

Before

```
Prince
```

After

```
PRINCE
```

---

# UPDATE with Multiple Conditions

```sql
UPDATE users
SET city = 'Delhi'
WHERE age > 25
AND city = 'Mumbai';
```

Only rows satisfying both conditions are updated.

---

# UPDATE in Node.js

```javascript
const sql = `
UPDATE users
SET city = ?
WHERE id = ?
`;

const [result] = await connection.execute(
    sql,
    [
        "Hyderabad",
        1
    ]
);

console.log(result);
```

Output

```javascript
ResultSetHeader {
    affectedRows: 1,
    changedRows: 1,
    warningStatus: 0
}
```

---

# Understanding UPDATE Result

Example

```javascript
console.log(result);
```

```javascript
ResultSetHeader {
    affectedRows: 1,
    changedRows: 1
}
```

Meaning

| Property | Description |
|----------|-------------|
| affectedRows | Rows matched by the WHERE clause |
| changedRows | Rows whose values actually changed |

Example

Suppose the city is already

```
Hyderabad
```

Executing

```sql
UPDATE users
SET city = 'Hyderabad'
WHERE id = 1;
```

may produce

```javascript
affectedRows = 1
changedRows = 0
```

because

- One row matched.
- Nothing actually changed.

---

# What Happens If No Row Matches?

Example

```sql
UPDATE users
SET city = 'Delhi'
WHERE id = 1000;
```

Suppose user 1000 doesn't exist.

Output

```javascript
affectedRows = 0
```

This is **not** an error.

It simply means no matching records were found.

---

# Best Practices

✅ Always use a WHERE clause.

✅ Update only the required columns.

✅ Validate user input before updating.

✅ Use parameterized queries.

✅ Keep transactions small.

---

# Common Mistakes

❌ Forgetting the WHERE clause.

❌ Updating thousands of rows accidentally.

❌ Updating indexed columns unnecessarily.

❌ Building SQL using string concatenation.

❌ Assuming an UPDATE always modifies data.

---

# Performance Notes

- Updating indexed columns is slower because indexes must also be updated.
- Large UPDATE operations generate significant redo log activity.
- Batch updates inside a transaction are generally more efficient than many individual transactions.
- Updating fewer columns reduces the amount of work MySQL performs.

---

# Interview Notes

### Q1. Difference between `affectedRows` and `changedRows`?

`affectedRows`

Rows matched by the WHERE clause.

`changedRows`

Rows whose values actually changed.

---

### Q2. Why is UPDATE without WHERE dangerous?

Because every row in the table is modified.

---

### Q3. Does UPDATE create new rows?

No.

It only modifies existing rows.

---

### Q4. Why can updating indexed columns be slower?

Because MySQL must update both the table data and the corresponding indexes.

---

# Quick Revision

- UPDATE modifies existing records.
- Always use a WHERE clause unless updating every row intentionally.
- UPDATE can modify one or multiple columns.
- Expressions like `age = age + 1` are allowed.
- `affectedRows` and `changedRows` are different.
- Indexed columns are more expensive to update.
- Parameterized queries should always be used in Node.js.

---

The next section covers **DELETE**, where we'll explore safe row deletion, `DELETE` vs `TRUNCATE`, cascading deletes, internal working, and best practices for removing data efficiently.

# 7. DELETE

## What is DELETE?

The `DELETE` statement removes one or more existing rows from a table.

Unlike `UPDATE`, which modifies data, DELETE permanently removes the selected records.

Common examples include:

- Deleting a user account
- Removing an old product
- Cancelling a draft post
- Deleting expired sessions
- Removing inactive users
- Clearing temporary data

Once a row is deleted and the transaction is committed, it cannot be recovered unless a backup exists.

---

# Basic Syntax

```sql
DELETE FROM table_name
WHERE condition;
```

Example

```sql
DELETE FROM users
WHERE id = 1;
```

Before

| id | name | city |
|----|------|------|
| 1 | Rahul | Delhi |
| 2 | Prince | Bengaluru |
| 3 | Amit | Mumbai |

After

| id | name | city |
|----|------|------|
| 2 | Prince | Bengaluru |
| 3 | Amit | Mumbai |

The row with

```
id = 1
```

has been removed.

---

# How DELETE Works

Unlike UPDATE,

DELETE completely removes the row.

```
Before

+----+---------+
| 1  | Rahul   |
| 2  | Prince  |
| 3  | Amit    |
+----+---------+

DELETE id = 2

↓

+----+---------+
| 1  | Rahul   |
| 3  | Amit    |
+----+---------+
```

Notice

- No remaining rows are modified.
- The deleted row disappears.
- AUTO_INCREMENT values are **not** renumbered.

---

# Internal Working of DELETE

When MySQL executes a DELETE statement,

it performs several operations internally.

```
Client
    │
    ▼
MySQL Server
    │
    ▼
SQL Parser
    │
    ▼
Permission Check
    │
    ▼
Optimizer
    │
    ▼
Locate Matching Rows
    │
    ▼
Acquire Row Locks
    │
    ▼
Mark/Delete Records
    │
    ▼
Update Indexes
    │
    ▼
Redo Log
    │
    ▼
Commit Transaction
```

---

## SQL Parser

Checks whether the SQL statement is valid.

---

## Permission Check

Verifies whether the current user has DELETE permission.

---

## Optimizer

Determines the fastest method to locate the rows.

If the WHERE column is indexed,

the lookup is extremely fast.

Otherwise,

MySQL performs a full table scan.

---

## Row Lock

Before deleting,

InnoDB acquires an exclusive lock on the matching rows.

Other transactions attempting to modify the same rows must wait.

---

## Delete Row

Internally,

InnoDB doesn't immediately erase the row from disk.

Instead,

it marks the row as deleted.

The actual physical cleanup is handled later by a background process called the **Purge Thread**.

This behavior is possible because of **MVCC (Multi-Version Concurrency Control)**.

We'll study MVCC in detail in the Transactions chapter.

---

## Update Indexes

Every index referencing the deleted row must also be updated.

If a table contains many indexes,

DELETE operations become more expensive.

---

## Redo Log

Before committing,

the delete operation is recorded in the redo log,

allowing MySQL to recover after crashes.

---

# Deleting Multiple Rows

DELETE can remove several rows simultaneously.

Example

```sql
DELETE FROM users
WHERE city = 'Delhi';
```

Suppose

Before

| id | name | city |
|----|------|------|
| 1 | Rahul | Delhi |
| 2 | Amit | Delhi |
| 3 | Prince | Bengaluru |

After

| id | name | city |
|----|------|------|
| 3 | Prince | Bengaluru |

Two rows have been deleted.

---

# DELETE Without WHERE

If the WHERE clause is omitted,

every row is deleted.

```sql
DELETE FROM users;
```

Before

```
100 rows
```

After

```
0 rows
```

The table still exists,

but every record has been removed.

---

# ⚠️ Production Warning

One of the most dangerous SQL statements is

```sql
DELETE FROM users;
```

If executed accidentally,

the table becomes empty.

Always verify

- database
- table
- WHERE clause

before pressing Enter.

A common habit among experienced engineers is

1. Write the WHERE clause first.
2. Verify it using SELECT.
3. Convert it into DELETE.

Example

```sql
SELECT *
FROM users
WHERE id = 10;
```

Verify the returned row.

Then execute

```sql
DELETE FROM users
WHERE id = 10;
```

This simple habit prevents many production incidents.

---

# DELETE Using Multiple Conditions

```sql
DELETE FROM users
WHERE age < 18
AND city = 'Delhi';
```

Only rows satisfying both conditions are removed.

---

# DELETE with LIMIT

MySQL supports deleting a limited number of rows.

```sql
DELETE FROM users
LIMIT 5;
```

Only five rows are deleted.

This is useful for

- cleaning logs
- deleting old records in batches
- reducing lock duration

---

# DELETE in Node.js

```javascript
const sql = `
DELETE FROM users
WHERE id = ?
`;

const [result] = await connection.execute(
    sql,
    [1]
);

console.log(result);
```

Output

```javascript
ResultSetHeader {
    affectedRows: 1,
    warningStatus: 0
}
```

---

# Understanding DELETE Result

Example

```javascript
console.log(result);
```

Output

```javascript
ResultSetHeader {
    affectedRows: 1
}
```

Meaning

| Property | Description |
|----------|-------------|
| affectedRows | Number of rows deleted |

Example

```javascript
if (result.affectedRows === 0) {
    console.log("User not found");
}
```

---

# What Happens If No Row Exists?

Suppose

```sql
DELETE FROM users
WHERE id = 5000;
```

If user

```
5000
```

doesn't exist,

the query succeeds,

but

```javascript
affectedRows
```

equals

```
0
```

This is not considered an error.

---

# DELETE vs TRUNCATE

Both remove data,

but they work differently.

| Feature | DELETE | TRUNCATE |
|----------|---------|-----------|
| Removes selected rows | ✅ | ❌ |
| Supports WHERE | ✅ | ❌ |
| Removes all rows | ✅ | ✅ |
| Transactional (InnoDB) | ✅ | Generally behaves like DDL with implicit commit |
| Fires DELETE triggers | ✅ | ❌ |
| Resets AUTO_INCREMENT | ❌ | ✅ |
| Usually Faster | ❌ | ✅ |

---

## DELETE Example

```sql
DELETE FROM users;
```

Result

```
Rows removed

AUTO_INCREMENT continues
```

Suppose

IDs

```
1
2
3
```

Delete all rows.

Next insert

```
4
```

---

## TRUNCATE Example

```sql
TRUNCATE TABLE users;
```

Result

```
Table emptied

AUTO_INCREMENT reset
```

Next insert

```
1
```

---

# When Should You Use DELETE?

Use DELETE when

- removing one user
- deleting selected orders
- deleting inactive accounts
- deleting records matching conditions

---

# When Should You Use TRUNCATE?

Use TRUNCATE when

- clearing test data
- resetting development databases
- removing every row
- improving speed

Never use TRUNCATE if you only want to remove a subset of rows.

---

# Best Practices

✅ Always verify the WHERE clause.

✅ Execute a SELECT before DELETE.

✅ Use parameterized queries.

✅ Delete data in batches for very large tables.

✅ Keep backups for production databases.

---

# Common Mistakes

❌ Forgetting WHERE.

❌ Deleting production data accidentally.

❌ Assuming deleted IDs will be reused.

❌ Using TRUNCATE when only a few rows should be removed.

❌ Building DELETE statements using string concatenation.

---

# Performance Notes

- Indexed searches make DELETE much faster.
- More indexes mean more work during deletion.
- Large DELETE operations generate substantial redo log activity.
- Deleting millions of rows at once can lock tables and impact performance.
- Batch deletion is often preferable for large datasets.

---

# Interview Notes

### Q1. DELETE vs TRUNCATE?

DELETE

- DML command
- Supports WHERE
- Can delete selected rows
- Doesn't reset AUTO_INCREMENT

TRUNCATE

- DDL command
- Removes every row
- Resets AUTO_INCREMENT
- Usually much faster

---

### Q2. Why is DELETE slower than TRUNCATE?

DELETE removes rows individually,

updates indexes,

writes undo/redo information,

and can fire triggers.

TRUNCATE simply recreates the table structure internally, making it much faster.

---

### Q3. Does DELETE free disk space immediately?

Not necessarily.

InnoDB marks rows for deletion,

and the purge thread later reclaims storage as part of MVCC cleanup.

---

### Q4. What happens if DELETE matches no rows?

The query succeeds,

but

```javascript
affectedRows = 0
```

---

# Quick Revision

- DELETE removes existing rows.
- Always use a WHERE clause unless intentionally deleting every row.
- Verify records with SELECT before deleting.
- `affectedRows` tells how many rows were removed.
- DELETE does **not** reset AUTO_INCREMENT.
- TRUNCATE removes every row and resets AUTO_INCREMENT.
- Large DELETE operations should usually be performed in batches.

---

# CRUD Completed

Congratulations!

You now understand the four fundamental SQL operations:

| Operation | SQL Command | Purpose |
|-----------|-------------|---------|
| Create | INSERT | Add new records |
| Read | SELECT | Retrieve records |
| Update | UPDATE | Modify existing records |
| Delete | DELETE | Remove records |

These four commands form the foundation of almost every backend application.

However, writing CRUD queries correctly is **not enough**.

A secure backend must also protect itself from malicious input and SQL Injection.

The next section introduces **Parameterized Queries and Prepared Statements**, where you'll learn how to safely execute SQL from Node.js using `mysql2`.

# 8. Parameterized Queries

## What are Parameterized Queries?

A parameterized query is a SQL statement where the SQL structure remains fixed, while the data values are supplied separately.

Instead of embedding user input directly into the SQL string, placeholders (`?`) are used.

Example

```sql
SELECT *
FROM users
WHERE email = ?;
```

The actual value is supplied separately by the application.

```javascript
const email = "prince@gmail.com";

await connection.execute(
    `
    SELECT *
    FROM users
    WHERE email = ?
    `,
    [email]
);
```

The SQL statement and the data are transmitted independently.

This approach improves

- Security
- Readability
- Performance
- Maintainability

---

# Why Do We Need Parameterized Queries?

Imagine a login API.

```javascript
const email = req.body.email;

const sql = `
SELECT *
FROM users
WHERE email = '${email}'
`;
```

Suppose

```
email = prince@gmail.com
```

The generated SQL becomes

```sql
SELECT *
FROM users
WHERE email = 'prince@gmail.com';
```

Everything appears fine.

Now suppose an attacker enters

```text
' OR 1=1 --
```

The generated SQL becomes

```sql
SELECT *
FROM users
WHERE email = ''
OR 1=1 --';
```

The condition

```sql
1 = 1
```

is always true.

The attacker may receive every record in the table.

This is called **SQL Injection**.

---

# Safe Alternative

Instead,

write

```javascript
const email = req.body.email;

await connection.execute(
`
SELECT *
FROM users
WHERE email = ?
`,
[email]
);
```

Even if the user enters

```text
' OR 1=1 --
```

MySQL treats it as a normal string.

It is **not executed as SQL**.

---

# Placeholders

The question mark

```sql
?
```

represents a placeholder.

Example

```sql
SELECT *
FROM users
WHERE id = ?;
```

Later,

Node.js supplies

```javascript
[10]
```

The query becomes

```
Find the user whose id equals 10.
```

The SQL statement itself never changes.

---

# Multiple Placeholders

Example

```sql
SELECT *
FROM users
WHERE city = ?
AND age > ?;
```

Node.js

```javascript
await connection.execute(
`
SELECT *
FROM users
WHERE city = ?
AND age > ?
`,
[
    "Delhi",
    25
]
);
```

MySQL automatically substitutes

First placeholder

```
Delhi
```

Second placeholder

```
25
```

The order of values must match the order of placeholders.

---

# INSERT Using Parameters

```javascript
await connection.execute(
`
INSERT INTO users
(
name,
email,
age,
city
)
VALUES
(
?,
?,
?,
?
)
`,
[
    "Prince Raj",
    "prince@gmail.com",
    25,
    "Bengaluru"
]
);
```

---

# UPDATE Using Parameters

```javascript
await connection.execute(
`
UPDATE users
SET city = ?
WHERE id = ?
`,
[
    "Mumbai",
    1
]
);
```

---

# DELETE Using Parameters

```javascript
await connection.execute(
`
DELETE FROM users
WHERE id = ?
`,
[
    5
]
);
```

---

# Benefits of Parameterized Queries

| Benefit | Explanation |
|----------|-------------|
| Prevent SQL Injection | User input is never executed as SQL |
| Cleaner Code | SQL and data remain separate |
| Better Readability | Easier to maintain |
| Faster Repeated Execution | Can reuse prepared statements |
| Safer APIs | Prevents accidental syntax errors |

---

# Parameter Order Matters

Incorrect

```javascript
await connection.execute(
`
SELECT *
FROM users
WHERE city = ?
AND age > ?
`,
[
    30,
    "Delhi"
]
);
```

Here

```
30
```

is assigned to

```
city
```

which is incorrect.

Correct

```javascript
[
"Delhi",
30
]
```

Always match the order of placeholders.

---

# NULL Values

Parameters can also be

```javascript
null
```

Example

```javascript
await connection.execute(
`
INSERT INTO users
(
name,
age
)
VALUES
(
?,
?
)
`,
[
    "Rahul",
    null
]
);
```

Provided the column allows NULL,

MySQL stores

```
NULL
```

---

# Arrays Cannot Replace Placeholders

Incorrect

```javascript
const ids = [1,2,3];

await connection.execute(
`
SELECT *
FROM users
WHERE id IN (?)
`,
[ids]
);
```

Some drivers may expand arrays,

others may not.

For portability,

construct placeholders dynamically when needed.

We'll discuss dynamic SQL later.

---

# Best Practices

✅ Always use placeholders.

✅ Never concatenate user input.

✅ Keep SQL readable.

✅ Validate data before executing queries.

---

# Common Mistakes

❌ Mixing placeholders and string concatenation.

Example

```javascript
const sql =
`
SELECT *
FROM users
WHERE city = '${city}'
AND age > ?
`;
```

Only part of the query is protected.

Always parameterize **every** user-provided value.

---

# Interview Notes

### Why are parameterized queries important?

Because they separate SQL from data,

preventing SQL Injection and improving maintainability.

---

### Does using placeholders make queries slower?

No.

In many cases,

they improve performance,

especially when combined with prepared statements.

---

# Quick Revision

- Parameterized queries separate SQL and user input.
- Use `?` placeholders.
- Values are supplied separately.
- They prevent SQL Injection.
- They improve readability and maintainability.

---

# 9. Prepared Statements

## What is a Prepared Statement?

A prepared statement is a SQL statement that is

1. Parsed
2. Validated
3. Optimized

only once,

and then executed multiple times with different values.

Instead of repeatedly parsing the same SQL,

MySQL can reuse the execution plan.

---

# Why Prepared Statements?

Suppose your application performs

```
Login
```

10,000 times every minute.

Without prepared statements,

MySQL repeatedly performs

```
Receive SQL

↓

Parse

↓

Validate

↓

Optimize

↓

Execute
```

for every request.

With prepared statements,

the first execution performs

```
Parse

↓

Optimize

↓

Store Execution Plan
```

Subsequent executions become

```
Receive Values

↓

Execute Existing Plan
```

Much less work is required.

---

# How mysql2 Uses Prepared Statements

The mysql2 package provides two commonly used methods.

```javascript
connection.query()
```

and

```javascript
connection.execute()
```

Although both execute SQL,

they behave differently.

---

# query()

```javascript
await connection.query(
`
SELECT *
FROM users
WHERE id = 1
`
);
```

Characteristics

- Sends SQL directly.
- Suitable for general-purpose queries.
- Does not automatically use server-side prepared statements for every execution.
- Flexible for dynamic SQL.

---

# execute()

```javascript
await connection.execute(
`
SELECT *
FROM users
WHERE id = ?
`,
[
1
]
);
```

Characteristics

- Uses placeholders.
- Designed for prepared statements.
- Safer for user input.
- Recommended for CRUD APIs.

---

# query() vs execute()

| Feature | query() | execute() |
|----------|----------|-----------|
| Supports placeholders | ✅ | ✅ |
| Prepared statement protocol | ❌ | ✅ |
| SQL Injection protection | Only if placeholders are used correctly | ✅ Recommended |
| Best for repeated CRUD | ❌ | ✅ |
| Performance for repeated execution | Good | Better |

> **Note:** `query()` can still be used safely with placeholders, but `execute()` is specifically designed to use MySQL's prepared statement protocol and is generally the preferred choice for repeated parameterized queries with `mysql2`.

---

# Internal Working

Using

```javascript
connection.execute()
```

```
Node.js

↓

mysql2

↓

Prepare Statement

↓

MySQL Server

↓

Execution Plan

↓

Execute

↓

Return Result
```

On later executions,

only new parameter values are sent,

reducing parsing overhead.

---

# Real Example

```javascript
const sql = `
SELECT *
FROM users
WHERE email = ?
`;

await connection.execute(
sql,
["rahul@gmail.com"]
);

await connection.execute(
sql,
["amit@gmail.com"]
);

await connection.execute(
sql,
["prince@gmail.com"]
);
```

The SQL statement remains identical.

Only the parameter values change.

This is exactly the scenario where prepared statements are most effective.

---
# 10. SQL Injection

## What is SQL Injection?

SQL Injection is one of the oldest and most dangerous security vulnerabilities in web applications.

It occurs when an application treats **user input as part of the SQL query** instead of as plain data.

An attacker can manipulate the SQL statement to

- Read unauthorized data
- Modify records
- Delete records
- Bypass authentication
- Execute unintended SQL commands

In severe cases, SQL Injection can compromise the entire database.

---

# How SQL Injection Happens

Suppose a login API receives

```json
{
    "email": "prince@gmail.com"
}
```

A developer writes

```javascript
const email = req.body.email;

const sql = `
SELECT *
FROM users
WHERE email = '${email}'
`;

const [rows] = await connection.query(sql);
```

If the user enters

```
prince@gmail.com
```

The generated SQL becomes

```sql
SELECT *
FROM users
WHERE email = 'prince@gmail.com';
```

Everything works correctly.

---

# Now Imagine an Attacker

Instead of entering a real email,

the attacker enters

```text
' OR 1=1 --
```

The generated SQL becomes

```sql
SELECT *
FROM users
WHERE email = ''
OR 1=1 --';
```

Let's break this down.

```sql
email = ''
```

OR

```sql
1 = 1
```

Since

```sql
1 = 1
```

is always true,

every row matches.

The attacker receives every user record.

---

# Understanding the Comment

The double hyphen

```sql
--
```

starts a SQL comment.

Everything after it is ignored.

Example

```sql
SELECT *
FROM users
WHERE email = ''
OR 1=1 --';
```

MySQL actually executes

```sql
SELECT *
FROM users
WHERE email = ''
OR 1=1;
```

The trailing quote never matters because it has become part of the comment.

---

# Visualizing the Attack

Application

```
User Input

↓

Build SQL String

↓

Send SQL

↓

MySQL Executes
```

The application cannot distinguish

between

```
SQL
```

and

```
User Input
```

because they have been mixed together.

This is the root cause of SQL Injection.

---

# Another Example

Suppose authentication is written as

```javascript
const sql = `
SELECT *
FROM users
WHERE email='${email}'
AND password='${password}'
`;
```

Normal input

```text
Email

↓

prince@gmail.com

Password

↓

secret123
```

Generated SQL

```sql
SELECT *
FROM users
WHERE email='prince@gmail.com'
AND password='secret123';
```

Now imagine

Password

```text
' OR 1=1 --
```

Generated SQL

```sql
SELECT *
FROM users
WHERE email='prince@gmail.com'
AND password=''
OR 1=1 --';
```

The condition becomes true,

allowing authentication to be bypassed.

---

# Why String Concatenation is Dangerous

Avoid writing code like this.

```javascript
const sql =
`
SELECT *
FROM users
WHERE id = ${id}
`;
```

or

```javascript
const sql =
`
SELECT *
FROM users
WHERE email='${email}'
`;
```

or

```javascript
const sql =
`
DELETE FROM users
WHERE id=${id}
`;
```

These examples directly insert user input into the SQL statement.

---

# Safe Alternative

Instead,

always use placeholders.

```javascript
const sql =
`
SELECT *
FROM users
WHERE email = ?
`;

const [rows] =
await connection.execute(
sql,
[email]
);
```

Now suppose the attacker enters

```text
' OR 1=1 --
```

MySQL receives

SQL

```sql
SELECT *
FROM users
WHERE email = ?
```

Parameters

```
[
"' OR 1=1 --"
]
```

Notice the difference.

The SQL statement never changes.

Only the parameter value changes.

The attacker input is treated as plain text,

not executable SQL.

---

# How MySQL Sees It

Unsafe approach

```
SQL

↓

User Input

↓

SQL Statement Changes
```

Safe approach

```
SQL Statement

↓

Fixed

↓

Parameter Values

↓

Separate
```

The database knows exactly which part is SQL

and which part is data.

---

# SQL Injection Example

Suppose your table contains

| id | email |
|----|--------|
| 1 | rahul@gmail.com |
| 2 | prince@gmail.com |

Attacker input

```text
' OR 1=1 --
```

Unsafe SQL

```sql
SELECT *
FROM users
WHERE email=''
OR 1=1;
```

Output

| id | email |
|----|--------|
| 1 | rahul@gmail.com |
| 2 | prince@gmail.com |

Every record is returned.

---

# Safe Version

```javascript
await connection.execute(
`
SELECT *
FROM users
WHERE email = ?
`,
[
"' OR 1=1 --"
]
);
```

Output

```
0 rows
```

because no user actually has the email

```
' OR 1=1 --
```

---

# SQL Injection in INSERT

Unsafe

```javascript
const sql =
`
INSERT INTO users
(name)
VALUES('${name}')
`;
```

Safe

```javascript
await connection.execute(
`
INSERT INTO users(name)
VALUES(?)
`,
[
name
]
);
```

---

# SQL Injection in UPDATE

Unsafe

```javascript
const sql =
`
UPDATE users
SET city='${city}'
WHERE id=${id}
`;
```

Safe

```javascript
await connection.execute(
`
UPDATE users
SET city = ?
WHERE id = ?
`,
[
city,
id
]
);
```

---

# SQL Injection in DELETE

Unsafe

```javascript
const sql =
`
DELETE FROM users
WHERE id=${id}
`;
```

Safe

```javascript
await connection.execute(
`
DELETE FROM users
WHERE id = ?
`,
[
id
]
);
```

---

# Is Parameterization Enough?

Parameterized queries eliminate the most common SQL injection vulnerabilities, but secure applications should also follow additional practices.

### Validate Input

If

```
age
```

must be a number,

validate it before executing SQL.

```javascript
if (!Number.isInteger(age)) {
    throw new Error("Invalid age");
}
```

---

### Never Trust Client Input

Everything received from

- browsers
- mobile apps
- APIs

should be considered untrusted until validated.

---

### Use the Principle of Least Privilege

Suppose your application only needs to

- SELECT
- INSERT
- UPDATE

There is no reason to grant

```sql
DROP DATABASE
```

permission.

Restrict database users to only the permissions they require.

---

### Store Passwords Securely

Never store passwords like

```text
password123
```

Instead,

store password hashes.

Example

```
$2b$10$...
```

We'll cover password hashing later when building authentication APIs.

---

### Avoid Displaying SQL Errors to Users

Bad

```
Duplicate entry 'abc@gmail.com'
for key users.email
```

Good

```
Something went wrong.
Please try again later.
```

Detailed SQL errors should be logged internally,

not shown to end users.

---

# Best Practices

✅ Always use `execute()` with placeholders.

✅ Validate every input.

✅ Escape identifiers only when necessary (never user values).

✅ Restrict database permissions.

✅ Log errors securely.

✅ Never build SQL using string concatenation.

---

# Common Mistakes

❌ Trusting frontend validation.

❌ Assuming numeric values cannot be exploited.

❌ Mixing placeholders with string interpolation.

❌ Showing raw SQL errors to users.

❌ Giving the application full administrative privileges.

---

# Interview Notes

### Q1. What is SQL Injection?

A security vulnerability where user input is interpreted as SQL commands instead of plain data.

---

### Q2. How do parameterized queries prevent SQL Injection?

They separate SQL statements from user-supplied values, ensuring the input is treated as data rather than executable SQL.

---

### Q3. Is using `execute()` enough to secure an application?

No.

Applications should also validate input, follow least-privilege access, hash passwords, and avoid exposing database errors.

---

### Q4. Is SQL Injection still relevant today?

Yes.

Although frameworks and drivers make prevention easier, SQL Injection remains one of the most common web application vulnerabilities when developers build SQL using string concatenation.

---

# Quick Revision

- SQL Injection occurs when user input changes the intended SQL statement.
- Never concatenate user input into SQL.
- Use placeholders (`?`) with `connection.execute()`.
- Validate all external input.
- Restrict database permissions.
- Do not expose SQL errors to users.

---

# 11. Understanding mysql2 Result Objects

Every SQL query executed using **mysql2** returns data to your Node.js application.

Understanding these return values is essential because backend APIs rely on them to determine whether an operation succeeded and what data should be returned to the client.

The exact structure depends on the type of SQL statement executed.

| SQL Operation | Primary Return Value |
|--------------|----------------------|
| SELECT | Rows + Fields |
| INSERT | ResultSetHeader |
| UPDATE | ResultSetHeader |
| DELETE | ResultSetHeader |

In the next section, we'll examine these objects in detail, explain every property they contain, and learn how to use them effectively in production applications.

# 11. Understanding mysql2 Result Objects

Throughout this chapter, we've executed SQL statements using

```javascript
connection.execute()
```

or

```javascript
connection.query()
```

You may have noticed code like this.

```javascript
const [rows] = await connection.execute(sql);
```

or

```javascript
const [result] = await connection.execute(sql);
```

Why do we sometimes use

```javascript
rows
```

and other times

```javascript
result
```

To answer that, we first need to understand what **mysql2** actually returns.

---

# The Return Value

Every call to

```javascript
await connection.execute(...)
```

returns **two values**.

```javascript
[
    data,
    fields
]
```

or in JavaScript destructuring syntax

```javascript
const [data, fields] = await connection.execute(sql);
```

Notice that

```javascript
data
```

is just a variable name.

It can represent

- rows
- result
- users
- products
- anything you choose.

The variable name has **no special meaning**.

---

# Why Do We Write rows?

Consider

```javascript
const [rows] =
await connection.execute(
`
SELECT *
FROM users
`
);
```

Here

```javascript
rows
```

is simply a descriptive variable name.

We could also write

```javascript
const [users] =
await connection.execute(sql);
```

or

```javascript
const [records] =
await connection.execute(sql);
```

or

```javascript
const [data] =
await connection.execute(sql);
```

All are perfectly valid.

The following statements are equivalent.

```javascript
const [rows] = await connection.execute(sql);
```

```javascript
const [users] = await connection.execute(sql);
```

```javascript
const [records] = await connection.execute(sql);
```

Choose names that make your code easier to understand.

---

# Why Do We Write result?

Consider an INSERT query.

```javascript
const [result] =
await connection.execute(
`
INSERT INTO users(name)
VALUES(?)
`,
["Prince"]
);
```

The returned object is **not** rows.

Instead,

MySQL returns metadata describing the INSERT operation.

Therefore,

calling it

```javascript
result
```

makes more sense.

---

# SELECT Returns Rows

Example

```javascript
const [rows] =
await connection.execute(
`
SELECT *
FROM users
`
);

console.log(rows);
```

Output

```javascript
[
    {
        id: 1,
        name: "Rahul",
        email: "rahul@gmail.com"
    },
    {
        id: 2,
        name: "Prince",
        email: "prince@gmail.com"
    }
]
```

Every object inside the array represents one database row.

---

# INSERT Returns ResultSetHeader

Example

```javascript
const [result] =
await connection.execute(
`
INSERT INTO users(name)
VALUES(?)
`,
["Rahul"]
);

console.log(result);
```

Output

```javascript
ResultSetHeader {
    fieldCount: 0,
    affectedRows: 1,
    insertId: 8,
    info: '',
    serverStatus: 2,
    warningStatus: 0
}
```

Notice

There are **no rows**.

Instead,

MySQL tells us

- how many rows were inserted
- generated ID
- warning count
- server status

---

# UPDATE Returns ResultSetHeader

```javascript
const [result] =
await connection.execute(
`
UPDATE users
SET city = ?
WHERE id = ?
`,
[
    "Delhi",
    1
]
);

console.log(result);
```

Output

```javascript
ResultSetHeader {
    affectedRows: 1,
    changedRows: 1,
    warningStatus: 0
}
```

Again,

there are no returned rows.

Only metadata.

---

# DELETE Returns ResultSetHeader

```javascript
const [result] =
await connection.execute(
`
DELETE FROM users
WHERE id = ?
`,
[
    5
]
);

console.log(result);
```

Output

```javascript
ResultSetHeader {
    affectedRows: 1
}
```

---

# Understanding the Fields Object

Remember

```javascript
const [rows, fields] =
await connection.execute(sql);
```

What is

```javascript
fields
```

The

```javascript
fields
```

array contains metadata describing each returned column.

Example

```javascript
console.log(fields);
```

Output (simplified)

```javascript
[
    {
        name: "id",
        type: "LONG"
    },
    {
        name: "name",
        type: "VARCHAR"
    },
    {
        name: "email",
        type: "VARCHAR"
    }
]
```

This information is useful when building

- ORM libraries
- Admin dashboards
- Generic SQL clients
- Database explorers

In most backend APIs,

you rarely need the

```javascript
fields
```

array.

Therefore,

developers often ignore it.

```javascript
const [rows] =
await connection.execute(sql);
```

---

# Why Doesn't Everyone Destructure fields?

Suppose we write

```javascript
const [rows] =
await connection.execute(sql);
```

This is completely valid.

JavaScript simply ignores the second returned value.

Internally,

the function still returns

```javascript
[
    rows,
    fields
]
```

but we only capture

```javascript
rows
```

---

# Accessing Both Values

If needed,

capture both.

```javascript
const [rows, fields] =
await connection.execute(
`
SELECT *
FROM users
`
);

console.log(rows);

console.log(fields);
```

---

# Destructuring Refresher

Suppose

```javascript
const data = [
    "Apple",
    "Banana"
];
```

Destructuring

```javascript
const [first, second] = data;
```

Results in

```javascript
first = "Apple"

second = "Banana"
```

mysql2 behaves exactly the same way.

It returns

```javascript
[
    rows,
    fields
]
```

and JavaScript destructures the array.

---

# ResultSetHeader Properties

Let's understand the most commonly used properties.

---

## affectedRows

Represents the number of rows affected by the query.

INSERT

```javascript
affectedRows = 1
```

UPDATE

```javascript
affectedRows = 5
```

DELETE

```javascript
affectedRows = 10
```

Very commonly used.

---

## insertId

Available after

```sql
INSERT
```

Example

```javascript
insertId = 42
```

Meaning

The newly inserted row received

```
id = 42
```

Very useful when creating related records.

Example

Insert user

↓

Receive

```
insertId = 15
```

↓

Insert user profile

using

```
user_id = 15
```

---

## changedRows

Available after

```sql
UPDATE
```

Suppose

City already equals

```
Delhi
```

Executing

```sql
UPDATE users
SET city='Delhi'
WHERE id=1;
```

Produces

```javascript
affectedRows = 1

changedRows = 0
```

The row matched,

but nothing changed.

---

## warningStatus

Number of warnings generated.

Usually

```javascript
0
```

---

## fieldCount

For INSERT,

UPDATE,

DELETE

this is usually

```javascript
0
```

SELECT queries instead return row data.

---

# Choosing Good Variable Names

Good

```javascript
const [users] =
await connection.execute(...);
```

Good

```javascript
const [products] =
await connection.execute(...);
```

Good

```javascript
const [orders] =
await connection.execute(...);
```

Good

```javascript
const [result] =
await connection.execute(...);
```

Avoid

```javascript
const [abc] =
await connection.execute(...);
```

Choose names that describe the returned data.

---

# Best Practices

✅ Use descriptive variable names.

✅ Ignore `fields` unless required.

✅ Check

```javascript
affectedRows
```

after UPDATE and DELETE.

✅ Use

```javascript
insertId
```

after INSERT.

---

# Common Mistakes

❌ Assuming every query returns rows.

❌ Ignoring

```javascript
affectedRows
```

after UPDATE.

❌ Expecting

```javascript
insertId
```

after SELECT.

❌ Using confusing variable names.

---

# Interview Notes

### Q1. Why do we write

```javascript
const [rows]
```

instead of

```javascript
const rows
```

Because mysql2 returns an array

```javascript
[
rows,
fields
]
```

JavaScript array destructuring extracts the first element.

---

### Q2. Can we rename

```javascript
rows
```

to something else?

Yes.

```javascript
const [users] =
await connection.execute(sql);
```

is perfectly valid.

---

### Q3. What does

```javascript
affectedRows
```

represent?

The number of rows modified by INSERT,

UPDATE,

or DELETE.

---

### Q4. When is

```javascript
insertId
```

available?

After successful INSERT operations involving AUTO_INCREMENT columns.

---

# Quick Revision

- `execute()` returns an array containing **data** and **field metadata**.
- For `SELECT`, the first value is an array of row objects.
- For `INSERT`, `UPDATE`, and `DELETE`, the first value is a `ResultSetHeader`.
- `rows` and `result` are just variable names—you can choose any meaningful name.
- `fields` contains metadata about returned columns and is rarely needed in everyday CRUD APIs.

---

# 12. CRUD Flow Inside MySQL

Now that you understand every CRUD operation individually, let's connect everything together and see how a SQL statement travels from your Node.js application to the MySQL storage engine and back. This end-to-end flow will help you understand what happens "behind the scenes" whenever your application executes a query.

# 12. CRUD Flow Inside MySQL

So far, we've learned how to perform

- INSERT
- SELECT
- UPDATE
- DELETE

using Node.js and MySQL.

But what actually happens when you execute

```javascript
await connection.execute(sql, values);
```

Understanding this internal flow is extremely valuable because it helps explain

- why indexes improve performance
- why some queries are slow
- why locks occur
- why transactions exist
- how MySQL processes every request

---

# High-Level Architecture

A typical request follows this path.

```
Node.js Application
        │
        ▼
mysql2 Driver
        │
        ▼
TCP Connection
        │
        ▼
MySQL Server
        │
        ▼
SQL Parser
        │
        ▼
Permission Check
        │
        ▼
Query Optimizer
        │
        ▼
Storage Engine (InnoDB)
        │
        ▼
Disk / Buffer Pool
        │
        ▼
Result
        │
        ▼
Node.js Application
```

Let's study every stage.

---

# Step 1 — Node.js Builds the Query

Suppose we execute

```javascript
const [users] =
await connection.execute(
`
SELECT *
FROM users
WHERE id = ?
`,
[
5
]
);
```

Your application doesn't directly communicate with the database files.

Instead,

it sends the SQL statement to the mysql2 driver.

---

# Step 2 — mysql2 Driver

The mysql2 package acts as a translator between

```
JavaScript
```

and

```
MySQL Protocol
```

Responsibilities include

- Managing TCP connections
- Sending SQL statements
- Sending parameter values
- Receiving responses
- Converting binary protocol into JavaScript objects

```
Node.js

↓

mysql2

↓

MySQL
```

Without mysql2,

Node.js wouldn't know how to communicate with MySQL.

---

# Step 3 — TCP Connection

mysql2 sends packets over TCP.

```
Node.js

↓

TCP

↓

3306

↓

MySQL Server
```

Default MySQL port

```
3306
```

For Amazon RDS,

the process is identical.

The only difference is that the database resides on another machine.

---

# Step 4 — SQL Parser

The parser checks

- SQL syntax
- keywords
- table names
- column names

Example

```sql
SELEC *
FROM users;
```

The parser immediately rejects it.

```
Syntax Error
```

No further processing occurs.

---

# Step 5 — Permission Check

MySQL verifies

Does this database user have permission?

Example

```
SELECT
```

allowed

↓

Continue

```
DROP DATABASE
```

not allowed

↓

Permission denied

This prevents unauthorized operations.

---

# Step 6 — Query Optimizer

The optimizer decides

**How should this query be executed?**

Example

```sql
SELECT *
FROM users
WHERE id = 5;
```

Suppose

```
id
```

is indexed.

Optimizer chooses

```
Index Lookup
```

instead of

```
Full Table Scan
```

Another query

```sql
SELECT *
FROM users
WHERE city = 'Delhi';
```

Suppose

```
city
```

is not indexed.

Optimizer chooses

```
Full Table Scan
```

The optimizer's decisions have a major impact on performance.

---

# Step 7 — Storage Engine

After the optimizer decides the execution plan,

the query is handed to the storage engine.

In MySQL,

the default storage engine is

```
InnoDB
```

The storage engine is responsible for

- Reading rows
- Writing rows
- Locking
- Transactions
- Recovery
- Index management

Think of InnoDB as the component that actually interacts with the data.

---

# Step 8 — Buffer Pool

Before reading from disk,

InnoDB checks its memory cache,

called the **Buffer Pool**.

```
Request

↓

Buffer Pool

↓

Found?

↓

Yes → Return Data

↓

No

↓

Read From Disk

↓

Store In Buffer Pool

↓

Return Data
```

Because RAM is much faster than disk,

queries that hit the buffer pool are significantly faster.

---

# Step 9 — Disk Access

If the requested page isn't in memory,

InnoDB reads it from disk.

```
SSD

↓

Database Pages

↓

Memory

↓

MySQL
```

Disk access is one of the slowest parts of query execution.

That's why repeated queries often become faster after the data has been cached.

---

# Step 10 — Execute the Operation

Depending on the SQL statement,

different work is performed.

### SELECT

Reads rows.

### INSERT

Adds new rows.

### UPDATE

Modifies existing rows.

### DELETE

Marks rows for deletion.

Each operation may also involve

- updating indexes
- writing redo logs
- acquiring locks

---

# Step 11 — Result Generation

After execution,

MySQL prepares a response.

For SELECT

```javascript
[
rows,
fields
]
```

For INSERT

```javascript
ResultSetHeader
```

For UPDATE

```javascript
ResultSetHeader
```

For DELETE

```javascript
ResultSetHeader
```

---

# Step 12 — mysql2 Converts the Result

mysql2 receives the response

and converts it into JavaScript objects.

Example

MySQL

```
Binary Protocol
```

↓

mysql2

↓

JavaScript Objects

↓

```javascript
[
{
id:1,
name:"Prince"
}
]
```

---

# Step 13 — Node.js Receives the Result

Finally,

your code receives

```javascript
const [rows] =
await connection.execute(...);
```

and can use it however it likes.

Example

```javascript
res.json(rows);
```

or

```javascript
console.log(rows);
```

---

# Complete Flow Diagram

```
Client Request
       │
       ▼
Express Route
       │
       ▼
Controller
       │
       ▼
mysql2 Driver
       │
       ▼
TCP Connection
       │
       ▼
MySQL Server
       │
       ▼
SQL Parser
       │
       ▼
Permission Check
       │
       ▼
Query Optimizer
       │
       ▼
InnoDB Storage Engine
       │
       ▼
Buffer Pool
       │
       ▼
Disk (If Needed)
       │
       ▼
Execute Query
       │
       ▼
Prepare Result
       │
       ▼
mysql2
       │
       ▼
JavaScript Object
       │
       ▼
Express Response
```

---

# Example Walkthrough

Suppose your API executes

```javascript
const [users] =
await connection.execute(
`
SELECT *
FROM users
WHERE id = ?
`,
[
7
]
);
```

Internally

```
Express

↓

mysql2

↓

TCP Packet

↓

MySQL

↓

Parser

↓

Optimizer

↓

Index Lookup

↓

Buffer Pool

↓

Disk (If Necessary)

↓

Read Row

↓

Return Result

↓

mysql2

↓

JavaScript Object

↓

users Variable
```

All of this typically happens in just a few milliseconds.

---

# Where Performance Problems Can Occur

```
Slow Network

↓

Connection Delay

↓

Parser

↓

Poor Execution Plan

↓

Missing Index

↓

Disk Reads

↓

Lock Wait

↓

Large Result Set

↓

Serialization

↓

Node.js
```

Understanding this pipeline makes it much easier to diagnose slow queries.

---

# Best Practices

✅ Use indexes on frequently searched columns.

✅ Use parameterized queries.

✅ Return only the required columns instead of `SELECT *`.

✅ Keep transactions short.

✅ Avoid unnecessary disk reads by designing efficient queries.

---

# Common Mistakes

❌ Assuming MySQL reads directly from disk every time.

❌ Ignoring the query optimizer.

❌ Fetching unnecessary columns.

❌ Believing mysql2 executes SQL itself—it only communicates with MySQL.

❌ Thinking indexes always improve performance (they also make INSERT, UPDATE, and DELETE more expensive).

---

# Interview Notes

### Q1. Does Node.js execute SQL?

No.

Node.js sends SQL to the MySQL server using the mysql2 driver.

---

### Q2. What is the role of mysql2?

It implements the MySQL client protocol, manages connections, sends queries, and converts MySQL responses into JavaScript objects.

---

### Q3. What is the Buffer Pool?

The InnoDB Buffer Pool is an in-memory cache of database pages. If requested data is already in memory, MySQL avoids slower disk I/O.

---

### Q4. Which component decides whether to use an index?

The Query Optimizer.

---

### Q5. Which component actually reads and writes the data?

The InnoDB Storage Engine.

---

# Quick Revision

- `mysql2` is the client driver between Node.js and MySQL.
- Queries travel over TCP (typically port 3306).
- The SQL Parser validates syntax.
- Permission checks verify database privileges.
- The Query Optimizer chooses the execution plan.
- InnoDB performs the actual data operations.
- The Buffer Pool caches frequently accessed data.
- Results are converted into JavaScript objects by mysql2 before being returned to your application.

---

# Phase 1 Complete 🎉

You have now mastered the fundamentals of MySQL:

- ✅ Databases and Tables
- ✅ Data Types
- ✅ CREATE DATABASE / TABLE
- ✅ INSERT
- ✅ SELECT
- ✅ UPDATE
- ✅ DELETE
- ✅ Parameterized Queries
- ✅ Prepared Statements
- ✅ SQL Injection Prevention
- ✅ mysql2 Result Objects
- ✅ End-to-End CRUD Flow Inside MySQL

You now have a solid foundation to build real-world backend applications.

---

# Phase 2 Preview — Querying Data Like a Pro

In Phase 2, you'll move beyond basic CRUD and learn how to retrieve data efficiently using SQL. Topics include:

- WHERE clause
- Comparison operators
- Logical operators (`AND`, `OR`, `NOT`)
- `IN`, `BETWEEN`, `LIKE`
- `ORDER BY`
- `LIMIT` and pagination
- Aggregate functions (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`)
- `GROUP BY`
- `HAVING`
- Aliases
- Real-world filtering and reporting queries

By the end of Phase 2, you'll be able to write the majority of SQL queries used in production backend services.
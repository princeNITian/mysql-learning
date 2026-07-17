# 04. Filtering Data

# Introduction

So far, we've learned how to

- Create databases
- Create tables
- Insert data
- Read all data
- Update data
- Delete data

However, in real-world applications, we almost never retrieve **every row** from a table.

Instead, we retrieve **only the rows we need**.

Examples

- Find a user by email.
- Find products cheaper than ₹10,000.
- Find employees working in Bengaluru.
- Find orders placed today.
- Find students scoring above 90%.
- Find active users.
- Find products whose stock is less than 5.

This process is called **Filtering Data**.

Filtering allows MySQL to return only the records that satisfy a given condition.

---

# Why is Filtering Important?

Imagine an e-commerce website having

```
Products Table

10,000,000 rows
```

Suppose a customer searches

```
iPhone
```

Should the database return

```
10 million products?
```

Obviously not.

Instead,

it should return

```
Only products matching "iPhone"
```

Similarly,

when a user logs in,

the backend should not retrieve

```
Every user
```

Instead,

it should retrieve

```
Exactly one user
```

using the email.

Efficient filtering is one of the most important skills in SQL.

---

# Real World Examples

## Login API

```sql
SELECT *
FROM users
WHERE email = 'prince@gmail.com';
```

---

## Search Product

```sql
SELECT *
FROM products
WHERE name = 'iPhone 16';
```

---

## Dashboard

```sql
SELECT *
FROM orders
WHERE status = 'Pending';
```

---

## HR Portal

```sql
SELECT *
FROM employees
WHERE salary > 100000;
```

---

## Banking

```sql
SELECT *
FROM transactions
WHERE account_number = '123456789';
```

Every backend application performs filtering continuously.

---

# Sample Database

Throughout this chapter,

we'll use the following table.

## users

| id | name | age | city | salary | email |
|----|------|-----|------|---------|-----------------------|
| 1 | Prince | 25 | Bengaluru | 80000 | prince@gmail.com |
| 2 | Rahul | 28 | Delhi | 90000 | rahul@gmail.com |
| 3 | Amit | 32 | Mumbai | 120000 | amit@gmail.com |
| 4 | Sneha | 24 | Pune | 70000 | sneha@gmail.com |
| 5 | Priya | 29 | Hyderabad | 95000 | priya@gmail.com |
| 6 | Karan | 31 | Jaipur | 110000 | karan@gmail.com |
| 7 | Neha | 26 | Kolkata | 85000 | neha@gmail.com |
| 8 | Rohit | 30 | Ahmedabad | 100000 | rohit@gmail.com |

Every query in this chapter will use this table.

---

# The WHERE Clause

Filtering in SQL is performed using the **WHERE** clause.

Basic syntax

```sql
SELECT column_name
FROM table_name
WHERE condition;
```

General form

```
SELECT

↓

FROM

↓

WHERE

↓

Return Matching Rows
```

Example

```sql
SELECT *
FROM users
WHERE city = 'Delhi';
```

Output

| id | name | age | city |
|----|------|-----|------|
| 2 | Rahul | 28 | Delhi |

Notice

Only one row is returned.

---

# Without WHERE

```sql
SELECT *
FROM users;
```

Output

```
8 rows
```

---

# With WHERE

```sql
SELECT *
FROM users
WHERE city = 'Delhi';
```

Output

```
1 row
```

The WHERE clause filters out unwanted rows.

---

# Internal Working of WHERE

Let's understand what MySQL does internally.

```
Client

↓

Node.js

↓

mysql2

↓

MySQL Server

↓

SQL Parser

↓

Permission Check

↓

Query Optimizer

↓

Locate Rows

↓

Evaluate WHERE Condition

↓

Return Matching Rows
```

Notice

The WHERE clause is **not** evaluated first.

MySQL first

- validates SQL
- checks permissions
- builds an execution plan

Only then does it evaluate the filtering condition.

---

# Step 1 — SQL Parser

Example

```sql
SELECT *
FROM users
WHERE city = 'Delhi';
```

The parser checks

- SQL syntax
- keywords
- table names
- column names

Incorrect SQL

```sql
SELEC *
FROM users;
```

Results in

```
Syntax Error
```

---

# Step 2 — Permission Check

MySQL verifies whether the current user has

```
SELECT
```

permission.

If permission is denied,

query execution stops immediately.

---

# Step 3 — Query Optimizer

The optimizer decides

```
How should this query be executed?
```

Example

```sql
SELECT *
FROM users
WHERE id = 5;
```

If

```
id
```

is indexed,

the optimizer chooses

```
Index Lookup
```

Otherwise,

it performs

```
Full Table Scan
```

We'll study indexes in depth later.

---

# Step 4 — Row Evaluation

Suppose the table contains

| id | city |
|----|------|
|1|Bengaluru|
|2|Delhi|
|3|Mumbai|
|4|Delhi|

Query

```sql
SELECT *
FROM users
WHERE city = 'Delhi';
```

Internally

```
Row 1

↓

Bengaluru == Delhi ?

↓

False

↓

Discard


Row 2

↓

Delhi == Delhi ?

↓

True

↓

Return


Row 3

↓

Mumbai == Delhi ?

↓

False

↓

Discard


Row 4

↓

Delhi == Delhi ?

↓

True

↓

Return
```

Only rows satisfying the condition are returned.

---

# Conceptually

Think of WHERE as a filter.

```
Entire Table

↓

Condition

↓

Matching Rows

↓

Application
```

The original table remains unchanged.

WHERE only controls **what is returned**, not what is stored.

---

# WHERE Does NOT Modify Data

This query

```sql
SELECT *
FROM users
WHERE age > 25;
```

does **not**

- update data
- delete data
- insert data

It simply returns matching rows.

This is why SELECT is considered a **read-only operation**.

---

# Multiple Conditions

The WHERE clause can contain

- one condition
- multiple conditions
- nested conditions
- function calls
- expressions

Example

```sql
SELECT *
FROM users
WHERE age > 25
AND city = 'Delhi';
```

We'll study each of these throughout this chapter.

---

# Best Practices

✅ Always filter using indexed columns when possible.

✅ Retrieve only the rows you actually need.

✅ Avoid filtering in application code when SQL can do it.

---

# Common Mistakes

❌ Retrieving the entire table and filtering in JavaScript.

Bad

```javascript
const [users] = await connection.execute(
`SELECT * FROM users`
);

const result = users.filter(user => user.city === "Delhi");
```

Good

```javascript
const [users] = await connection.execute(
`
SELECT *
FROM users
WHERE city = ?
`,
["Delhi"]
);
```

Filtering should happen inside MySQL, not inside Node.js.

---

# Quick Revision

- Filtering retrieves only the required rows.
- SQL uses the `WHERE` clause for filtering.
- WHERE does not modify data.
- MySQL evaluates the WHERE condition after parsing, permission checks, and optimization.
- Efficient filtering reduces network traffic and improves performance.

---

The next section introduces **Comparison Operators**, which form the foundation of almost every WHERE clause.

# Comparison Operators

The `WHERE` clause becomes truly powerful when combined with **comparison operators**.

Comparison operators compare two values and determine whether a condition is

- True
- False

Only rows for which the condition evaluates to **TRUE** are returned.

---

# Available Comparison Operators

| Operator | Meaning |
|----------|---------|
| = | Equal to |
| != | Not equal to |
| <> | Not equal to (standard SQL) |
| > | Greater than |
| < | Less than |
| >= | Greater than or equal to |
| <= | Less than or equal to |

These operators work with

- Numbers
- Strings
- Dates
- DateTime
- Time
- Boolean values

---

# How MySQL Evaluates Comparisons

Consider

```sql
SELECT *
FROM users
WHERE age > 25;
```

Internally,

MySQL evaluates every row.

```
Row 1

Age = 25

↓

25 > 25 ?

↓

False

↓

Discard


Row 2

Age = 28

↓

28 > 25 ?

↓

True

↓

Return


Row 3

Age = 32

↓

32 > 25 ?

↓

True

↓

Return
```

Notice

Every row is tested independently.

---

# Equality Operator (=)

The most commonly used comparison operator.

Syntax

```sql
SELECT *
FROM table_name
WHERE column = value;
```

Example

```sql
SELECT *
FROM users
WHERE city = 'Delhi';
```

Output

| id | name | city |
|----|------|------|
|2|Rahul|Delhi|

---

Another example

```sql
SELECT *
FROM users
WHERE id = 3;
```

Output

| id | name |
|----|------|
|3|Amit|

---

# Internal Working

Suppose the table contains

```
Delhi

Mumbai

Delhi

Pune
```

Query

```sql
WHERE city = 'Delhi'
```

Evaluation

```
Delhi

↓

Match

↓

Return


Mumbai

↓

No Match

↓

Discard


Delhi

↓

Match

↓

Return


Pune

↓

No Match

↓

Discard
```

---

# Numeric Comparison

```sql
SELECT *
FROM users
WHERE salary = 100000;
```

Output

| id | name | salary |
|----|------|---------|
|8|Rohit|100000|

---

# String Comparison

```sql
SELECT *
FROM users
WHERE name = 'Prince';
```

Output

| id | name |
|----|------|
|1|Prince|

---

# Date Comparison

Suppose

```text
created_at
```

contains

```
2026-07-10
```

Query

```sql
SELECT *
FROM orders
WHERE created_at = '2026-07-10';
```

Returns all orders placed on that date.

---

# Not Equal (!=)

Returns rows that are **not equal** to the given value.

Example

```sql
SELECT *
FROM users
WHERE city != 'Delhi';
```

Output

| name | city |
|------|------|
|Prince|Bengaluru|
|Amit|Mumbai|
|Sneha|Pune|
|Priya|Hyderabad|
|Karan|Jaipur|
|Neha|Kolkata|
|Rohit|Ahmedabad|

Only Delhi is excluded.

---

# Not Equal (<>)

Standard SQL also supports

```sql
<>
```

Example

```sql
SELECT *
FROM users
WHERE city <> 'Delhi';
```

Output

Exactly the same as

```sql
!=
```

In MySQL,

both operators are equivalent.

---

# Greater Than (>)

Returns rows having values greater than the specified value.

Example

```sql
SELECT *
FROM users
WHERE age > 28;
```

Output

| name | age |
|------|-----|
|Amit|32|
|Priya|29|
|Karan|31|
|Rohit|30|

---

Another example

```sql
SELECT *
FROM users
WHERE salary > 90000;
```

Output

```
95000

100000

110000

120000
```

---

# Less Than (<)

Example

```sql
SELECT *
FROM users
WHERE age < 26;
```

Output

| name | age |
|------|-----|
|Prince|25|
|Sneha|24|

---

# Greater Than or Equal (>=)

Example

```sql
SELECT *
FROM users
WHERE salary >= 100000;
```

Output

| name | salary |
|------|---------|
|Amit|120000|
|Karan|110000|
|Rohit|100000|

Notice

```
100000
```

is included.

---

# Less Than or Equal (<=)

Example

```sql
SELECT *
FROM users
WHERE age <= 25;
```

Output

| name | age |
|------|-----|
|Prince|25|
|Sneha|24|

---

# Comparison on Strings

Strings can also be compared.

Example

```sql
SELECT *
FROM users
WHERE name > 'M';
```

This comparison is based on the column's collation and lexical ordering, not simply the first character.

In practice,

string range comparisons are less common than equality or pattern matching (`LIKE`).

---

# Comparison on Dates

Dates are naturally comparable.

Example

```sql
SELECT *
FROM orders
WHERE created_at > '2026-07-10';
```

Returns all orders created after July 10.

---

# Comparison on DateTime

```sql
SELECT *
FROM orders
WHERE created_at >= '2026-07-10 10:00:00';
```

Very common in

- logs
- payments
- order history
- audit tables

---

# Node.js Example

```javascript
const minAge = 25;

const [users] =
await connection.execute(
`
SELECT *
FROM users
WHERE age > ?
`,
[
minAge
]
);

console.log(users);
```

Notice

The comparison operator remains inside SQL,

while the value comes from JavaScript.

---

# Another Node.js Example

```javascript
const city = "Delhi";

const [users] =
await connection.execute(
`
SELECT *
FROM users
WHERE city = ?
`,
[
city
]
);
```

---

# Internal Execution Flow

Suppose

```sql
SELECT *
FROM users
WHERE salary > 90000;
```

Execution

```
Parser

↓

Optimizer

↓

Locate Rows

↓

Compare Salary

↓

TRUE ?

↓

Return

↓

FALSE ?

↓

Discard
```

Every returned row has satisfied the comparison.

---

# Index Optimization

Suppose

```
salary
```

is indexed.

Query

```sql
WHERE salary > 90000
```

MySQL can jump directly to the first matching index entry.

```
Index

↓

90001

↓

95000

↓

100000

↓

110000

↓

120000
```

Instead of scanning every row,

MySQL scans only the relevant portion of the index.

We'll explore this in detail during the Indexes phase.

---

# Best Practices

✅ Compare values using their correct data types.

✅ Parameterize comparison values.

✅ Index columns that are frequently compared.

---

# Common Mistakes

❌ Comparing numbers as strings without understanding implicit conversions.

❌ Building comparison queries using string concatenation.

❌ Using floating-point values for exact equality when precision matters.

---

# Interview Notes

### Q1. What comparison operators does MySQL support?

```
=
!=
<>
>
<
>=
<=
```

---

### Q2. Are `!=` and `<>` different?

No.

In MySQL,

both mean **not equal**.

---

### Q3. Can comparison operators work on dates?

Yes.

Dates and DateTime values can be compared directly.

---

### Q4. Why are indexed comparisons faster?

Because MySQL can use the index to locate matching rows instead of scanning the entire table.

---

# Quick Revision

- Comparison operators evaluate conditions row by row.
- Only rows where the condition is **TRUE** are returned.
- Operators work with numbers, strings, and dates.
- `=` checks equality.
- `!=` and `<>` both mean "not equal."
- `>`, `<`, `>=`, and `<=` perform range comparisons.
- Parameterized queries should always be used with comparison values.

---

The next section introduces **Logical Operators (`AND`, `OR`, and `NOT`)**, allowing multiple conditions to be combined into powerful filtering expressions.

# Logical Operators

So far, we've learned how to filter data using a **single condition**.

Example

```sql
SELECT *
FROM users
WHERE age > 25;
```

But in real-world applications, one condition is rarely enough.

Consider the following scenarios.

- Find employees older than 25 **and** working in Bengaluru.
- Find products that belong to Electronics **or** Mobiles.
- Find users who are **not** inactive.
- Find orders that are **paid and delivered**.
- Find students with marks above 90 **or** rank less than 10.

To combine multiple conditions, SQL provides **Logical Operators**.

---

# Available Logical Operators

| Operator | Meaning |
|----------|---------|
| AND | All conditions must be TRUE |
| OR | At least one condition must be TRUE |
| NOT | Reverses a condition |

---

# Visual Overview

Suppose we have

```
Condition A

Age > 25

Condition B

City = Delhi
```

Using

```
AND
```

Both must be true.

```
Condition A

TRUE

AND

Condition B

TRUE

↓

Return Row
```

Using

```
OR
```

Only one must be true.

```
TRUE

OR

FALSE

↓

Return Row
```

Using

```
NOT
```

Reverses the result.

```
TRUE

↓

NOT

↓

FALSE
```

---

# AND Operator

The AND operator returns rows only if **every condition** evaluates to TRUE.

Syntax

```sql
SELECT *
FROM table_name
WHERE condition1
AND condition2;
```

---

## Example

```sql
SELECT *
FROM users
WHERE city = 'Delhi'
AND age > 25;
```

Output

| id | name | age | city |
|----|------|-----|------|
|2|Rahul|28|Delhi|

---

# Internal Evaluation

Suppose

| Name | Age | City |
|------|-----|------|
|Prince|25|Bengaluru|
|Rahul|28|Delhi|
|Amit|32|Mumbai|

Query

```sql
WHERE age > 25
AND city = 'Delhi'
```

Evaluation

```
Prince

Age >25 ?

False

↓

Discard


Rahul

Age >25 ?

True

↓

City = Delhi ?

True

↓

Return


Amit

Age >25 ?

True

↓

City = Delhi ?

False

↓

Discard
```

Only Rahul satisfies both conditions.

---

# Another Example

```sql
SELECT *
FROM users
WHERE salary > 90000
AND age < 30;
```

Output

| Name | Salary | Age |
|------|---------|-----|
|Priya|95000|29|

---

# OR Operator

The OR operator returns rows when **at least one condition** is TRUE.

Syntax

```sql
SELECT *
FROM users
WHERE condition1
OR condition2;
```

---

## Example

```sql
SELECT *
FROM users
WHERE city = 'Delhi'
OR city = 'Mumbai';
```

Output

| Name | City |
|------|------|
|Rahul|Delhi|
|Amit|Mumbai|

---

# Internal Evaluation

```
Prince

Delhi ?

False

OR

Mumbai ?

False

↓

Discard


Rahul

Delhi ?

True

↓

Return


Amit

Mumbai ?

True

↓

Return
```

Notice

Only one condition must succeed.

---

# Another Example

```sql
SELECT *
FROM users
WHERE age < 25
OR salary > 100000;
```

Output

| Name | Age | Salary |
|------|-----|---------|
|Sneha|24|70000|
|Amit|32|120000|
|Karan|31|110000|

---

# NOT Operator

NOT reverses a condition.

Syntax

```sql
SELECT *
FROM users
WHERE NOT condition;
```

---

## Example

```sql
SELECT *
FROM users
WHERE NOT city = 'Delhi';
```

Output

Returns every user except Rahul.

---

Equivalent query

```sql
SELECT *
FROM users
WHERE city != 'Delhi';
```

---

# Internal Evaluation

```
Delhi

↓

City = Delhi ?

True

↓

NOT

↓

False

↓

Discard


Mumbai

↓

False

↓

NOT

↓

True

↓

Return
```

---

# Combining AND and OR

Multiple logical operators can appear together.

Example

```sql
SELECT *
FROM users
WHERE city = 'Delhi'
OR city = 'Mumbai'
AND age > 30;
```

Many beginners expect

```
(Delhi OR Mumbai)
AND Age >30
```

But SQL does **not** evaluate it that way.

It follows operator precedence.

---

# Operator Precedence

SQL evaluates operators in the following order.

```
()

↓

NOT

↓

AND

↓

OR
```

Therefore,

```sql
WHERE city='Delhi'
OR city='Mumbai'
AND age>30
```

is interpreted as

```sql
WHERE city='Delhi'
OR (
city='Mumbai'
AND age>30
)
```

NOT

```sql
(
city='Delhi'
OR city='Mumbai'
)
AND age>30
```

This is one of the most common SQL interview questions.

---

# Using Parentheses

Always use parentheses when mixing AND and OR.

Good

```sql
SELECT *
FROM users
WHERE
(
city='Delhi'
OR city='Mumbai'
)
AND age > 30;
```

Now the intent is crystal clear.

---

# Real-World Login Example

```sql
SELECT *
FROM users
WHERE email = ?
AND password = ?;
```

Both conditions must match.

---

# Product Search

```sql
SELECT *
FROM products
WHERE category = 'Mobile'
AND price < 50000;
```

---

# Order Dashboard

```sql
SELECT *
FROM orders
WHERE status = 'Pending'
OR status = 'Processing';
```

---

# Employee Portal

```sql
SELECT *
FROM employees
WHERE department = 'Engineering'
AND salary > 150000;
```

---

# Node.js Example

```javascript
const city = "Delhi";
const minAge = 25;

const [users] =
await connection.execute(
`
SELECT *
FROM users
WHERE city = ?
AND age > ?
`,
[
city,
minAge
]
);

console.log(users);
```

---

# Another Node.js Example

```javascript
const city1 = "Delhi";
const city2 = "Mumbai";

const [users] =
await connection.execute(
`
SELECT *
FROM users
WHERE city = ?
OR city = ?
`,
[
city1,
city2
]
);
```

---

# Internal Execution Flow

Suppose

```sql
SELECT *
FROM users
WHERE age > 25
AND city='Delhi';
```

Execution

```
Parser

↓

Optimizer

↓

Read Row

↓

Age >25 ?

↓

False

↓

Discard


↓

True

↓

City = Delhi ?

↓

True

↓

Return
```

Every row must satisfy the complete logical expression.

---

# Truth Table

## AND

| A | B | Result |
|---|---|--------|
| True | True | True |
| True | False | False |
| False | True | False |
| False | False | False |

---

## OR

| A | B | Result |
|---|---|--------|
| True | True | True |
| True | False | True |
| False | True | True |
| False | False | False |

---

## NOT

| A | Result |
|---|--------|
| True | False |
| False | True |

---

# Best Practices

✅ Keep conditions simple.

✅ Use parentheses whenever AND and OR are mixed.

✅ Parameterize every user input.

✅ Put highly selective indexed conditions in the WHERE clause.

---

# Common Mistakes

❌ Forgetting parentheses.

❌ Assuming SQL evaluates left to right.

❌ Writing unnecessary nested conditions.

❌ Filtering in JavaScript instead of SQL.

---

# Interview Notes

### Q1. What are the logical operators in SQL?

```
AND

OR

NOT
```

---

### Q2. Which operator has higher precedence?

```
NOT

↓

AND

↓

OR
```

---

### Q3. Why should parentheses be used?

They make the evaluation order explicit, improve readability, and prevent subtle logic bugs.

---

### Q4. What is the difference between AND and OR?

- **AND** requires every condition to be true.
- **OR** requires at least one condition to be true.

---

# Quick Revision

- `AND` returns rows only when all conditions are true.
- `OR` returns rows when any condition is true.
- `NOT` reverses a condition.
- SQL evaluates `NOT` before `AND`, and `AND` before `OR`.
- Parentheses should be used to clearly express complex filtering logic.
- Logical operators are used in almost every production SQL query.

---

The next section introduces the **IN operator**, which provides a cleaner and more efficient way to compare a column against multiple possible values without writing long chains of `OR` conditions.

# IN Operator

In the previous section, we learned how to combine multiple conditions using the `OR` operator.

Example

```sql
SELECT *
FROM users
WHERE city = 'Delhi'
OR city = 'Mumbai'
OR city = 'Pune';
```

This works perfectly.

However, imagine checking for **20 different cities**.

The query quickly becomes difficult to read and maintain.

SQL provides the **IN** operator to solve this problem.

---

# What is the IN Operator?

The `IN` operator checks whether a value exists in a list of values.

Instead of writing multiple `OR` conditions,

you provide a list.

General syntax

```sql
SELECT column_list
FROM table_name
WHERE column_name IN (value1, value2, value3);
```

Think of it as asking

> "Is this value present inside the given list?"

---

# OR vs IN

Without IN

```sql
SELECT *
FROM users
WHERE city = 'Delhi'
OR city = 'Mumbai'
OR city = 'Pune';
```

Using IN

```sql
SELECT *
FROM users
WHERE city IN ('Delhi', 'Mumbai', 'Pune');
```

Both queries return exactly the same result.

The second query is cleaner and easier to maintain.

---

# Internal Representation

Conceptually,

MySQL treats

```sql
WHERE city IN ('Delhi','Mumbai','Pune')
```

similar to

```sql
WHERE
city='Delhi'
OR city='Mumbai'
OR city='Pune'
```

However,

the optimizer may execute them differently for better performance.

---

# Example 1

Find users living in Delhi or Mumbai.

```sql
SELECT *
FROM users
WHERE city IN ('Delhi', 'Mumbai');
```

Output

| id | name | city |
|----|------|------|
|2|Rahul|Delhi|
|3|Amit|Mumbai|

---

# Example 2

Find users whose age is

25,

28,

or

31.

```sql
SELECT *
FROM users
WHERE age IN (25, 28, 31);
```

Output

| Name | Age |
|------|-----|
|Prince|25|
|Rahul|28|
|Karan|31|

---

# Example 3

Find employees having salaries

80000,

95000,

or

120000.

```sql
SELECT *
FROM users
WHERE salary IN (
80000,
95000,
120000
);
```

Output

| Name | Salary |
|------|---------|
|Prince|80000|
|Priya|95000|
|Amit|120000|

---

# How MySQL Evaluates IN

Suppose

```
IN ('Delhi','Mumbai')
```

Table

| Name | City |
|------|------|
|Prince|Bengaluru|
|Rahul|Delhi|
|Amit|Mumbai|
|Sneha|Pune|

Evaluation

```
Prince

↓

Bengaluru

↓

Inside List?

↓

No

↓

Discard


Rahul

↓

Delhi

↓

Inside List?

↓

Yes

↓

Return


Amit

↓

Mumbai

↓

Inside List?

↓

Yes

↓

Return


Sneha

↓

Pune

↓

Inside List?

↓

No

↓

Discard
```

---

# IN with Numbers

```sql
SELECT *
FROM users
WHERE id IN (
1,
3,
5,
8
);
```

Output

| id | Name |
|----|------|
|1|Prince|
|3|Amit|
|5|Priya|
|8|Rohit|

---

# IN with Strings

```sql
SELECT *
FROM users
WHERE name IN (
'Prince',
'Rahul',
'Sneha'
);
```

Output

Returns only those users.

---

# IN with Dates

Suppose

```
orders
```

contains

```
created_at
```

Query

```sql
SELECT *
FROM orders
WHERE created_at IN (
'2026-07-10',
'2026-07-15'
);
```

Returns orders created on either date.

---

# Combining IN with AND

Example

```sql
SELECT *
FROM users
WHERE city IN (
'Delhi',
'Mumbai'
)
AND salary > 80000;
```

Output

| Name | City | Salary |
|------|------|---------|
|Rahul|Delhi|90000|
|Amit|Mumbai|120000|

Notice

Both conditions must be satisfied.

---

# Combining IN with OR

```sql
SELECT *
FROM users
WHERE city IN (
'Delhi',
'Mumbai'
)
OR age < 25;
```

Returns

- users from Delhi
- users from Mumbai
- users younger than 25

---

# NOT IN

Sometimes we need the opposite.

Find everyone except selected cities.

```sql
SELECT *
FROM users
WHERE city NOT IN (
'Delhi',
'Mumbai'
);
```

Output

| Name | City |
|------|------|
|Prince|Bengaluru|
|Sneha|Pune|
|Priya|Hyderabad|
|Karan|Jaipur|
|Neha|Kolkata|
|Rohit|Ahmedabad|

---

# Internal Working of NOT IN

```
City

↓

Inside List?

↓

Yes

↓

Discard


No

↓

Return
```

---

# Node.js Example

```javascript
const cities = ["Delhi", "Mumbai"];

// mysql2 cannot automatically expand an array into (?, ?)
// so create placeholders dynamically.
const placeholders = cities.map(() => "?").join(",");

const sql = `
SELECT *
FROM users
WHERE city IN (${placeholders})
`;

const [users] = await connection.execute(sql, cities);

console.log(users);
```

Output

```javascript
[
  { id: 2, name: "Rahul", city: "Delhi" },
  { id: 3, name: "Amit", city: "Mumbai" }
]
```

> **Important:** `mysql2` does **not** expand arrays automatically when using prepared statements. You must generate the correct number of placeholders (`?, ?, ?`) based on the array length.

---

# Alternative (Incorrect)

Many beginners try

```javascript
await connection.execute(
`
SELECT *
FROM users
WHERE city IN (?)
`,
[
["Delhi", "Mumbai"]
]
);
```

This does **not** work as expected with `mysql2` prepared statements.

Always build the placeholders dynamically.

---

# Performance Considerations

Small lists

```sql
IN (
1,
2,
3,
4
)
```

are very fast.

Large lists

```
5000 values
```

can become inefficient.

For very large datasets,

consider

- temporary tables
- JOINs
- subqueries

We'll study these in later phases.

---

# Index Usage

Suppose

```
city
```

is indexed.

Query

```sql
WHERE city IN (
'Delhi',
'Mumbai'
)
```

MySQL can use the index to locate matching values efficiently.

Conceptually

```
Index

↓

Delhi

↓

Return Rows

↓

Mumbai

↓

Return Rows
```

Instead of scanning every row in the table.

---

# Real-World Examples

## Product Categories

```sql
SELECT *
FROM products
WHERE category IN (
'Mobile',
'Laptop',
'Tablet'
);
```

---

## Allowed Roles

```sql
SELECT *
FROM users
WHERE role IN (
'Admin',
'Manager'
);
```

---

## Order Status

```sql
SELECT *
FROM orders
WHERE status IN (
'Pending',
'Processing',
'Shipped'
);
```

---

# Best Practices

✅ Prefer `IN` over long chains of `OR`.

✅ Parameterize every value.

✅ Use indexes on columns frequently used with `IN`.

✅ Keep value lists reasonably small.

---

# Common Mistakes

❌ Writing ten or twenty `OR` conditions instead of using `IN`.

❌ Building SQL strings using user input.

❌ Passing an array directly into a single `?` placeholder when using `mysql2`.

❌ Using very large `IN` lists where a JOIN would be more appropriate.

---

# Interview Notes

### Q1. What is the purpose of the `IN` operator?

It checks whether a column's value matches **any value** in a specified list.

---

### Q2. Is `IN` equivalent to multiple `OR` conditions?

Logically, yes.

```sql
city IN ('Delhi','Mumbai')
```

is equivalent to

```sql
city='Delhi'
OR city='Mumbai'
```

---

### Q3. What is `NOT IN`?

It returns rows whose values are **not** present in the given list.

---

### Q4. Can indexes be used with `IN`?

Yes.

If the filtered column is indexed, MySQL can often use the index to efficiently locate matching values.

---

### Q5. Can we pass an array directly to `IN (?)` in `mysql2`?

No.

With prepared statements in `mysql2`, you should generate the required number of placeholders (`?, ?, ?`) dynamically and pass the array elements as parameters.

---

# Quick Revision

- `IN` is a concise alternative to multiple `OR` conditions.
- `NOT IN` excludes values present in a list.
- `IN` works with numbers, strings, and dates.
- Combine `IN` with `AND` or `OR` for more complex filters.
- For `mysql2`, dynamically generate placeholders when using arrays.
- Indexed columns make `IN` queries significantly faster.

---

The next section covers the **BETWEEN operator**, which allows filtering values within an inclusive range, making range-based queries cleaner than using `>=` and `<=` together.

# BETWEEN Operator

In the previous section, we learned how to filter multiple discrete values using the `IN` operator.

However, many real-world queries involve **ranges** rather than individual values.

Examples

- Find employees aged between 25 and 30.
- Find products priced between ₹20,000 and ₹50,000.
- Find orders placed between two dates.
- Find students whose marks are between 80 and 90.

Writing these queries using multiple comparison operators is possible, but SQL provides a cleaner alternative:

The **BETWEEN** operator.

---

# What is BETWEEN?

The `BETWEEN` operator filters rows whose values lie within a specified range.

General syntax

```sql
SELECT column_list
FROM table_name
WHERE column_name BETWEEN lower_value AND upper_value;
```

Think of it as asking

> "Is this value inside the specified range?"

---

# Equivalent Comparison

These two queries are identical.

Using BETWEEN

```sql
SELECT *
FROM users
WHERE age BETWEEN 25 AND 30;
```

Using comparison operators

```sql
SELECT *
FROM users
WHERE age >= 25
AND age <= 30;
```

The second query is exactly how BETWEEN can be understood conceptually.

---

# Important Property

**BETWEEN is inclusive.**

This means

```
Lower Limit

Included

Upper Limit

Included
```

Example

```sql
BETWEEN 25 AND 30
```

Matches

```
25

26

27

28

29

30
```

Both

```
25

and

30
```

are included.

---

# Example 1

Find users aged between

25

and

30.

```sql
SELECT *
FROM users
WHERE age BETWEEN 25 AND 30;
```

Output

| Name | Age |
|------|-----|
|Prince|25|
|Rahul|28|
|Priya|29|
|Rohit|30|
|Neha|26|

---

# Internal Evaluation

Suppose

```
BETWEEN 25 AND 30
```

Evaluation

```
Age = 24

↓

24 >=25 ?

↓

False

↓

Discard


Age = 25

↓

25 >=25 ?

↓

True

↓

25 <=30 ?

↓

True

↓

Return


Age = 31

↓

31 <=30 ?

↓

False

↓

Discard
```

Internally,

MySQL evaluates BETWEEN similarly to

```sql
>=

AND

<=
```

---

# Example 2

Find employees earning between

80000

and

100000.

```sql
SELECT *
FROM users
WHERE salary BETWEEN 80000 AND 100000;
```

Output

| Name | Salary |
|------|---------|
|Prince|80000|
|Rahul|90000|
|Priya|95000|
|Rohit|100000|
|Neha|85000|

---

# BETWEEN with Dates

One of the most common uses.

Suppose

```
orders
```

contains

```
created_at
```

Query

```sql
SELECT *
FROM orders
WHERE created_at
BETWEEN '2026-07-01'
AND '2026-07-10';
```

Returns

All orders created from July 1 through July 10.

---

# BETWEEN with DateTime

Suppose

```
created_at
```

contains

```
2026-07-10 15:45:10
```

Query

```sql
SELECT *
FROM orders
WHERE created_at
BETWEEN
'2026-07-10 00:00:00'
AND
'2026-07-10 23:59:59';
```

Returns

Every order created on that day.

---

# BETWEEN with Strings

BETWEEN also works on strings because strings have an ordering based on the column's collation.

Example

```sql
SELECT *
FROM users
WHERE name
BETWEEN 'A'
AND 'M';
```

This returns names that sort within that range.

Although supported,

BETWEEN is far more commonly used with

- numbers
- dates
- timestamps

than with strings.

---

# Reversed Range

Suppose we write

```sql
SELECT *
FROM users
WHERE age
BETWEEN 30 AND 20;
```

Output

```
No Rows
```

Why?

Internally

```text
age >= 30

AND

age <=20
```

No value can satisfy both conditions simultaneously.

Always place the **smaller value first**.

---

# Combining BETWEEN with AND

Example

```sql
SELECT *
FROM users
WHERE age BETWEEN 25 AND 30
AND city = 'Delhi';
```

Output

| Name | Age | City |
|------|-----|------|
|Rahul|28|Delhi|

---

# Combining BETWEEN with OR

```sql
SELECT *
FROM users
WHERE age BETWEEN 25 AND 30
OR salary > 110000;
```

Returns

- users aged 25–30
- users earning more than 110000

---

# NOT BETWEEN

The opposite of BETWEEN.

Example

```sql
SELECT *
FROM users
WHERE age NOT BETWEEN 25 AND 30;
```

Output

| Name | Age |
|------|-----|
|Sneha|24|
|Amit|32|
|Karan|31|

Equivalent query

```sql
SELECT *
FROM users
WHERE age < 25
OR age > 30;
```

---

# Node.js Example

```javascript
const minAge = 25;
const maxAge = 30;

const [users] =
await connection.execute(
`
SELECT *
FROM users
WHERE age BETWEEN ? AND ?
`,
[
minAge,
maxAge
]
);

console.log(users);
```

Output

```javascript
[
  { id: 1, name: "Prince", age: 25 },
  { id: 2, name: "Rahul", age: 28 },
  { id: 5, name: "Priya", age: 29 },
  { id: 7, name: "Neha", age: 26 },
  { id: 8, name: "Rohit", age: 30 }
]
```

---

# Date Range Example

```javascript
const startDate = "2026-07-01";
const endDate = "2026-07-10";

const [orders] =
await connection.execute(
`
SELECT *
FROM orders
WHERE created_at
BETWEEN ? AND ?
`,
[
startDate,
endDate
]
);
```

---

# Internal Execution Flow

Suppose

```sql
SELECT *
FROM users
WHERE salary BETWEEN 80000 AND 100000;
```

Execution

```
Parser

↓

Optimizer

↓

Locate Rows

↓

salary >=80000 ?

↓

salary <=100000 ?

↓

TRUE ?

↓

Return

↓

FALSE ?

↓

Discard
```

Every row must satisfy **both** comparisons.

---

# Index Optimization

Suppose

```
salary
```

is indexed.

Query

```sql
WHERE salary
BETWEEN 80000 AND 100000;
```

Instead of scanning the entire table,

MySQL can perform an **Index Range Scan**.

Conceptually

```
Index

↓

80000

↓

85000

↓

90000

↓

95000

↓

100000

↓

Stop
```

This is much faster than checking every row.

We'll study index range scans in detail during the Indexes and Query Optimization phases.

---

# Real-World Examples

## Salary Report

```sql
SELECT *
FROM employees
WHERE salary
BETWEEN 50000 AND 100000;
```

---

## Product Price Filter

```sql
SELECT *
FROM products
WHERE price
BETWEEN 20000 AND 50000;
```

---

## Marks Report

```sql
SELECT *
FROM students
WHERE marks
BETWEEN 80 AND 90;
```

---

## Order History

```sql
SELECT *
FROM orders
WHERE order_date
BETWEEN '2026-01-01'
AND '2026-12-31';
```

---

# Best Practices

✅ Use BETWEEN for inclusive range queries.

✅ Ensure the lower bound is smaller than the upper bound.

✅ Index columns frequently used in range searches.

✅ Use parameterized queries when supplying range values.

---

# Common Mistakes

❌ Forgetting that BETWEEN includes both boundary values.

❌ Reversing the lower and upper bounds.

❌ Comparing values of incompatible data types.

❌ Using string-formatted dates inconsistently.

---

# Interview Notes

### Q1. Is BETWEEN inclusive?

Yes.

Both the lower and upper values are included.

---

### Q2. Is BETWEEN equivalent to `>=` and `<=`?

Conceptually, yes.

```sql
BETWEEN 25 AND 30
```

is equivalent to

```sql
>=25

AND

<=30
```

---

### Q3. Can BETWEEN be used with dates?

Yes.

It is one of the most common ways to query date ranges.

---

### Q4. Can indexes improve BETWEEN queries?

Yes.

MySQL can often perform an **Index Range Scan** instead of a full table scan when the filtered column is indexed.

---

### Q5. What is NOT BETWEEN?

It returns values **outside** the specified inclusive range.

---

# Quick Revision

- `BETWEEN` filters values within an inclusive range.
- It is equivalent to using `>=` and `<=` together.
- `NOT BETWEEN` returns values outside the range.
- BETWEEN works with numbers, dates, timestamps, and strings.
- Indexed columns make BETWEEN queries much faster through range scans.
- Always use the correct lower and upper bounds.

---

The next section introduces the **LIKE operator**, one of the most frequently used SQL operators for text searching, including wildcards (`%` and `_`), prefix searches, suffix searches, and substring matching.

# LIKE Operator

So far, we've learned how to filter

- Exact values (`=`)
- Multiple values (`IN`)
- Value ranges (`BETWEEN`)

However, users often search using **partial text** rather than exact matches.

Examples

- Search users whose names start with "Pri"
- Find products containing the word "iPhone"
- Search emails ending with "@gmail.com"
- Find cities beginning with "New"

Exact comparison using `=` is not enough.

SQL provides the **LIKE** operator for pattern matching.

---

# What is LIKE?

The `LIKE` operator compares a string against a **pattern** instead of an exact value.

General syntax

```sql
SELECT column_list
FROM table_name
WHERE column_name LIKE pattern;
```

Unlike

```sql
=
```

which requires an exact match,

`LIKE` allows partial matching.

---

# Wildcards

LIKE becomes powerful because of **wildcards**.

SQL provides two wildcard characters.

| Wildcard | Meaning |
|----------|---------|
| % | Zero or more characters |
| _ | Exactly one character |

---

# Understanding %

The

```
%
```

wildcard means

```
Any number of characters

including

zero characters.
```

Examples

```
P%

↓

Prince

Priya

Pooja

Pankaj

P
```

All match.

---

# Example 1

Find users whose names start with

```
P
```

```sql
SELECT *
FROM users
WHERE name LIKE 'P%';
```

Output

| Name |
|------|
|Prince|
|Priya|

---

# Internal Evaluation

Suppose

```
Pattern

P%
```

Table

```
Prince

Rahul

Priya

Sneha
```

Evaluation

```
Prince

↓

Starts with P ?

↓

Yes

↓

Return


Rahul

↓

Starts with P ?

↓

No

↓

Discard


Priya

↓

Starts with P ?

↓

Yes

↓

Return
```

---

# Example 2

Ends With

Suppose

```sql
LIKE '%a'
```

Query

```sql
SELECT *
FROM users
WHERE name LIKE '%a';
```

Output

```
Priya
```

Meaning

Any name ending with

```
a
```

---

# Example 3

Contains

```sql
SELECT *
FROM users
WHERE name LIKE '%it%';
```

Output

```
Amit
```

Because

```
it
```

appears inside

```
Amit
```

---

# Prefix Search

```
P%
```

Diagram

```
P

↓

Anything

↓

Anything

↓

Anything
```

Matches

```
Prince

Priya

Pooja
```

---

# Suffix Search

```
%a
```

Diagram

```
Anything

↓

Anything

↓

Ends With

↓

a
```

Matches

```
Priya

Pooja
```

---

# Contains Search

```
%it%
```

Diagram

```
Anything

↓

it

↓

Anything
```

Matches

```
Amit

Ritika

Nitin
```

---

# Matching Email Domains

Very common.

```sql
SELECT *
FROM users
WHERE email LIKE '%@gmail.com';
```

Output

Returns every Gmail user.

---

# Searching Products

```sql
SELECT *
FROM products
WHERE name LIKE '%iPhone%';
```

Matches

```
Apple iPhone 16

iPhone 15 Pro

Used iPhone 14
```

---

# Searching Cities

```sql
SELECT *
FROM users
WHERE city LIKE 'B%';
```

Output

```
Bengaluru
```

---

# Understanding _

The underscore

```
_
```

matches **exactly one character**.

Unlike

```
%
```

which matches many characters.

---

# Example

```sql
SELECT *
FROM users
WHERE name LIKE '_mit';
```

Matches

```
Amit
```

Explanation

```
_

↓

A

↓

mit
```

Exactly one character before

```
mit
```

---

# Another Example

Pattern

```
P_____
```

Means

```
P

+

Exactly five characters
```

Matches

```
Prince
```

But not

```
Priya
```

because

```
Priya

↓

P + four letters
```

---

# Difference Between % and _

Pattern

```
P%
```

Matches

```
P

Pa

Priya

Prince

Pankaj
```

Pattern

```
P_
```

Matches

```
Pa

Pi

Po
```

Only one character after

```
P
```

---

# Escaping Wildcards

Suppose your data literally contains

```
100%
```

Query

```sql
SELECT *
FROM products
WHERE name LIKE '100\%';
```

The backslash tells MySQL

```
Treat %

as a normal character,

not a wildcard.
```

Similarly

```sql
LIKE 'A\_B'
```

matches

```
A_B
```

instead of treating

```
_
```

as a wildcard.

---

# Case Sensitivity

Whether

```sql
LIKE
```

is case-sensitive depends on the **collation** of the column.

Most MySQL installations use a case-insensitive collation such as

```
utf8mb4_0900_ai_ci
```

where

```sql
LIKE 'prince%'
```

matches

```
Prince

PRINCE

prince
```

If the column uses a case-sensitive collation,

results will differ.

We'll discuss collations in a later phase.

---

# Node.js Example

```javascript
const keyword = "Pri%";

const [users] =
await connection.execute(
`
SELECT *
FROM users
WHERE name LIKE ?
`,
[
keyword
]
);

console.log(users);
```

Output

```javascript
[
    {
        id:1,
        name:"Prince"
    },
    {
        id:5,
        name:"Priya"
    }
]
```

---

# Dynamic Search API

A common REST API pattern.

```javascript
const search = "Phone";

const [products] =
await connection.execute(
`
SELECT *
FROM products
WHERE name LIKE ?
`,
[
`%${search}%`
]
);
```

User searches

```
Phone
```

Generated pattern

```
%Phone%
```

This matches

```
iPhone

Phone Case

Smart Phone
```

---

# Internal Working

Suppose

```sql
LIKE 'Pri%'
```

Execution

```
Parser

↓

Optimizer

↓

Read Row

↓

Pattern Match

↓

TRUE ?

↓

Return

↓

FALSE ?

↓

Discard
```

Unlike exact comparisons,

pattern matching often requires more work.

---

# Performance Considerations

Consider

```sql
LIKE 'Pri%'
```

MySQL may use an index because the prefix is known.

Conceptually

```
Index

↓

Pri...

↓

Scan Matching Prefix
```

This is generally efficient.

---

However,

```sql
LIKE '%Pri%'
```

is different.

Because the string can begin with **any character**,

MySQL usually cannot jump directly to a starting point in a normal B-tree index.

Conceptually

```
Read Every Row

↓

Check Pattern

↓

Return Matches
```

This often results in a full table scan.

---

# Prefix vs Contains Search

Efficient

```sql
LIKE 'Pri%'
```

Less efficient

```sql
LIKE '%Pri%'
```

For very large datasets,

contains searches may require

- Full-Text Indexes
- Search engines (OpenSearch, Elasticsearch)
- Dedicated search services

We'll revisit this in later phases.

---

# Real-World Examples

## Login Suggestion

```sql
SELECT *
FROM users
WHERE email LIKE 'pri%';
```

---

## Product Search

```sql
SELECT *
FROM products
WHERE name LIKE '%Laptop%';
```

---

## City Search

```sql
SELECT *
FROM users
WHERE city LIKE 'New%';
```

---

## Domain Search

```sql
SELECT *
FROM users
WHERE email LIKE '%@gmail.com';
```

---

# Best Practices

✅ Use `LIKE 'prefix%'` whenever possible.

✅ Parameterize search values.

✅ Understand the difference between `%` and `_`.

✅ Consider Full-Text Search for advanced text searching.

---

# Common Mistakes

❌ Using `=` when partial matching is required.

❌ Assuming `%` matches exactly one character.

❌ Expecting `LIKE '%text%'` to efficiently use a normal B-tree index.

❌ Concatenating user input directly into SQL strings.

---

# Interview Notes

### Q1. What is the purpose of the LIKE operator?

It performs pattern matching on string values.

---

### Q2. What does `%` represent?

Zero or more characters.

---

### Q3. What does `_` represent?

Exactly one character.

---

### Q4. Which query is generally faster?

```sql
LIKE 'Pri%'
```

because MySQL can often use an index.

---

### Q5. Why is `LIKE '%Pri%'` slower?

Because MySQL usually cannot use a standard B-tree index efficiently when the pattern starts with a wildcard, so it often scans many or all rows.

---

# Quick Revision

- `LIKE` performs pattern matching.
- `%` matches zero or more characters.
- `_` matches exactly one character.
- `LIKE 'prefix%'` is generally index-friendly.
- `LIKE '%text%'` often requires scanning many rows.
- Use parameterized queries for all search patterns.
- `LIKE` is widely used in search bars, autocomplete, and filtering APIs.

---

The next section covers **NULL Handling (`IS NULL` and `IS NOT NULL`)**, explaining SQL's three-valued logic and why `= NULL` never works as many beginners expect.

# NULL Handling (IS NULL and IS NOT NULL)

So far, we've filtered data using

- Comparison operators
- Logical operators
- IN
- BETWEEN
- LIKE

However, real-world databases often contain **missing information**.

Examples

- A customer hasn't provided a phone number.
- An employee hasn't been assigned a manager.
- An order hasn't been shipped yet.
- A product doesn't have a discount.

In SQL, missing values are represented using **NULL**.

Understanding NULL is one of the most important SQL concepts because it behaves differently from every other value.

---

# What is NULL?

NULL means

```
Unknown

or

Missing

or

Not Available
```

It **does not mean**

- Zero
- Empty string
- False

These are completely different values.

---

# Example Table

| id | name | phone |
|----|------|--------|
|1|Prince|9876543210|
|2|Rahul|NULL|
|3|Priya|9988776655|
|4|Sneha|NULL|

Notice

Rahul doesn't have a phone number.

That phone number is

```
NULL
```

---

# Common Beginner Mistake

Many beginners write

```sql
SELECT *
FROM users
WHERE phone = NULL;
```

Result

```
No Rows
```

Even though NULL values exist.

Why?

Because

```
NULL

is NOT a value.

It represents

UNKNOWN.
```

---

# SQL Three-Valued Logic

Unlike programming languages,

SQL does **not** have only

```
TRUE

FALSE
```

It has

```
TRUE

FALSE

UNKNOWN
```

This is called

**Three-Valued Logic**.

---

# Example

Suppose

```
phone = NULL
```

Evaluation

```
9876543210 = NULL

↓

UNKNOWN


NULL = NULL

↓

UNKNOWN


1234567890 = NULL

↓

UNKNOWN
```

Notice

Nothing becomes

```
TRUE
```

Therefore,

no rows are returned.

---

# Correct Way

SQL provides

```
IS NULL
```

instead of

```
= NULL
```

Syntax

```sql
SELECT *
FROM table_name
WHERE column IS NULL;
```

---

# Example

```sql
SELECT *
FROM users
WHERE phone IS NULL;
```

Output

| id | name |
|----|------|
|2|Rahul|
|4|Sneha|

---

# Internal Evaluation

Suppose

```
phone

↓

NULL
```

Evaluation

```
NULL

↓

IS NULL ?

↓

TRUE

↓

Return


9876543210

↓

IS NULL ?

↓

FALSE

↓

Discard
```

---

# IS NOT NULL

Returns rows having a valid value.

Syntax

```sql
SELECT *
FROM users
WHERE phone IS NOT NULL;
```

Output

| Name | Phone |
|------|---------|
|Prince|9876543210|
|Priya|9988776655|

---

# NULL vs Empty String

Many beginners confuse these.

Suppose

| Name | Phone |
|------|---------|
|Prince|NULL|
|Rahul|""|

These are different.

```
NULL

↓

Unknown


""

↓

Known

↓

Empty Text
```

Query

```sql
WHERE phone IS NULL
```

returns

```
Prince
```

Only.

---

# NULL vs Zero

Suppose

| Salary Bonus |
|--------------|
|NULL|
|0|

Meaning

```
NULL

↓

Bonus not recorded


0

↓

Bonus is zero
```

These have completely different meanings.

---

# Using NULL with Dates

Suppose

| Order | Shipped Date |
|--------|--------------|
|101|NULL|
|102|2026-07-10|

Query

```sql
SELECT *
FROM orders
WHERE shipped_date IS NULL;
```

Returns

Orders that haven't shipped yet.

---

# Using NULL with Foreign Keys

Suppose

Employees

| Employee | Manager ID |
|----------|------------|
|Prince|NULL|
|Rahul|1|

Query

```sql
SELECT *
FROM employees
WHERE manager_id IS NULL;
```

Returns

Top-level managers.

---

# Combining NULL with AND

```sql
SELECT *
FROM users
WHERE phone IS NULL
AND city='Delhi';
```

Returns

Delhi users without phone numbers.

---

# Combining NULL with OR

```sql
SELECT *
FROM users
WHERE phone IS NULL
OR email IS NULL;
```

Returns

Users missing either phone or email.

---

# NOT with NULL

```sql
SELECT *
FROM users
WHERE NOT phone IS NULL;
```

Equivalent to

```sql
SELECT *
FROM users
WHERE phone IS NOT NULL;
```

---

# NULL in Comparisons

Suppose

```sql
SELECT *
FROM users
WHERE age > NULL;
```

Result

```
No Rows
```

Because

```
25 > NULL

↓

UNKNOWN
```

---

# NULL in Arithmetic

Suppose

```sql
SELECT
salary + bonus
FROM employees;
```

If

```
bonus = NULL
```

Then

```
100000 + NULL

↓

NULL
```

Any arithmetic involving NULL generally produces NULL.

---

# NULL in Concatenation

```sql
SELECT
CONCAT(name, phone)
FROM users;
```

If

```
phone=NULL
```

The result depends on the function used.

We'll study NULL-safe functions like

```
COALESCE()

IFNULL()
```

later.

---

# Node.js Example

Find users without phone numbers.

```javascript
const [users] =
await connection.execute(
`
SELECT *
FROM users
WHERE phone IS NULL
`
);

console.log(users);
```

---

# Another Example

Users having phone numbers.

```javascript
const [users] =
await connection.execute(
`
SELECT *
FROM users
WHERE phone IS NOT NULL
`
);
```

---

# Internal Execution Flow

Suppose

```sql
SELECT *
FROM users
WHERE phone IS NULL;
```

Execution

```
Parser

↓

Optimizer

↓

Read Row

↓

Phone Exists?

↓

No

↓

Return


Yes

↓

Discard
```

---

# Index Usage

If

```
phone
```

is indexed,

MySQL can often use the index when searching for

```sql
IS NULL
```

or

```sql
IS NOT NULL
```

depending on the query and optimizer.

---

# Real-World Examples

## Orders Not Delivered

```sql
SELECT *
FROM orders
WHERE delivered_at IS NULL;
```

---

## Employees Without Managers

```sql
SELECT *
FROM employees
WHERE manager_id IS NULL;
```

---

## Products Without Discounts

```sql
SELECT *
FROM products
WHERE discount IS NULL;
```

---

## Users Missing Email

```sql
SELECT *
FROM users
WHERE email IS NULL;
```

---

# Best Practices

✅ Always use

```sql
IS NULL
```

instead of

```sql
= NULL
```

---

✅ Use

```sql
IS NOT NULL
```

for existing values.

---

✅ Understand that NULL is different from

- Zero
- Empty String
- False

---

✅ Design schemas carefully to avoid unnecessary NULL values.

---

# Common Mistakes

❌

```sql
WHERE column = NULL
```

---

❌

```sql
WHERE column != NULL
```

---

❌ Assuming NULL equals an empty string.

---

❌ Forgetting SQL's three-valued logic.

---

# Interview Notes

### Q1. What is NULL?

A missing or unknown value.

---

### Q2. Why doesn't `= NULL` work?

Because NULL is **not a value**.

Comparisons involving NULL evaluate to

```
UNKNOWN
```

not TRUE.

---

### Q3. How do you check for NULL?

```sql
IS NULL
```

---

### Q4. How do you check for non-NULL values?

```sql
IS NOT NULL
```

---

### Q5. Is NULL equal to zero?

No.

```
NULL

↓

Unknown

0

↓

Known numeric value
```

---

### Q6. Is NULL equal to an empty string?

No.

An empty string is a valid string value.

NULL represents missing information.

---

# Quick Revision

- NULL represents unknown or missing data.
- NULL is not zero, an empty string, or false.
- Never use `= NULL` or `!= NULL`.
- Use `IS NULL` and `IS NOT NULL` instead.
- SQL uses three-valued logic: **TRUE**, **FALSE**, and **UNKNOWN**.
- Comparisons and arithmetic involving NULL often produce UNKNOWN or NULL.
- NULL handling is fundamental in production databases and a common interview topic.

---

The next section covers **DISTINCT**, which removes duplicate values from query results and explains how MySQL determines uniqueness across one or multiple columns.

# DISTINCT

So far, every query has returned **all matching rows**.

However, many real-world queries require **unique values only**.

Examples

- Show all unique cities.
- Show all departments.
- Show all product categories.
- Show all order statuses.

Without removing duplicates, the result may contain repeated values.

SQL provides the **DISTINCT** keyword for this purpose.

---

# What is DISTINCT?

`DISTINCT` removes duplicate rows from the result set.

General syntax

```sql
SELECT DISTINCT column_name
FROM table_name;
```

Instead of returning every row,

MySQL returns only **unique values**.

---

# Example Table

Suppose the `users` table contains

| id | name | city |
|----|------|-----------|
|1|Prince|Bengaluru|
|2|Rahul|Delhi|
|3|Amit|Mumbai|
|4|Sneha|Delhi|
|5|Priya|Hyderabad|
|6|Neha|Mumbai|
|7|Rohit|Delhi|

---

# Without DISTINCT

```sql
SELECT city
FROM users;
```

Output

| city |
|------|
|Bengaluru|
|Delhi|
|Mumbai|
|Delhi|
|Hyderabad|
|Mumbai|
|Delhi|

Notice

```
Delhi

↓

Repeated

Mumbai

↓

Repeated
```

---

# Using DISTINCT

```sql
SELECT DISTINCT city
FROM users;
```

Output

| city |
|------|
|Bengaluru|
|Delhi|
|Mumbai|
|Hyderabad|

Every city appears only once.

---

# Internal Working

Conceptually

```
Read Rows

↓

Bengaluru

↓

Store

↓

Delhi

↓

Store

↓

Mumbai

↓

Store

↓

Delhi

↓

Already Exists

↓

Ignore

↓

Mumbai

↓

Already Exists

↓

Ignore
```

Only unique values remain.

---

# DISTINCT on Numbers

Suppose

| Age |
|-----|
|25|
|28|
|25|
|30|
|28|

Query

```sql
SELECT DISTINCT age
FROM users;
```

Output

| age |
|-----|
|25|
|28|
|30|

---

# DISTINCT on Multiple Columns

DISTINCT also works on **multiple columns**.

Example

```sql
SELECT DISTINCT city, age
FROM users;
```

Now,

MySQL checks uniqueness based on the **entire combination**.

---

Suppose

| City | Age |
|------|-----|
|Delhi|25|
|Delhi|25|
|Delhi|28|
|Mumbai|25|

Output

| City | Age |
|------|-----|
|Delhi|25|
|Delhi|28|
|Mumbai|25|

Only identical combinations are removed.

---

# Important Concept

Consider

```sql
SELECT DISTINCT city, age
```

This does **not** mean

- Unique cities
- Unique ages

Instead,

it means

```
Unique

(city, age)

pairs.
```

---

# DISTINCT vs GROUP BY

These two queries often produce the same result.

Using DISTINCT

```sql
SELECT DISTINCT city
FROM users;
```

Using GROUP BY

```sql
SELECT city
FROM users
GROUP BY city;
```

Output

```
Bengaluru

Delhi

Mumbai

Hyderabad
```

Although the output is similar,

their purposes are different.

- `DISTINCT` removes duplicates.
- `GROUP BY` groups rows for aggregation (COUNT, SUM, AVG, etc.).

We'll study `GROUP BY` in detail later.

---

# DISTINCT with WHERE

DISTINCT is applied **after filtering**.

Example

```sql
SELECT DISTINCT city
FROM users
WHERE age > 25;
```

Execution order (conceptually)

```
Read Rows

↓

WHERE age >25

↓

Remaining Rows

↓

Remove Duplicates

↓

Return Result
```

---

# DISTINCT with ORDER BY

```sql
SELECT DISTINCT city
FROM users
ORDER BY city;
```

Output

| city |
|------|
|Bengaluru|
|Delhi|
|Hyderabad|
|Mumbai|

Duplicates are removed first,

then the remaining rows are sorted.

---

# DISTINCT with NULL

Consider

| phone |
|--------|
|NULL|
|NULL|
|9876543210|
|9988776655|

Query

```sql
SELECT DISTINCT phone
FROM users;
```

Output

| phone |
|--------|
|NULL|
|9876543210|
|9988776655|

Even though multiple NULL values exist,

only **one NULL** appears in the DISTINCT result.

---

# Node.js Example

Find all unique cities.

```javascript
const [cities] =
await connection.execute(
`
SELECT DISTINCT city
FROM users
`
);

console.log(cities);
```

Output

```javascript
[
  { city: "Bengaluru" },
  { city: "Delhi" },
  { city: "Mumbai" },
  { city: "Hyderabad" }
]
```

---

# Another Example

Unique combinations.

```javascript
const [records] =
await connection.execute(
`
SELECT DISTINCT city, age
FROM users
`
);

console.log(records);
```

---

# Internal Execution Flow

Suppose

```sql
SELECT DISTINCT city
FROM users;
```

Execution

```
Parser

↓

Optimizer

↓

Read Rows

↓

Temporary Set

↓

Duplicate?

↓

Yes

↓

Ignore

↓

No

↓

Store

↓

Return Result
```

Depending on the query,

MySQL may use

- Sorting
- Hashing
- Indexes

to eliminate duplicates.

---

# Performance Considerations

For small tables,

DISTINCT is inexpensive.

For very large tables,

removing duplicates may require

- Sorting
- Temporary tables
- Additional memory

If an appropriate index exists,

MySQL may satisfy DISTINCT directly from the index, making the query much faster.

---

# Index Optimization

Suppose

```
city
```

is indexed.

Query

```sql
SELECT DISTINCT city
FROM users;
```

MySQL may scan only the index entries.

Conceptually

```
Index

↓

Bengaluru

↓

Delhi

↓

Delhi

↓

Mumbai

↓

Mumbai

↓

Hyderabad

↓

Return Unique Values
```

Scanning an index is generally cheaper than scanning the entire table.

---

# Real-World Examples

## Unique Departments

```sql
SELECT DISTINCT department
FROM employees;
```

---

## Product Categories

```sql
SELECT DISTINCT category
FROM products;
```

---

## Order Statuses

```sql
SELECT DISTINCT status
FROM orders;
```

---

## Customer Countries

```sql
SELECT DISTINCT country
FROM customers;
```

---

# Best Practices

✅ Use DISTINCT only when duplicate removal is required.

✅ Index columns frequently queried with DISTINCT.

✅ Remember that DISTINCT applies to the entire selected column list.

---

# Common Mistakes

❌ Assuming DISTINCT works on each column independently.

❌ Using DISTINCT to hide duplicate rows caused by incorrect JOINs.

❌ Using DISTINCT unnecessarily, which can increase query cost.

---

# Interview Notes

### Q1. What does DISTINCT do?

It removes duplicate rows from the query result.

---

### Q2. Does DISTINCT work on multiple columns?

Yes.

It removes duplicate **combinations** of the selected columns.

---

### Q3. Does DISTINCT remove multiple NULL values?

Yes.

Multiple NULL values are treated as duplicates for the purpose of DISTINCT, so only one NULL appears in the result.

---

### Q4. Is DISTINCT the same as GROUP BY?

No.

`DISTINCT` removes duplicates.

`GROUP BY` groups rows, usually for aggregate calculations.

---

### Q5. Can DISTINCT use indexes?

Yes.

If suitable indexes exist, MySQL can often use them to efficiently return unique values.

---

# Quick Revision

- `DISTINCT` removes duplicate rows from the result set.
- It works with one or multiple columns.
- With multiple columns, uniqueness is based on the entire column combination.
- DISTINCT is applied after filtering (`WHERE`).
- DISTINCT and GROUP BY can produce similar output but serve different purposes.
- Proper indexing can significantly improve DISTINCT performance.

---

The next section covers **LIMIT and OFFSET**, which are essential for pagination, APIs, dashboards, and efficiently retrieving subsets of large result sets.

# LIMIT and OFFSET

So far, every query we've executed has returned **all matching rows**.

This is acceptable for small tables.

However, imagine a production database containing

- 5 million users
- 20 million orders
- 100 million products

Returning every row would

- consume excessive memory
- increase network traffic
- slow down applications
- provide a poor user experience

Instead, applications usually request only a **small subset** of rows.

This is where **LIMIT** and **OFFSET** come in.

---

# What is LIMIT?

`LIMIT` restricts the number of rows returned by a query.

General syntax

```sql
SELECT column_list
FROM table_name
LIMIT number_of_rows;
```

---

# Example

Suppose

```
users
```

contains

| id | name |
|----|------|
|1|Prince|
|2|Rahul|
|3|Amit|
|4|Sneha|
|5|Priya|
|6|Neha|

Query

```sql
SELECT *
FROM users
LIMIT 3;
```

Output

| id | name |
|----|------|
|1|Prince|
|2|Rahul|
|3|Amit|

Only three rows are returned.

---

# Internal Working

```
Read First Row

↓

Return

↓

Read Second Row

↓

Return

↓

Read Third Row

↓

Return

↓

Limit Reached

↓

Stop Reading
```

Notice

MySQL stops once the required number of rows has been produced.

---

# LIMIT Without ORDER BY

Consider

```sql
SELECT *
FROM users
LIMIT 5;
```

Many beginners assume

```
First five inserted rows.
```

This is **not guaranteed**.

Without an

```sql
ORDER BY
```

clause,

SQL does **not guarantee** the order of returned rows.

If row order matters,

always use

```sql
ORDER BY
```

together with

```sql
LIMIT
```

---

# LIMIT with ORDER BY

Example

```sql
SELECT *
FROM users
ORDER BY age DESC
LIMIT 3;
```

Output

| Name | Age |
|------|-----|
|Amit|32|
|Karan|31|
|Rohit|30|

This query returns

The three oldest users.

---

# Another Example

Top five highest salaries.

```sql
SELECT *
FROM employees
ORDER BY salary DESC
LIMIT 5;
```

Very common in dashboards.

---

# LIMIT 1

Frequently used when only one row is expected.

Example

```sql
SELECT *
FROM users
WHERE email='prince@gmail.com'
LIMIT 1;
```

Even if duplicate data exists accidentally,

only one row is returned.

---

# What is OFFSET?

OFFSET tells MySQL

```
Skip

N

rows

then

start returning rows.
```

General syntax

```sql
SELECT *
FROM table_name
LIMIT row_count
OFFSET rows_to_skip;
```

---

# Example

```sql
SELECT *
FROM users
LIMIT 3
OFFSET 2;
```

Suppose

| id | name |
|----|------|
|1|Prince|
|2|Rahul|
|3|Amit|
|4|Sneha|
|5|Priya|
|6|Neha|

Execution

```
Skip

Prince

↓

Skip

Rahul

↓

Return

Amit

↓

Return

Sneha

↓

Return

Priya
```

Output

| id | name |
|----|------|
|3|Amit|
|4|Sneha|
|5|Priya|

---

# MySQL Shortcut Syntax

Instead of

```sql
LIMIT 5 OFFSET 10
```

MySQL also supports

```sql
LIMIT 10,5
```

Meaning

```
Skip

10

Return

5
```

Both queries are identical.

However,

the first syntax is more readable and is part of the SQL standard.

Recommended

```sql
LIMIT 5 OFFSET 10
```

---

# Pagination

LIMIT and OFFSET are primarily used for pagination.

Suppose

```
10 rows per page
```

Page 1

```sql
LIMIT 10 OFFSET 0
```

Page 2

```sql
LIMIT 10 OFFSET 10
```

Page 3

```sql
LIMIT 10 OFFSET 20
```

General formula

```
OFFSET

=

(Page Number - 1)

×

Page Size
```

---

# Example

Page Size

```
20
```

Page

```
4
```

Offset

```
(4−1)

×

20

=

60
```

Query

```sql
SELECT *
FROM users
LIMIT 20
OFFSET 60;
```

---

# Node.js Example

Suppose

```
page = 2

pageSize = 10
```

```javascript
const page = 2;
const pageSize = 10;

const offset =
(page - 1) * pageSize;

const [users] =
await connection.execute(
`
SELECT *
FROM users
ORDER BY id
LIMIT ?
OFFSET ?
`,
[
pageSize,
offset
]
);

console.log(users);
```

---

# REST API Example

```
GET

/users?page=3&limit=20
```

Node.js

```javascript
const page = Number(req.query.page);
const limit = Number(req.query.limit);

const offset =
(page - 1) * limit;
```

SQL

```sql
SELECT *
FROM users
ORDER BY id
LIMIT ?
OFFSET ?;
```

This is one of the most common SQL queries in backend development.

---

# Internal Execution Flow

Suppose

```sql
SELECT *
FROM users
ORDER BY id
LIMIT 5
OFFSET 10;
```

Execution

```
Parser

↓

Optimizer

↓

Sort

↓

Skip

10 Rows

↓

Return

Next 5 Rows

↓

Stop
```

---

# Performance Considerations

Small OFFSET

```
OFFSET 20
```

Fast.

Large OFFSET

```
OFFSET 500000
```

Potentially slow.

Why?

Because MySQL must still traverse many rows before reaching the requested starting point.

Conceptually

```
Read

↓

Skip

↓

Skip

↓

Skip

↓

Skip

↓

Return
```

Large offsets become increasingly expensive.

---

# Better Pagination (Keyset Pagination)

Instead of

```sql
LIMIT 20 OFFSET 100000
```

Many production systems use

```sql
SELECT *
FROM users
WHERE id > 100000
ORDER BY id
LIMIT 20;
```

This is called

**Keyset Pagination** (also known as Seek Pagination).

Benefits

- Faster on large tables
- Uses indexes efficiently
- Avoids scanning skipped rows

We'll study this in the Query Optimization phase.

---

# LIMIT with WHERE

Example

```sql
SELECT *
FROM users
WHERE city='Delhi'
LIMIT 2;
```

Execution

```
Filter

↓

Delhi Users

↓

Return

First Two
```

---

# LIMIT with DISTINCT

```sql
SELECT DISTINCT city
FROM users
LIMIT 3;
```

Duplicates are removed first,

then the limit is applied.

---

# Real-World Examples

## Latest Orders

```sql
SELECT *
FROM orders
ORDER BY created_at DESC
LIMIT 10;
```

---

## Highest Paid Employees

```sql
SELECT *
FROM employees
ORDER BY salary DESC
LIMIT 5;
```

---

## Dashboard

```sql
SELECT *
FROM notifications
ORDER BY created_at DESC
LIMIT 20;
```

---

## Infinite Scroll

```sql
SELECT *
FROM posts
ORDER BY id DESC
LIMIT 20
OFFSET 40;
```

---

# Best Practices

✅ Always combine `LIMIT` with `ORDER BY` when row order matters.

✅ Use pagination in APIs instead of returning every row.

✅ Prefer keyset pagination for very large datasets.

✅ Keep page sizes reasonable (10–100 rows in most APIs).

---

# Common Mistakes

❌ Using `LIMIT` without `ORDER BY` and expecting a consistent order.

❌ Using very large `OFFSET` values on huge tables.

❌ Returning thousands of rows when only a page is needed.

❌ Calculating offsets incorrectly.

---

# Interview Notes

### Q1. What does LIMIT do?

It restricts the number of rows returned by a query.

---

### Q2. What does OFFSET do?

It skips a specified number of rows before returning results.

---

### Q3. Why should LIMIT usually be combined with ORDER BY?

Without `ORDER BY`, SQL does not guarantee the order of returned rows, so pagination can become inconsistent.

---

### Q4. Why is a very large OFFSET slow?

Because MySQL typically has to scan or traverse many rows before reaching the desired starting point.

---

### Q5. What is keyset pagination?

A pagination technique that uses a unique, ordered column (such as `id`) instead of OFFSET, allowing MySQL to efficiently continue from the last retrieved row.

---

# Quick Revision

- `LIMIT` restricts the number of returned rows.
- `OFFSET` skips rows before returning results.
- Use `ORDER BY` with `LIMIT` for deterministic results.
- Pagination commonly uses `LIMIT` and `OFFSET`.
- Large OFFSET values can hurt performance.
- Keyset pagination is a scalable alternative for large datasets.

---

The next section begins **05-Sorting-and-Pagination.md**, where we'll explore `ORDER BY` in depth, including ascending and descending sorting, multi-column sorting, sorting with `NULL` values, custom ordering, and advanced pagination strategies used in production systems.


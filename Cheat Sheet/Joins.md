## SQL JOIN Types Cheat Sheet Table

| JOIN Type | Purpose | How It Works | Key Use Case |
| :--- | :--- | :--- | :--- |
| **INNER JOIN** | **Intersection** | Returns **only** rows that have matching values in **both** tables. | Finding records that have a relationship (e.g., customers who have placed orders). |
| **LEFT (OUTER) JOIN** | **Primary Dominance** | Returns **all** rows from the **left** table, and the matched rows from the right table. **NULL** is returned for non-matches on the right. | Listing all items from one table and seeing if they have a match in the second (e.g., all employees and their respective tasks, if any). |
| **RIGHT (OUTER) JOIN** | **Secondary Dominance** | Returns **all** rows from the **right** table, and the matched rows from the left table. **NULL** is returned for non-matches on the left. | Listing all items from one table and seeing if they have a match in the first (less common than LEFT JOIN, as tables can often be reordered). |
| **FULL (OUTER) JOIN** | **Union** | Returns **all** rows when there is a match in **either** the left or the right table. **NULL** is returned where there is no match in the other table. | Comparing two sets of data to find all records, including those unique to each set (e.g., comparing user lists from two different systems). |

-----

## SQL JOIN Types Mini Playbook (Realistic Queries)

These snippets illustrate when to use each `JOIN` type with sample tables `Users` and `Orders`.

### 1\. INNER JOIN (Matching Records Only)

**Use Case:** Find the **names of users who have placed at least one order**.

```sql
SELECT
    u.user_id,
    u.user_name,
    o.order_id
FROM
    Users AS u
INNER JOIN
    Orders AS o ON u.user_id = o.user_id;
```

-----

### 2\. LEFT JOIN (All Left Records)

**Use Case:** List **all users**, and show their order ID if they have one. If a user hasn't placed an order, their order ID will be **NULL**.

```sql
SELECT
    u.user_id,
    u.user_name,
    o.order_id,
    o.order_date
FROM
    Users AS u
LEFT JOIN
    Orders AS o ON u.user_id = o.user_id;
```

-----

### 3\. RIGHT JOIN (All Right Records)

**Use Case:** List **all orders**, and show the user name who placed them. If an order exists without a matching user (data integrity issue), the user name will be **NULL**.

```sql
SELECT
    u.user_name,
    o.order_id,
    o.order_date
FROM
    Users AS u
RIGHT JOIN
    Orders AS o ON u.user_id = o.user_id;
```

-----

### 4\. FULL OUTER JOIN (All Records from Both)

**Use Case:** Find **all users AND all orders**, linking them where possible, and showing **NULL** where a match doesn't exist (i.e., users without orders, and orders without users).

```sql
SELECT
    u.user_id AS user_id_from_users,
    u.user_name,
    o.order_id,
    o.order_date
FROM
    Users AS u
FULL OUTER JOIN
    Orders AS o ON u.user_id = o.user_id;
```

-----

### Bonus: Finding Non-Matching Records (Anti-Join Pattern)

A crucial use case is finding records in one table that **do not** have a match in the other. This is achieved using a **LEFT JOIN** combined with a `WHERE` clause.

**Use Case:** Find all **users who have NEVER placed an order**.

```sql
SELECT
    u.user_id,
    u.user_name
FROM
    Users AS u
LEFT JOIN
    Orders AS o ON u.user_id = o.user_id
WHERE
    o.order_id IS NULL; -- The key filter to isolate non-matches
```

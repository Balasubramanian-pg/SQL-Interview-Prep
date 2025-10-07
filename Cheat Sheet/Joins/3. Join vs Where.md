This cheat sheet compares two fundamental methods for restricting data in SQL: **filtering via the `WHERE` clause** and **filtering via the `JOIN` clause**. Understanding their distinct effects—especially with `LEFT JOIN`—is crucial for accurate query results.

## JOIN vs. WHERE Clause Filtering Cheat Sheet Table

| Feature | Filtering in `WHERE` Clause | Filtering in `JOIN` Clause |
| :--- | :--- | :--- |
| **Purpose** | Restricts the **final result set** *after* the join or Cartesian product is complete. | Defines the **conditions** for joining the tables; restricts the data *before* or *during* the join. |
| **Impact on `INNER JOIN`** | **Identical** result. The effect is the same regardless of placement. | **Identical** result. Conditions can be placed in `ON` or `WHERE`. |
| **Impact on `LEFT JOIN`** | **DRAMATICALLY different** (turns the `LEFT JOIN` into an `INNER JOIN`). | **Retains all rows** from the left table; filters the right table only. |
| **When to Use** | For general filtering on columns from *either* table, or for filtering the final result after a complex join. | To specify the exact linkage conditions, especially in a **`LEFT JOIN`** to preserve unmatched rows. |

## JOIN vs. WHERE Clause Filtering Mini Playbook (Realistic Queries)

These snippets illustrate the critical difference using a `LEFT JOIN` on two tables: `Users` (left) and `Orders` (right). We want all users, but only their orders with a `status = 'SHIPPED'`.

### Scenario 1: Filtering in the `ON` Clause (Correct LEFT JOIN)

**Goal:** List **all users**. Include only their **SHIPPED** orders. Users with no orders, or users whose orders aren't shipped, will still appear, but the order columns will be `NULL`.

```sql
SELECT
    u.user_id,
    u.user_name,
    o.order_id,
    o.status
FROM
    Users AS u
LEFT JOIN
    Orders AS o
    -- Match user ID AND filter the RIGHT table (Orders)
    ON u.user_id = o.user_id
    AND o.status = 'SHIPPED';
    
-- RESULT: All users are present. Orders are only included if 'SHIPPED'.
--         Users who placed 'PENDING' orders will appear with NULL order data.
```

### Scenario 2: Filtering in the `WHERE` Clause (Turns into INNER JOIN)

**Goal:** List all users and their SHIPPED orders, but the `WHERE` clause fundamentally changes the logic.

```sql
SELECT
    u.user_id,
    u.user_name,
    o.order_id,
    o.status
FROM
    Users AS u
LEFT JOIN
    Orders AS o
    -- Only match user ID
    ON u.user_id = o.user_id
WHERE
    -- This filters the final result, removing any row where o.status IS NULL.
    -- This eliminates all users who had no matching orders or no SHIPPED orders.
    o.status = 'SHIPPED';
    
-- RESULT: Identical to an INNER JOIN. Only users who have a 'SHIPPED' order are returned.
--         Users with no orders or only 'PENDING' orders are completely removed.
```

### 3\. INNER JOIN Equivalence

For an **`INNER JOIN`**, the placement of the non-key condition makes no difference.

```sql
-- Inner Join: Condition in ON
SELECT * FROM Users u INNER JOIN Orders o ON u.user_id = o.user_id AND o.amount > 100;

-- Inner Join: Condition in WHERE
SELECT * FROM Users u INNER JOIN Orders o ON u.user_id = o.user_id WHERE o.amount > 100;

-- RESULT: Both queries return the exact same rows: only users and orders linked by ID,
--         and where the order amount is greater than 100.
```

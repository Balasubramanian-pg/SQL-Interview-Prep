## CTEs vs. Subqueries Cheat Sheet Table

| Feature | Common Table Expression (CTE) | Subquery (Derived Table) |
| :--- | :--- | :--- |
| **Syntax/Keyword** | Defined with **`WITH`** clause. | Nested directly within the `FROM`, `SELECT`, or `WHERE` clauses. |
| **Readability** | Generally **higher** (especially for complex queries). Code is sequential (top-down). | Can be **low** when heavily nested. Difficult to debug deep nesting. |
| **Reusability** | **High**. A single CTE can be referenced multiple times within the same subsequent query. | **Low**. Must be redefined (copied/pasted) every time it's needed. |
| **Self-Reference** | **Possible** (Allows for **Recursive CTEs**). | **Not possible**. Cannot reference itself. |
| **Scope** | Limited to the single immediate `SELECT`, `INSERT`, `UPDATE`, or `DELETE` statement that follows the `WITH` clause. | Limited to the specific clause (`FROM`, `SELECT`, or `WHERE`) it is defined in. |

-----

## CTEs vs. Subqueries Mini Playbook (Realistic Queries)

These snippets illustrate the common use cases and structural differences between the two methods for calculating the **average order value** for *high-value users* (those with lifetime spending over $1000).

### 1\. Using Common Table Expressions (CTEs)

**Use Case:** Define a list of high-value users once, then calculate their average order value in the main query.

```sql
WITH HighValueUsers AS (
    -- CTE 1: Identify users who spent over $1000 total
    SELECT
        user_id
    FROM
        Orders
    GROUP BY
        user_id
    HAVING
        SUM(amount) > 1000
),
UserOrders AS (
    -- CTE 2: Filter all orders for only the High Value Users
    SELECT
        o.order_id,
        o.amount
    FROM
        Orders AS o
    INNER JOIN
        HighValueUsers AS h ON o.user_id = h.user_id
)
-- Main Query: Calculate the average of the filtered orders
SELECT
    AVG(amount) AS avg_order_value_high_spenders
FROM
    UserOrders;
```

-----

### 2\. Using Subqueries (Derived Tables)

**Use Case:** Achieve the same result by nesting subqueries in the `FROM` clause.

```sql
SELECT
    AVG(FilteredOrders.amount) AS avg_order_value_high_spenders
FROM
    (
        -- Derived Table 1: Filter all orders for only the High Value Users
        SELECT
            o.amount
        FROM
            Orders AS o
        INNER JOIN
            (
                -- Derived Table 2: Identify users who spent over $1000 total
                SELECT
                    user_id
                FROM
                    Orders
                GROUP BY
                    user_id
                HAVING
                    SUM(amount) > 1000
            ) AS HighValueUsers ON o.user_id = HighValueUsers.user_id
    ) AS FilteredOrders;
    
-- Note: Subquery nesting can become complex quickly, hindering readability.
```

-----

### 3\. Key Difference: Recursive CTE

**Use Case:** Traverse a hierarchy (like an organizational chart) of unknown depth. This is **only possible** with a **Recursive CTE**.

```sql
WITH RECURSIVE OrgHierarchy AS (
    -- Anchor Member: Start with the top-level employee (CEO/no manager)
    SELECT
        employee_id,
        manager_id,
        employee_name,
        1 AS level
    FROM
        Employees
    WHERE
        manager_id IS NULL

    UNION ALL

    -- Recursive Member: Join to find all direct reports
    SELECT
        e.employee_id,
        e.manager_id,
        e.employee_name,
        oh.level + 1 AS level
    FROM
        Employees AS e
    INNER JOIN
        OrgHierarchy AS oh ON e.manager_id = oh.employee_id
)
-- Final Select: Display the full hierarchy
SELECT
    employee_name,
    level
FROM
    OrgHierarchy
ORDER BY
    level, employee_name;
```

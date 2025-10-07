## Subquery in SELECT vs FROM vs WHERE Cheat Sheet Table

| Clause | Subquery Type | Result Required | Purpose & Key Behavior |
| :--- | :--- | :--- | :--- |
| **`SELECT`** | **Scalar** (Must be Correlated) | **Single value** (1 row, 1 column) | Used to bring a single, summarized value from another table alongside each row of the outer query. It acts as a calculated column. |
| **`FROM`** | **Table** (Non-correlated) | **Table/Relation** (multiple rows/columns) | Used to create a temporary, derived table that the outer query can then join to, filter, or select from. Often called a **Derived Table**. |
| **`WHERE`** | **List/Scalar/Existence** | **List** (1 column) or **Single value** | Used for filtering rows: checks if a value exists in a list (`IN`), compares to a single value (Scalar), or checks for row existence (`EXISTS`). |

## Subquery in SELECT vs FROM vs WHERE Mini Playbook (Realistic Queries)

These snippets illustrate the distinct functionality of subqueries based on their placement.

### 1\. Subquery in SELECT (Scalar Subquery)

**Use Case:** Retrieve the **count of orders** for *each* customer alongside their name. This requires a correlated subquery to run for every customer.

```sql
SELECT
    c.customer_id,
    c.customer_name,
    (
        -- Inner query runs for EACH row of the outer query (correlated)
        SELECT
            COUNT(o.order_id)
        FROM
            Orders AS o
        WHERE
            o.customer_id = c.customer_id
    ) AS total_orders
FROM
    Customers AS c;
```

### 2\. Subquery in FROM (Derived Table)

**Use Case:** Find the average number of orders per day. First, calculate the daily order count (the derived table), then average those counts.

```sql
SELECT
    AVG(daily_order_count) AS avg_orders_per_day
FROM
    (
        -- Inner query creates a temporary table/view (Derived Table)
        SELECT
            order_date,
            COUNT(order_id) AS daily_order_count
        FROM
            Orders
        GROUP BY
            order_date
    ) AS DailyCounts; -- Must assign an alias to the derived table
```

### 3\. Subquery in WHERE (Filtering)

#### A. Filtering with `IN` (List Subquery)

**Use Case:** Find all products that have been included in an order placed in the last 7 days.

```sql
SELECT
    p.product_name
FROM
    Products AS p
WHERE
    p.product_id IN (
        -- Inner query returns a list of product_ids to check against
        SELECT
            DISTINCT oi.product_id
        FROM
            Order_Items AS oi
        INNER JOIN
            Orders AS o ON oi.order_id = o.order_id
        WHERE
            o.order_date >= CURRENT_DATE - INTERVAL '7' DAY
    );
```

#### B. Filtering with Scalar Subquery

**Use Case:** Find all employees with a salary higher than the average salary for their department (similar to the correlated example in the `WHERE` clause cheat sheet).

```sql
SELECT
    e.employee_name,
    e.salary,
    e.department
FROM
    Employees AS e
WHERE
    e.salary > (
        -- Inner query returns a single average value per department (correlated)
        SELECT
            AVG(e2.salary)
        FROM
            Employees AS e2
        WHERE
            e2.department = e.department
    );
```

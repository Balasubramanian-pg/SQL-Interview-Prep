This cheat sheet compares **Correlated Subqueries** and **Non-correlated Subqueries**, two ways to use subqueries (queries nested within a larger query). The key difference lies in their dependency on the outer query's rows.

## Correlated vs. Non-correlated Subqueries Cheat Sheet Table

| Feature | Non-correlated Subquery | Correlated Subquery |
| :--- | :--- | :--- |
| **Dependency** | Independent (Runs once). | Dependent (Runs once **per row** of the outer query). |
| **Execution** | The inner query runs first and completes. | The outer query provides values to the inner query, which then runs, often repeatedly. |
| **Data Flow** | Inner query result (a set of values or a single value) is passed to the outer query. | Outer query row values are passed to the inner query as parameters. |
| **Syntax Clue** | References only columns from the inner query. | References columns from the outer query (using an alias) within the inner query. |
| **Performance** | Generally **faster** as it executes only once. | Generally **slower** as it re-executes for every row processed by the outer query. |

## Correlated vs. Non-correlated Subqueries Mini Playbook (Realistic Queries)

These snippets illustrate the distinct syntax and usage of both subquery types.

### 1\. Non-correlated Subquery (Independent)

**Use Case:** Find all products whose price is **above the average price** of *all* products. The average price is calculated once.

```sql
SELECT
    product_id,
    price
FROM
    Products
WHERE
    price > (
        -- Inner query runs once to find the global average
        SELECT
            AVG(price)
        FROM
            Products
    );
```

### 2\. Correlated Subquery (Dependent on Outer Query)

**Use Case:** Find the **most recent order** for *each* customer. The inner query needs the `customer_id` from the specific row being processed by the outer query.

```sql
SELECT
    o1.customer_id,
    o1.order_date,
    o1.order_amount
FROM
    Orders AS o1 -- Outer query table
WHERE
    o1.order_date = (
        -- Inner query depends on o1.customer_id
        SELECT
            MAX(o2.order_date)
        FROM
            Orders AS o2
        WHERE
            o2.customer_id = o1.customer_id -- The correlation link
    );
```

### 3\. Correlated Subquery with `EXISTS`

**Use Case:** Find all customers who have placed an order **in the last 30 days**. This is often used for efficient existence checks.

```sql
SELECT
    c.customer_id,
    c.customer_name
FROM
    Customers AS c -- Outer query table
WHERE
    EXISTS (
        -- Inner query checks for the existence of a recent order for the specific customer (c)
        SELECT
            1
        FROM
            Orders AS o
        WHERE
            o.customer_id = c.customer_id -- The correlation link
            AND o.order_date >= CURRENT_DATE - INTERVAL '30' DAY -- Date condition (syntax varies by DB)
    );
```

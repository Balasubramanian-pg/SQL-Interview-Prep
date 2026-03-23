Multiple **Common Table Expressions (CTEs)** can be defined and used within a single SQL query by listing them sequentially in the initial `WITH` clause, separated by **commas**. This structure is essential for breaking down complex queries into manageable, logical steps, as each CTE can refer to the CTEs defined *before* it.

## Structure and Syntax

The general syntax for defining multiple CTEs is:

```sql
WITH
    CTE_Name_1 AS (
        -- The first CTE's SELECT statement
        SELECT column_a, column_b FROM Table1 WHERE condition_1
    ),
    
    CTE_Name_2 AS (
        -- The second CTE, which can reference CTE_Name_1
        SELECT column_c, COUNT(*)
        FROM CTE_Name_1
        GROUP BY column_c
    ),
    
    CTE_Name_3 AS (
        -- The third CTE, which can reference any previous CTE
        SELECT *
        FROM Table2
        WHERE status IN (SELECT status FROM CTE_Name_2)
    )

-- Final SELECT statement, which must reference at least one of the CTEs
SELECT * FROM CTE_Name_3
WHERE ...;
```

-----

## Multiple CTEs Mini Playbook (Realistic Query)

This example demonstrates a common three-step analytical process using multiple CTEs to find the **average order value (AOV) for high-spending customers**.

### Scenario: Find AOV for Top Spenders

| CTE | Purpose | Dependency |
| :--- | :--- | :--- |
| **`CustomerTotalSpend`** | Calculates the lifetime spending for every customer. | Base Table (`Orders`) |
| **`HighValueCustomers`** | Filters the results from the first CTE to identify top spenders (e.g., spending over $1,000). | `CustomerTotalSpend` |
| **`HighValueOrders`** | Filters the original `Orders` table to include only the transactions from the high-value customers. | `HighValueCustomers`, Base Table (`Orders`) |

```sql
WITH
    -- 1. Calculate the total spending per customer
    CustomerTotalSpend AS (
        SELECT
            customer_id,
            SUM(amount) AS lifetime_spend
        FROM
            Orders
        GROUP BY
            customer_id
    ),
    
    -- 2. Identify the high-value customers (spending > $1000)
    HighValueCustomers AS (
        SELECT
            customer_id
        FROM
            CustomerTotalSpend -- References CTE 1
        WHERE
            lifetime_spend > 1000
    ),
    
    -- 3. Filter all orders to include only those from high-value customers
    HighValueOrders AS (
        SELECT
            o.amount
        FROM
            Orders AS o
        INNER JOIN
            HighValueCustomers AS hvc -- References CTE 2
            ON o.customer_id = hvc.customer_id
    )

-- Final Query: Calculate the Average Order Value (AOV) from the filtered set
SELECT
    AVG(amount) AS average_order_value_for_top_spenders
FROM
    HighValueOrders;
```

-----

## Key Rules for Multiple CTEs

1.  **Sequential Definition:** All CTEs must be declared at the beginning of the query, starting with a single `WITH` keyword, and separated by commas.
2.  **Referencing Order:** A CTE can reference **any CTE defined *before* it** in the list. It cannot reference itself (unless it is a recursive CTE) or a CTE defined *after* it.
3.  **Single Final Query:** The entire `WITH` block must be followed by exactly one final `SELECT`, `INSERT`, `UPDATE`, or `DELETE` statement, which must reference at least one of the defined CTEs.

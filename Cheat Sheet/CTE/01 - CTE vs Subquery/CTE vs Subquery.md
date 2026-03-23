Choosing between **Common Table Expressions (CTEs)**, **Temporary Tables**, and **Subqueries** depends on the complexity, reusability, and performance needs of your query.

Here is a concise comparison table and a playbook for when to use each:

## CTE vs. Temp Table vs. Subquery Cheat Sheet

| Feature | Common Table Expression (CTE) | Temporary Table (Temp Table) | Subquery (Derived Table) |
| :--- | :--- | :--- | :--- |
| **Scope/Persistence** | **Single Query.** Exists only for the duration of the one query statement immediately following the `WITH` clause. | **Session/Connection.** Persists for the duration of the current database session or until explicitly dropped. | **Single Clause/Query.** Exists only within the clause (`FROM`, `SELECT`, or `WHERE`) it's nested in. |
| **Reusability** | **High within query.** Can be referenced multiple times within the same final `SELECT/INSERT/UPDATE/DELETE`. | **High across queries.** Can be referenced by subsequent queries in the same session. | **Low.** Must be redefined or repeated if needed elsewhere in the query or in subsequent queries. |
| **Indexing** | **No** (usually). Generally non-materialized, so indexing is not possible. | **Yes.** Can be indexed after creation, significantly improving join performance. | **No.** Cannot be indexed. |
| **Complexity** | Best for **readability** and **recursive** logic (Hierarchical data). | Best for **complex multi-step processing** and **large datasets** that require indexing. | Best for **simple, one-off logic** (e.g., finding an average for filtering). |
| **Syntax** | `WITH MyCTE AS (...) SELECT * FROM MyCTE` | `CREATE #Table (...) INSERT INTO #Table ...` | `SELECT * FROM (SELECT ... ) AS T` |

-----

## CTE vs. Temp Table vs. Subquery Mini Playbook

### 1\. When to Use a Common Table Expression (CTE)

Use CTEs when the primary goal is **readability, modularity, or recursion** within a single, complex statement.

  * **Recursion:** Absolutely required for handling hierarchical data (e.g., manager/employee structure) or graph traversals.
  * **Readability:** Breaking down a multi-step calculation (e.g., calculate sales, then filter sales, then aggregate final results) into logical blocks.
  * **Self-Referencing:** Referencing the same result set multiple times in the final query without redefining it.

<!-- end list -->

```sql
-- CTE Example: Calculate total sales and then filter customers based on that total
WITH CustomerSales AS (
    SELECT customer_id, SUM(amount) AS total_spent
    FROM Orders
    GROUP BY customer_id
)
SELECT *
FROM CustomerSales
WHERE total_spent > 500;
```

### 2\. When to Use a Temporary Table

Use temporary tables when you need **performance optimization, multi-step processing, or persistent storage** for the session.

  * **Large Data/Indexing:** When a preceding calculation produces a **large intermediate result** that will be joined or filtered many times. Creating a temporary table and adding an index dramatically boosts performance.
  * **Transaction Management:** When you need to process data in multiple stages (e.g., `INSERT` data, then `UPDATE` based on a complex rule, then `SELECT` the final result) and persist results between queries.
  * **Cross-Query Use:** When the results of the first query are needed in several subsequent, separate queries within the same session.

<!-- end list -->

```sql
-- Temp Table Example: Optimize join on large intermediate data
CREATE TABLE #HighValueCustomers (customer_id INT PRIMARY KEY);

INSERT INTO #HighValueCustomers
SELECT customer_id FROM Orders GROUP BY customer_id HAVING SUM(amount) > 1000;
-- Index created automatically via PRIMARY KEY or manually for performance

SELECT o.*
FROM Orders AS o
INNER JOIN #HighValueCustomers AS hvc ON o.customer_id = hvc.customer_id;
```

### 3\. When to Use a Subquery (Derived Table)

Use subqueries for **simple, immediate logic** that feeds directly into the parent query, particularly within the `FROM` clause.

  * **One-Time Use:** When the intermediate result is used only once and is not overly complex.
  * **Immediate Filtering:** Filtering rows against an aggregate or list calculated inside the subquery (`WHERE column IN (SELECT...)`).
  * **Table Aliasing:** Creating a derived table for simple joins.

<!-- end list -->

```sql
-- Subquery Example: Finding customers with above-average order frequency
SELECT customer_id, orders_count
FROM (
    -- Subquery (Derived Table) to calculate orders per customer
    SELECT customer_id, COUNT(order_id) AS orders_count
    FROM Orders
    GROUP BY customer_id
) AS CustomerCounts
WHERE orders_count > (
    -- Scalar Subquery to find the overall average order count
    SELECT AVG(orders_count) FROM (
        SELECT COUNT(order_id) AS orders_count FROM Orders GROUP BY customer_id
    ) AS AvgCalc
);
```

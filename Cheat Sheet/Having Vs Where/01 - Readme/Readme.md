## HAVING vs. WHERE Cheat Sheet Table

| Feature | WHERE Clause | HAVING Clause |
| :--- | :--- | :--- |
| **Stage of Execution** | **Before** aggregation (`GROUP BY`). | **After** aggregation (`GROUP BY`). |
| **Data Filtered** | **Individual Rows** (records from the tables). | **Groups** (results of the `GROUP BY` operation). |
| **Can Reference** | **Non-aggregated columns** (i.e., columns directly from the tables). | **Aggregated functions** (`SUM()`, `AVG()`, `COUNT()`, etc.) and non-aggregated columns. |
| **Order in Query** | Comes after `FROM` and `JOIN` clauses. | Comes after the `GROUP BY` clause. |
| **Primary Use** | Filtering out specific rows that do not meet initial criteria (e.g., `price > 100`). | Filtering out entire groups based on a summary condition (e.g., `AVG(price) > 500`). |

## HAVING vs. WHERE Mini Playbook (Realistic Queries)

These snippets illustrate the distinct use cases for `WHERE` and `HAVING` in a scenario involving filtering orders by date and then filtering the resulting customer groups by total spend.

### 1\. Using WHERE (Filtering Individual Rows)

**Use Case:** Find the total sales amount for orders placed **only in 2025**. The filter applies to the `order_date` *before* grouping.

```sql
SELECT
    customer_id,
    SUM(amount) AS total_sales
FROM
    Orders
WHERE
    -- Filters individual order rows BEFORE grouping
    order_date BETWEEN '2025-01-01' AND '2025-12-31'
GROUP BY
    customer_id;
```

### 2\. Using HAVING (Filtering Aggregated Groups)

**Use Case:** Calculate the total sales for *every* customer, and then only show the customers whose **total sales exceeded $500**.

```sql
SELECT
    customer_id,
    SUM(amount) AS total_sales
FROM
    Orders
GROUP BY
    customer_id
HAVING
    -- Filters the results of the grouping based on the aggregated value
    SUM(amount) > 500;
```

### 3\. Using BOTH WHERE and HAVING (Combined Filtering)

**Use Case:** Find customers who placed orders **only in 2025** (using `WHERE`), and whose total sales for that period **exceeded $1000** (using `HAVING`).

```sql
SELECT
    customer_id,
    SUM(amount) AS total_sales_2025
FROM
    Orders
WHERE
    -- Stage 1: Filter individual rows (orders) to 2025
    order_date BETWEEN '2025-01-01' AND '2025-12-31'
GROUP BY
    customer_id
HAVING
    -- Stage 2: Filter the resulting groups based on the aggregated sum
    SUM(amount) > 1000;
```

### 4\. WHERE vs. HAVING on Non-Aggregated Columns

While you **can** use `HAVING` on a non-aggregated column (like `customer_id`), it is generally **slower and less efficient** than using `WHERE` and is strongly discouraged.

```sql
-- CORRECT and RECOMMENDED (Runs BEFORE aggregation)
SELECT customer_id, SUM(amount) FROM Orders WHERE customer_id = 10 GROUP BY customer_id;

-- DISCOURAGED (Runs AFTER aggregation)
SELECT customer_id, SUM(amount) FROM Orders GROUP BY customer_id HAVING customer_id = 10;
```

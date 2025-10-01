
## Multiple Aggregations on Same Column Cheat Sheet Table

| Problem Type | What They're Testing | Solution Pattern | Key Technique |
| :--- | :--- | :--- | :--- |
| **Basic Multiple Metrics** | Calculating standard aggregates (Min, Max, Avg, Sum) on a column in one go. | Use standard aggregate functions with different aliases in the `SELECT` clause. | `$SELECT MIN(col), MAX(col), AVG(col), SUM(col) FROM Table$` |
| **Conditional Aggregation** | Calculating aggregates based on different conditions/segments of the data. | Use the **`CASE`** statement *inside* the aggregate function (often with `SUM` or `COUNT`). | `$SUM(CASE WHEN condition THEN column ELSE 0 END)$` |
| **Proportional Aggregation** | Calculating the percentage of a part relative to the whole, grouped by a field. | Use a **Window Function** (e.g., `SUM() OVER ()`) to get the total and divide the group total by it. | `$SUM(group\_col) / SUM(SUM(group\_col)) OVER ()$` |
| **Filter-First Aggregation** | Aggregating a column only after filtering the initial rows (before grouping). | Use the **`WHERE`** clause to filter rows, then aggregate the result. | `$SELECT SUM(col) FROM Table WHERE condition GROUP BY group\_col$` |

-----

## Multiple Aggregations on Same Column Mini Playbook (Realistic Queries)

These snippets illustrate ways to extract various statistics from the `amount` column in a single pass.

### 1\. Basic Multiple Metrics

**Use Case:** Get the total, average, minimum, and maximum sale `amount` for each region.

```sql
SELECT
    region_id,
    SUM(sale_amount) AS total_region_sales,
    AVG(sale_amount) AS average_region_sale,
    MIN(sale_amount) AS lowest_sale,
    MAX(sale_amount) AS highest_sale
FROM
    Sales
GROUP BY
    region_id;
```

-----

### 2\. Conditional Aggregation (SUM and COUNT with CASE)

**Use Case:** For each user, calculate the **total successful transactions** and the **total failed transactions** using the same `transaction_amount` column.

```sql
SELECT
    user_id,
    -- Sum of amounts only for 'SUCCESS' transactions
    SUM(CASE WHEN status = 'SUCCESS' THEN transaction_amount ELSE 0 END) AS total_successful_amount,
    -- Count of transactions only for 'FAILED' status
    COUNT(CASE WHEN status = 'FAILED' THEN 1 ELSE NULL END) AS count_failed_transactions
FROM
    Transactions
GROUP BY
    user_id;
```

-----

### 3\. Proportional Aggregation (Percent of Total)

**Use Case:** Calculate each product's sales **amount** and its percentage contribution to the **grand total sales amount**.

```sql
SELECT
    product_id,
    SUM(amount) AS product_sales,
    -- Divide product sales by the total sales across the entire dataset (no PARTITION BY)
    SUM(amount) / SUM(SUM(amount)) OVER () AS percent_of_total
FROM
    Orders
GROUP BY
    product_id;
```

-----

### 4\. Filter-First Aggregation (WHERE vs. HAVING)

**Use Case:** Find the **total amount** for orders placed **this year** (using `WHERE`) and the **average amount** only for customers whose total spending **exceeds $500** (using `HAVING`).

```sql
SELECT
    customer_id,
    SUM(order_amount) AS current_year_total_spend,
    AVG(order_amount) AS average_order_value
FROM
    Orders
WHERE
    -- Filter rows before aggregation to include only current year orders
    order_date >= DATE_TRUNC('year', CURRENT_DATE) -- Example for current year filter
GROUP BY
    customer_id
HAVING
    -- Filter the groups after aggregation
    SUM(order_amount) > 500;
```

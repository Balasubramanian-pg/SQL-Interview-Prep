This cheat sheet covers the essential SQL techniques for **limiting the number of rows** returned by a query. This is critical for performance (especially in reporting), pagination in applications, and retrieving top-N results.

## Limiting Result Sets Cheat Sheet Table

| Technique | Database Support | Syntax Pattern | Key Use Case |
| :--- | :--- | :--- | :--- |
| **`LIMIT`** | MySQL, PostgreSQL, SQLite, CockroachDB | `$SELECT ... LIMIT N [OFFSET M]$` | Standard for simple limiting and basic pagination. |
| **`TOP`** | SQL Server, MS Access | `$SELECT TOP N ...$` | Microsoft standard for retrieving the first N rows. |
| **`FETCH NEXT`** | ANSI SQL Standard, Oracle, SQL Server (2012+) | `$ORDER BY col [OFFSET M ROWS] FETCH NEXT N ROWS ONLY$` | The modern, portable ANSI standard for pagination. |
| **Window Functions** | All major databases | `$ROW_NUMBER() OVER (ORDER BY col)$` | Complex top-N per-group analysis (e.g., finding the top 3 products per department). |

## Limiting Result Sets Mini Playbook (Realistic Queries)

These snippets illustrate the primary methods for limiting results.

### 1\. LIMIT (Simple Limiting)

**Use Case:** Retrieve the 10 most recent orders. **Requires `ORDER BY` for meaningful results.**

```sql
SELECT
    order_id,
    order_date,
    amount
FROM
    Orders
ORDER BY
    order_date DESC -- Must sort to ensure you get the "most recent"
LIMIT 10;
```

### 2\. LIMIT with OFFSET (Pagination)

**Use Case:** Retrieve results for **Page 3** of a list, where each page contains 50 records.

  * **Page 1:** `LIMIT 50 OFFSET 0`
  * **Page 2:** `LIMIT 50 OFFSET 50`
  * **Page 3:** `LIMIT 50 OFFSET 100`

<!-- end list -->

```sql
SELECT
    product_name,
    price
FROM
    Products
ORDER BY
    product_name
LIMIT 50 OFFSET 100;
```

### 3\. TOP (SQL Server)

**Use Case:** Select the top 5 highest-paid employees.

```sql
SELECT TOP 5
    employee_name,
    salary
FROM
    Employees
ORDER BY
    salary DESC;
```

### 4\. FETCH NEXT / OFFSET (ANSI Standard)

**Use Case:** Retrieve results for **Page 2** with 20 rows per page (equivalent to `LIMIT 20 OFFSET 20`). This is the most standard and recommended syntax for pagination across modern SQL databases.

```sql
SELECT
    report_id,
    report_date
FROM
    Reports
ORDER BY
    report_date DESC
OFFSET 20 ROWS        -- Skip the first 20 rows (Page 1)
FETCH NEXT 20 ROWS ONLY; -- Retrieve the next 20 rows (Page 2)
```

### 5\. Window Function (Top N Per Group)

**Use Case:** Find the **single highest-rated product** within each `category`.

```sql
WITH RankedProducts AS (
    SELECT
        category,
        product_name,
        rating,
        -- Assigns a rank starting from 1 for the highest rating within each category
        ROW_NUMBER() OVER (PARTITION BY category ORDER BY rating DESC) AS rn
    FROM
        Products
)
SELECT
    category,
    product_name,
    rating
FROM
    RankedProducts
WHERE
    rn = 1; -- Filter for only the top-ranked item in each category
```

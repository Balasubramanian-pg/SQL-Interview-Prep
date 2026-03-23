## NTILE (Dividing into Buckets) Cheat Sheet Table

| Problem Type | What They're Testing | Solution Pattern | Key Technique |
| :--- | :--- | :--- | :--- |
| **Quartile Analysis** | Dividing a population into 4 equal groups (25% ranges). | Use **`NTILE(4)`** to create four buckets. | `$NTILE(4) OVER (ORDER BY sorting\_col)$` |
| **Decile Analysis** | Dividing a population into 10 equal groups (10% ranges). | Use **`NTILE(10)`** to create ten buckets. | `$NTILE(10) OVER (ORDER BY sorting\_col)$` |
| **Custom Bucketing** | Dividing data into an arbitrary number ($N$) of approximately equal groups. | Use **`NTILE(N)`** with appropriate ordering. | `$NTILE(N) OVER (ORDER BY sorting\_col)$` |
| **Grouped Bucketing** | Dividing groups into buckets *within* each group (e.g., quartiles per department). | Use **`NTILE(N)`** with a `PARTITION BY` clause. | `$NTILE(N) OVER (PARTITION BY group\_col ORDER BY sorting\_col)$` |

## NTILE (Dividing into Buckets) Mini Playbook (SQL Snippets)

These snippets illustrate the common use cases for the `NTILE` function.

### 1\. Quartile Analysis

Determine which **quartile (1 to 4)** a customer's total spending falls into. Lower quartile number means lower spending.

```sql
SELECT
    customer_id,
    total_spend,
    NTILE(4) OVER (
        -- Orders by spend to rank customers
        ORDER BY total_spend
    ) AS spend_quartile
FROM
    customer_data;
```

### 2\. Decile Analysis

Assign a **performance decile (1 to 10)** to employees based on their annual review score, where 10 is the top decile.

```sql
SELECT
    employee_name,
    review_score,
    NTILE(10) OVER (
        -- Orders by score descending so the highest score is in decile 1 (can be reversed)
        ORDER BY review_score DESC
    ) AS performance_decile
FROM
    employee_reviews;
```

### 3\. Custom Bucketing

Divide a set of sales leads into **5 priority buckets**, where leads with the highest scores get the lowest bucket number (e.g., 'Bucket 1' is highest priority).

```sql
SELECT
    lead_id,
    qualification_score,
    NTILE(5) OVER (
        -- Orders by score descending for priority assignment
        ORDER BY qualification_score DESC
    ) AS priority_bucket
FROM
    sales_leads;
```

### 4\. Grouped Bucketing

Determine the **salary quartile** for each employee *within their specific department*.

```sql
SELECT
    employee_id,
    department,
    salary,
    NTILE(4) OVER (
        -- Partitions by department to reset the quartile count for each group
        PARTITION BY department
        ORDER BY salary
    ) AS department_salary_quartile
FROM
    employees;
```

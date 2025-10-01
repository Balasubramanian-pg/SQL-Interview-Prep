## GROUP BY with CASE Statements Cheat Sheet Table

| Problem Type | What They're Testing | Solution Pattern | Key Technique |
| :--- | :--- | :--- | :--- |
| **Custom Bucketing/Segmentation** | Grouping rows based on dynamic criteria (e.g., spending tiers, age ranges). | Define the categories using a `CASE` statement in the `SELECT` clause, then repeat that same `CASE` logic in the `GROUP BY` clause. | `$GROUP BY (CASE WHEN condition THEN 'Bucket A' ELSE 'Bucket B' END)$` |
| **Conditional Aggregation** (Reminder) | Aggregating specific subsets without changing the grouping columns. | Use the `CASE` statement *inside* the aggregate function (e.g., `SUM` or `COUNT`). | `$SUM(CASE WHEN condition THEN column ELSE 0 END)$` |
| **Grouping Status/Flags** | Grouping data based on complex status logic derived from multiple columns. | Use `CASE` to simplify or standardize status codes into meaningful groups. | `$GROUP BY (CASE WHEN col1=A AND col2=B THEN 'Special' ELSE 'Standard' END)$` |
| **Grouping by Null/Missing Values** | Standardizing the treatment of `NULL` or zero values into a single reporting group. | Use `CASE` to explicitly assign a name (e.g., 'Unknown' or 'No Value') to the missing group. | `$GROUP BY (CASE WHEN col IS NULL THEN 'Unknown' ELSE col END)$` |

-----

## GROUP BY with CASE Statements Mini Playbook (Realistic Queries)

These snippets illustrate the use of `CASE` within the `GROUP BY` clause for dynamic segmentation.

### 1\. Custom Bucketing/Segmentation (Spending Tiers)

**Use Case:** Group customers into custom spending tiers (High, Medium, Low) and count how many customers fall into each tier.

```sql
SELECT
    -- Define the group name
    CASE
        WHEN lifetime_spend >= 500 THEN 'High Value'
        WHEN lifetime_spend >= 100 THEN 'Medium Value'
        ELSE 'Low Value'
    END AS spending_tier,
    COUNT(customer_id) AS customer_count,
    AVG(lifetime_spend) AS avg_spend_in_tier
FROM
    Customers
GROUP BY
    -- Must repeat the exact same CASE logic here to group by the new category
    CASE
        WHEN lifetime_spend >= 500 THEN 'High Value'
        WHEN lifetime_spend >= 100 THEN 'Medium Value'
        ELSE 'Low Value'
    END;
```

-----

### 2\. Grouping Status/Flags (Risk Categories)

**Use Case:** Group accounts based on a derived risk score where different combinations of flags map to the same category.

```sql
SELECT
    -- Define the group name
    CASE
        WHEN (credit_score < 600 OR default_flag = TRUE) THEN 'High Risk'
        WHEN (age < 25 AND loan_amount > 50000) THEN 'Medium Risk'
        ELSE 'Low Risk'
    END AS risk_category,
    COUNT(account_id) AS account_count,
    SUM(loan_balance) AS total_exposure
FROM
    Accounts
GROUP BY
    -- Group by the derived risk category
    CASE
        WHEN (credit_score < 600 OR default_flag = TRUE) THEN 'High Risk'
        WHEN (age < 25 AND loan_amount > 50000) THEN 'Medium Risk'
        ELSE 'Low Risk'
    END;
```

-----

### 3\. Grouping by Null/Missing Values

**Use Case:** Group sales by representative, treating all sales without an assigned representative (`NULL`) as a single 'Unassigned' group.

```sql
SELECT
    -- Standardize the rep name
    CASE
        WHEN sales_rep_name IS NULL THEN 'UNASSIGNED'
        ELSE sales_rep_name
    END AS representative_group,
    SUM(sale_amount) AS total_sales
FROM
    Sales
GROUP BY
    -- Group by the standardized name
    CASE
        WHEN sales_rep_name IS NULL THEN 'UNASSIGNED'
        ELSE sales_rep_name
    END;
```

-----

### 4\. Grouping by Column Alias (If supported by your SQL dialect)

*Note: While many modern SQL dialects (like PostgreSQL, MySQL) allow grouping by the column alias defined in the `SELECT` clause, SQL standards (and older systems like SQL Server) generally require repeating the full `CASE` expression in the `GROUP BY` clause. Use the full expression (as shown in 1-3) for maximum portability.*

```sql
-- *** ONLY WORKS IN SOME DBs (e.g., PostgreSQL, MySQL) ***
SELECT
    CASE
        WHEN amount > 100 THEN 'Large'
        ELSE 'Small'
    END AS size_category,
    COUNT(*)
FROM
    Transactions
GROUP BY
    size_category; -- Grouping by the alias
```

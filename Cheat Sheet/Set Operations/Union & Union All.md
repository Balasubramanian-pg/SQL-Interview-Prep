This cheat sheet covers the difference between **`UNION`** and **`UNION ALL`**, two operators used to combine the result sets of two or more `SELECT` statements into a single result set.

## UNION vs. UNION ALL Cheat Sheet Table

| Feature | UNION | UNION ALL |
| :--- | :--- | :--- |
| **Duplicate Rows** | **Removes** duplicate rows from the final result set. | **Retains** all duplicate rows from all combined result sets. |
| **Performance** | **Slower.** Requires an internal sort operation to identify and remove duplicates. | **Faster.** Simply appends the data; no sorting or duplicate checking is performed. |
| **Data Integrity** | Useful when you need a perfectly distinct list of records. | Useful when you need the complete, original record count (e.g., counting total transactions from multiple sources). |
| **Sorting** | The entire result set is implicitly sorted for duplicate removal. | No implicit sorting occurs, maintaining the order of the combined tables. |
| **Requirement** | Both queries **must** have the same number of columns, and the corresponding columns **must** have compatible data types. | Both queries **must** have the same number of columns, and the corresponding columns **must** have compatible data types. |

-----

## UNION vs. UNION ALL Mini Playbook (Realistic Queries)

These snippets illustrate when to use each operator using two tables: `Sales_2024` and `Sales_2025`.

### 1\. UNION (Removing Duplicates)

**Use Case:** Get a **distinct list of all product IDs** that were sold in either 2024 **or** 2025. If a product was sold in both years, it appears only once.

```sql
SELECT
    product_id,
    product_name
FROM
    Sales_2024

UNION

SELECT
    product_id,
    product_name
FROM
    Sales_2025;
    
-- RESULT: A single, de-duplicated list of products.
```

-----

### 2\. UNION ALL (Retaining All Rows)

**Use Case:** Combine all sales records from both 2024 and 2025 to calculate the **total transaction count and total revenue** across both years. Duplicates (if they exist in the raw source data) are preserved.

```sql
SELECT
    order_id,
    sale_date,
    amount
FROM
    Sales_2024

UNION ALL

SELECT
    order_id,
    sale_date,
    amount
FROM
    Sales_2025;
    
-- RESULT: A large, combined list that includes every single transaction from both tables.
```

-----

### 3\. Combining UNION with Literal Columns

**Use Case:** Combine data while adding a literal string column to easily identify the source table for each record. This is a common practice when combining historical data.

```sql
SELECT
    customer_id,
    transaction_amount,
    'Source_2024' AS source_year
FROM
    Sales_2024

UNION ALL

SELECT
    customer_id,
    transaction_amount,
    'Source_2025' AS source_year
FROM
    Sales_2025

ORDER BY
    source_year, customer_id;
    
-- NOTE: ORDER BY can only be used once at the end of the combined result set.
```

Avoiding `SELECT *` (selecting all columns) is a crucial best practice in SQL for improving query performance, reducing network traffic, enhancing security, and maintaining code stability.

## Why You Should Avoid `SELECT *` 🛑

| Reason | Explanation | Performance Impact |
| :--- | :--- | :--- |
| **Network & Memory Overhead** | You retrieve data you don't need, consuming unnecessary bandwidth and database server memory, especially when dealing with tables that have many columns (e.g., 50+). | **Worse.** Slower query execution and higher server load. |
| **Index Use (Covering Indexes)** | When you specify only necessary columns, the database might be able to use a **covering index** (an index that includes all requested columns). If you use `SELECT *`, the database is usually forced to access the full table data. | **Worse.** Prevents optimized *index-only* scans. |
| **Query Stability** | If the underlying table schema changes (e.g., columns are reordered, renamed, or new ones are added), `SELECT *` breaks code in downstream applications that rely on column position or implicitly adds unexpected data to your reports. | **Worse.** Leads to broken reports and unstable code. |
| **Security Principle** | It exposes sensitive or unnecessary data fields (e.g., internal IDs, audit columns, or unmasked PII) to an application layer or user who doesn't need to see them. | **Better.** Reduces exposure of sensitive information. |

-----

## Recommended Alternatives (Best Practices) ✅

The best practice is always to **explicitly list the columns** you need.

### 1\. Explicit Column Listing

Always list every required column. This is the foundation of maintainable, high-performance SQL.

```sql
-- BAD ❌: Retrieves the entire row unnecessarily
SELECT *
FROM Employees
WHERE status = 'Active';

-- GOOD ✅: Retrieves only the necessary columns
SELECT employee_id, first_name, last_name, email, hire_date
FROM Employees
WHERE status = 'Active';
```

### 2\. Qualified Column Listing

When joining tables, using a table alias prefix (`t1.column_name`) prevents naming ambiguity and improves readability, even when you aren't using `SELECT *`.

```sql
SELECT
    c.customer_id,
    c.first_name,
    o.order_id,
    o.order_date,
    o.amount
FROM
    Customers AS c
INNER JOIN
    Orders AS o ON c.customer_id = o.customer_id;
```

### 3\. Using Aliases for Clarity

When columns have identical names across tables (e.g., `last_modified_date`), using aliases in the `SELECT` list prevents confusion in the final result set.

```sql
SELECT
    c.last_modified_date AS customer_last_update,
    o.last_modified_date AS order_last_update
FROM
    Customers AS c
INNER JOIN
    Orders AS o ON ...;
```

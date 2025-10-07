Removing duplicate records is a crucial data cleaning task in SQL. The most common and reliable method uses **Window Functions** to rank the duplicates, allowing you to selectively delete the unwanted copies while preserving the desired "original."

## 1\. The Recommended Method: Using `ROW_NUMBER()` (The Safest Way) 
This technique is the safest because it explicitly defines which record to **keep** (the original) and which to **delete** (the copies).

### Technique

1.  **Rank Duplicates:** Use the **`ROW_NUMBER()`** window function partitioned by the columns that define a duplicate (e.g., `email`). The `ORDER BY` clause determines which row gets rank `1` (the one to keep).
2.  **Filter/Delete:** Use a CTE (Common Table Expression) or subquery to target the records where the rank (`rn`) is greater than 1.

### Example: Deleting Duplicates, Keeping the Oldest Record

Assume a `Leads` table has duplicate `email` addresses, and you want to **keep the record with the earliest `created_at` timestamp**.

```sql
WITH RankedLeads AS (
    SELECT
        lead_id, -- Primary key needed for the DELETE statement
        email,
        created_at,
        -- Assigns a row number partitioned by email
        ROW_NUMBER() OVER (
            PARTITION BY email  -- Group by the duplicate-defining column
            ORDER BY created_at ASC -- The oldest record (earliest created_at) gets rank 1
        ) AS rn
    FROM
        Leads
)
-- Target the rows with rank > 1 for deletion
DELETE FROM Leads 
WHERE lead_id IN (
    SELECT lead_id
    FROM RankedLeads
    WHERE rn > 1 
);
```

### Key Considerations for `ROW_NUMBER()`

  * **`PARTITION BY`**: Crucial. List all columns that together define a duplicate (e.g., `(first_name, last_name, phone)`).
  * **`ORDER BY`**: Defines the "winner." For example, use `ORDER BY created_at ASC` to keep the oldest record, or `ORDER BY last_updated_at DESC` to keep the newest record.

## 2\. Using `GROUP BY` and `MIN/MAX` (When Keeping One Specific Value)

If you only need to keep one specific unique key (like the `MIN(primary_key)`) for each duplicate group, you can use `GROUP BY` and `HAVING`.

### Technique

1.  Group the records by the duplicate-defining column (`email`).
2.  Find the key to keep (e.g., `MIN(lead_id)`).
3.  Delete all records whose keys are **not** the one you kept.

### Example: Deleting Duplicates, Keeping the Record with the Lowest ID

```sql
DELETE FROM Leads
WHERE lead_id NOT IN (
    SELECT MIN(lead_id) -- Find the smallest (oldest) ID for each unique email
    FROM Leads
    GROUP BY email      -- Group all rows that are duplicates
);
```

  * **Warning:** This method is concise but lacks the flexibility of `ROW_NUMBER()` if your criteria for keeping the original record is more complex than just `MIN()` or `MAX()`.

## 3\. Using `DISTINCT` (Read-Only) 

The `DISTINCT` keyword removes duplicate rows from a result set but **does not modify the underlying table data**. It is only for viewing or inserting into a new table.

### Example (Viewing Unique Rows)

```sql
-- Shows only the unique combination of first_name and last_name in the result set
SELECT DISTINCT
    first_name,
    last_name
FROM
    Employees;
```

### Example (Inserting Unique Rows into a New Table)

```sql
-- Creates a new table containing only unique emails
INSERT INTO Unique_Leads (email, created_at)
SELECT DISTINCT email, created_at 
FROM Leads;
```

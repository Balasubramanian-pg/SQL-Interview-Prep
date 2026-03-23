Finding duplicate records in SQL is a common data quality task, typically accomplished by identifying rows that share the same values across a specific set of columns.

The most effective and common method involves using **Window Functions** or the **`GROUP BY`** clause with the **`HAVING`** clause.

## 1\. Identifying and Viewing Duplicates (GROUP BY + HAVING) 

This method is the most straightforward way to find the values that have duplicates and count how many times they appear.

### Technique

1.  **Group:** Use `GROUP BY` on the column(s) you suspect have duplicates (e.g., `email`, `first_name`, `last_name`).
2.  **Count:** Use the `COUNT(*)` aggregate function to count the occurrences within each group.
3.  **Filter:** Use the **`HAVING`** clause to filter these groups, keeping only those where the count is greater than 1.

### Example: Finding Duplicate Email Addresses

Assume a `Users` table. We want to find all email addresses that appear more than once.

```sql
SELECT
    email,
    COUNT(*) AS total_duplicates
FROM
    Users
GROUP BY
    email
HAVING
    COUNT(*) > 1; -- Filter for groups with more than one record
```

## 2\. Viewing the Full Duplicate Rows (Window Function) 💡

If you need to see all the data for every row that is a duplicate (including the original and all copies), the **`ROW_NUMBER()`** window function is the standard, high-performance solution.

### Technique

1.  **Partition:** Use `ROW_NUMBER() OVER (PARTITION BY ... ORDER BY ...)` to divide the data into groups based on the columns that define the duplicate (e.g., `email`).
2.  **Rank:** Within each group, assign a sequential rank (`rn`).
3.  **Filter:** Use a CTE or subquery to filter for any row where the rank (`rn`) is greater than 1. These rows are the true duplicates.

### Example: Finding All Duplicates of the Same Email

```sql
WITH RankedUsers AS (
    SELECT
        user_id,
        first_name,
        email,
        -- Assigns a row number to each record within a group of identical emails
        ROW_NUMBER() OVER (
            PARTITION BY email  -- Defines what constitutes a duplicate group
            ORDER BY user_id    -- Determines which row gets rank 1 (the 'original')
        ) AS rn
    FROM
        Users
)
SELECT
    user_id,
    first_name,
    email
FROM
    RankedUsers
WHERE
    rn > 1; -- Keeps all rows that are not the first occurrence
```

## 3\. Deleting Duplicates (Window Function) 

The `ROW_NUMBER()` method is also the safest way to delete duplicates, as it allows you to explicitly define which copy is the "original" to keep (rank 1) and which copies to delete (rank \> 1).

### Technique

Use the same `ROW_NUMBER()` CTE as above, but target the duplicate ranks (`rn > 1`) in the `DELETE` statement.

### Example: Deleting Duplicate Emails, Keeping the Lowest ID

```sql
-- DELETE requires the source to be a CTE/subquery referencing the base table
DELETE FROM Users 
WHERE user_id IN (
    SELECT user_id
    FROM (
        SELECT
            user_id,
            email,
            ROW_NUMBER() OVER (
                PARTITION BY email
                ORDER BY user_id -- Keep the row with the lowest user_id (the first one created)
            ) AS rn
        FROM
            Users
    ) AS Duplicates
    WHERE rn > 1 -- Target all records that are not the 'original' (rank 1)
);
```

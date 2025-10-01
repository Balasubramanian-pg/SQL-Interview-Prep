This cheat sheet covers the crucial distinction between correctly checking for `NULL` values using **`IS NULL`** and the common mistake of using the standard equality operator **`= NULL`** in SQL comparisons. Understanding this behavior is fundamental because `NULL` represents an unknown or missing value, not a zero or an empty string.

## NULL in Comparisons Cheat Sheet Table

| Operator | Usage | Logic and Result | Key Technique |
| :--- | :--- | :--- | :--- |
| **`IS NULL`** | Checks if a value **is** the `NULL` state. | Correctly evaluates to **TRUE** if the column contains `NULL`. | Use this **always** to identify missing or unknown values. |
| **`IS NOT NULL`** | Checks if a value **is not** the `NULL` state. | Correctly evaluates to **TRUE** if the column contains any non-NULL data (even an empty string or zero). | Use this **always** to filter out missing or unknown values. |
| **`= NULL`** | Checks for standard equality against the literal word `NULL`. | Evaluates to **UNKNOWN** (and thus fails the `WHERE` clause condition). | **Never use** for checking `NULL` values. This will always return an empty result set. |
| **`<> NULL` or `!= NULL`** | Checks for standard inequality against the literal word `NULL`. | Also evaluates to **UNKNOWN** (and thus fails the `WHERE` clause condition). | **Never use** for checking non-`NULL` values. |

-----

## NULL in Comparisons Mini Playbook (Realistic Queries)

These snippets illustrate the correct and incorrect ways to handle `NULL` in `WHERE` clauses, assuming a table `Tasks` with a nullable `completed_date` column.

### 1\. Correctly Filtering for NULLs

**Use Case:** Find all tasks that are **still incomplete** (where `completed_date` is `NULL`).

```sql
SELECT
    task_id,
    task_description
FROM
    Tasks
WHERE
    completed_date IS NULL;
    
-- RESULT: Returns all rows where the date is unknown/missing.
```

-----

### 2\. Correctly Filtering for Non-NULLs

**Use Case:** Find all tasks that **have been completed** (where `completed_date` is a specific date value, i.e., not `NULL`).

```sql
SELECT
    task_id,
    completed_date
FROM
    Tasks
WHERE
    completed_date IS NOT NULL;
    
-- RESULT: Returns all rows where the date is a known value.
```

-----

### 3\. The Incorrect (and Failing) Equality Check

**Use Case:** Attempt to find incomplete tasks using the standard equality operator. **This query will fail to return any rows.**

```sql
SELECT
    task_id,
    task_description
FROM
    Tasks
WHERE
    completed_date = NULL;
    
-- RESULT: Zero rows returned, even if NULLs exist, because the comparison evaluates to UNKNOWN.
```

-----

### 4\. Handling Comparisons with COALESCE/ISNULL (Workaround)

**Use Case:** You need to compare two nullable columns (`T1.col` and `T2.col`) and want to treat two `NULL`s as equal for comparison logic (e.g., in a `WHERE` clause, not a `JOIN`). This requires replacing `NULL` with a consistent, non-existent sentinel value (like a specific date or string).

```sql
SELECT
    T1.record_id
FROM
    Table1 AS T1
INNER JOIN
    Table2 AS T2 ON T1.record_id = T2.record_id
WHERE
    -- Replaces NULL in both columns with a common, impossible value ('-NULL-')
    COALESCE(T1.optional_field, '-NULL-') = COALESCE(T2.optional_field, '-NULL-');
    
-- RESULT: If T1.optional_field is NULL and T2.optional_field is NULL, the comparison is '-NULL-' = '-NULL-', which is TRUE.
```

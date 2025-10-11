Finding **consecutive events** in SQL involves identifying sequences of records where a key value or status is repeated immediately after the preceding row, usually defined by a chronological column like a date or a sequence ID.

The most robust, standard, and high-performance method for this is the **Gaps and Islands** technique, which relies on **Window Functions** to group the consecutive runs of identical values.

## Gaps and Islands Technique for Consecutive Events 

This technique works by creating a unique identifier for each run (or "island") of consecutive, identical values.

### Example: Finding a Run of Consecutive 'Success' Statuses

Assume a table, `ProcessLogs`, with columns `log_time` and `status` ('Success' or 'Failure'). We want to find runs where the process status was 'Success' for 3 or more consecutive logs.

### Step 1: Identify Status Changes

First, use the **`LAG()`** window function to compare the current row's `status` to the previous row's `status` (ordered by `log_time`). Assign a value of **1** when the status changes, and **0** when it remains the same.

```sql
WITH StatusChanges AS (
    SELECT
        log_time,
        status,
        -- Check if the current status is different from the previous status
        CASE
            WHEN status = LAG(status) OVER (ORDER BY log_time) THEN 0
            ELSE 1
        END AS is_new_group
    FROM
        ProcessLogs
)
```

### Step 2: Create a Group ID (The Island)

Next, use a **cumulative sum** over the `is_new_group` column. Every time the status changes (where the value is 1), the running sum increments, effectively creating a unique `status_group_id` for each consecutive run of an identical status.

```sql
, GroupedStatus AS (
    SELECT
        log_time,
        status,
        -- Cumulative SUM creates a unique ID for each consecutive run ('Island')
        SUM(is_new_group) OVER (ORDER BY log_time) AS status_group_id
    FROM
        StatusChanges
)
```

### Step 3: Aggregate and Filter

Finally, group the data by the unique `status_group_id` and the `status` value. Use the `HAVING` clause to filter for groups that have a count greater than or equal to your required length (e.g., 3).

```sql
SELECT
    status,
    MIN(log_time) AS run_start,
    MAX(log_time) AS run_end,
    COUNT(*) AS run_length
FROM
    GroupedStatus
WHERE
    status = 'Success' -- Filter to check only for 'Success' runs
GROUP BY
    status, status_group_id
HAVING
    COUNT(*) >= 3 -- Filter for runs of 3 or more logs
ORDER BY
    run_start;
```
## Alternative: Finding Consecutive Sequential IDs

When the problem involves finding consecutive **sequential numbers** (like $ID, ID+1, ID+2, \dots$), you can use a simpler approach by comparing the sequence ID to its corresponding row number.

### Technique

1.  Assign a **`ROW_NUMBER()`** partitioned and ordered by the relevant grouping column (if needed).
2.  Subtract the row number from the sequence ID: $\text{Sequence ID} - \text{Row Number} = \text{Constant}$.
3.  Any rows that are consecutive will have the same difference (the constant value), thus forming a group that can be aggregated.

### Example: Finding Consecutive IDs

```sql
WITH SequenceDifference AS (
    SELECT
        event_id,
        user_id,
        -- Calculate the difference (event_id - row_number)
        event_id - ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY event_id) AS group_key
    FROM
        Events
)
SELECT
    user_id,
    MIN(event_id) AS run_start,
    MAX(event_id) AS run_end,
    COUNT(*) AS run_length
FROM
    SequenceDifference
GROUP BY
    user_id, group_key
HAVING
    COUNT(*) >= 3;
```

  * **How it works:** If the IDs are 10, 11, 12, the row numbers are 1, 2, 3. The difference is $10-1=9$, $11-2=9$, $12-3=9$. The constant difference (`group_key = 9`) identifies the consecutive run.

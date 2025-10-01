Finding **consecutive records** in SQL involves identifying sequences of data where a value in a key column follows immediately after the value in the preceding row (e.g., consecutive dates, sequential IDs, or matching status entries).

The most reliable and standard technique for solving this problem is using **Window Functions**, specifically **`LAG()`** or **`LEAD()`**, to compare the current row's data against its predecessor or successor.

-----

## 1\. Finding Consecutive IDs or Dates (Sequence-Based)

This technique identifies records where the current value is exactly one step higher (or one day later) than the previous value.

### Technique

1.  Use the **`LAG()`** function to retrieve the value of the key column from the row immediately preceding the current row, based on the sorting order.
2.  In the outer query or a CTE, compare the current value to the lagged value. If `(current_value - lagged_value) = 1` (or 1 day), the records are consecutive.

### Example: Finding Consecutive IDs

Assume an `Events` table with `event_id` and `user_id`. We want to find events where a user recorded two events sequentially.

```sql
WITH LaggedEvents AS (
    SELECT
        user_id,
        event_id,
        -- Get the event_id from the previous row for the current user
        LAG(event_id) OVER (
            PARTITION BY user_id
            ORDER BY event_id
        ) AS previous_event_id
    FROM
        Events
)
SELECT
    user_id,
    previous_event_id AS start_of_sequence,
    event_id AS end_of_sequence
FROM
    LaggedEvents
WHERE
    -- Condition for numerical consecutiveness
    event_id = previous_event_id + 1
ORDER BY
    user_id, start_of_sequence;
```

  * **`PARTITION BY user_id`**: Ensures the comparison only happens within the same user's events.
  * **`event_id = previous_event_id + 1`**: Confirms the current ID is immediately sequential to the last one.

-----

## 2\. Finding Runs of Identical Statuses (Value-Based)

This technique identifies sequences of three or more rows that share the same categorical value (e.g., three consecutive days where a machine was 'down'). This requires a multi-step approach using a combination of window functions and grouping.

### Technique (Gaps and Islands)

1.  **Identify Groups:** Create a running group counter (`group_rank`) that increments *only* when the key value changes. This is done by comparing the current row's value to the lagged value.
2.  **Filter/Count:** Count the number of rows within each identified group (`group_rank`).
3.  **Result:** Filter for groups that have a count greater than the minimum required run length (e.g., $\ge 3$).

### Example: Finding 3 or More Consecutive Status Matches

Assume a `DailyStatus` table with `status_date` and `system_status`. We want to find systems that had a status of 'down' for 3 or more consecutive days.

```sql
WITH StatusChanges AS (
    SELECT
        status_date,
        system_status,
        -- Check if the current status is different from the previous status
        CASE
            WHEN system_status = LAG(system_status) OVER (ORDER BY status_date) THEN 0
            ELSE 1
        END AS is_new_group
    FROM
        DailyStatus
),
GroupedStatus AS (
    SELECT
        status_date,
        system_status,
        -- Create a unique ID for each consecutive run (Island)
        SUM(is_new_group) OVER (ORDER BY status_date) AS status_group_id
    FROM
        StatusChanges
)
SELECT
    system_status,
    MIN(status_date) AS run_start,
    MAX(status_date) AS run_end,
    COUNT(*) AS run_length
FROM
    GroupedStatus
WHERE
    system_status = 'down' -- Only check for 'down' statuses
GROUP BY
    system_status, status_group_id
HAVING
    COUNT(*) >= 3 -- Filter for runs of 3 or more
ORDER BY
    run_start;
```

  * The `SUM(is_new_group)` creates the `status_group_id`. Every time `is_new_group` is 1 (a status change occurs), the cumulative sum increments, effectively assigning a unique ID to the current run of identical statuses.
  * The final `GROUP BY status_group_id` aggregates the run, and `HAVING COUNT(*) >= 3` ensures we only return long consecutive runs.

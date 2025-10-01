## Date Ranges and BETWEEN Cheat Sheet Table

| Problem Type | SQL Concept | Solution Pattern | Key Technique |
| :--- | :--- | :--- | :--- |
| **Simple Inclusive Range** | Filter data where a date falls exactly between two boundary dates (inclusive). | Use the **`BETWEEN`** operator with two date values. | `$date\_col BETWEEN 'start\_date' AND 'end\_date'$` |
| **Inclusive/Exclusive (Best Practice)** | Filter data up to the last millisecond of the end date, avoiding time zone/timestamp issues. | Use explicit comparison operators: `>=` for the start date and `<` for the day *after* the end date. | `$date\_col >= 'start\_date' AND date\_col < 'day\_after\_end'$` |
| **Range Overlap Check** | Identify if two date ranges intersect with each other. | Use a compound condition that checks if neither range starts entirely after the other ends. | `$R1.Start < R2.End AND R2.Start < R1.End$` |
| **Missing/Gaps Check** | Finding periods where data is absent or an expected sequence of dates is broken. | Use **`LAG`** (or **`LEAD`**) to compare the current date to the previous date. | `$IF (current\_date = LAG(current\_date) + 1, 'No Gap', 'Gap')$` |

-----

## Date Ranges and BETWEEN Mini Playbook (Realistic Queries)

These snippets illustrate the proper methods for filtering data using date ranges, emphasizing the distinction between `BETWEEN` and the safer `> / <` approach for timestamps.

### 1\. Simple Inclusive Range (BETWEEN)

**Use Case:** Retrieve all orders placed during the calendar month of January 2025.

*Note: This works reliably if `order_date` is a pure **DATE** type. If it's a **TIMESTAMP**, any order placed after `2025-01-31 00:00:00` will be excluded.*

```sql
SELECT
    order_id,
    order_date
FROM
    Orders
WHERE
    order_date BETWEEN '2025-01-01' AND '2025-01-31';
```

-----

### 2\. Inclusive/Exclusive (Best Practice for Timestamps)

**Use Case:** Retrieve all log entries that occurred during the calendar month of January 2025, robustly handling the `log_timestamp` field.

*This pattern ensures all timestamps from 2025-01-01 00:00:00 up to (but not including) 2025-02-01 00:00:00 are captured.*

```sql
SELECT
    log_id,
    log_timestamp
FROM
    System_Logs
WHERE
    -- Start date is inclusive
    log_timestamp >= '2025-01-01'
    -- End date is exclusive (the first moment of the next day)
    AND log_timestamp < '2025-02-01';
```

-----

### 3\. Range Overlap Check

**Use Case:** Find all active subscription plans that overlap with a specific holiday blackout period, defined by `blackout_start` and `blackout_end`.

```sql
SELECT
    plan_id,
    plan_start_date,
    plan_end_date
FROM
    Subscription_Plans
WHERE
    -- The plan must start before the blackout ends
    plan_start_date < '2025-12-24' -- (Blackout End)
    -- AND the blackout must start before the plan ends
    AND '2025-12-18' < plan_end_date; -- (Blackout Start)
```

-----

### 4\. Missing/Gaps Check

**Use Case:** Identify days when no data was logged, by comparing the current log date to the previous log date.

```sql
SELECT
    current_log_date,
    LAG(current_log_date) OVER (ORDER BY current_log_date) AS previous_log_date,
    CASE
        -- If the difference between current and previous date is greater than 1 day, a gap exists
        WHEN current_log_date - LAG(current_log_date) OVER (ORDER BY current_log_date) > 1 THEN 'Gap Found'
        ELSE 'Sequential'
    END AS sequence_status
FROM
    (SELECT DISTINCT DATE(log_timestamp) AS current_log_date FROM System_Logs) AS DailyLogs;
-- PostgreSQL date subtraction syntax is used here (current_date - previous_date yields an interval/integer).
```

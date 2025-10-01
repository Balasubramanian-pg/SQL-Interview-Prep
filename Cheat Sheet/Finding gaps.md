Finding gaps in a sequential series of data (like dates, IDs, or invoice numbers) is a common SQL task. The most robust and high-performing methods leverage **Window Functions** (specifically `LAG` or `LEAD`) or utilize **Recursive CTEs** when the sequence is known to be continuous.

Here are the primary techniques for detecting missing records in a sequence.

-----

## 1\. Using Window Functions (LAG/LEAD) 🎣

This is the most common and efficient method for non-continuous sequences (i.e., when you only have the existing numbers and need to find gaps between them). It works by comparing the current row's value to the previous row's value.

### Technique

1.  **Identify the Previous Value:** Use the **`LAG()`** window function to fetch the preceding value in the sequence, ordered by the sequence column.
2.  **Calculate the Difference:** Compare the current value with the lagged value. A difference greater than 1 indicates a gap.

### Example (Finding Missing IDs)

Assume an `Invoices` table with a sequential `invoice_id`.

```sql
WITH GapCheck AS (
    SELECT
        invoice_id,
        -- Get the previous invoice_id, ordered by the sequence
        LAG(invoice_id) OVER (ORDER BY invoice_id) AS previous_id
    FROM
        Invoices
)
SELECT
    previous_id + 1 AS gap_starts_at,
    invoice_id - 1 AS gap_ends_at
FROM
    GapCheck
WHERE
    invoice_id - previous_id > 1
ORDER BY
    gap_starts_at;
```

**Result Interpretation:** If `previous_id` is 105 and `invoice_id` is 108, the difference is 3. The query returns a gap from 106 to 107.

-----

## 2\. Using Date Arithmetic (Date Gaps) 🗓️

This is a specific application of the `LAG` technique for date sequences. The comparison must check the date difference, not just the numerical difference.

### Example (Finding Missing Days)

Assume a `DailyLogs` table with a `log_date` column.

```sql
WITH DateCheck AS (
    SELECT DISTINCT
        log_date,
        -- Get the previous day's log date
        LAG(log_date) OVER (ORDER BY log_date) AS previous_log_date
    FROM
        DailyLogs
)
SELECT
    previous_log_date + INTERVAL '1 day' AS gap_starts_at,
    log_date - INTERVAL '1 day' AS gap_ends_at
FROM
    DateCheck
WHERE
    -- Check if the current date is more than one day after the previous date
    log_date > previous_log_date + INTERVAL '1 day'
ORDER BY
    gap_starts_at;
-- PostgreSQL/MySQL syntax shown. Use DATEDIFF in SQL Server.
```

-----

## 3\. Using Left Join / EXCEPT with a Sequence Generator ⚙️

If you know the expected start and end of the full sequence, you can generate all possible numbers/dates and then check which ones are missing from your table. This is often the most explicit way to list all gaps.

### Technique

1.  **Generate a Full Sequence:** Create a series of all possible IDs or dates (using a table, a numbers table, or a Recursive CTE).
2.  **Find the Difference:** Use `LEFT JOIN` or `EXCEPT`/`MINUS` to find the generated numbers that do not exist in the actual data table.

### Example (EXCEPT for Missing IDs)

```sql
-- Step 1: Generate a sequence of all expected IDs (1 to 1000)
WITH ExpectedIDs AS (
    SELECT generate_series(1, 1000) AS expected_id -- PostgreSQL function shown
    -- Replace with a dedicated numbers table or Recursive CTE for other DBs
)
SELECT
    e.expected_id
FROM
    ExpectedIDs AS e

EXCEPT 

-- Step 2: Subtract the IDs that actually exist in the table
SELECT
    i.invoice_id
FROM
    Invoices AS i
ORDER BY
    expected_id;

-- Result: A list of every single missing invoice ID between 1 and 1000.
```

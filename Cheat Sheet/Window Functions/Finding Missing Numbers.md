Finding missing numbers in a sequential series of data (like primary keys, invoice IDs, or log entry IDs) is a classic SQL problem.

The most effective methods rely on either **Window Functions** for partial analysis or a **Sequence Generator** for finding all missing numbers between a known start and end point.

## 1\. Using Window Functions (`LAG`/`LEAD`)
This is the most efficient method when you only have the existing numbers and want to find gaps between them. It compares the current value to the previous value.

### Technique

1.  **Identify the Previous Value:** Use the **`LAG()`** window function to retrieve the value of the sequence column from the row immediately preceding the current row, ordered by the sequence.
2.  **Calculate the Difference:** Compare the current value with the lagged value. A difference **greater than 1** indicates a gap.

### Example: Finding Gaps in Invoice IDs

Assume an `Invoices` table with a sequential `invoice_id`.

```sql
WITH GapCheck AS (
    SELECT
        invoice_id,
        -- Get the ID of the previous invoice
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
    invoice_id - previous_id > 1 -- Check if the difference is more than 1
ORDER BY
    gap_starts_at;
```

**Result Interpretation:** If `previous_id` is 105 and `invoice_id` is 108, the difference is 3. The query identifies a gap that starts at 106 and ends at 107.

## 2\. Using a Sequence Generator (`EXCEPT` or `LEFT JOIN`) 

If you know the expected start and end points of the entire sequence (e.g., IDs should run from 1 to 1000), you can generate a full set of numbers and then subtract the ones that actually exist in your table.

### Technique

1.  **Generate a Full Sequence:** Use a dedicated numbers table, a **Recursive CTE**, or a database-specific function (like `generate_series` in PostgreSQL) to create every expected number.
2.  **Find the Difference:** Use the **`EXCEPT`** (or `MINUS`) set operator or a **`LEFT JOIN` with `WHERE IS NULL`** to identify the numbers in the generated set that are missing from your data table.

### Example: Finding All Missing IDs (1 to MAX)

This example uses a **Recursive CTE** (ANSI SQL standard) to generate the sequence, making the pattern portable.

```sql
WITH RECURSIVE NumberSeries AS (
    -- Anchor Member: Start the sequence at 1
    SELECT 1 AS n
    
    UNION ALL
    
    -- Recursive Member: Increment the number up to the maximum ID found in the table
    SELECT n + 1 
    FROM NumberSeries
    WHERE n < (SELECT MAX(invoice_id) FROM Invoices)
)
SELECT
    ns.n AS missing_invoice_id
FROM
    NumberSeries AS ns
LEFT JOIN
    Invoices AS i ON ns.n = i.invoice_id
WHERE
    i.invoice_id IS NULL -- Only keep the numbers from the series that had no match
ORDER BY
    missing_invoice_id;
```

  * **Performance Note:** Generating a very large series with a Recursive CTE can be resource-intensive. For sequences exceeding $10^5$ or $10^6$ numbers, a pre-existing "Numbers Table" is usually the most performant method.

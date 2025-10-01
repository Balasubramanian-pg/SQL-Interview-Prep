This cheat sheet covers the `LAG` and `LEAD` window functions for accessing data from preceding or succeeding rows in a result set.

## LAG/LEAD Cheat Sheet Table

| Problem Type | What They're Testing | Solution Pattern | Key Technique |
| :--- | :--- | :--- | :--- |
| **Row-over-Row Comparison** | Comparing a current row's value to the previous row's value. | Use `LAG` to retrieve the previous value for calculation. | `$LAG(column, 1, 0) OVER (PARTITION BY group_col ORDER BY sort_col)$` |
| **Calculating Differences/Deltas** | Finding the change between the current and the previous row. | Subtract the previous value (`LAG`) from the current value. | `$column - LAG(column) OVER (ORDER BY sort_col)$` |
| **Forecasting/Lookahead** | Accessing the next row's value from the current row. | Use `LEAD` to retrieve the subsequent value. | `$LEAD(column, 1, 0) OVER (PARTITION BY group_col ORDER BY sort_col)$` |
| **Checking Data Gaps/Skips** | Identifying if there's a break in a sequential series (e.g., dates, IDs). | Compare the current value to the next value (`LEAD`) or previous value (`LAG`). | `$IF (col = LAG(col) + 1, 'No Gap', 'Gap')$` or similar logic. |

-----

## LAG/LEAD Mini Playbook (SQL Snippets)

These snippets illustrate the common use cases for `LAG` and `LEAD` functions.

### 1\. Row-over-Row Comparison (LAG)

Retrieve the **previous day's closing price** for comparison.

```sql
SELECT
    stock_id,
    trade_date,
    closing_price,
    LAG(closing_price, 1) OVER (
        PARTITION BY stock_id
        ORDER BY trade_date
    ) AS previous_day_close
FROM
    daily_stock_data;
```

-----

### 2\. Calculating Differences/Deltas (LAG)

Calculate the **daily change (delta)** in the `amount` of a transaction within a user's history.

```sql
SELECT
    user_id,
    transaction_date,
    amount,
    amount - LAG(amount) OVER (
        PARTITION BY user_id
        ORDER BY transaction_date
    ) AS amount_delta_from_previous
FROM
    transactions;
```

-----

### 3\. Forecasting/Lookahead (LEAD)

Find the **next scheduled delivery date** for a specific order.

```sql
SELECT
    order_id,
    scheduled_date,
    status,
    LEAD(scheduled_date, 1) OVER (
        PARTITION BY order_id
        ORDER BY scheduled_date
    ) AS next_delivery_date
FROM
    deliveries;
```

-----

### 4\. Checking Data Gaps/Skips (LAG/LEAD)

Identify if a **sequential ID** in a log is missing (not sequential) by checking if the current ID is one greater than the previous ID. The default value of `0` ensures the very first row doesn't error and can be easily compared.

```sql
SELECT
    log_id,
    log_timestamp,
    CASE
        WHEN log_id = LAG(log_id, 1, 0) OVER (ORDER BY log_id) + 1 THEN 'Sequential'
        ELSE 'ID Gap Found'
    END AS sequence_check
FROM
    system_logs;
```

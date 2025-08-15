---
Company:
  - PayPal
Difficulty: Intermediate
Created:
Status:
Category:
Sub category:
Question Link:
---
### 1. The Question
The business question being asked by this SQL query is:

**"What is the final balance for each account, based on the sum of all its transactions? Deposits should be treated as positive amounts and all other transaction types as negative amounts."**

---

### 2. Table Schema

The query references a single table, `transactions`. A plausible schema for this table would be:

```sql
-- Table: transactions
-- Stores a record for every transaction associated with an account.
CREATE TABLE transactions (
    transaction_id   INT PRIMARY KEY,
    account_id       INT NOT NULL,            -- The ID of the account this transaction belongs to
    transaction_type VARCHAR(20) NOT NULL,    -- The type, e.g., 'Deposit', 'Withdrawal'
    amount           DECIMAL(12, 2) NOT NULL, -- The absolute monetary value of the transaction
    transaction_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- An index on the account_id is crucial for the GROUP BY performance.
CREATE INDEX idx_transactions_account ON transactions (account_id);
```

---

### 3. Structured SQL Query (Method 1: Conditional `SUM` with `CASE`)

This is your original query, which is the most efficient and standard way to solve this problem.

```sql
SELECT
    account_id,
    SUM(
        CASE
            WHEN transaction_type = 'Deposit' THEN amount
            ELSE -amount
        END
    ) AS final_balance
FROM
    transactions
GROUP BY
    account_id;
```

---

### 4. Explanation of the Query

This query calculates the net balance for each account by using conditional logic inside a `SUM` function.

1.  **`FROM transactions`**: The query specifies the `transactions` table as its data source.
2.  **`GROUP BY account_id`**: This is the most critical step. It groups all rows with the same `account_id` together. This ensures that the `SUM` function will calculate a separate total for each individual account.
3.  **`SELECT account_id, SUM(...) AS final_balance`**: This clause operates on each group created by the `GROUP BY`.
    *   `account_id`: It selects the ID for the account group.
    *   `SUM(...)`: It calculates the sum for that group.
4.  **The `CASE` Statement: `CASE WHEN ... END`**:
    *   This logic is applied to the `amount` of **every single row** before it is summed up.
    *   `WHEN transaction_type = 'Deposit' THEN amount`: If the transaction is a 'Deposit', it uses the `amount` as a positive value.
    *   `ELSE -amount`: For any other transaction type (e.g., 'Withdrawal', 'Fee'), it multiplies the `amount` by -1, making it a negative value.
    *   The `SUM` function then adds all these positive and negative values together for a given account, resulting in the correct final net balance.

---

### 5. Another SQL Method (Method 2: Using Separate CTEs and a `JOIN`)

An alternative, though much less performant and more verbose, way to solve this is to calculate deposits and withdrawals separately and then join them together.

```sql
WITH deposits AS (
    -- First, calculate total deposits for each account
    SELECT
        account_id,
        SUM(amount) AS total_deposits
    FROM
        transactions
    WHERE
        transaction_type = 'Deposit'
    GROUP BY
        account_id
),
withdrawals AS (
    -- Second, calculate total withdrawals for each account
    SELECT
        account_id,
        SUM(amount) AS total_withdrawals
    FROM
        transactions
    WHERE
        transaction_type != 'Deposit'
    GROUP BY
        account_id
)
-- Finally, join the two summaries and calculate the difference
SELECT
    d.account_id,
    COALESCE(d.total_deposits, 0) - COALESCE(w.total_withdrawals, 0) AS final_balance
FROM
    deposits d
FULL JOIN
    withdrawals w ON d.account_id = w.account_id;
```

---

### 6. Explanation of the Alternative Query

This method breaks the problem into three distinct steps using Common Table Expressions (CTEs).

1.  **`deposits` CTE**: This first query scans the `transactions` table to find only the 'Deposit' rows, groups them by `account_id`, and calculates the total deposit amount for each account.
2.  **`withdrawals` CTE**: This second query does the same for all non-deposit transactions, calculating the total withdrawal amount for each account.
3.  **Final `SELECT` with `FULL JOIN`**:
    *   This is the final step that combines the two summaries.
    *   **`FULL JOIN`**: A `FULL JOIN` is essential here. It ensures that accounts with *only* deposits or *only* withdrawals are included in the final result. An `INNER JOIN` would miss them.
    *   **`COALESCE(..., 0)`**: The `COALESCE` function is used to handle `NULL` values. If an account has no deposits, `d.total_deposits` will be `NULL`. `COALESCE` replaces this `NULL` with `0`, ensuring the subtraction works correctly.
    *   `... - ... AS final_balance`: The final balance is calculated by subtracting the total withdrawals from the total deposits.

**Comparison:** While the second method works, the first method (`SUM(CASE...END)`) is vastly superior. It is more concise, more readable, and significantly more performant as it only requires a single scan of the `transactions` table.
# Calculate a Transactional Running Total

## 1. Overview
This document explains how to solve a common SQL interview question that involves calculating a **running total** or cumulative sum, where the calculation must account for both additions (deposits) and subtractions (withdrawals). This problem tests your ability to use window functions, conditional logic with `CASE` statements, and handle ordering to ensure the calculation is performed correctly over a sequence of transactions.

## 2. Problem Statement

### 2.1. The Scenario
You are given a `transactions` table that logs financial transactions in chronological order. The `type` column indicates whether a transaction is a 'Deposit' (added to the balance) or a 'Withdrawal' (subtracted from the balance).

**transactions:**
| transaction_id | type       | amount | transaction_date |
|----------------|------------|--------|------------------|
| 1              | Deposit    | 178    | 2024-07-08       |
| 2              | Withdrawal | 25     | 2024-07-08       |
| 3              | Withdrawal | 45     | 2024-07-08       |
| 4              | Deposit    | 65     | 2024-07-10       |
| 5              | Deposit    | 32     | 2024-07-10       |

### 2.2. The Requirement
Write a SQL query to calculate the running total (or current balance) after each transaction.

-   After transaction 1 (Deposit 178): Running total is **178**.
-   After transaction 2 (Withdrawal 25): Running total is 178 - 25 = **153**.
-   After transaction 3 (Withdrawal 45): Running total is 153 - 45 = **108**.
-   After transaction 4 (Deposit 65): Running total is 108 + 65 = **173**.
-   After transaction 5 (Deposit 32): Running total is 173 + 32 = **205**.

**Expected Output:**
| transaction_id | type       | amount | transaction_date | running_total |
|----------------|------------|--------|------------------|---------------|
| 1              | Deposit    | 178    | 2024-07-08       | 178           |
| 2              | Withdrawal | 25     | 2024-07-08       | 153           |
| 3              | Withdrawal | 45     | 2024-07-08       | 108           |
| 4              | Deposit    | 65     | 2024-07-10       | 173           |
| 5              | Deposit    | 32     | 2024-07-10       | 205           |

## 3. Setup Script
You can use the following T-SQL script to create the table and populate it with sample data.

```sql
-- Create the table
CREATE TABLE transactions (
    transaction_id INT,
    type VARCHAR(20),
    amount INT,
    transaction_date DATE
);
GO

-- Insert sample data
INSERT INTO transactions (transaction_id, type, amount, transaction_date) VALUES
(1, 'Deposit', 178, '2024-07-08'),
(2, 'Withdrawal', 25, '2024-07-08'),
(3, 'Withdrawal', 45, '2024-07-08'),
(4, 'Deposit', 65, '2024-07-10'),
(5, 'Deposit', 32, '2024-07-10');
GO

-- Verify the initial data
SELECT * FROM transactions;
```

## 4. Solution and Explanation
The key to this problem is to use a `SUM()` window function combined with a `CASE` statement to handle the different transaction types.

### 4.1. The Logic

#### 4.1.1. Handling Deposits and Withdrawals
We need to treat withdrawals as negative numbers. A `CASE` statement inside the `SUM()` function can achieve this:
-   `CASE WHEN type = 'Deposit' THEN amount ELSE -amount END`: This expression returns the `amount` as a positive number for deposits and as a negative number for withdrawals.

#### 4.1.2. Calculating the Cumulative Sum
The `SUM()` window function can then be applied to this conditional value.
-   `OVER (ORDER BY transaction_date, transaction_id)`: The `ORDER BY` clause within the `OVER()` clause is critical. It tells the `SUM()` function to calculate the total in a specific sequence. We order first by `transaction_date` and then by `transaction_id` to ensure that transactions on the same day are processed in the correct order.

> [!IMPORTANT]
> The default frame for a window function with `ORDER BY` is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. This means for each row, it sums up all values from the beginning of the partition up to and including the current row, which is exactly what a running total requires.

### 4.2. The Complete SQL Query
This query directly applies the `SUM()` window function with the `CASE` statement to the `transactions` table.

```sql
SELECT
    transaction_id,
    type,
    amount,
    transaction_date,
    SUM(CASE
            WHEN type = 'Deposit' THEN amount
            ELSE -amount
        END) OVER (ORDER BY transaction_date, transaction_id) AS running_total
FROM
    transactions
ORDER BY
    transaction_date, transaction_id;
```

This solution is efficient and directly calculates the running total by dynamically converting withdrawal amounts to negative values before summing them up in the correct chronological order.

# First Transaction Spend Analysis

## 1. The Business Question
The objective of this query is to analyze the behavior of new users at the point of their very first transaction. The specific business question is:

**"How many unique users made their very first transaction with a value of $50 or more?"**

> [!IMPORTANT]
> This analysis is focused exclusively on the **first transaction**. Any subsequent purchases, regardless of their value, are not considered. The query must correctly identify and isolate only the initial transaction for every user.

## 2. Assumed Table Schema
The query references a single table, `user_transactions`. A plausible schema for this table would be:

```sql
-- Table: user_transactions
-- Stores a record for every transaction made by a user.
CREATE TABLE user_transactions (
    transaction_id   INT PRIMARY KEY,
    user_id          INT NOT NULL,           -- The ID of the user making the transaction
    spend            DECIMAL(10, 2) NOT NULL,-- The amount of money spent
    transaction_date TIMESTAMP NOT NULL      -- The exact date and time of the transaction
);
```

> [!TIP]
> For this query to perform well, a composite index on the columns used in the window function (`user_id`, `transaction_date`) is crucial. This allows the database to efficiently partition and order the data without a full table sort.
>
> `CREATE INDEX idx_user_trans_date ON user_transactions (user_id, transaction_date);`

## 3. Solution Approaches

### 3.1. Method 1: Using a Window Function in a CTE (Recommended)
This is a modern, efficient, and highly readable way to solve this common type of "first event" analysis.

> [!NOTE]
> This approach uses a Common Table Expression (CTE) to first rank all transactions for each user. This pre-processing step makes the final filtering logic simple and clear.

```sql
-- Step 1: Rank all transactions for each user chronologically.
WITH transaction_ranking AS (
    SELECT
        user_id,
        spend,
        DENSE_RANK() OVER (
          PARTITION BY user_id
          ORDER BY transaction_date ASC
        ) AS transaction_num
    FROM
        user_transactions
)
-- Step 2: From the ranked list, count users whose #1 transaction meets the criteria.
SELECT
    COUNT(DISTINCT user_id) AS users
FROM
    transaction_ranking
WHERE
    transaction_num = 1
    AND spend >= 50;
```

#### 3.1.1. Explanation

1.  **The `transaction_ranking` CTE**:
    -   This CTE is the core of the solution. It assigns a chronological rank to every transaction for every user.
    -   `DENSE_RANK() OVER (...)`: This is the window function that generates the rank.
        -   `PARTITION BY user_id`: This is a critical instruction. It tells the function to operate in separate groups for each `user_id`. The ranking will restart at `1` for every new user.
        -   `ORDER BY transaction_date ASC`: Within each user's partition, transactions are sorted by date, ensuring the earliest one comes first.
    -   `DENSE_RANK()` is used here. If a user has two transactions at the exact same earliest timestamp, both will be ranked as `1`. This is the correct behavior for finding all "first" transactions.

2.  **The Final `SELECT` Statement**:
    -   This query operates on the pre-ranked data from the CTE.
    -   `WHERE transaction_num = 1`: This filter isolates only the very first transaction(s) for each user.
    -   `AND spend >= 50`: This further filters that set, keeping only the first transactions where the value was $50 or more.
    -   `COUNT(DISTINCT user_id)`: Finally, it counts the number of unique `user_id`s that remain. `DISTINCT` is important in the rare case that a user had multiple "first" transactions (at the same time) that both met the spend criteria.

> [!CAUTION]
> While `RANK()` would also work, `DENSE_RANK()` is a safe choice. `ROW_NUMBER()` would be incorrect if a user had two transactions at the exact same time, as it would arbitrarily assign one as rank 1 and the other as rank 2, potentially causing you to miss a valid first transaction.

### 3.2. Method 2: Using a Subquery with `GROUP BY` and `JOIN`
A more traditional way to solve this, without window functions, is to first find the minimum transaction date for each user and then join that information back.

> [!TIP]
> This method is very explicit. It involves creating a temporary "derived table" that contains the first transaction date for every user, which is then used to filter the main table.

```sql
SELECT
    COUNT(DISTINCT t.user_id) AS users
FROM
    user_transactions AS t
INNER JOIN (
    -- Subquery to find the first transaction date for each user
    SELECT
        user_id,
        MIN(transaction_date) AS first_transaction_date
    FROM
        user_transactions
    GROUP BY
        user_id
) AS first_transactions
ON t.user_id = first_transactions.user_id
AND t.transaction_date = first_transactions.first_transaction_date
WHERE
    t.spend >= 50;
```

#### 3.2.1. Explanation

1.  **The Subquery (Derived Table)**:
    -   The inner `SELECT ... GROUP BY user_id` query is executed first. It finds the earliest (`MIN`) `transaction_date` for each unique `user_id`.
    -   This produces a temporary table named `first_transactions` containing each user and their specific first transaction timestamp.

2.  **The Outer Query**:
    -   The main `user_transactions` table (`t`) is joined to this temporary `first_transactions` table.
    -   **The `ON` Clause**: The join condition `ON t.user_id = ... AND t.transaction_date = ...` is crucial. It finds the full row(s) from the original table that match *both* the user and their specific first transaction date, effectively isolating the first transaction record(s).

> [!WARNING]
> A common mistake is to only join on `user_id`. You must join on both the user and the timestamp to uniquely identify the first transaction record.

3.  **Final Filtering and Counting**:
    -   `WHERE t.spend >= 50`: This clause filters these first-transaction records.
    -   `COUNT(DISTINCT t.user_id)`: Counts the unique users who are left.

## 4. Comparison of Methods
-   **Method 1 (Window Function)**: Generally more concise, modern, and readable once you are familiar with the syntax. It can be easily extended to find the 2nd, 3rd, or Nth transaction by simply changing `WHERE transaction_num = 1`.
-   **Method 2 (`GROUP BY`/`JOIN`)**: A very solid and fundamental SQL pattern that works on virtually all database systems. It can be more verbose but is sometimes seen as more explicit.

> [!IMPORTANT]
> The window function approach is often more performant as the database can rank and filter in a more optimized, single-pass operation compared to the aggregate-then-join pattern. It should be the preferred method in modern SQL environments.

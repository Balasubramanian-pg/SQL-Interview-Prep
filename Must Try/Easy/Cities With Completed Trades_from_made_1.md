---
Created:
Company:
  -
Difficulty:
Status:
Category:
Sub category:
Question Link:
---
### 1. The Question

The business question being asked by this SQL query is:

**"Which are the top 3 cities with the most completed trade orders? Provide the city name and the total count of completed orders for each."**

---

### 2. Table Schema

The query involves two tables, `users` and `trades`, linked by `user_id`. Here is a plausible schema for these tables:

```sql
-- Table: users
-- Stores information about each client or user.
CREATE TABLE users (
    user_id     INT PRIMARY KEY,          -- Unique identifier for the user
    full_name   VARCHAR(255) NOT NULL,    -- User's full name
    city        VARCHAR(100),             -- The city where the user is located
    signup_date DATE                      -- The date the user registered
);

-- Table: trades
-- Stores information about each trade order placed by a user.
CREATE TABLE trades (
    order_id    INT PRIMARY KEY,          -- Unique identifier for the trade order
    user_id     INT,                      -- Foreign key linking to the users table
    status      VARCHAR(50) NOT NULL,     -- Status of the trade, e.g., 'Completed', 'Pending', 'Cancelled'
    trade_value DECIMAL(12, 2),           -- The monetary value of the trade
    trade_date  TIMESTAMP,                -- The date and time the trade was placed
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

---

### 3. Structured SQL Query (Method 1: `JOIN` with `WHERE`)

This is your original query, formatted for better readability.

```sql
SELECT
    u.city,
    COUNT(t.order_id) AS total_orders
FROM
    trades AS t
INNER JOIN
    users AS u ON t.user_id = u.user_id
WHERE
    t.status = 'Completed'
GROUP BY
    u.city
ORDER BY
    total_orders DESC
LIMIT 3;
```

---

### 4. Explanation of the Query

This query follows a logical sequence to find the top-performing cities based on trade volume. The database executes these steps in a specific order (which is slightly different from how we write it).

1.  **`FROM trades AS t INNER JOIN users AS u ON t.user_id = u.user_id`**
    *   First, the query combines the `trades` table (aliased as `t`) and the `users` table (aliased as `u`).
    *   The `INNER JOIN` creates a new, temporary virtual table where each trade record is matched with the information of the user who made it, using `user_id` as the link.

2.  **`WHERE t.status = 'Completed'`**
    *   Next, it filters this combined set of data. It discards all rows where the trade `status` is anything other than 'Completed'. This ensures that only completed trades are considered for the count.

3.  **`GROUP BY u.city`**
    *   The remaining rows are then grouped together based on the user's city. All completed trades from 'New York' go into one group, all from 'London' into another, and so on.

4.  **`SELECT u.city, COUNT(t.order_id) AS total_orders`**
    *   This clause is now executed on each group.
    *   `u.city`: It selects the name of the city for the group.
    *   `COUNT(t.order_id)`: It counts the number of `order_id`s within that city's group, giving the total completed orders. This count is aliased as `total_orders`.

5.  **`ORDER BY total_orders DESC`**
    *   The resulting list of cities and their counts is sorted in descending order (`DESC`), so the city with the highest `total_orders` appears first.

6.  **`LIMIT 3`**
    *   Finally, the query discards all but the first 3 rows of the sorted result, giving you only the top 3 cities.

---

### 5. Another SQL Method (Method 2: Using a Common Table Expression - CTE)

For more complex queries, using a Common Table Expression (CTE) can make the logic cleaner and easier to follow by breaking it into logical steps.

```sql
WITH CompletedTrades AS (
    -- Step 1: Get a list of all completed trades and their corresponding city
    SELECT
        u.city
    FROM
        trades AS t
    INNER JOIN
        users AS u ON t.user_id = u.user_id
    WHERE
        t.status = 'Completed'
)
-- Step 2: Aggregate the results from the CTE
SELECT
    city,
    COUNT(*) AS total_orders
FROM
    CompletedTrades
GROUP BY
    city
ORDER BY
    total_orders DESC
LIMIT 3;
```

---

### 6. Explanation of the Alternative Query

This method achieves the exact same result but separates the data preparation from the final aggregation.

1.  **`WITH CompletedTrades AS (...)`**
    *   This defines a CTE named `CompletedTrades`. A CTE is a temporary, named result set that exists only for the duration of the query.
    *   The query inside the parentheses is executed first. It joins the `trades` and `users` tables and filters for `status = 'Completed'`, just like the original method.
    *   The result is a temporary "table" (`CompletedTrades`) that contains a single column, `city`, with one row for every single completed trade. For example, if New York had 500 completed trades, there would be 500 rows with the value 'New York' in this CTE.

2.  **The Final `SELECT` Statement**
    *   This outer query now runs against the `CompletedTrades` CTE as if it were a real table.
    *   `FROM CompletedTrades`: It reads from our temporary result set.
    *   `GROUP BY city`: It groups the rows by city.
    *   `SELECT city, COUNT(*) AS total_orders`: It selects the city name and counts all rows (`COUNT(*)`) for that city group. This gives the total number of completed trades.
    *   `ORDER BY total_orders DESC LIMIT 3`: Finally, it sorts the aggregated results and keeps only the top 3.

This CTE approach is highly valued for its readability, especially when queries become much longer and involve multiple steps of filtering and aggregation.
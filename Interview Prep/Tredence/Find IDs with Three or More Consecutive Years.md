# Find IDs with Three or More Consecutive Years

## 1. Overview
This document explains how to solve a common SQL interview question that involves identifying "gaps and islands" in a time-series dataset. The specific task is to find all IDs that have data for three or more consecutive years. This problem tests your ability to use window functions to create logical groups from sequential data.

> [!IMPORTANT]
> The core challenge is to group consecutive years together. A simple `GROUP BY` on the `id` and `year` won't work. The solution lies in a clever trick using `ROW_NUMBER()` to create a consistent value for each "island" of consecutive years.

## 2. Problem Statement

### 2.1. The Scenario
You are given an `events` table that logs events for different IDs over several years.

**events:**
| id  | year |
|-----|------|
| 1   | 2019 |
| 1   | 2020 |
| 1   | 2021 |
| 2   | 2021 |
| 2   | 2022 |
| 3   | 2019 |
| 3   | 2021 |
| 3   | 2022 |

### 2.2. The Requirement
Write a SQL query to find all `id`s that have event data for at least three consecutive years.

-   **ID 1**: Has data for 2019, 2020, 2021. These are three consecutive years, so ID 1 should be in the result.
-   **ID 2**: Has data for 2021, 2022. This is only two consecutive years, so it should be excluded.
-   **ID 3**: Has data for 2019, 2021, 2022. There is a gap in 2020, so there is no sequence of three consecutive years. It should be excluded.

**Expected Output:** `1`

## 3. Setup Script
You can use the following T-SQL script to create the table and populate it with the sample data.

```sql
-- Create the table
CREATE TABLE events (
    id INT,
    year INT
);
GO

-- Insert sample data
INSERT INTO events (id, year) VALUES
(1, 2019),
(1, 2020),
(1, 2021),
(2, 2021),
(2, 2022),
(3, 2019),
(3, 2021),
(3, 2022);
GO

-- Verify the initial data
SELECT * FROM events;
```

## 4. Solution and Explanation
The solution uses a window function to identify consecutive sequences and then aggregation to count the length of each sequence.

### 4.1. Step 1: Create a Grouping Key
This is the key step. We create a value that remains constant for each island of consecutive years.

-   **Logic**:
    1.  First, we generate a row number for each `id`, ordered by `year`. For a consecutive sequence, this `ROW_NUMBER()` will increment by 1 each time (1, 2, 3, ...).
    2.  The `year` column is also incrementing by 1 for each consecutive year.
    3.  Therefore, the difference `year - ROW_NUMBER()` will be a constant value for every row within a single consecutive sequence. This constant value acts as our "grouping key".

-   **Example for ID 1**:
| id  | year | rn | year - rn (group_key) |
|-----|------|----|-----------------------|
| 1   | 2019 | 1  | 2018                  |
| 1   | 2020 | 2  | 2018                  |
| 1   | 2021 | 3  | 2018                  |

-   **Example for ID 3**:
| id  | year | rn | year - rn (group_key) |
|-----|------|----|-----------------------|
| 3   | 2019 | 1  | 2018                  |  *(Island 1)*
| 3   | 2021 | 2  | 2019                  |  *(Island 2)*
| 3   | 2022 | 3  | 2019                  |  *(Island 2)*

### 4.2. Step 2: Count the Length of Each Group
Now that we have a `group_key`, we can use a standard `GROUP BY` to count how many years are in each consecutive sequence.

-   **Logic**: We `GROUP BY id, group_key` and use `COUNT(*)` to find the length of each island.

### 4.3. Step 3: Filter for Groups of 3 or More
The final step is to filter these groups, keeping only those with a count of 3 or more.
-   **Logic**: A `HAVING COUNT(*) >= 3` clause is used to filter the grouped results.

## 5. The Complete SQL Query (Using a CTE)
This query combines all the steps into a single, readable block using a Common Table Expression (CTE).

```sql
WITH ConsecutiveGroups AS (
    -- Step 1: Generate the grouping key to identify islands of consecutive years
    SELECT
        id,
        year,
        (year - ROW_NUMBER() OVER (PARTITION BY id ORDER BY year)) AS group_key
    FROM
        events
)
-- Step 2 & 3: Group by ID and the new key, then filter for counts of 3 or more
SELECT
    id
FROM
    ConsecutiveGroups
GROUP BY
    id, group_key
HAVING
    COUNT(*) >= 3;
```

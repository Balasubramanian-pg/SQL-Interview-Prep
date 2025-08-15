---
Company:
  - NY Times
Difficulty: Easy
Created:
Status:
Category:
Sub category:
Question Link:
---
### 1. The Question

The business question being asked by this SQL query is:

**"What is the total count of views from laptops versus the total count of views from mobile devices (defined as tablets and phones)?"**

This query produces a single row with two columns, allowing for a direct side-by-side comparison of the two device categories.

---

### 2. Table Schema

The query references a single table, `viewership`. A plausible schema for this table would be:

```sql
-- Table: viewership
-- Stores a log of every time a piece of content is viewed.
CREATE TABLE viewership (
    view_id     INT PRIMARY KEY AUTO_INCREMENT, -- A unique ID for the view event
    user_id     INT,                            -- The ID of the user who viewed the content
    content_id  INT,                            -- The ID of the content that was viewed
    view_time   TIMESTAMP DEFAULT CURRENT_TIMESTAMP, -- The time of the view
    device_type VARCHAR(50) NOT NULL              -- The type of device used, e.g., 'laptop', 'tablet', 'phone'
);
```

---

### 3. Structured SQL Query (Method 1: Conditional `SUM` with `CASE`)

This is your original query, formatted for clarity. This is a very common and portable method that works across almost all SQL databases.

```sql
SELECT
    SUM(
        CASE
            WHEN device_type = 'laptop' THEN 1
            ELSE 0
        END
    ) AS laptop_views,
    SUM(
        CASE
            WHEN device_type IN ('tablet', 'phone') THEN 1
            ELSE 0
        END
    ) AS mobile_views
FROM
    viewership;
```

---

### 4. Explanation of the Query

This query uses **conditional aggregation** to count different categories of data within the same query and present them as separate columns. Since there is no `GROUP BY` clause, the aggregation is performed over the entire table.

1.  **`FROM viewership`**
    *   This specifies that we are pulling data from the `viewership` table.

2.  **`SUM(CASE WHEN device_type = 'laptop' THEN 1 ELSE 0 END) AS laptop_views`**
    *   This expression calculates the total number of laptop views.
    *   The `CASE` statement is evaluated for **every single row** in the table.
    *   If a row's `device_type` is 'laptop', the statement returns the number `1`.
    *   For all other device types, it returns `0`.
    *   The `SUM()` function then adds up all these 1s and 0s. The final result is a count of only the rows that matched the 'laptop' condition.
    *   This total is given the alias `laptop_views`.

3.  **`SUM(CASE WHEN device_type IN ('tablet', 'phone') THEN 1 ELSE 0 END) AS mobile_views`**
    *   This expression works in the same way to calculate the total mobile views.
    *   The `CASE` statement checks if the `device_type` is one of the values in the list `('tablet', 'phone')`.
    *   If it matches, it returns `1`; otherwise, it returns `0`.
    *   `SUM()` adds up these values to get the total count for mobile devices, which is aliased as `mobile_views`.

The query returns a single row because we are aggregating across the entire table without grouping.

---

### 5. Another SQL Method (Method 2: Using `COUNT` with `FILTER`)

A more modern and often more declarative alternative, supported by databases like PostgreSQL and SQLite, is to use the `FILTER` clause with an aggregate function.

```sql
SELECT
    COUNT(*) FILTER (WHERE device_type = 'laptop') AS laptop_views,
    COUNT(*) FILTER (WHERE device_type IN ('tablet', 'phone')) AS mobile_views
FROM
    viewership;
```

---

### 6. Explanation of the Alternative Query

This method achieves the exact same result but with a syntax that can be more intuitive and readable.

1.  **`FROM viewership`**
    *   The data source remains the `viewership` table.

2.  **`COUNT(*) FILTER (WHERE device_type = 'laptop') AS laptop_views`**
    *   This is a conditional count.
    *   `COUNT(*)` is instructed to count rows, but the `FILTER (WHERE ...)` clause tells it to *only* consider the rows that meet the specified condition.
    *   In this case, it counts only the rows where `device_type` is 'laptop'. The result is aliased as `laptop_views`.

3.  **`COUNT(*) FILTER (WHERE device_type IN ('tablet', 'phone')) AS mobile_views`**
    *   Similarly, this expression counts only the rows where the `device_type` is either 'tablet' or 'phone'.

**Comparison:**
*   The `SUM(CASE...END)` method is the most universally compatible way to perform conditional aggregation.
*   The `COUNT(FILTER...)` method is often preferred for its clarity and readability when counting, as it directly states the intention: "Count the rows, but only where this condition is true." It separates the aggregation (`COUNT`) from the condition (`FILTER`) more cleanly than the `CASE` statement does.
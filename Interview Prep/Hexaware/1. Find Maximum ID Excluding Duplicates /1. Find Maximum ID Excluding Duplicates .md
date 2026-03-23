# Find Maximum ID Excluding Duplicates

## 1. Overview
This document breaks down a common SQL interview question that requires you to find the maximum value in a column while ignoring any values that appear more than once. This problem tests your understanding of aggregation with `GROUP BY`, filtering with `HAVING`, and the use of subqueries to solve multi-step problems.

## 2. Problem Statement

### 2.1. The Scenario
You are given a table named `employees` with an `ID` column containing the following values:

**employees:**
| ID |
|----|
| 2  |
| 5  |
| 6  |
| 6  |
| 7  |
| 8  |
| 8  |

### 2.2. The Requirement
Write a SQL query to find the maximum `ID` from the table, but only consider IDs that are unique (i.e., appear exactly once).

-   The absolute maximum `ID` is **8**. However, it is a duplicate value, so it must be ignored.
-   The `ID` **6** is also a duplicate and must be ignored.
-   The remaining, non-duplicate IDs are `2`, `5`, and `7`.
-   The maximum value among these non-duplicates is **7**.

**Expected Output:** `7`

## 3. Setup Script
You can use the following T-SQL script to create the `employees` table and populate it with the sample data.

```sql
-- Create the table
CREATE TABLE employees (
    ID INT
);
GO

-- Insert sample data
INSERT INTO employees (ID) VALUES
(2),
(5),
(6),
(6),
(7),
(8),
(8);
GO

-- Verify the data
SELECT * FROM employees;
```

## 4. Solution and Explanation
The solution involves a two-step process: first, identify all the IDs that are duplicates, and second, query the table again to find the maximum ID from the set that excludes those duplicates.

### 4.1. Step 1: Identifying the Duplicate IDs
To find the IDs that appear more than once, we can use `GROUP BY` on the `ID` column and then filter the groups with `HAVING` where the count is greater than one.

-   **Query to find duplicates:**
    ```sql
    SELECT ID
    FROM employees
    GROUP BY ID
    HAVING COUNT(ID) > 1;
    ```
-   **Explanation:** This query groups all identical IDs together, counts how many there are in each group, and returns only those IDs whose count is greater than 1. For our data, this will return `6` and `8`.

### 4.2. Step 2: Filtering and Finding the Maximum
Now that we have a way to identify the duplicates, we can use this logic in a subquery to filter them out from our main query. The outer query will then calculate the `MAX()` value from the remaining (unique) IDs.

-   **The Complete Query:**
    ```sql
    SELECT MAX(ID) AS MaxUniqueID
    FROM employees
    WHERE ID NOT IN (
        -- This subquery returns all duplicate IDs (6, 8)
        SELECT ID
        FROM employees
        GROUP BY ID
        HAVING COUNT(ID) > 1
    );
    ```

-   **Explanation:**
    1.  The inner query (the subquery) executes first and returns a list of duplicate IDs: `(6, 8)`.
    2.  The outer query then selects from the `employees` table, applying the condition `WHERE ID NOT IN (6, 8)`. This effectively filters the table down to only the rows with unique IDs: `2`, `5`, and `7`.
    3.  Finally, the `MAX(ID)` aggregate function is applied to this filtered result set `(2, 5, 7)`, returning `7` as the final answer.

## 5. Summary
By using a subquery to first isolate the non-unique values, we can easily exclude them from our main query and then perform the final aggregation to get the desired result.

| Query Type            | Method                                     | Result |
|-----------------------|--------------------------------------------|--------|
| Find Max Unique ID    | `MAX()` with a `NOT IN` subquery           | 7      |

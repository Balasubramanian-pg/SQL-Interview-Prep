# Update Scores to Department Maximum

## 1. Overview
This document explains how to solve a common SQL interview question that involves updating records in a table based on an aggregate value calculated over a specific group or partition. The solution requires using a correlated subquery within an `UPDATE` statement to set the score for every employee to the maximum score within their respective department.

## 2. Problem Statement

### 2.1. The Scenario
You are given a table named `employee_department` which contains employee IDs, their department, and individual scores.

**employee_department (Before Update):**
| employee_id | department | scores |
|-------------|------------|--------|
| 1           | Dept 1     | 1.00   |
| 2           | Dept 1     | 5.28   |
| 3           | Dept 1     | 4.00   |
| 4           | Dept 1     | 2.50   |
| 5           | Dept 2     | 8.00   |
| 6           | Dept 2     | 7.00   |
| 7           | Dept 3     | 10.20  |
| 8           | Dept 4     | 9.50   |

### 2.2. The Requirement
Write a single SQL query to **update** the `scores` column for all rows. The new score for each row should be the maximum score found within that row's department.

For example, for `Dept 1`, the maximum score is `5.28`. All four records for `Dept 1` should have their `scores` column updated to `5.28`. Similarly, for `Dept 2`, the maximum score is `8.00`, so both records should be updated to `8.00`.

**Expected Output (After Update):**
| employee_id | department | scores |
|-------------|------------|--------|
| 1           | Dept 1     | 5.28   |
| 2           | Dept 1     | 5.28   |
| 3           | Dept 1     | 5.28   |
| 4           | Dept 1     | 5.28   |
| 5           | Dept 2     | 8.00   |
| 6           | Dept 2     | 8.00   |
| 7           | Dept 3     | 10.20  |
| 8           | Dept 4     | 9.50   |

## 3. Setup Script
You can use the following T-SQL script to create the table and populate it with the sample data.

```sql
-- Create the table
CREATE TABLE employee_department (
    employee_id INT,
    department VARCHAR(50),
    scores FLOAT
);
GO

-- Insert sample data
INSERT INTO employee_department (employee_id, department, scores) VALUES
(1, 'Dept 1', 1.00),
(2, 'Dept 1', 5.28),
(3, 'Dept 1', 4.00),
(4, 'Dept 1', 2.50),
(5, 'Dept 2', 8.00),
(6, 'Dept 2', 7.00),
(7, 'Dept 3', 10.20),
(8, 'Dept 4', 9.50);
GO

-- Verify the initial data
SELECT * FROM employee_department;
```

## 4. Solution and Explanation
The most direct way to solve this is to use an `UPDATE` statement that incorporates a **correlated subquery**.

### 4.1. The Correlated Subquery
A correlated subquery is an inner query that depends on the outer query for its values. In this case, for each row that the outer `UPDATE` statement processes, the inner subquery will execute and calculate the maximum score specifically for that row's department.

> [!IMPORTANT]
> The key to a correlated subquery is the `WHERE` clause that links the inner query to the outer query. Here, `T2.department = ED.department` ensures the `MAX(scores)` is calculated per department.

-   **The Subquery Logic:**
    ```sql
    (SELECT MAX(T2.scores)
     FROM employee_department AS T2
     WHERE T2.department = ED.department)
    ```
    For each row `ED` being updated, this subquery finds the maximum score from all rows `T2` that share the same department as `ED`.

### 4.2. The Complete Update Query
We embed this subquery directly into the `SET` clause of our `UPDATE` statement.

```sql
UPDATE ED
SET
    scores = (
        SELECT MAX(T2.scores)
        FROM employee_department AS T2
        WHERE T2.department = ED.department
    )
FROM
    employee_department AS ED;
```

-   **How it Works:**
    1.  The `UPDATE` statement iterates through each row of the `employee_department` table (aliased as `ED`).
    2.  For each row, it executes the correlated subquery.
    3.  The subquery finds the maximum score for the department of the current row.
    4.  This maximum score is returned and used as the new value for the `scores` column in the current row.
    5.  This process repeats for all eight rows in the table.

## 5. Verification
After running the `UPDATE` query, you can verify the result by selecting all data from the table.

```sql
-- Run this query after the UPDATE to see the final result
SELECT * FROM employee_department;
```
The output will match the expected result, with all scores updated to their department's maximum value.

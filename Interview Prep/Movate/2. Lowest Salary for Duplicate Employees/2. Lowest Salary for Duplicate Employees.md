# Select Lowest Salary for Duplicate Employees

## 1. Overview
This document explains how to solve a common SQL interview question where you need to combine data from two tables and, in cases of duplicate records, select only the one that meets a specific criterion—in this case, having the lowest salary. This problem tests your ability to use `UNION`, aggregate functions like `MIN()`, and window functions like `ROW_NUMBER()`.

## 2. Problem Statement

### 2.1. The Scenario
You are given two tables, `TableA` and `TableB`, both containing employee data with the same structure: `EmployeeID`, `EmployeeName`, and `Salary`.

**TableA:**
| EmployeeID | EmployeeName | Salary |
|------------|--------------|--------|
| 1          | Emp1         | 1000   |
| 2          | Emp2         | 400    |

**TableB:**
| EmployeeID | EmployeeName | Salary |
|------------|--------------|--------|
| 2          | Emp2         | 300    |
| 3          | Emp3         | 100    |

### 2.2. The Requirement
Write a SQL query that produces a consolidated list of employees from both tables. If an `EmployeeID` appears in both tables (or multiple times in general), only the record with the **lowest salary** for that employee should be included in the final output.

**Expected Output:**
| EmployeeID | EmployeeName | Salary |
|------------|--------------|--------|
| 1          | Emp1         | 1000   |
| 2          | Emp2         | 300    |
| 3          | Emp3         | 100    |

## 3. Setup Script
You can use the following T-SQL script to create the tables and populate them with the sample data to test the solutions.

```sql
-- Create TableA
CREATE TABLE TableA (
    EmployeeID INT,
    EmployeeName VARCHAR(50),
    Salary INT
);
INSERT INTO TableA (EmployeeID, EmployeeName, Salary) VALUES
(1, 'Emp1', 1000),
(2, 'Emp2', 400);

-- Create TableB
CREATE TABLE TableB (
    EmployeeID INT,
    EmployeeName VARCHAR(50),
    Salary INT
);
INSERT INTO TableB (EmployeeID, EmployeeName, Salary) VALUES
(2, 'Emp2', 300),
(3, 'Emp3', 100);

-- Verify the data
SELECT * FROM TableA;
SELECT * FROM TableB;
```

## 4. Solutions and Explanations
There are multiple ways to solve this problem. We will cover two common and effective approaches.

### 4.1. Solution 1: Using `GROUP BY` and `MIN()`
This approach first combines all records from both tables and then uses aggregation to find the minimum salary for each employee.

-   **Explanation:**
    1.  **Combine Data**: Use `UNION ALL` to create a single, consolidated result set from `TableA` and `TableB`.
    2.  **Group Records**: Group the combined records by `EmployeeID` and `EmployeeName`.
    3.  **Find Minimum Salary**: Use the `MIN(Salary)` aggregate function to calculate the lowest salary within each group.

> [!NOTE]
> We use `UNION ALL` instead of `UNION` because `UNION` automatically removes duplicate rows, which might interfere with our logic. `UNION ALL` includes all rows from both tables, which is safer for this approach.

-   **SQL Query:**
    ```sql
    SELECT
        EmployeeID,
        EmployeeName,
        MIN(Salary) AS Salary
    FROM (
        SELECT EmployeeID, EmployeeName, Salary FROM TableA
        UNION ALL
        SELECT EmployeeID, EmployeeName, Salary FROM TableB
    ) AS CombinedEmployees
    GROUP BY
        EmployeeID,
        EmployeeName
    ORDER BY
        EmployeeID;
    ```

### 4.2. Solution 2: Using a Window Function (`ROW_NUMBER`)
This is a more flexible and often preferred method for "top-N-per-group" problems. It allows you to select the entire row associated with the lowest salary, not just the minimum value.

-   **Explanation:**
    1.  **Combine Data**: Just as before, use `UNION ALL` to combine the tables.
    2.  **Assign Ranks**: Use the `ROW_NUMBER()` window function. We `PARTITION BY EmployeeID` to create separate rankings for each employee. We `ORDER BY Salary ASC` to assign the rank of `1` to the record with the lowest salary for each employee.
    3.  **Filter**: Wrap the query in a Common Table Expression (CTE) or a derived table, and then select only the rows where the assigned row number is `1`.

> [!TIP]
> This method is very powerful. If the requirement was to find the highest salary, you would simply change the `ORDER BY` clause to `Salary DESC`.

-   **SQL Query:**
    ```sql
    WITH CombinedEmployees AS (
        -- Step 1: Combine the tables
        SELECT EmployeeID, EmployeeName, Salary FROM TableA
        UNION ALL
        SELECT EmployeeID, EmployeeName, Salary FROM TableB
    ),
    RankedEmployees AS (
        -- Step 2: Assign a rank based on the lowest salary for each employee
        SELECT
            EmployeeID,
            EmployeeName,
            Salary,
            ROW_NUMBER() OVER(PARTITION BY EmployeeID ORDER BY Salary ASC) AS rn
        FROM CombinedEmployees
    )
    -- Step 3: Filter for the top-ranked row (the one with the lowest salary)
    SELECT
        EmployeeID,
        EmployeeName,
        Salary
    FROM RankedEmployees
    WHERE rn = 1
    ORDER BY
        EmployeeID;
    ```

## 5. Summary of Approaches

| Method                | Pros                                                                              | Cons                                                                   |
|-----------------------|-----------------------------------------------------------------------------------|------------------------------------------------------------------------|
| **`GROUP BY`**        | Simple and direct for this specific problem.                                      | Less flexible; can become complex if you need other columns from the "winning" row. |
| **`ROW_NUMBER()`**    | Highly flexible and scalable. Easily selects the entire desired row. | Slightly more complex syntax than a simple `GROUP BY`.                    |

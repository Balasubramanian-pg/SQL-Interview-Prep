# Find Employees with the Same Salary

## 1. Overview
This document explains how to solve a classic SQL interview question that requires finding all employees who share a salary with at least one other employee. This problem is an excellent test of the **self-join** technique, where a table is joined to itself to compare rows within it.

## 2. Problem Statement

### 2.1. The Scenario
You are given an `employees` table with employee names and their salaries.

**employees:**
| employee_id | employee_name | salary |
|-------------|---------------|--------|
| 1           | Alice         | 50000  |
| 2           | Bob           | 60000  |
| 3           | Charlie       | 50000  |
| 4           | Diana         | 75000  |
| 5           | Edward        | 80000  |
| 6           | Fiona         | 70000  |
| 7           | Grace         | 75000  |
| 8           | Hank          | 60000  |

### 2.2. The Requirement
Write a SQL query to return the names of all employees who have the same salary as other employees.

-   Bob and Hank share a salary of 60000.
-   Diana and Grace share a salary of 75000.
-   Alice and Charlie share a salary of 50000.

**Expected Output:**
| employee_name |
|---------------|
| Alice         |
| Bob           |
| Charlie       |
| Diana         |
| Grace         |
| Hank          |

## 3. Setup Script
You can use the following T-SQL script to create the table and populate it with sample data.

```sql
-- Create the table
CREATE TABLE employees (
    employee_id INT,
    employee_name VARCHAR(50),
    salary INT
);
GO

-- Insert sample data
INSERT INTO employees (employee_id, employee_name, salary) VALUES
(1, 'Alice', 50000),
(2, 'Bob', 60000),
(3, 'Charlie', 50000),
(4, 'Diana', 75000),
(5, 'Edward', 80000),
(6, 'Fiona', 70000),
(7, 'Grace', 75000),
(8, 'Hank', 60000);
GO
```

## 4. Solution and Explanation
The key to this problem is to perform a `self-join` on the `employees` table. This allows us to compare every employee's record with every other employee's record.

### 4.1. The SQL Query
```sql
SELECT DISTINCT
    T1.employee_name
FROM
    employees AS T1
JOIN
    employees AS T2 ON T1.salary = T2.salary
WHERE
    T1.employee_id <> T2.employee_id
ORDER BY
    T1.employee_name;
```

### 4.2. How it Works
1.  **Table Aliases**: We reference the `employees` table twice, giving it two different aliases: `T1` and `T2`. This treats it as two separate tables for the purpose of the join.
2.  **Join Condition**: `ON T1.salary = T2.salary` connects rows from `T1` and `T2` where the salaries are identical.
3.  **Filtering Condition**: `WHERE T1.employee_id <> T2.employee_id` is the crucial step. It removes rows where an employee is matched with themselves. This ensures we only find matches between *different* employees.
4.  **`SELECT DISTINCT`**: We use `DISTINCT` because if three employees share the same salary, a simple join would return each employee multiple times. `DISTINCT` ensures each employee appears only once in the final result.

> [!TIP]
> It is best practice to use a unique identifier like `employee_id` for the inequality check rather than `employee_name`, as two different employees could potentially have the same name.

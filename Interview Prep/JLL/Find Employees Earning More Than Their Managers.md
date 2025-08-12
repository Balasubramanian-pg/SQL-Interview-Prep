# Find Employees Earning More Than Their Managers

## 1. Overview
This document explains a classic SQL interview question that tests your understanding of **self-joins**. The task is to query a single table containing both employee and manager information to find employees who earn a higher salary than their direct managers. This requires joining a table to itself to create a relationship between employee rows and their corresponding manager rows.

## 2. Problem Statement

### 2.1. The Scenario
You are given a single `employees` table that contains employee details, including their salary and the ID of their manager. The `manager_id` column refers to the `employee_id` of another employee in the same table.

**employees:**
| employee_id | employee_name | salary | manager_id |
|-------------|---------------|--------|------------|
| 1           | John          | 50000  | NULL       |
| 2           | Alice         | 40000  | 1          |
| 3           | Bob           | 70000  | 1          |
| 4           | Emily         | 55000  | NULL       |
| 5           | Charlie       | 65000  | 4          |
| 6           | David         | 50000  | 4          |

### 2.2. The Requirement
Write a SQL query to find the names of all employees whose salary is greater than the salary of their immediate manager.

-   **Bob's** salary (70,000) is greater than his manager **John's** salary (50,000).
-   **Charlie's** salary (65,000) is greater than his manager **Emily's** salary (55,000).

**Expected Output:** `Bob`, `Charlie`

## 3. Setup Script
You can use the following T-SQL script to create the table and populate it with the sample data.

```sql
-- Create the table
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    employee_name VARCHAR(50),
    salary INT,
    manager_id INT
);
GO

-- Insert sample data
INSERT INTO employees (employee_id, employee_name, salary, manager_id) VALUES
(1, 'John', 50000, NULL),
(2, 'Alice', 40000, 1),
(3, 'Bob', 70000, 1),
(4, 'Emily', 55000, NULL),
(5, 'Charlie', 65000, 4),
(6, 'David', 50000, 4);
GO

-- Verify the initial data
SELECT * FROM employees;
```

## 4. Solution and Explanation

### 4.1. The Self-Join Concept
Since the relationship between an employee and their manager exists within the same table, we must join the table to itself. This technique is called a **self-join**. We treat the table as two separate entities: one representing the 'Employee' and the other representing the 'Manager'.

### 4.2. The SQL Query
The solution uses a `JOIN` (specifically a `LEFT JOIN` or `INNER JOIN`) to link employee rows to their corresponding manager rows and then filters the results.

```sql
SELECT
    e.employee_name
FROM
    employees AS e
JOIN
    employees AS m ON e.manager_id = m.employee_id
WHERE
    e.salary > m.salary;
```

### 4.3. Explanation

1.  **Table Aliases**: We query the `employees` table twice, giving it two different aliases:
    -   `e` for the employee's record.
    -   `m` for the manager's record.

2.  **The Join Condition**: `ON e.manager_id = m.employee_id` is the core of the solution. It connects an employee row (`e`) to their manager's row (`m`) by matching the employee's `manager_id` with the manager's `employee_id`.

3.  **The Filter**: `WHERE e.salary > m.salary` is applied after the join. It compares the salary from the employee alias (`e.salary`) with the salary from the manager alias (`m.salary`) and keeps only the rows where the employee's salary is higher.

> [!NOTE]
> An `INNER JOIN` is used here because we only care about employees who have a manager to compare against. A `LEFT JOIN` would also work but would include employees without managers (where manager details would be `NULL`), who would then be filtered out by the `WHERE` clause anyway.

## 5. Final Result and Formatted Output
To get a more detailed output showing both the employee and their manager for context, you can select columns from both aliases.

-   **Query for Detailed Output:**
    ```sql
    SELECT
        e.employee_name AS EmployeeName,
        e.salary AS EmployeeSalary,
        m.employee_name AS ManagerName,
        m.salary AS ManagerSalary
    FROM
        employees AS e
    JOIN
        employees AS m ON e.manager_id = m.employee_id
    WHERE
        e.salary > m.salary;
    ```
-   **Final Output:**
| EmployeeName | EmployeeSalary | ManagerName | ManagerSalary |
|--------------|----------------|-------------|---------------|
| Bob          | 70000          | John        | 50000         |
| Charlie      | 65000          | Emily       | 55000         |

# Calculate and Store Grades

## 1. Overview
This document explains a common SQL interview question that tests your ability to use conditional logic to categorize data and modify a table's structure to store the results. The solution involves using the `CASE` statement for logic and Data Definition Language (DDL) commands like `ALTER TABLE` to permanently add a new column.

## 2. Problem Statement

### 2.1. The Scenario
You are given a `students` table containing student names and their corresponding marks.

**students:**
| student_name | marks |
|--------------|-------|
| G            | 75    |
| H            | 30    |
| C            | 55    |
| A            | 60    |
| B            | 91    |
| ...          | ...   |

### 2.2. The Requirement
The task has two parts:
1.  Write a query to **display** a `grade` for each student based on their `marks` according to the following rules:
    -   Marks >= 80: **'Excellent'**
    -   Marks >= 60 and < 80: **'Very Good'**
    -   Marks >= 35 and < 60: **'Good'**
    -   Marks < 35: **'Poor'**
2.  Modify the `students` table to **permanently add** this `grade` column and populate it with the correct values for all existing records.

## 3. Setup Script
You can use the following T-SQL script to create and populate the `students` table to test the solutions.

```sql
-- Create the table
CREATE TABLE students (
    student_name VARCHAR(50),
    marks INT
);
GO

-- Insert sample data
INSERT INTO students (student_name, marks) VALUES
('G', 75),
('H', 30),
('C', 55),
('A', 60),
('B', 91),
('D', 49),
('E', 36),
('F', 81),
('I', 19);
GO

-- Verify the initial data
SELECT * FROM students;
```

## 4. Solutions and Explanations

### 4.1. Solution Part 1: Displaying Grades with a CASE Statement
First, to simply display the grades without changing the table, we use a `SELECT` query with a `CASE` statement. The `CASE` statement goes through conditions sequentially and returns a value when the first condition is met.

-   **SQL Query:**
    ```sql
    SELECT
        student_name,
        marks,
        CASE
            WHEN marks >= 80 THEN 'Excellent'
            WHEN marks >= 60 THEN 'Very Good'
            WHEN marks >= 35 THEN 'Good'
            ELSE 'Poor'
        END AS grade
    FROM students;
    ```
-   **Explanation:**
    -   `WHEN marks >= 80`: Checks for the highest category first.
    -   `WHEN marks >= 60`: This is only evaluated if the first condition (`>= 80`) is false, so it effectively checks for marks between 60 and 79.
    -   `WHEN marks >= 35`: This is only evaluated if the prior conditions are false, checking for marks between 35 and 59.
    -   `ELSE 'Poor'`: This is the default case, catching any marks that are less than 35.

### 4.2. Solution Part 2: Permanently Adding and Populating the Grade Column
To permanently add the `grade` column and its values to the table, a two-step process using DDL and DML is required in standard SQL.

> [!IMPORTANT]
> Modifying a table's structure and data is a two-step operation:
> 1.  **`ALTER TABLE`**: Add the new column (initially it will be `NULL` for all rows).
> 2.  **`UPDATE`**: Populate the new column with values based on your logic.

#### 4.2.1. Step 1: Add the New Column
Use the `ALTER TABLE` command to add the `grade` column to the `students` table. We define it as `VARCHAR` to hold the text-based grades.

-   **SQL Query:**
    ```sql
    ALTER TABLE students
    ADD grade VARCHAR(20);
    ```

#### 4.2.2. Step 2: Populate the New Column
Now that the column exists, use an `UPDATE` statement with the same `CASE` logic to fill in the grade for each student.

-   **SQL Query:**
    ```sql
    UPDATE students
    SET grade =
        CASE
            WHEN marks >= 80 THEN 'Excellent'
            WHEN marks >= 60 THEN 'Very Good'
            WHEN marks >= 35 THEN 'Good'
            ELSE 'Poor'
        END;
    ```

## 5. Final Result
After executing both the `ALTER` and `UPDATE` commands, you can query the table to see the final, modified structure with the populated `grade` column.

-   **Verification Query:**
    ```sql
    SELECT * FROM students;
    ```
-   **Expected Output Table:**
| student_name | marks | grade       |
|--------------|-------|-------------|
| G            | 75    | Very Good   |
| H            | 30    | Poor        |
| C            | 55    | Good        |
| A            | 60    | Very Good   |
| B            | 91    | Excellent   |
| D            | 49    | Good        |
| E            | 36    | Good        |
| F            | 81    | Excellent   |
| I            | 19    | Poor        |

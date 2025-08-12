# Forward Fill NULL Values in a Column

## 1. Overview
This document explains a sophisticated SQL interview question that requires you to perform a "forward fill" or "last observation carried forward" (LOCF). The goal is to replace `NULL` values in a column with the most recent preceding non-`NULL` value. This problem tests advanced knowledge of SQL window functions, particularly for creating sequential groups and retrieving values within those groups.

> [!IMPORTANT]
> The core challenge is that there's no built-in SQL function for a direct forward fill. The solution requires a multi-step, creative use of window functions to first create logical groups and then propagate the correct value within each group.

## 2. Problem Statement

### 2.1. The Scenario
You are given a table named `brand_details` with `category` and `brand_name` columns. The `category` column contains `NULL` values that need to be filled.

**brand_details:**
| category  | brand_name |
|-----------|------------|
| Chocolate | Dairy Milk |
| NULL      | 5 Star     |
| NULL      | Kit Kat    |
| NULL      | Perk       |
| Biscuit   | Oreo       |
| NULL      | Good day   |
| NULL      | Marie      |

### 2.2. The Requirement
Write a SQL query to update the `category` column so that each `NULL` value is replaced by the last non-`NULL` category that appeared before it in the sequence.

**Expected Output:**
| category  | brand_name |
|-----------|------------|
| Chocolate | Dairy Milk |
| Chocolate | 5 Star     |
| Chocolate | Kit Kat    |
| Chocolate | Perk       |
| Biscuit   | Oreo       |
| Biscuit   | Good day   |
| Biscuit   | Marie      |

## 3. Setup Script
You can use the following T-SQL script to create the table and populate it with the sample data.

```sql
-- Create the table
CREATE TABLE brand_details (
    category VARCHAR(50),
    brand_name VARCHAR(50)
);
GO

-- Insert sample data
INSERT INTO brand_details (category, brand_name) VALUES
('Chocolate', 'Dairy Milk'),
(NULL, '5 Star'),
(NULL, 'Kit Kat'),
(NULL, 'Perk'),
('Biscuit', 'Oreo'),
(NULL, 'Good day'),
(NULL, 'Marie');
GO

-- Verify the initial data
SELECT * FROM brand_details;
```

## 4. Solution and Explanation
The solution is a three-step process that uses a chain of window functions.

### 4.1. Step 1: Establish a Consistent Order with `ROW_NUMBER`
Since the table lacks a dedicated ordering column (like an ID or timestamp), we first need to generate one to ensure our operations are performed in a consistent, predictable sequence.

-   **Logic**: The `ROW_NUMBER()` function can generate a unique sequence number for each row. `ORDER BY (SELECT NULL)` is a T-SQL technique to generate this number based on the table's current physical order, without depending on any specific column.
-   **Query**:
    ```sql
    SELECT
        *,
        ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) as rn
    FROM brand_details;
    ```

### 4.2. Step 2: Create Data Groups with a Windowed `COUNT`
The key to the solution is to assign a "group ID" to each non-`NULL` value and all the `NULL`s that follow it. We can do this with a running count on the `category` column.

-   **Logic**: The `COUNT(category)` window function increments its count only when it encounters a non-`NULL` value. For all subsequent `NULL` rows, the count remains the same, effectively creating a group.
-   **Query**:
    ```sql
    SELECT
        *,
        COUNT(category) OVER (ORDER BY rn) as category_group
    FROM (
        SELECT *, ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) as rn
        FROM brand_details
    ) AS with_rownum;
    ```
-   **Result**: This will produce groups. The "Chocolate" block will have `category_group = 1`, and the "Biscuit" block will have `category_group = 2`.

### 4.3. Step 3: Apply the Forward Fill with `FIRST_VALUE`
Now that we have logical groups (`category_group`), we can use the `FIRST_VALUE` window function to find the first non-`NULL` value within each group and apply it to all rows in that group.

-   **Logic**: We `PARTITION BY` our newly created `category_group`. Within each partition, `FIRST_VALUE(category)` will find the first value (which is our desired non-`NULL` category) and propagate it across all rows of that partition.
-   **Query**: This is the final step, combining all logic.

## 5. The Complete Solution (Using CTEs)
Using Common Table Expressions (CTEs) makes the multi-step logic clean and readable.

```sql
WITH AddRowNumber AS (
    -- Step 1: Assign a consistent row number to preserve order
    SELECT
        category,
        brand_name,
        ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS rn
    FROM brand_details
),
AddGroup AS (
    -- Step 2: Create a grouping ID using a running count
    SELECT
        category,
        brand_name,
        rn,
        COUNT(category) OVER (ORDER BY rn) AS category_group
    FROM AddRowNumber
)
-- Step 3: Use FIRST_VALUE within each group to fill the NULLs
SELECT
    FIRST_VALUE(category) OVER (PARTITION BY category_group ORDER BY rn) AS category,
    brand_name
FROM AddGroup;
```

# SQL Query: Delete Every Nth Row from a Table  

## 1. **Problem Statement**  
Given a table (`your_table`) with:  
- `id` (unique identifier)  
- Other columns  

We need to delete every **Nth** row (e.g., every 5th row) from the table.  

> [!WARNING]  
> **Caution**: Deleting data is irreversible. Always back up your data before executing `DELETE` operations.  

---

## 2. **Solution 1: Using `ROW_NUMBER()` and CTEs**  
```sql
-- Delete every 5th row (n=5)
WITH numbered AS (
    SELECT
        *,
        ROW_NUMBER() OVER (ORDER BY id) AS row_num
    FROM your_table
)
DELETE FROM your_table
WHERE id IN (SELECT id FROM numbered WHERE row_num % 5 = 0);
```  

---

## 3. **Explanation**  

### 3.1 **CTE: `numbered`**  
- **Purpose**: Assigns a sequential number (`row_num`) to each row based on the `id` column.  
- **Window Function**: `ROW_NUMBER() OVER (ORDER BY id)`.  

> [!TIP]  
> The `ORDER BY` clause ensures consistent numbering based on the `id` column.  

### 3.2 **DELETE Operation**  
- **Filter**: Uses `row_num % 5 = 0` to identify every 5th row.  
- **Subquery**: Selects the `id` of rows to delete from the `numbered` CTE.  

> [!IMPORTANT]  
> The `%` operator checks if the row number is divisible by `N` (5 in this case).  

---

## 4. **Solution 2: For Databases Without CTEs (MySQL Example)**  
```sql
-- MySQL example
DELETE FROM your_table
WHERE id IN (
    SELECT id FROM (
        SELECT
            id,
            @row := @row + 1 AS row_num
        FROM your_table, (SELECT @row := 0) r
        ORDER BY id
    ) t WHERE t.row_num % 5 = 0
);
```  

> [!NOTE]  
> This solution uses a user-defined variable (`@row`) to simulate row numbering in MySQL.  

---

## 5. **Alternative Approach: Recreate the Table**  
```sql
-- Create new table without nth rows
CREATE TABLE new_table AS
SELECT * FROM (
    SELECT
        *,
        ROW_NUMBER() OVER (ORDER BY id) AS row_num
    FROM original_table
) t WHERE row_num % 5 != 0;

-- Then drop original and rename new one
DROP TABLE original_table;
ALTER TABLE new_table RENAME TO original_table;
```  

> [!TIP]  
> This approach is safer for large tables as it avoids direct deletion and preserves data integrity.  

---

## 6. **Key Concepts**  

### 6.1 **ROW_NUMBER() Window Function**  
- **Purpose**: Assigns a unique sequential number to each row within a partition.  
- **Use Case**: Essential for row-based operations like deleting every Nth row.  

> [!IMPORTANT]  
> The `ORDER BY` clause determines the sequence of numbering.  

### 6.2 **Modulo Operator (`%`)**  
- **Purpose**: Checks if a number is divisible by another number.  
- **Use Case**: Identifies every Nth row for deletion.  

> [!NOTE]  
> `row_num % N = 0` selects rows where the row number is a multiple of `N`.  

### 6.3 **Subqueries in DELETE Statements**  
- **Purpose**: Allows complex filtering logic in `DELETE` operations.  
- **Example**: Identifying rows to delete based on calculated row numbers.  

> [!WARNING]  
> Always test subqueries with a `SELECT` statement before using them in `DELETE`.  

### 6.4 **Table Recreation**  
- **Benefit**: Avoids direct deletion, reducing the risk of data loss.  
- **Steps**: Create a new table, drop the old one, and rename the new table.  

> [!TIP]  
> Use this method when dealing with critical tables or large datasets.  

---

## 7. **Additional Notes**  
- **Backup**: Always back up your data before performing `DELETE` operations.  
- **Testing**: Test the query with a `SELECT` statement first to verify the rows to be deleted.  
- **Performance**: For large tables, consider the impact of `DELETE` operations on database performance.  

> [!WARNING]  
> Deleting rows directly can lock the table and affect concurrent operations. Use table recreation for large tables.  

---

This solution demonstrates how to delete every Nth row from a table using SQL. It covers methods for databases with and without support for CTEs, as well as an alternative approach to recreate the table safely.  

> [!TIP]  
> Practice these techniques with sample data to reinforce your understanding of row-based operations and deletion in SQL.  

---

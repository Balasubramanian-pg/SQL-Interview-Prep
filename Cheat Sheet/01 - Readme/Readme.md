## SQL Interview Cheat Sheet Topics - Brainstorm

### **High-Priority (Most Common in Interviews)**

1. **Window Functions** (you already have ranking functions)
   - LAG/LEAD (accessing previous/next rows)
   - Running totals & moving averages
   - FIRST_VALUE/LAST_VALUE
   - NTILE (dividing into buckets)

2. **JOIN Patterns**
   - When to use INNER vs LEFT vs RIGHT vs FULL OUTER
   - CROSS JOIN use cases
   - Self-joins (comparing rows within same table)
   - Multiple join conditions
   - Join vs WHERE clause filtering

3. **Subquery Patterns**
   - Correlated vs Non-correlated
   - IN vs EXISTS vs JOIN
   - Subquery in SELECT vs FROM vs WHERE
   - Common Table Expressions (CTEs) vs Subqueries

4. **GROUP BY Traps & Tricks**
   - HAVING vs WHERE
   - Multiple aggregations on same column
   - GROUP BY with CASE statements
   - Grouping sets, ROLLUP, CUBE

5. **Date/Time Manipulation**
   - Date differences (DATEDIFF, date arithmetic)
   - Date parts (YEAR, MONTH, QUARTER, DATEPART)
   - Date ranges and between
   - First/last day of month/year
   - Working with timestamps

6. **String Functions**
   - CONCAT vs || vs +
   - SUBSTRING/LEFT/RIGHT
   - LIKE patterns and wildcards
   - String splitting (STRING_SPLIT, SUBSTRING_INDEX)
   - TRIM, LTRIM, RTRIM
   - Pattern matching (REGEXP, LIKE)

### **Medium-Priority (Still Very Common)**

7. **NULL Handling**
   - COALESCE vs ISNULL vs IFNULL
   - NULL in JOINs
   - NULL in aggregations
   - NULL in comparisons (IS NULL vs = NULL)

8. **Set Operations**
   - UNION vs UNION ALL
   - INTERSECT
   - EXCEPT/MINUS
   - When to use each

9. **Aggregate Function Patterns**
   - COUNT(*) vs COUNT(column) vs COUNT(DISTINCT)
   - SUM with CASE (conditional aggregation)
   - String aggregation (GROUP_CONCAT, STRING_AGG)
   - MIN/MAX with multiple columns

10. **Common Table Expressions (CTEs)**
    - Recursive CTEs
    - Multiple CTEs in one query
    - CTE vs temp table vs subquery

11. **CASE Statement Patterns**
    - Simple CASE vs Searched CASE
    - CASE in SELECT, WHERE, ORDER BY
    - Pivot-style transformations with CASE
    - NULL handling in CASE

### **Specialized (Shows Advanced Knowledge)**

12. **Performance Patterns**
    - When indexes help
    - Avoiding SELECT *
    - EXISTS vs IN for large datasets
    - Limiting result sets

13. **Data Type Conversions**
    - CAST vs CONVERT
    - Implicit vs explicit conversion
    - Common casting patterns

14. **Complex Filtering Patterns**
    - Finding gaps in sequences
    - Finding consecutive records
    - Date range overlaps
    - Finding records that don't exist

15. **Pivot/Unpivot Operations**
    - Manual pivot with CASE
    - PIVOT syntax (SQL Server)
    - Unpivot patterns

### **Interview-Specific Question Types**

16. **Classic Interview Problems**
    - Finding duplicates
    - Removing duplicates
    - Finding missing numbers in sequence
    - Employee-Manager hierarchies
    - Running calculations
    - First/Last record in group
    - Consecutive events

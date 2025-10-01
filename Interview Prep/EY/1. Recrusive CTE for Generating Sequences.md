# SQL Recursive CTE Solutions  

## 1. **Generating Number Sequences**  

### 1.1 **Basic Sequence (1-10)**  
```sql
WITH RECURSIVE numbers AS (
    -- Anchor query (base case)
    SELECT 1 AS num

    UNION ALL

    -- Recursive query
    SELECT num + 1
    FROM numbers
    WHERE num < 10  -- Termination condition
)
SELECT * FROM numbers;
```  

> [!TIP]  
> The **anchor query** initializes the sequence with the starting value (1).  

> [!IMPORTANT]  
> The **termination condition** (`num < 10`) prevents infinite recursion.  

> [!NOTE]  
> `UNION ALL` combines the anchor and recursive results without deduplication.  

---

### 1.2 **Even Numbers (2-10)**  
```sql
WITH RECURSIVE even_numbers AS (
    SELECT 2 AS num  -- First even number

    UNION ALL

    SELECT num + 2
    FROM even_numbers
    WHERE num < 10
)
SELECT * FROM even_numbers;
```  

> [!TIP]  
> Start with the first even number (2) in the anchor query.  

> [!IMPORTANT]  
> Increment by 2 in the recursive part to maintain even numbers.  

> [!NOTE]  
> The termination condition (`num < 10`) ensures the sequence stops at 10.  

---

### 1.3 **Odd Numbers (1-9)**  
```sql
WITH RECURSIVE odd_numbers AS (
    SELECT 1 AS num  -- First odd number

    UNION ALL

    SELECT num + 2
    FROM odd_numbers
    WHERE num < 9
)
SELECT * FROM odd_numbers;
```  

> [!TIP]  
> Start with the first odd number (1) in the anchor query.  

> [!IMPORTANT]  
> Increment by 2 in the recursive part to maintain odd numbers.  

> [!NOTE]  
> The termination condition (`num < 9`) ensures the sequence stops at 9.  

---

## 2. **Generating Date Series**  

### 2.1 **Daily Dates (Dec 15, 2024 - Jan 10, 2025)**  
```sql
WITH RECURSIVE date_series AS (
    SELECT CAST('2024-12-15' AS DATE) AS date_value

    UNION ALL

    SELECT date_value + INTERVAL '1 day'
    FROM date_series
    WHERE date_value < CAST('2025-01-10' AS DATE)
)
SELECT date_value FROM date_series
ORDER BY date_value;
```  

> [!TIP]  
> Use `CAST` to convert string literals to `DATE` type for accurate calculations.  

> [!IMPORTANT]  
> Add `INTERVAL '1 day'` in the recursive part to generate consecutive dates.  

> [!NOTE]  
> The termination condition (`date_value < '2025-01-10'`) ensures the sequence stops at the target date.  

---

### 2.2 **Alternate Days**  
```sql
WITH RECURSIVE date_series AS (
    SELECT CAST('2024-12-15' AS DATE) AS date_value

    UNION ALL

    SELECT date_value + INTERVAL '2 days'
    FROM date_series
    WHERE date_value < CAST('2025-01-08' AS DATE)
)
SELECT date_value FROM date_series;
```  

> [!TIP]  
> Start with the first date in the anchor query.  

> [!IMPORTANT]  
> Increment by `INTERVAL '2 days'` in the recursive part to generate alternate dates.  

> [!NOTE]  
> Adjust the termination date (`'2025-01-08'`) to ensure the sequence ends correctly.  

---

## 3. **Calculating Factorials**  

### 3.1 **Factorial of 5**  
```sql
WITH RECURSIVE factorial_cte AS (
    -- Base case: factorial of 1 is 1
    SELECT 1 AS n, 1 AS factorial

    UNION ALL

    -- Recursive part
    SELECT
        n + 1,
        factorial * (n + 1)
    FROM factorial_cte
    WHERE n < 5  -- Stop at factorial of 5
)
SELECT n, factorial FROM factorial_cte;
```  

> [!TIP]  
> The anchor query initializes with `n = 1` and `factorial = 1`.  

> [!IMPORTANT]  
> Multiply the previous factorial by `n + 1` in the recursive part to compute the next factorial.  

> [!NOTE]  
> The termination condition (`n < 5`) ensures the calculation stops at the factorial of 5.  

---

## 4. **Key Concepts Explained**  

1. **Recursive CTE Structure**:  
    - **`WITH RECURSIVE`**: Defines the CTE.  
    - **Anchor query**: Provides the starting point.  
    - **Recursive part**: Builds on previous results.  
    - **Termination condition**: Prevents infinite loops.  

2. **Number Sequences**:  
    - Start with the first number in the anchor.  
    - Increment by 1 (or 2 for even/odd) in the recursive part.  
    - Stop when reaching the upper limit.  

3. **Date Series**:  
    - Cast string literals to `DATE` type.  
    - Add intervals (1 day, 2 days, etc.).  
    - Order results chronologically.  

4. **Factorial Calculation**:  
    - Track both the number (`n`) and its factorial.  
    - Multiply the previous factorial by the next number.  
    - Builds up results incrementally (1!, 2!, 3!, etc.).  

> [!IMPORTANT]  
> Recursive CTEs are powerful for generating sequences, hierarchical data traversal, and solving mathematical problems in SQL.  

> [!TIP]  
> Always include a termination condition to avoid infinite recursion.  

> [!NOTE]  
> Test recursive queries with small inputs first to verify correctness before running on large datasets.  

--- 

This document covers the fundamentals of recursive CTEs in SQL, demonstrating their use in generating sequences, date series, and calculating factorials.  

> [!TIP]  
> Practice these patterns with different parameters to reinforce your understanding of recursive logic in SQL.  

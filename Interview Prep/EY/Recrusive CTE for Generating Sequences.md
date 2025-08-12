---
Difficulty: Easy
---
# SQL Recursive CTE Solutions

## 1. Generating Number Sequences

### Basic Sequence (1-10)

```SQL
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

### Even Numbers (2-10)

```SQL
WITH RECURSIVE even_numbers AS (
    SELECT 2 AS num  -- First even number

    UNION ALL

    SELECT num + 2
    FROM even_numbers
    WHERE num < 10
)
SELECT * FROM even_numbers;
```

### Odd Numbers (1-9)

```SQL
WITH RECURSIVE odd_numbers AS (
    SELECT 1 AS num  -- First odd number

    UNION ALL

    SELECT num + 2
    FROM odd_numbers
    WHERE num < 9
)
SELECT * FROM odd_numbers;
```

## 2. Generating Date Series

### Daily Dates (Dec 15, 2024 - Jan 10, 2025)

```SQL
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

### Alternate Days

```SQL
WITH RECURSIVE date_series AS (
    SELECT CAST('2024-12-15' AS DATE) AS date_value

    UNION ALL

    SELECT date_value + INTERVAL '2 days'
    FROM date_series
    WHERE date_value < CAST('2025-01-08' AS DATE)
)
SELECT date_value FROM date_series;
```

## 3. Calculating Factorials

### Factorial of 5

```SQL
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

## Key Concepts Explained

1. **Recursive CTE Structure**:
    - `WITH RECURSIVE` defines the CTE
    - Anchor query provides the starting point
    - Recursive part builds on previous results
    - Termination condition prevents infinite loops
2. **Number Sequences**:
    - Start with first number in anchor
    - Increment by 1 (or 2 for even/odd) in recursive part
    - Stop when reaching the upper limit
3. **Date Series**:
    - Cast string literals to DATE type
    - Add intervals (1 day, 2 days, etc.)
    - Order results chronologically
4. **Factorial Calculation**:
    - Track both the number (n) and its factorial
    - Multiply previous factorial by next number
    - Builds up results incrementally (1!, 2!, 3!, etc.)

These recursive techniques are powerful for generating sequences, hierarchical data traversal, and solving mathematical problems in SQL.

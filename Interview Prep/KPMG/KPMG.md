---
Difficulty: Easy
---
# SQL Interview Question: Top Two Subjects by Student

## Problem Statement

Given a students table with:

- `student_name`
- `subject`
- `marks`

We need to:

1. For each student, identify their top two highest-scoring subjects
2. Calculate the total marks from these top two subjects
3. Return results with student names and their total marks

## Solution 1: Using Subquery with ROW_NUMBER()

```SQL
SELECT
    student_name,
    SUM(marks) AS total_marks
FROM (
    SELECT
        student_name,
        marks,
        ROW_NUMBER() OVER (
            PARTITION BY student_name
            ORDER BY marks DESC
        ) AS row_num
    FROM students
) ranked_students
WHERE row_num <= 2
GROUP BY student_name;
```

## Solution 2: Using Common Table Expression (CTE)

```SQL
WITH ranked_students AS (
    SELECT
        student_name,
        marks,
        ROW_NUMBER() OVER (
            PARTITION BY student_name
            ORDER BY marks DESC
        ) AS row_num
    FROM students
)
SELECT
    student_name,
    SUM(marks) AS total_marks
FROM ranked_students
WHERE row_num <= 2
GROUP BY student_name;
```

## Key Concepts Explained

1. **ROW_NUMBER() Window Function**:
    - Assigns sequential numbers to rows within each partition
    - `PARTITION BY student_name` creates groups for each student
    - `ORDER BY marks DESC` sorts marks in descending order within each group
2. **Filtering Top Two Rows**:
    - `WHERE row_num <= 2` keeps only the top two subjects per student
    - For students with fewer than two subjects (like Daniel), it returns all available
3. **Aggregation**:
    - `SUM(marks)` calculates the total of the top two marks
    - `GROUP BY student_name` ensures we get one row per student
4. **Approach Comparison**:
    - Both solutions produce identical results
    - CTE approach is often more readable for complex queries
    - Subquery approach may be preferred for simpler cases

This solution demonstrates common SQL patterns for:

- Ranking data within groups
- Filtering based on rankings
- Aggregating filtered results
- Handling edge cases (students with fewer than two subjects)
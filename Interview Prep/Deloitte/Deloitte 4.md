---
Difficulty: Easy
---
# SQL Interview Question: Employee Performance Classification

## Problem Statement

Given an employees table with:

- `employee_id`
- `name`
- `performance_rating` (values: "excellent", "good", "average", etc.)
- `salary`

We need to classify employees into three performance levels:

1. **Top Performer**: "excellent" rating AND salary > 7000
2. **Good Performer**: "good" rating AND salary > 5000
3. **Others**: All remaining cases

## Solution Code

```SQL
SELECT
    employee_id,
    name,
    performance_rating,
    salary,
    CASE
        WHEN performance_rating = 'excellent' AND salary > 7000
            THEN 'Top Performer'
        WHEN performance_rating = 'good' AND salary > 5000
            THEN 'Good Performer'
        ELSE 'Others'
    END AS performance_level
FROM employees;
```

## Explanation

1. **CASE Statement Structure**:
    - Evaluates conditions in order
    - Returns the first matching result
    - `ELSE` catches all remaining cases
2. **Classification Logic**:
    - Top performers must meet BOTH conditions (rating AND salary)
    - Good performers must meet BOTH their conditions
    - All others fall into the "Others" category
3. **Result Column**:
    - Creates a new column `performance_level`
    - Contains the classification string

## Key Concepts Demonstrated

1. **Conditional Logic in SQL**:
    - Using CASE WHEN for row-by-row evaluation
    - Handling multiple conditions with AND
2. **Data Categorization**:
    - Creating meaningful business categories
    - Combining multiple criteria for classification
3. **Result Presentation**:
    - Adding a derived column to the output
    - Maintaining all original columns plus the new classification

This solution efficiently categorizes employees based on multiple criteria, a common requirement for HR analytics and performance reporting systems. The CASE statement is a fundamental SQL tool for such conditional transformations.
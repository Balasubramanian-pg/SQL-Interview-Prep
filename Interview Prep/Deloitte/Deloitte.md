---
Difficulty: Easy
---
# SQL Interview Question: Time Difference Between User Events

## Problem Summary

You're given a user activity table with:

- `user_identifier`
- `event_type` (login, logout, purchase)
- `event_date`

You need to:

1. For each user, find the time difference (in days) between their most recent event and their second-to-last event
2. Sort the results by `user_identifier` in ascending order

## Solution

```SQL
WITH event_rank AS (
  SELECT
    user_identifier,
    event_date,
    LAG(event_date) OVER (PARTITION BY user_identifier ORDER BY event_date) AS previous_event_date
  FROM user_activity
)
SELECT
  user_identifier,
  DATEDIFF(day, previous_event_date, event_date) AS days_elapsed
FROM event_rank
WHERE previous_event_date IS NOT NULL
ORDER BY user_identifier;
```

## Code Explanation

1. **CTE (Common Table Expression) named** `**event_rank**`:
    - Uses the `LAG()` window function to get the previous event date for each user
    - `PARTITION BY user_identifier` groups events by each user
    - `ORDER BY event_date` sorts events chronologically for each user
2. **Main Query**:
    - Selects `user_identifier` and calculates the day difference using `DATEDIFF`
    - `WHERE previous_event_date IS NOT NULL` filters out the first event for each user (which has no previous event)
    - `ORDER BY user_identifier` sorts results as required

## Key Concepts Used

1. **Window Functions (**`**LAG()**`**)**:
    - Accesses data from a previous row in the same result set
    - Essential for comparing current and previous events
2. **DATEDIFF**:
    - Calculates the difference between two dates (in days in this case)
3. **CTE (WITH clause)**:
    - Creates a temporary result set that can be referenced in the main query
    - Makes complex queries more readable and manageable

This solution efficiently calculates the time difference between consecutive events for each user while handling the data grouping and sorting requirements.
# SQL Interview Question: Find the Nth Weekday of the Current Month

## 1. Overview
This document explains how to solve a challenging SQL interview question that requires dynamic date calculation without any input tables. The task is to find a specific occurrence of a weekday within the current month (e.g., the second Wednesday). This problem tests your advanced understanding of date functions, Common Table Expressions (CTEs), and particularly, **recursive CTEs** for generating data on the fly.

## 2. Problem Statement

### 2.1. The Scenario
You are not given any input tables. The query must operate based on the current system date.

### 2.2. The Requirement
Write a SQL query to find the date of the **second Wednesday** of the current month. The query must be dynamic, meaning if it is executed in July 2024, it should return `2024-07-10`. If the same query is run in August 2024, it should automatically return the date of the second Wednesday of August.

## 3. Solution and Explanation
The solution involves generating a calendar for the current month, filtering for the desired weekday, ranking the occurrences, and finally selecting the Nth one. This is achieved through a series of steps, best organized using CTEs.

### 3.1. Step 1: Generate All Dates for the Current Month using a Recursive CTE
The core of the solution is to create a temporary calendar. A recursive CTE is the perfect tool for this.

#### 3.1.1. The Anchor Member: Finding the First Day
First, we need a starting point. We can dynamically calculate the first day of the current month using date functions.

-   **Logic**: `DATEFROMPARTS(YEAR(GETDATE()), MONTH(GETDATE()), 1)` constructs a date using the year and month from the current date (`GETDATE()`) and sets the day to `1`.
-   **Example**: If today is July 30, 2024, this will produce `2024-07-01`.

#### 3.1.2. The Recursive Member: Adding Days
Next, we recursively add one day at a time to the anchor date until we reach the end of the month.

-   **Logic**:
    -   The `UNION ALL` combines the anchor result with the recursive results.
    -   `DATEADD(day, 1, [CurrentDate])` takes the date from the previous iteration and adds one day to it.
    -   **Termination Condition**: The recursion must stop. A `WHERE` clause checks if the month of the newly generated date is still the same as the current month. Once the date flips to the next month (e.g., from July 31st to August 1st), the condition fails and the recursion stops.

### 3.2. Step 2: Filter and Rank the Target Weekdays
Once we have a list of all dates in the current month, we can identify and rank our target weekday.

#### 3.2.1. Identifying the Weekday
We can use the `DATENAME` function to get the name of the weekday for each date.
-   **Logic**: `DATENAME(weekday, [Date])` returns a string like 'Monday', 'Tuesday', etc. We can then use a `WHERE` clause to filter for `'Wednesday'`.

#### 3.2.2. Ranking the Occurrences
After filtering, we have a list of all Wednesdays in the month. We use `ROW_NUMBER()` to assign a rank to each one.
-   **Logic**: `ROW_NUMBER() OVER (ORDER BY [Date])` assigns a sequential number (1, 2, 3, ...) to each Wednesday in chronological order.

### 3.3. Step 3: Select the Final Date
With the ranks assigned, the final step is to simply select the row with the desired rank.
-   **Logic**: A `WHERE` clause filters for `rank = 2` to get the second Wednesday.

> [!TIP]
> This approach is highly flexible. By changing the weekday name in the filter and the rank number in the final `WHERE` clause, you can find any occurrence of any weekday (e.g., the 4th Friday or the 1st Monday).

## 4. The Complete SQL Query (Using CTEs)
This query combines all the steps into a single, readable block.

```sql
-- CTE to generate all dates of the current month
WITH DateGenerator AS (
    -- Anchor Member: Get the first day of the current month
    SELECT CAST(DATEFROMPARTS(YEAR(GETDATE()), MONTH(GETDATE()), 1) AS DATE) AS CalendarDate
    UNION ALL
    -- Recursive Member: Add one day at a time until the month changes
    SELECT
        DATEADD(day, 1, CalendarDate)
    FROM
        DateGenerator
    WHERE
        MONTH(DATEADD(day, 1, CalendarDate)) = MONTH(GETDATE())
),
-- CTE to find and rank all Wednesdays in the generated calendar
RankedWednesdays AS (
    SELECT
        CalendarDate,
        DATENAME(weekday, CalendarDate) AS DayName,
        ROW_NUMBER() OVER (ORDER BY CalendarDate) AS WeekdayRank
    FROM
        DateGenerator
    WHERE
        DATENAME(weekday, CalendarDate) = 'Wednesday'
)
-- Final SELECT to get the 2nd Wednesday
SELECT
    CalendarDate
FROM
    RankedWednesdays
WHERE
    WeekdayRank = 2;
```

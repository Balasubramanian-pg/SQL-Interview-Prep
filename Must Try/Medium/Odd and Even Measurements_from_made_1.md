# Odd vs. Even Numbered Measurements

## 1. Problem Statement

### 1.1. Objective
Assume you're given a table with measurement values obtained from a Google sensor over multiple days, with measurements taken multiple times within each day.

Write a query to calculate the sum of odd-numbered and even-numbered measurements separately for a particular day and display the results in two different columns.

### 1.2. Definition
-   Within a day, measurements taken at 1st, 3rd, and 5th times are considered odd-numbered measurements, and measurements taken at 2nd, 4th, and 6th times are considered even-numbered measurements.

> [!IMPORTANT]
> The definition of "odd-numbered" and "even-numbered" is based on the **chronological order** of measurements within a single day. This is a crucial requirement that necessitates the use of ranking or window functions.

### 1.3. Input Table: `measurements`

|Column Name|Type|
|---|---|
|measurement_id|integer|
|measurement_value|decimal|
|measurement_time|datetime|

### 1.4. Example

#### 1.4.1. `measurements` Example Input:

|measurement_id|measurement_value|measurement_time|
|---|---|---|
|131233|1109.51|07/10/2022 09:00:00|
|135211|1662.74|07/10/2022 11:00:00|
|523542|1246.24|07/10/2022 13:15:00|
|143562|1124.50|07/11/2022 15:00:00|
|346462|1234.14|07/11/2022 16:45:00|

#### 1.4.2. Example Output:

|measurement_day|odd_sum|even_sum|
|---|---|---|
|07/10/2022 00:00:00|2355.75|1662.74|
|07/11/2022 00:00:00|1124.50|1234.14|

## 2. Conceptual Approach
To solve this, we need to first rank the measurements within each day and then use conditional aggregation to sum the odd-ranked and even-ranked values separately.

1.  **Extract the Day**: We need to isolate the day component from the `measurement_time` column so we can group by it later.
2.  **Rank Measurements**: Use the `ROW_NUMBER()` window function to assign a sequential number (1, 2, 3...) to each measurement within a single day, ordered by `measurement_time`.
3.  **Classify Odd/Even**: Use the modulus operator (`%` or `MOD()`) on the assigned row numbers to determine if a measurement is odd or even.
4.  **Conditional Aggregation**: Use `CASE` statements inside `SUM()` to calculate the totals for odd and even measurements in separate columns.

> [!TIP]
> The `ROW_NUMBER()` function is essential here. `RANK()` and `DENSE_RANK()` would not work because if two measurements are taken at the exact same time, they would get the same rank, which would disrupt the odd/even sequence. `ROW_NUMBER()` is guaranteed to be unique.

## 3. SQL Solution

```sql
WITH numbered_measurements AS (
  -- Step 1 & 2: Extract the date and rank measurements chronologically within each day.
  SELECT
    DATE(measurement_time) AS measurement_day,
    measurement_value,
    ROW_NUMBER() OVER (
      PARTITION BY DATE(measurement_time)
      ORDER BY measurement_time
    ) AS measurement_order
  FROM measurements
)
-- Step 3 & 4: Calculate the sum of odd-numbered and even-numbered measurements separately.
SELECT
  measurement_day,
  SUM(CASE WHEN measurement_order % 2 = 1 THEN measurement_value ELSE 0 END) AS odd_sum,
  SUM(CASE WHEN measurement_order % 2 = 0 THEN measurement_value ELSE 0 END) AS even_sum
FROM numbered_measurements
GROUP BY measurement_day
ORDER BY measurement_day;
```

## 4. Code Breakdown

### 4.1. CTE: `numbered_measurements`
This CTE performs the initial data transformation by ranking the measurements.
-   `DATE(measurement_time) AS measurement_day`: This extracts the date part from the timestamp, which is used for partitioning the data by day.
-   `ROW_NUMBER() OVER (...)`: This assigns the chronological order of the measurements.
    -   `PARTITION BY DATE(measurement_time)`: This is crucial. It tells the ranking function to reset the counter for each new day.
    -   `ORDER BY measurement_time`: This orders the rows within each day by time, ensuring that the earliest measurement gets `measurement_order = 1`.

> [!NOTE]
> The output of this CTE includes `measurement_day`, `measurement_value`, and the `measurement_order` (1, 2, 3...) for each measurement within a day.

### 4.2. Main `SELECT` Statement
This part calculates the final sums using conditional aggregation on the results of the CTE.

-   `GROUP BY measurement_day`: This aggregates the results for each day.
-   **Conditional Aggregation (`SUM(CASE WHEN ...)`):**
    -   `measurement_order % 2 = 1`: This condition checks if the `measurement_order` (1st, 2nd, 3rd, etc.) is odd. The modulus operator (`%`) returns the remainder after division. If the order is odd, the remainder when divided by 2 is 1.
    -   `measurement_order % 2 = 0`: This condition checks if the `measurement_order` is even.

> [!IMPORTANT]
> The `CASE` statements are used inside the `SUM` function. If the condition is met (odd or even), the `measurement_value` is included in the sum; otherwise, `0` is used. This allows us to calculate the two sums separately in parallel.

-   `odd_sum` and `even_sum`: The `SUM()` functions apply the conditional logic, resulting in two separate columns for the totals.
-   `ORDER BY measurement_day`: Ensures the final output is chronologically ordered by day.

> [!TIP]
> This pattern of using `ROW_NUMBER()` over a partition and then applying conditional aggregation based on the row number's parity is a standard and efficient way to handle "alternate row" or "even/odd" analysis problems.

---
Difficulty: Easy
---
# SQL Time Series Data Analysis

## Problem Statement

Given a temperature readings table with:

- `reading_id`
- `sensor_id`
- `reading_time` (timestamp)
- `temperature`

We need to:

1. Extract just the date from the timestamp
2. Calculate the average temperature for each date
3. Round the results to 2 decimal places
4. Order the results by date

## Solution Code

```SQL
SELECT
    DATE(reading_time) AS reading_date,
    ROUND(AVG(temperature), 2) AS average_temperature
FROM
    temperature_readings
GROUP BY
    DATE(reading_time)
ORDER BY
    DATE(reading_time);
```

## Explanation

1. **DATE() Function**:
    - Extracts just the date portion from the timestamp
    - Converts values like "2023-07-01 08:30:00" to "2023-07-01"
2. **AVG() Function**:
    - Calculates the average temperature for all readings on each date
    - Works with the GROUP BY clause to aggregate by date
3. **ROUND() Function**:
    - Formats the average temperature to 2 decimal places
    - Makes the output more readable (e.g., 27.47 instead of 27.4666667)
4. **GROUP BY**:
    - Groups all readings by their date for the average calculation
    - Must match the expression in the SELECT clause
5. **ORDER BY**:
    - Sorts the results chronologically by date
    - Ensures a logical presentation of the time series data

## Key Concepts

1. **Time Series Analysis**:
    - Common requirement for IoT/sensor data
    - Focuses on aggregating measurements over time periods
2. **Date Extraction**:
    - Essential when you need to analyze data by day/month/year
    - Different databases may use slightly different functions (DATE(), CAST(), etc.)
3. **Data Aggregation**:
    - AVG() calculates the mean temperature
    - Other useful functions: MIN(), MAX(), SUM(), COUNT()
4. **Result Formatting**:
    - ROUND() improves presentation
    - Can also use TRUNCATE() or database-specific formatting functions

This solution efficiently transforms raw timestamp data into meaningful daily averages, which is a common requirement for monitoring and reporting systems.
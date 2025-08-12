# INSERT INTO Query for Nth Weekday Problem

To create a test environment for the Nth weekday problem, we need to insert sample data that represents dates and their corresponding weekdays. Here's how we can set this up:

## Option 1: Inserting Static Test Data

```sql
-- Create a table to store dates and weekdays
CREATE TABLE CalendarDates (
    DateValue DATE,
    DayName VARCHAR(10)
);

-- Insert sample dates for testing (July 2024 as example)
INSERT INTO CalendarDates (DateValue, DayName)
VALUES
    ('2024-07-01', 'Monday'),
    ('2024-07-02', 'Tuesday'),
    ('2024-07-03', 'Wednesday'),
    ('2024-07-04', 'Thursday'),
    ('2024-07-05', 'Friday'),
    ('2024-07-06', 'Saturday'),
    ('2024-07-07', 'Sunday'),
    ('2024-07-08', 'Monday'),
    ('2024-07-09', 'Tuesday'),
    ('2024-07-10', 'Wednesday'),  -- Second Wednesday
    ('2024-07-11', 'Thursday'),
    ('2024-07-12', 'Friday'),
    ('2024-07-13', 'Saturday'),
    ('2024-07-14', 'Sunday'),
    ('2024-07-15', 'Monday'),
    ('2024-07-16', 'Tuesday'),
    ('2024-07-17', 'Wednesday'),  -- Third Wednesday
    ('2024-07-18', 'Thursday'),
    ('2024-07-19', 'Friday'),
    ('2024-07-20', 'Saturday'),
    ('2024-07-21', 'Sunday'),
    ('2024-07-22', 'Monday'),
    ('2024-07-23', 'Tuesday'),
    ('2024-07-24', 'Wednesday'),  -- Fourth Wednesday
    ('2024-07-25', 'Thursday'),
    ('2024-07-26', 'Friday'),
    ('2024-07-27', 'Saturday'),
    ('2024-07-28', 'Sunday'),
    ('2024-07-29', 'Monday'),
    ('2024-07-30', 'Tuesday'),
    ('2024-07-31', 'Wednesday');  -- Fifth Wednesday
```

## Option 2: Dynamic Insert for Any Month/Year

```sql
-- Create table if it doesn't exist
IF NOT EXISTS (SELECT * FROM sys.tables WHERE name = 'CalendarDates')
BEGIN
    CREATE TABLE CalendarDates (
        DateValue DATE,
        DayName VARCHAR(10),
        MonthName VARCHAR(10),
        DayOfWeek INT,
        WeekdayRank INT
    );
END
ELSE
BEGIN
    TRUNCATE TABLE CalendarDates;
END

-- Insert all dates for current month with calculated fields
WITH DateGenerator AS (
    SELECT CAST(DATEFROMPARTS(YEAR(GETDATE()), MONTH(GETDATE()), 1) AS DATE) AS CalendarDate
    UNION ALL
    SELECT DATEADD(day, 1, CalendarDate)
    FROM DateGenerator
    WHERE MONTH(DATEADD(day, 1, CalendarDate)) = MONTH(GETDATE())
)
INSERT INTO CalendarDates (DateValue, DayName, MonthName, DayOfWeek)
SELECT 
    CalendarDate,
    DATENAME(weekday, CalendarDate),
    DATENAME(month, CalendarDate),
    DATEPART(weekday, CalendarDate)
FROM DateGenerator
OPTION (MAXRECURSION 31);

-- Update weekday ranks for each day of week
;WITH RankedDays AS (
    SELECT 
        DateValue,
        DayName,
        ROW_NUMBER() OVER (PARTITION BY DayName ORDER BY DateValue) AS WeekdayRank
    FROM CalendarDates
)
UPDATE cd
SET cd.WeekdayRank = rd.WeekdayRank
FROM CalendarDates cd
JOIN RankedDays rd ON cd.DateValue = rd.DateValue;
```

## Query to Find the Second Wednesday

After inserting the data, you can find the second Wednesday with:

```sql
SELECT DateValue 
FROM CalendarDates 
WHERE DayName = 'Wednesday' AND WeekdayRank = 2;
```

Or for a more dynamic approach that doesn't rely on the table:

```sql
WITH DateGenerator AS (
    SELECT CAST(DATEFROMPARTS(YEAR(GETDATE()), MONTH(GETDATE()), 1) AS DATE) AS CalendarDate
    UNION ALL
    SELECT DATEADD(day, 1, CalendarDate)
    FROM DateGenerator
    WHERE MONTH(DATEADD(day, 1, CalendarDate)) = MONTH(GETDATE())
),
RankedWednesdays AS (
    SELECT
        CalendarDate,
        ROW_NUMBER() OVER (ORDER BY CalendarDate) AS WeekdayRank
    FROM
        DateGenerator
    WHERE
        DATENAME(weekday, CalendarDate) = 'Wednesday'
)
SELECT CalendarDate
FROM RankedWednesdays
WHERE WeekdayRank = 2;
```

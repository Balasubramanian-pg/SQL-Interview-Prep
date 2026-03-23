## Date Parts (YEAR, MONTH, QUARTER, DATEPART) Cheat Sheet Table

| Problem Type | SQL Concept | Solution Pattern (Conceptual) | Key Technique |
| :--- | :--- | :--- | :--- |
| **Extracting a Part** | Retrieving a specific numerical part (e.g., the number `3` for March). | Use the ISO standard **`EXTRACT(part FROM date)`** or the platform-specific **`DATEPART`** function. | ` $EXTRACT(YEAR FROM date\_col)$ (Standard) or  `$DATEPART(year, date\_col)$ (SQL Server)`  | | **Trimming/Rounding** | Reducing the precision of a timestamp to a specific boundary (e.g., reducing a timestamp to the start of the hour or month). | Use the ** `DATE\_TRUNC`** or **`TRUNC` ** function. |  `$DATE\_TRUNC('month', date\_col)$ (PostgreSQL/Redshift)`| | **Standard Naming** | Converting a numerical part into a recognizable name (e.g.,`3`to`March`). | Use the **`DATENAME`** or **`TO\_CHAR` ** function with a format mask. |  `$DATENAME(month, date\_col)$ (SQL Server) or `$TO\_CHAR(date\_col, 'Month')$ (Oracle/PostgreSQL)` |
| **Grouping/Filtering** | Grouping or filtering based solely on a specific date component. | Use the extraction function directly in the `GROUP BY` or `WHERE` clause. | `$GROUP BY EXTRACT(MONTH FROM date\_col)$` |

## Date Parts Mini Playbook (Realistic Queries)

These snippets illustrate common date part extraction, showing syntax variations where necessary.

### 1\. Extracting Year, Month, and Quarter

**Use Case:** Analyze sales by year, quarter, and month of the transaction.

| Database | Syntax for Quarter/Year |
| :--- | :--- |
| **PostgreSQL, Oracle, MySQL** | `EXTRACT(QUARTER FROM transaction_date)` |
| **SQL Server** | `DATEPART(qq, transaction_date)` |

```sql
SELECT
    EXTRACT(YEAR FROM transaction_date) AS txn_year,
    EXTRACT(QUARTER FROM transaction_date) AS txn_quarter,
    EXTRACT(MONTH FROM transaction_date) AS txn_month,
    SUM(amount) AS quarterly_sales
FROM
    Transactions
GROUP BY
    1, 2, 3 -- Group by SELECT column position is often supported
ORDER BY
    1, 2, 3;
```

### 2\. Trimming/Rounding (DATE\_TRUNC)

**Use Case:** Calculate the total number of new users who signed up each calendar **month**.

*This is often the most robust way to group data by month/year because it creates a comparable timestamp/date value for the first day of that period.*

```sql
SELECT
    -- Truncates the timestamp to the first day of the month (e.g., 2025-10-01 00:00:00)
    DATE_TRUNC('month', signup_timestamp) AS signup_month,
    COUNT(user_id) AS new_users_count
FROM
    Users
GROUP BY
    1
ORDER BY
    1;
-- PostgreSQL/Redshift/Snowflake syntax shown.
```

### 3\. Standard Naming (DATENAME / TO\_CHAR)

**Use Case:** Group website traffic by the day of the week (e.g., Monday, Tuesday) to find peak traffic days.

| Database | Syntax for Day Name |
| :--- | :--- |
| **SQL Server** | `DATENAME(weekday, session_start_date)` |
| **PostgreSQL** | `TO_CHAR(session_start_date, 'Day')` |
| **MySQL** | `DAYNAME(session_start_date)` |

```sql
SELECT
    -- Extracts the name of the day (e.g., 'Monday', 'Tuesday')
    DATENAME(weekday, session_start_date) AS day_of_week,
    AVG(duration_seconds) AS avg_session_duration
FROM
    Web_Sessions
GROUP BY
    1
ORDER BY
    -- Order by the numerical day part to sort chronologically (1=Sunday, 2=Monday, etc.)
    DATEPART(dw, session_start_date); 
-- SQL Server syntax shown.
```

### 4\. Grouping by Week Number

**Use Case:** Summarize production volumes by the week number of the year.

```sql
SELECT
    -- Use the database-specific function to get the week number
    EXTRACT(WEEK FROM production_date) AS week_number,
    SUM(units_produced) AS total_units
FROM
    Production_Log
GROUP BY
    1
ORDER BY
    1;
-- PostgreSQL syntax shown.
```

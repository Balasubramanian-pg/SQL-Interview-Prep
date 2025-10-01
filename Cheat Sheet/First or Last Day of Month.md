This cheat sheet covers the essential SQL techniques for deriving the **first and last days** of the current month, quarter, or year from any given date. The specific functions used vary significantly by database platform (SQL Server, PostgreSQL, MySQL, etc.).

## First/Last Day of Month/Year Cheat Sheet Table

| Problem Type | Goal | Solution Pattern (Conceptual) | Key Technique |
| :--- | :--- | :--- | :--- |
| **First Day of Month** | Get the exact date of the 1st of the month containing `input_date`. | **`DATE_TRUNC('month', input_date)`** or add/subtract to the day part. | `DATE_TRUNC('month', date)` / `DATEADD(day, 1-DAY(date), date)` |
| **Last Day of Month** | Get the exact date of the last day of the month containing `input_date`. | **`EOMONTH(input_date)`** or find the first day of the *next* month and subtract one day. | `EOMONTH(date)` / `LAST_DAY(date)` |
| **First Day of Year** | Get the exact date of **January 1st** of the year containing `input_date`. | **`DATE_TRUNC('year', input_date)`** or similar truncation function. | `DATE_TRUNC('year', date)` / `DATEFROMPARTS(YEAR(date), 1, 1)` |
| **Last Day of Year** | Get the exact date of **December 31st** of the year containing `input_date`. | Find the first day of the *next* year and subtract one day. | `DATE_TRUNC('year', date + 1 year) - 1 day` |

-----

## First/Last Day of Month/Year Mini Playbook (Realistic Queries)

These snippets illustrate common methods for deriving period boundaries, using syntax examples from different platforms.

### 1\. First Day of Month

**Use Case:** Truncate a transaction date to the start of the month for consistent grouping.

| Database | Syntax |
| :--- | :--- |
| **PostgreSQL, Redshift** | `DATE_TRUNC('month', transaction_date)` |
| **SQL Server** | `DATEADD(month, DATEDIFF(month, 0, transaction_date), 0)` |
| **MySQL** | `DATE_FORMAT(transaction_date, '%Y-%m-01')` |

```sql
SELECT
    transaction_date,
    DATE_TRUNC('month', transaction_date) AS first_day_of_month
FROM
    Transactions;
-- PostgreSQL syntax shown.
```

-----

### 2\. Last Day of Month

**Use Case:** Define the end date for a monthly report based on the current date.

| Database | Syntax |
| :--- | :--- |
| **SQL Server** | `EOMONTH(report_date)` |
| **Oracle, MySQL** | `LAST_DAY(report_date)` |
| **PostgreSQL** | `(DATE_TRUNC('month', report_date) + INTERVAL '1 month') - INTERVAL '1 day'` |

```sql
SELECT
    CURRENT_DATE,
    EOMONTH(CURRENT_DATE) AS last_day_of_current_month
FROM
    DUMMY_TABLE;
-- SQL Server syntax shown.
```

-----

### 3\. First Day of Year

**Use Case:** Calculate year-to-date (YTD) metrics by establishing the starting date of the year.

```sql
SELECT
    signup_date,
    DATE_TRUNC('year', signup_date) AS first_day_of_year
FROM
    Users;
-- PostgreSQL syntax shown.
```

-----

### 4\. Last Day of Year

**Use Case:** Calculate the final date boundary for a full year's worth of data.

\*The general technique is: **Find the first day of next year, then subtract one day/unit.**

| Database | Syntax |
| :--- | :--- |
| **SQL Server** | `DATEADD(day, -1, DATEADD(year, DATEDIFF(year, 0, date_col) + 1, 0))` |
| **PostgreSQL** | `DATE_TRUNC('year', date_col) + INTERVAL '1 year' - INTERVAL '1 day'` |
| **MySQL** | `DATE_SUB(DATE_ADD(DATE_TRUNC('year', date_col), INTERVAL 1 YEAR), INTERVAL 1 DAY)` |

```sql
SELECT
    audit_date,
    DATE_TRUNC('year', audit_date) + INTERVAL '1 year' - INTERVAL '1 day' AS last_day_of_year
FROM
    Audit_Records;
-- PostgreSQL syntax shown.
```

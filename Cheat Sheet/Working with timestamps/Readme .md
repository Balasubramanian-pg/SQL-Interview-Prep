## Working with Timestamps Cheat Sheet Table

| Problem Type | Goal | Solution Pattern (Conceptual) | Key Technique |
| :--- | :--- | :--- | :--- |
| **Get Current Time** | Retrieve the current server or session timestamp. | Use the database-specific function for current time/timestamp. | `NOW()`, `CURRENT_TIMESTAMP()`, `GETDATE()` |
| **Time Zone Conversion** | Convert a timestamp from one time zone to another (e.g., UTC to local time). | Use the database-specific time zone conversion function. | `CONVERT_TZ()`, `AT TIME ZONE`, `TIMESTAMPTZ` |
| **Extracting Time Parts** | Retrieving the hour, minute, second, or millisecond from a timestamp. | Use the platform-specific extraction function (e.g., `EXTRACT`, `DATEPART`). | `EXTRACT(HOUR FROM timestamp_col)`, `DATEPART(hour, timestamp_col)` |
| **Time Difference** | Calculate the precise difference between two timestamps in a specific unit (e.g., seconds, milliseconds). | Use `DATEDIFF` or subtraction with type casting for precise measurement. | `DATEDIFF(millisecond, T1, T2)` / `(T2 - T1)` |

## Working with Timestamps Mini Playbook (Realistic Queries)

These snippets illustrate common timestamp operations, highlighting the database-specific nature of many time functions.

### 1\. Retrieving the Current Timestamp

**Use Case:** Record the exact moment a log entry is created.

| Database | Syntax |
| :--- | :--- |
| **PostgreSQL, MySQL** | `NOW()` or `CURRENT_TIMESTAMP()` |
| **SQL Server** | `GETDATE()` or `SYSDATETIME()` (for higher precision) |

```sql
INSERT INTO
    audit_logs (action, timestamp)
VALUES
    ('Login Success', NOW());
-- PostgreSQL/MySQL syntax shown.
```

### 2\. Time Zone Conversion

**Use Case:** Convert transaction timestamps stored in UTC to the local EST time zone for reporting.

| Database | Syntax (UTC $\implies$ EST) |
| :--- | :--- |
| **PostgreSQL** | `transaction_timestamp AT TIME ZONE 'UTC' AT TIME ZONE 'EST'` |
| **SQL Server** | `transaction_timestamp AT TIME ZONE 'UTC' AT TIME ZONE 'Eastern Standard Time'` |
| **MySQL** | `CONVERT_TZ(transaction_timestamp, 'UTC', 'EST')` |

```sql
SELECT
    transaction_id,
    transaction_timestamp,
    transaction_timestamp AT TIME ZONE 'UTC' AT TIME ZONE 'EST' AS local_est_timestamp
FROM
    Transactions;
-- PostgreSQL syntax shown.
```

### 3\. Extracting Time Parts

**Use Case:** Group website events by the hour of the day to find peak usage times.

| Database | Syntax (Extract Hour) |
| :--- | :--- |
| **PostgreSQL, MySQL** | `EXTRACT(HOUR FROM event_timestamp)` |
| **SQL Server** | `DATEPART(hour, event_timestamp)` |

```sql
SELECT
    EXTRACT(HOUR FROM event_timestamp) AS event_hour,
    COUNT(event_id) AS total_events
FROM
    Website_Events
GROUP BY
    1
ORDER BY
    2 DESC;
-- PostgreSQL syntax shown.
```

### 4\. Time Difference (Latency Calculation)

**Use Case:** Calculate the exact latency (in milliseconds) between a request and its corresponding response.

| Database | Syntax (Difference in Milliseconds) |
| :--- | :--- |
| **SQL Server** | `DATEDIFF(millisecond, request_time, response_time)` |
| **PostgreSQL** | `EXTRACT(EPOCH FROM (response_time - request_time)) * 1000` |
| **MySQL** | `TIMESTAMPDIFF(MICROSECOND, request_time, response_time) / 1000` |

```sql
SELECT
    request_id,
    request_time,
    response_time,
    DATEDIFF(millisecond, request_time, response_time) AS latency_ms
FROM
    API_Log;
-- SQL Server syntax shown.
```

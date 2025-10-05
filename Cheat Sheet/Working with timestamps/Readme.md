## Working with Timestamps Cheat Sheet Table

| Problem Type | Goal | Solution Pattern (Conceptual) | Key Technique |
| :--- | :--- | :--- | :--- |
| **Get Current Time** | Retrieve the current server or session timestamp. | Use the database-specific function for current time/timestamp. | `NOW()`, `CURRENT_TIMESTAMP()`, `GETDATE()` |
| **Time Zone Conversion** | Convert a timestamp from one time zone to another (e.g., UTC to local time). | Use the database-specific time zone conversion function. | `CONVERT_TZ()`, `AT TIME ZONE`, `TIMESTAMPTZ` |
| **Extracting Time Parts** | Retrieving the hour, minute, second, or millisecond from a timestamp. | Use the platform-specific extraction function (e.g., `EXTRACT`, `DATEPART`). | `EXTRACT(HOUR FROM timestamp_col)`, `DATEPART(hour, timestamp_col)` |
| **Time Difference** | Calculate the precise difference between two timestamps in a specific unit (e.g., seconds, milliseconds). | Use `DATEDIFF` or subtraction with type casting for precise measurement. | `DATEDIFF(millisecond, T1, T2)` / `(T2 - T1)` |


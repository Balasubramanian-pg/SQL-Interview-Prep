Common casting patterns in SQL involve explicitly changing an expression from one data type to another using the standard **`CAST`** function or the proprietary **`CONVERT`** function (primarily SQL Server). These patterns are crucial for data cleaning, calculation, and ensuring proper comparison in queries.

Here are the most common and necessary casting patterns:

## Common Casting Patterns in SQL

### 1\. Numeric $\iff$ Numeric (Precision/Rounding)

This is used to control the precision of numerical values, often for rounding or standardizing calculations.

| Pattern | Use Case | Example |
| :--- | :--- | :--- |
| `CAST(num AS INT)` | Truncating a decimal number to an integer (e.g., stripping time from a timestamp). | `CAST(45.98 AS INT) = 45` |
| `CAST(num AS DECIMAL(p, s))` | Controlling precision for currency or statistical calculations (e.g., standardizing `DECIMAL` to 2 significant places). | `CAST(5.1234 AS DECIMAL(10, 2)) = 5.12` |
| `CAST(num AS BIGINT)` | Ensuring a large calculation doesn't overflow standard integer limits. | `CAST(total_count AS BIGINT)` |

### 2\. String $\iff$ Date/Time (Parsing/Extraction)

This is essential for parsing dates that are stored as text and for extracting parts of a date into a string format.

| Pattern | Use Case | Example |
| :--- | :--- | :--- |
| `CAST(str AS DATE)` | Converting a stored string (e.g., '2025-10-01') into a true `DATE` or `DATETIME` object for arithmetic. | `CAST('2025-10-01' AS DATE)` |
| `CAST(datetime AS DATE)` | Stripping the time component from a `DATETIME` or `TIMESTAMP` value. | `CAST(CURRENT_TIMESTAMP() AS DATE)` |
| `CAST(datetime AS TIME)` | Isolating only the time component. | `CAST('2025-10-01 14:30:00' AS TIME)` |
| `CAST(num AS VARCHAR)` | Converting numeric data (like a year or an ID) to text for display or concatenation. | `CAST(order_id AS VARCHAR(10))` |

### Example: Casting String to Date for Comparison

```sql
SELECT
    *
FROM
    Sales_Data
WHERE
    sale_date >= CAST('2025-09-01' AS DATE); -- Ensures the filter is treated as a DATE object
```

### 3\. String $\iff$ String (Length Control)

Used primarily to limit the length of a string, often to comply with schema constraints or reduce network payload.

| Pattern | Use Case | Example |
| :--- | :--- | :--- |
| `CAST(str AS VARCHAR(N))` | Truncating a long text field (`TEXT` or `NVARCHAR(MAX)`) to a defined maximum length (N). | `CAST(full_notes AS VARCHAR(255))` |

### Example: Truncating a Field

```sql
SELECT
    product_id,
    CAST(product_description AS VARCHAR(50)) AS short_description
FROM
    Products; -- Ensures the output column length is limited to 50 characters
```

### 4\. Special Use Cases (Boolean/JSON)

Casting to specialized types, though platform-dependent, follows the same standard syntax.

| Pattern | Use Case | Example |
| :--- | :--- | :--- |
| `CAST(val AS BOOLEAN)` | Converting 0/1 or 'T'/'F' strings into a true boolean type (PostgreSQL/MySQL). | `CAST(is_active_flag AS BOOLEAN)` |
| `CAST(str AS JSON)` | Converting a text string containing JSON data into a structured JSON data type (PostgreSQL/SQL Server). | `CAST(json_text AS JSON)` |

### Example: Casting to Boolean

```sql
SELECT
    user_id,
    CAST(active_status_int AS BOOLEAN) AS is_user_active
FROM
    Users;
```

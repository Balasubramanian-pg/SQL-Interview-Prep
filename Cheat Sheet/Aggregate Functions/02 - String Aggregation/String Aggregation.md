## String Aggregation (GROUP\_CONCAT, STRING\_AGG) Cheat Sheet Table

| Function | Standard | Database Support | Key Feature |
| :--- | :--- | :--- | :--- |
| **`STRING_AGG()`** | **ANSI SQL (modern)** | **SQL Server (2017+), PostgreSQL (9.0+)** | Allows explicit control over the **delimiter** and often supports an **`ORDER BY`** clause for sorting the aggregated string. |
| **`GROUP_CONCAT()`** | **Proprietary** | **MySQL** | A well-known MySQL-specific function that easily concatenates strings within a group, often supporting `DISTINCT` and `ORDER BY`. |
| **`LISTAGG()`** | **Proprietary** | **Oracle** | Oracle's equivalent function, which also supports delimiters and sorting. |
| **`string_agg()`** | **Proprietary** | **SQLite** | SQLite's implementation, similar to the ANSI standard, with delimiter and ordering support. |

-----

## String Aggregation Mini Playbook (Realistic Queries)

These snippets illustrate the primary ways to perform string aggregation, focusing on the most common functions.

### 1\. STRING\_AGG (SQL Server/PostgreSQL)

**Use Case:** For each customer, generate a single string listing all the distinct products they have purchased, ordered alphabetically.

```sql
SELECT
    customer_id,
    -- Concatenates the product names with a comma and space separator, ordered alphabetically
    STRING_AGG(p.product_name, ', ') WITHIN GROUP (ORDER BY p.product_name) AS purchased_products
FROM
    Orders AS o
INNER JOIN
    Products AS p ON o.product_id = p.product_id
GROUP BY
    customer_id;

-- SQL Server/PostgreSQL syntax shown.
```

-----

### 2\. GROUP\_CONCAT (MySQL)

**Use Case:** For each tag category, list all the unique tags belonging to that category, separated by a pipe (`|`).

```sql
SELECT
    category_name,
    -- Concatenates the distinct tag names with a pipe delimiter
    GROUP_CONCAT(DISTINCT tag_name SEPARATOR ' | ') AS unique_tags
FROM
    Tags
GROUP BY
    category_name;

-- MySQL syntax shown.
```

-----

### 3\. STRING\_AGG without Ordering

**Use Case:** Combine all error messages for a single log ID. If sorting isn't necessary, the `WITHIN GROUP (ORDER BY...)` clause can be omitted for simplicity and potentially slightly faster execution.

```sql
SELECT
    log_id,
    -- Simple concatenation with a newline character as the separator
    STRING_AGG(error_message, '\n') AS combined_error_log
FROM
    Error_Messages
GROUP BY
    log_id;
```

-----

### 4\. LISTAGG (Oracle)

**Use Case:** List all employee names working in the same department, sorted by last name.

```sql
SELECT
    department_id,
    -- Concatenates names using a semicolon, ordered by last_name
    LISTAGG(employee_name, '; ') WITHIN GROUP (ORDER BY last_name) AS department_members
FROM
    Employees
GROUP BY
    department_id;

-- Oracle syntax shown.
```

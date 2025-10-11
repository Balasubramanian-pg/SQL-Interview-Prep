This cheat sheet compares the three main ways to perform **string concatenation** in SQL: the standard `CONCAT` function, the ANSI SQL standard `||` operator, and the database-specific `+` operator.

## CONCAT vs. || vs. + Cheat Sheet Table

| Operator/Function | Standard | Database Support | Handles NULLs | Key Characteristic |
| :--- | :--- | :--- | :--- | :--- |
| **`CONCAT()`** | **ANSI SQL Standard** | **High** (MySQL, PostgreSQL, SQL Server 2012+, Oracle, SQLite) | **Varies**. Most modern versions (e.g., MySQL, SQL Server) **ignore NULL** strings, treating them as empty strings (`''`). | Most portable function name, often easier to read when joining many strings. |
| **`||`** | **ANSI SQL Standard** | **High** (PostgreSQL, Oracle, SQLite, MySQL) | **Varies**. Typically treats NULL as an empty string, except in **Oracle**, where it results in **NULL**. | Highly readable operator, considered the standard for concatenation. |
| **`+`** | **Proprietary** | **SQL Server, MS Access** | **Always results in NULL** if any operand is NULL (unless `CONCAT_NULL_YIELDS_NULL` is set to OFF). | Used for both arithmetic and string concatenation in SQL Server; requires careful handling of data types. |

-----

## CONCAT vs. || vs. + Mini Playbook (Realistic Queries)

These snippets illustrate the primary differences, focusing on the handling of NULL values and required data types.

### 1\. CONCAT() (Standard Function)

**Use Case:** Assemble a customer's full address line, robustly handling potential `NULL` values in the `address_2` field.

```sql
SELECT
    user_id,
    CONCAT(first_name, ' ', last_name, ', ', address_1, ' ', address_2) AS full_address
FROM
    Users;

-- NULL Handling Example:
-- CONCAT('Hello', NULL, 'World')  =>  'HelloWorld' (Most common behavior)
```

-----

### 2\. || Operator (ANSI Standard Operator)

**Use Case:** Combine product name and color into a descriptive SKU label.

```sql
SELECT
    product_sku,
    product_name || ' - ' || color AS descriptive_label
FROM
    Products;

-- NULL Handling Example (PostgreSQL/SQLite/MySQL):
-- 'Hello' || NULL || 'World'  =>  'HelloWorld' (Ignores NULL)

-- NULL Handling Example (Oracle):
-- 'Hello' || NULL || 'World'  =>  NULL (Propagates NULL, often requiring NVL/COALESCE)
```

-----

### 3\. + Operator (SQL Server/Proprietary)

**Use Case:** Construct a fully qualified employee name and job title in SQL Server.

*Note: In SQL Server, the `+` operator requires all components to be of string data type (`VARCHAR`, `NVARCHAR`, etc.). If used with a non-string NULL, the result is NULL.*

```sql
SELECT
    employee_id,
    first_name + ' ' + last_name + ' (' + job_title + ')' AS full_signature
FROM
    Employees;

-- NULL Handling Example (SQL Server):
-- 'Hello' + NULL + 'World'  =>  NULL (Propagates NULL)

-- To avoid NULL propagation in SQL Server, use ISNULL or COALESCE:
-- SELECT first_name + ' ' + ISNULL(middle_name, '') + ' ' + last_name FROM Employees;
```

-----

### 4\. CONCAT\_WS (Concatenate with Separator)

**Bonus Technique:** Some databases (like MySQL and PostgreSQL with extensions) offer `CONCAT_WS` which simplifies concatenating many fields with a constant separator and automatically ignores NULL values.

```sql
SELECT
    customer_id,
    -- Concatenates three fields using ',' as the separator, ignoring NULLs
    CONCAT_WS(', ', city, state, zip_code) AS location_string
FROM
    Customers;
```

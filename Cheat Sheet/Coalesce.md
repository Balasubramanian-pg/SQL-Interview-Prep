## COALESCE vs. ISNULL vs. IFNULL Cheat Sheet Table

| Function | Standard | Database Support | Data Type Behavior | Key Feature |
| :--- | :--- | :--- | :--- | :--- |
| **`COALESCE`** | **ANSI SQL Standard** | **High** (PostgreSQL, Oracle, SQL Server, MySQL, SQLite) | Returns the data type of the **first non-NULL argument** (standard type precedence rules apply). | **Accepts unlimited arguments** (multiple fallback values). |
| **`ISNULL`** | **Proprietary** | **SQL Server, MS Access** | Returns the data type of the **first argument** (coercing the second argument's type if necessary). | **Accepts only two arguments** (the column and the replacement value). |
| **`IFNULL`** | **Proprietary** | **MySQL, SQLite, Oracle** (as a synonym for `COALESCE`) | Returns the data type of the first non-NULL argument. | **Accepts only two arguments** (the column and the replacement value). |

-----

## COALESCE vs. ISNULL vs. IFNULL Mini Playbook (Realistic Queries)

These snippets illustrate the primary differences, focusing on portability and the handling of multiple fallback values.

### 1\. COALESCE (ANSI Standard and Multiple Fallbacks)

**Use Case:** Retrieve the first available email address from a hierarchy of three potential fields: a primary contact, a secondary contact, or a generic department email.

```sql
SELECT
    user_id,
    -- Returns the first non-NULL value from the list
    COALESCE(
        primary_email,
        secondary_email,
        department_email,
        'No Email Provided' -- Final fallback value
    ) AS contact_email
FROM
    Contacts;

-- Example: COALESCE(NULL, NULL, 'generic@org.com', 'default') => 'generic@org.com'
```

-----

### 2\. ISNULL (SQL Server Specific)

**Use Case:** Replace a NULL `sale_price` with the `list_price` if the sale price is missing.

*Note: This is SQL Server-specific and only accepts two arguments.*

```sql
SELECT
    product_id,
    -- If sale_price is NULL, use list_price; otherwise, use sale_price
    ISNULL(sale_price, list_price) AS final_price
FROM
    Products;
    
-- Data Type Behavior (SQL Server): If sale_price is INT, and list_price is DECIMAL,
-- the result will be of type INT (type of the first argument), which may truncate list_price.
```

-----

### 3\. IFNULL (MySQL/SQLite Specific)

**Use Case:** Replace a NULL `commission_rate` with a default rate of `0.05`.

*Note: This is syntactical sugar for `COALESCE` in many databases but is widely known in MySQL and SQLite.*

```sql
SELECT
    agent_id,
    -- If commission_rate is NULL, use 0.05
    IFNULL(commission_rate, 0.05) AS calculated_rate
FROM
    Sales_Agents;
    
-- Example (MySQL): IFNULL(NULL, 0.05) => 0.05
```

-----

### 4\. Portability and Preference

**The `COALESCE` function is the universally recommended choice** for new development due to its ANSI standard compliance and support across all major platforms, ensuring maximum query portability.

```sql
-- The portable way to do the ISNULL/IFNULL replacement:
SELECT
    product_id,
    COALESCE(sale_price, list_price) AS final_price
FROM
    Products;
```

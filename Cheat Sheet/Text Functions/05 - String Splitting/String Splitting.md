This cheat sheet covers the techniques for **splitting a delimited string** into multiple components or rows. Since SQL lacks a universal standard split function, two common, platform-specific methods are highlighted, along with a general technique using `SUBSTRING` and position functions.

## String Splitting Cheat Sheet Table

| Problem Type | Goal | Solution Pattern | Key Technique |
| :--- | :--- | :--- | :--- |
| **Simple Splitting (SQL Server/Azure)** | Convert a delimited string into a table of rows. | Use the built-in **`STRING_SPLIT`** function. | `$SELECT value FROM STRING\_SPLIT(string, delimiter)$` |
| **Simple Splitting (MySQL)** | Extract a specific segment from a delimited string (e.g., the 2nd part). | Use the built-in **`SUBSTRING_INDEX`** function. | `$SUBSTRING\_INDEX(string, delimiter, index)$` |
| **Generic Splitting (Position-Based)** | Extract the first, second, or third segment using standard functions. | Use **`SUBSTRING`** combined with a positional function (like `CHARINDEX` or `STRPOS`). | `$SUBSTRING(string, 1, CHARINDEX(delimiter, string) - 1)$` |
| **Row Splitting (Advanced/Legacy)** | Convert a string to multiple rows where `STRING_SPLIT` is unavailable (e.g., PostgreSQL/Oracle). | Use a recursive CTE or a function like **`regexp_split_to_table`** (PostgreSQL). | `$SELECT * FROM regexp\_split\_to\_table(string, delimiter)$` |

-----

## String Splitting Mini Playbook (Realistic Queries)

These snippets illustrate the primary ways to split or parse delimited data.

### 1\. Simple Splitting (SQL Server/Azure/Oracle)

**Use Case:** Take a comma-separated list of product IDs in a single column and return them as individual rows for processing.

```sql
SELECT
    user_id,
    p.value AS purchased_product_id
FROM
    User_Carts AS uc
    -- STRING_SPLIT converts the delimited string into a table ('value' column)
    CROSS APPLY STRING_SPLIT(uc.product_list, ',') AS p; 
    
-- SQL Server/Azure syntax shown. Oracle has similar built-in functions or techniques.
```

-----

### 2\. Simple Splitting (MySQL SUBSTRING\_INDEX)

**Use Case:** Extract the **domain name** (the second part) from an email address using the `@` as the delimiter.

```sql
SELECT
    email,
    -- Get everything after the first '@'
    SUBSTRING_INDEX(email, '@', -1) AS domain_name 
FROM
    Emails;
    
-- Example: SUBSTRING_INDEX('user@example.com', '@', -1) => 'example.com'
```

**Use Case:** Extract the **user name** (the first part) from the email address.

```sql
SELECT
    email,
    -- Get everything before the first '@'
    SUBSTRING_INDEX(email, '@', 1) AS user_name
FROM
    Emails;
    
-- Example: SUBSTRING_INDEX('user@example.com', '@', 1) => 'user'
```

-----

### 3\. Generic Splitting (Extracting the First Segment)

**Use Case:** Extract the **first tag** from a string of space-separated tags, where `CHARINDEX` (SQL Server) finds the position of the first space.

```sql
SELECT
    tags_string,
    SUBSTRING(
        tags_string,
        1,
        CHARINDEX(' ', tags_string) - 1 -- Length is up to the space
    ) AS first_tag
FROM
    Posts
WHERE
    CHARINDEX(' ', tags_string) > 0; -- Ensure there is a delimiter
    
-- Syntax for CHARINDEX/STRPOS varies by database.
```

-----

### 4\. Row Splitting (PostgreSQL REGEXP\_SPLIT\_TO\_TABLE)

**Use Case:** Use PostgreSQL's regular expression split function to unnest the string into multiple rows. This is an efficient alternative to recursive CTEs in PostgreSQL.

```sql
SELECT
    user_id,
    -- REGEXP_SPLIT_TO_TABLE splits the string by the comma and returns a rowset
    item
FROM
    User_Preferences,
    REGEXP_SPLIT_TO_TABLE(preferred_items, ',') AS item;
```

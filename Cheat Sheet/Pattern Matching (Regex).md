## Pattern Matching (REGEXP vs. LIKE) Cheat Sheet Table

| Technique | Goal | Solution Pattern | Key Characteristic |
| :--- | :--- | :--- | :--- |
| **Simple Matching (LIKE)** | Match any string containing, starting with, or ending with a substring. | `$column LIKE 'pattern\_with\_\%'` | Uses simple wildcards (`%`, `_`). Widely supported, but limited to basic patterns. |
| **Complex Matching (RegEx)** | Match strings against complex patterns (e.g., email format, phone numbers, alphanumeric codes). | `$REGEXP\_LIKE(column, 'regex\_pattern')` | Uses complex RegEx syntax. Highly flexible and powerful, but function name varies by DB. |
| **Case-Insensitive Match** | Perform matching without regard to case. | `$LOWER(column) LIKE LOWER('pattern')` | Standard method: convert both the column and the pattern to lower/upper case. |
| **Extracting Substrings** | Extract a specific part of a string that matches a complex RegEx pattern. | `$REGEXP\_SUBSTR(column, 'regex\_pattern', 1, 1, 'i', 1)` | Uses the RegEx function to return the matching substring or a captured group (syntax varies). |

-----

## Pattern Matching Mini Playbook (Realistic Queries)

These snippets illustrate the distinct power and syntax of both `LIKE` and Regular Expressions.

### 1\. Simple Matching with LIKE

**Use Case:** Find all product codes that start with 'PROD-' and end with a letter, followed by any two characters.

```sql
SELECT
    product_code
FROM
    Products
WHERE
    product_code LIKE 'PROD-_[A-Z]__'; -- '%' and '_' wildcards

-- Matches: PROD-1A01, PROD-9BZZ
-- Does not match: PROD-1100 (ends with two numbers)
```

-----

### 2\. Complex Matching with RegEx (Data Validation)

**Use Case:** Find all rows where the `phone_number` field matches the strict U.S. format: `(XXX) XXX-XXXX`.

```sql
SELECT
    phone_number
FROM
    Contacts
WHERE
    REGEXP_LIKE(phone_number, '^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$');
    
-- RegEx Breakdown:
-- ^: Start of string
-- \([0-9]{3}\): Three digits inside parentheses
-- [0-9]{3}: Three digits
-- -: Literal hyphen
-- $: End of string
```

-----

### 3\. Case-Insensitive Matching

**Use Case:** Find all log messages containing the word "ERROR" regardless of capitalization (e.g., 'Error', 'error', 'ERROR').

```sql
SELECT
    log_message
FROM
    System_Logs
WHERE
    LOWER(log_message) LIKE '%error%';

-- Alternative (RegEx with 'i' flag, if supported):
-- REGEXP_LIKE(log_message, 'error', 'i')
```

-----

### 4\. Extracting Data with RegEx

**Use Case:** Extract the numerical version number (e.g., '3.1.2') from a complex description string like 'App\_Name\_v3.1.2\_Build'.

*Note: This relies on capturing groups `()` within the pattern.*

```sql
SELECT
    full_string,
    REGEXP_SUBSTR(
        full_string,
        'v([0-9]+\.[0-9]+\.[0-9]+)', -- Pattern with capture group
        1,                            -- Start at position 1
        1,                            -- Find the 1st occurrence
        'i',                          -- Case-insensitive
        1                             -- Return the 1st capture group
    ) AS extracted_version
FROM
    Software_Versions;
```

-----

## TRIM, LTRIM, RTRIM Cheat Sheet Table

| Function | Purpose | Solution Pattern | Key Technique |
| :--- | :--- | :--- | :--- |
| **`TRIM`** | Removes characters from **both the beginning and end** of a string. | `$TRIM( [ [LEADING | TRAILING | BOTH] [characters] FROM ] string)$` | The ANSI standard function; allows specifying *which* characters to remove (default is space). |
| **`LTRIM`** | Removes characters only from the **beginning (Left)** of a string. | `$LTRIM(string [, characters])$` | Useful for removing leading spaces, prefixes, or unwanted symbols. |
| **`RTRIM`** | Removes characters only from the **end (Right)** of a string. | `$RTRIM(string [, characters])$` | Useful for removing trailing spaces, line breaks, or suffixes. |
| **Simple Space Removal** | The most common use: removing leading/trailing **whitespace**. | `$TRIM(string)$` | Most databases use this simple syntax for removing only spaces. |

## TRIM, LTRIM, RTRIM Mini Playbook (Realistic Queries)

These snippets illustrate the common use cases for trimming functions, emphasizing general syntax and platform variations.

### 1\. Simple Space Removal (TRIM)

**Use Case:** Clean a user-input field (e.g., `zip_code`) to remove accidental leading or trailing spaces before saving or joining.

```sql
SELECT
    id,
    TRIM(zip_code) AS cleaned_zip
FROM
    Address_Data
WHERE
    TRIM(zip_code) = '90210'; -- Ensures accurate comparison
    
-- Example: TRIM('   12345   ') => '12345'
```

### 2\. LTRIM (Removing Specific Leading Characters)

**Use Case:** Remove the leading currency symbol (`$`) from a price column that was imported as a string.

*Note: The optional `characters` argument works differently across platforms. The general ANSI standard syntax for specifying characters is shown in the pattern.*

```sql
SELECT
    product_id,
    LTRIM(price_string, '$') AS price_without_symbol
FROM
    Products;
    
-- Example: LTRIM('$$125.00', '$') => '125.00'
```

### 3\. RTRIM (Removing Specific Trailing Characters)

**Use Case:** Remove unwanted trailing commas (`,`) from a messy notes field.

```sql
SELECT
    log_id,
    RTRIM(notes_field, ',') AS cleaned_notes
FROM
    System_Logs;
    
-- Example: RTRIM('Warning, Error,,', ',') => 'Warning, Error'
```

### 4\. Advanced TRIM (ANSI Standard)

**Use Case:** Explicitly remove only the trailing asterisk (`*`) characters from a record, while preserving any leading asterisks.

```sql
SELECT
    raw_key,
    TRIM(TRAILING '*' FROM raw_key) AS key_no_suffix
FROM
    Data_Import;

-- Example: TRIM(TRAILING '*' FROM '***ABC**') => '***ABC'
```

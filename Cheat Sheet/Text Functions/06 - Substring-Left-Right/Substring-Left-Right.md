## SUBSTRING/LEFT/RIGHT Cheat Sheet Table

| Function | Purpose | Solution Pattern | Key Technique |
| :--- | :--- | :--- | :--- |
| **`SUBSTRING`** | Extract a segment of a string starting at a specific position for a specific length. | `$SUBSTRING(string, start\_position, length)$` | Ideal for extracting data from fixed-width fields or the middle of a string. |
| **`LEFT`** | Extract a specified number of characters from the **beginning** (left side) of a string. | `$LEFT(string, length)$` | Quick way to get prefixes, codes, or initial characters. |
| **`RIGHT`** | Extract a specified number of characters from the **end** (right side) of a string. | `$RIGHT(string, length)$` | Quick way to get suffixes, extensions, or trailing data. |
| **Finding Position** | Used *with* `SUBSTRING` to dynamically find the starting point of the substring. | `$SUBSTRING(string, INSTR(string, delimiter) + 1, length)$` | Requires a positional function like `INSTR`, `CHARINDEX`, or `POSITION` (syntax varies by DB). |

## SUBSTRING/LEFT/RIGHT Mini Playbook (Realistic Queries)

These snippets illustrate the common and combined use cases for these string functions.

### 1\. SUBSTRING (Extracting Middle Data)

**Use Case:** A product ID is formatted as `YYMMDD-SKU-REGION`. Extract the 4-digit **SKU** which starts at the 8th character and is 4 characters long.

```sql
SELECT
    product_code,
    SUBSTRING(product_code, 8, 4) AS extracted_sku
FROM
    Products;

-- Example: SUBSTRING('240115-A45B-NY', 8, 4) => 'A45B'
```

### 2\. LEFT (Extracting Prefixes)

**Use Case:** Retrieve the first three characters of a part number to identify its manufacturing batch.

```sql
SELECT
    part_number,
    LEFT(part_number, 3) AS batch_prefix
FROM
    Inventory;

-- Example: LEFT('PN-78592', 3) => 'PN-'
```

### 3\. RIGHT (Extracting Suffixes)

**Use Case:** Extract the file extension (last 3 characters) from a file name.

```sql
SELECT
    file_name,
    RIGHT(file_name, 3) AS file_extension
FROM
    Documents;

-- Example: RIGHT('report.pdf', 3) => 'pdf'
```

### 4\. SUBSTRING with Dynamic Position (Extracting Between Delimiters)

**Use Case:** Extract the domain name from an email address (everything after the `@` symbol). This requires finding the position of the `@` first.

*Note: `INSTR` is used conceptually here; the function name varies (e.g., `CHARINDEX` in SQL Server, `STRPOS` in PostgreSQL).*

```sql
SELECT
    email_address,
    SUBSTRING(
        email_address,
        INSTR(email_address, '@') + 1, -- Start position: character after '@'
        LENGTH(email_address)          -- Extract to the end of the string
    ) AS domain_name
FROM
    Users;

-- Example: SUBSTRING('test@example.com', 6, 15) => 'example.com'
```

This cheat sheet focuses on using the **`LIKE`** operator in SQL for pattern matching, utilizing **wildcards** to search for strings that match a specified format. This is essential for flexible text searching and data validation.

## LIKE Patterns and Wildcards Cheat Sheet Table

| Operator | Name | Matches | Key Use Case |
| :--- | :--- | :--- | :--- |
| **`%`** | **Percent Sign** | Matches any sequence of **zero or more** characters. | Finding strings that **start with**, **end with**, or **contain** a specific substring. |
| **`_`** | **Underscore** | Matches any **single character**. | Finding strings of a specific length, or where only one character varies (e.g., specific codes). |
| **`[]`** | **Bracket List** | Matches any single character within the specified set. | Finding strings where a character must be one of several options (e.g., `[aeiou]`). **(SQL Server/MySQL)** |
| **`[^]`** | **Bracket Negation** | Matches any single character **NOT** within the specified set. | Finding strings that explicitly *do not* contain certain characters (e.g., `[^0-9]`). **(SQL Server/MySQL)** |

-----

## LIKE Patterns and Wildcards Mini Playbook (Realistic Queries)

These snippets illustrate the common use of wildcards for various searching needs.

### 1\. Starts With (%)

**Use Case:** Find all customers whose last name begins with the letter 'S'.

```sql
SELECT
    customer_name
FROM
    Customers
WHERE
    last_name LIKE 'S%';
    
-- Matches: Smith, Sanchez, Sterling
-- Does not match: Asmith, Taylor
```

-----

### 2\. Ends With (%)

**Use Case:** Find all documents that are PDF files (i.e., end with '.pdf').

```sql
SELECT
    file_name
FROM
    Documents
WHERE
    file_name LIKE '%.pdf';
    
-- Matches: report.pdf, draft_v2.pdf
-- Does not match: image.jpg
```

-----

### 3\. Contains (%)

**Use Case:** Find all product descriptions that include the word 'waterproof' anywhere in the text.

```sql
SELECT
    product_name,
    description
FROM
    Products
WHERE
    description LIKE '%waterproof%';
    
-- Matches: "durable waterproof coating", "Waterproof and light"
```

-----

### 4\. Specific Length/Position (\_)

**Use Case:** Find all 5-character product codes that start with 'P' and end with 'Z', regardless of the three characters in the middle.

```sql
SELECT
    product_code
FROM
    Products
WHERE
    product_code LIKE 'P___Z';
    
-- Matches: P123Z, PX-AZ
-- Does not match: P12Z, P1234Z
```

-----

### 5\. Combining Wildcards

**Use Case:** Find email addresses from the `gmail.com` domain that have a single-digit number (0-9) as the second character of the local part (e.g., `a1test@gmail.com`).

```sql
SELECT
    email_address
FROM
    Users
WHERE
    email_address LIKE '_[0-9]%@gmail.com';
    
-- SQL Server/MySQL-style bracket usage:
-- Matches: a1xyz@gmail.com, b5_@gmail.com
-- Does not match: ab1@gmail.com, a10@gmail.com
```

-----

### 6\. Escaping Wildcards

**Use Case:** Find products that literally contain the percent sign (`%`) in their name (e.g., "Discount 50%").

*Note: The `ESCAPE` clause must be used to tell SQL that the following character (here, `\`) should be treated as a literal character, not a wildcard.*

```sql
SELECT
    product_name
FROM
    Products
WHERE
    product_name LIKE '%\%%' ESCAPE '\';
    
-- Matches: 'Item 20% Off', '100% Cotton'
-- Does not match: 'Item 20 Off'
```

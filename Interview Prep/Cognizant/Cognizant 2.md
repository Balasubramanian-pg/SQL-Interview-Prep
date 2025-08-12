---
Difficulty: Easy
---
# SQL Interview Question: Extracting Email Domains and Usernames

## Problem Statement

Given an employees table with:

- `id`
- `name`
- `email` (format: username@domain.com)

We need to:

1. Extract the domain name from email addresses
2. Return results with id, name, and domain
3. Bonus: Also extract the username portion

## Solution for Extracting Domains

```SQL
SELECT
    id,
    name,
    SUBSTRING(email FROM POSITION('@' IN email) + 1) AS domain
FROM
    employees;
```

## Solution for Extracting Usernames

```SQL
SELECT
    id,
    name,
    SUBSTRING(email FROM 1 FOR POSITION('@' IN email) - 1) AS username
FROM
    employees;
```

## Explanation

### Domain Extraction

1. `POSITION('@' IN email)` finds the index of '@' character
2. Adding 1 (`+ 1`) starts extraction from the character after '@'
3. Returns everything after '@' (the domain)

### Username Extraction

1. Starts from position 1 (beginning of string)
2. Extracts up to but not including '@' (`1`)
3. Returns everything before '@' (the username)

## Key Concepts

1. **SUBSTRING() Function**:
    - Extracts portions of strings
    - Syntax: `SUBSTRING(string FROM start [FOR length])`
2. **POSITION() Function**:
    - Returns the index of a substring
    - Alternative: `STRPOS(email, '@')` in some databases
3. **String Manipulation**:
    - Common requirement for data cleaning
    - Useful for parsing standardized formats like emails

## Alternative Solutions

### Using SPLIT_PART (PostgreSQL)

```SQL
-- Domain extraction
SELECT
    id,
    name,
    SPLIT_PART(email, '@', 2) AS domain
FROM employees;

-- Username extraction
SELECT
    id,
    name,
    SPLIT_PART(email, '@', 1) AS username
FROM employees;
```

### Using RIGHT/LEFT (SQL Server)

```SQL
-- Domain extraction
SELECT
    id,
    name,
    RIGHT(email, LEN(email) - CHARINDEX('@', email)) AS domain
FROM employees;

-- Username extraction
SELECT
    id,
    name,
    LEFT(email, CHARINDEX('@', email) - 1) AS username
FROM employees;
```

These solutions demonstrate essential string manipulation techniques commonly tested in SQL interviews.
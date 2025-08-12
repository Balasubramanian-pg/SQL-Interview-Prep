---
Difficulty: Easy
---
# SQL Query to Split Full Names into First and Last Names

## Problem Summary

You need to write a SQL query that:

1. Splits a `full_name` column into `first_name` and `last_name` columns
2. Assumes names are separated by single spaces
3. Handles cases with middle names by extracting only the first and last names
4. Returns results with ID, full_name, first_name, and last_name

## Solution Code

```SQL
SELECT
    id,
    full_name,
    -- Extract first name (first part before space)
    SPLIT_PART(full_name, ' ', 1) AS first_name,
    -- Extract last name (last part of the array)
    SPLIT_PART(full_name, ' ',
               ARRAY_LENGTH(STRING_TO_ARRAY(full_name, ' '), 1)
    ) AS last_name
FROM persons;
```

## Code Explanation

1. **SPLIT_PART for First Name**:
    - `SPLIT_PART(full_name, ' ', 1)` splits the full name at each space and returns the first part
    - This reliably gets the first name regardless of how many names follow
2. **SPLIT_PART with ARRAY_LENGTH for Last Name**:
    - `STRING_TO_ARRAY(full_name, ' ')` converts the full name into an array of names split by spaces
    - `ARRAY_LENGTH(...)` counts how many name parts exist
    - Using this count as the position parameter in `SPLIT_PART` ensures we always get the last name
3. **Handling Different Name Formats**:
    - For "John Doe" (2 parts): Returns "John" and "Doe"
    - For "Emily Rose Patrick" (3 parts): Returns "Emily" and "Patrick" (skips middle name "Rose")
    - Works consistently regardless of how many middle names exist

## Alternative Solution (More Readable)

```SQL
WITH name_parts AS (
    SELECT
        id,
        full_name,
        STRING_TO_ARRAY(full_name, ' ') AS parts
    FROM persons
)
SELECT
    id,
    full_name,
    parts[1] AS first_name,
    parts[ARRAY_LENGTH(parts, 1)] AS last_name
FROM name_parts;
```

This solution first creates a CTE that splits all names into arrays, then selects the first and last elements of each array. Both solutions achieve the same result.
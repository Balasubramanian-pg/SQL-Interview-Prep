# SQL Query to Split Full Names into First and Last Names  

## 1. **Problem Summary**  
You need to write a SQL query that:  
1. Splits a `full_name` column into `first_name` and `last_name` columns.  
2. Assumes names are separated by single spaces.  
3. Handles cases with middle names by extracting only the first and last names.  
4. Returns results with `ID`, `full_name`, `first_name`, and `last_name`.  

> [!NOTE]  
> This problem tests your ability to manipulate string data in SQL, particularly using array and split functions.  

---

## 2. **Solution Code**  
```sql
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

---

## 3. **Code Explanation**  

### 3.1 **SPLIT_PART for First Name**  
- **`SPLIT_PART(full_name, ' ', 1)`**:  
  - Splits the `full_name` at each space and returns the first part.  
  - Reliably extracts the `first_name` regardless of how many names follow.  

> [!TIP]  
> `SPLIT_PART` is a PostgreSQL-specific function. For other databases, use equivalent string functions like `SUBSTRING` or `REGEXP_SUBSTR`.  

### 3.2 **SPLIT_PART with ARRAY_LENGTH for Last Name**  
- **`STRING_TO_ARRAY(full_name, ' ')`**: Converts the `full_name` into an array of names split by spaces.  
- **`ARRAY_LENGTH(...)`**: Counts how many name parts exist.  
- Using this count as the position parameter in `SPLIT_PART` ensures we always get the `last_name`.  

> [!IMPORTANT]  
> This method dynamically handles names with any number of middle names by targeting the last element of the array.  

### 3.3 **Handling Different Name Formats**  
- **"John Doe" (2 parts)**: Returns "John" and "Doe".  
- **"Emily Rose Patrick" (3 parts)**: Returns "Emily" and "Patrick" (skips middle name "Rose").  
- Works consistently regardless of how many middle names exist.  

> [!NOTE]  
> This solution assumes names are separated by single spaces and does not handle suffixes or prefixes (e.g., "John Jr.").  

---

## 4. **Alternative Solution (More Readable)**  
```sql
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

> [!TIP]  
> This version uses a Common Table Expression (CTE) to split names into arrays first, making the logic easier to follow.  

---

## 5. **Key Concepts Demonstrated**  

### 5.1 **String Manipulation Functions**  
- **`SPLIT_PART`**: Splits a string at a delimiter and returns a specific part.  
- **`STRING_TO_ARRAY`**: Converts a string into an array based on a delimiter.  
- **`ARRAY_LENGTH`**: Returns the number of elements in an array.  

> [!IMPORTANT]  
> These functions are essential for parsing and transforming string data in SQL.  

### 5.2 **Handling Complex Data**  
- **Dynamic Extraction**: Adapts to varying numbers of name parts.  
- **Readability**: Uses CTEs to break down complex logic into manageable steps.  

> [!NOTE]  
> While this solution is written for PostgreSQL, similar logic can be implemented in other databases using their respective string and array functions.  

---

## 6. **Additional Notes**  
- **Performance**: Both solutions are efficient for moderate-sized datasets. For very large tables, consider indexing or optimizing further.  
- **Edge Cases**: Test with names containing hyphens, suffixes, or non-standard formats to ensure robustness.  

> [!WARNING]  
> This query assumes consistent name formatting. Irregular formats (e.g., "Van Gogh" or "Mary-Kate") may require additional handling.  

---

This solution efficiently splits full names into first and last names while handling middle names gracefully. It demonstrates the use of array and string functions in SQL, a common requirement for data cleaning and transformation tasks.  

> [!TIP]  
> Practice this pattern with different datasets to reinforce your understanding of string manipulation and array functions in SQL.  

---

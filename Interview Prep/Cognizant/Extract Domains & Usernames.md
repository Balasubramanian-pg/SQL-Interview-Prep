# SQL Interview Question: Extracting Email Domains and Usernames  

## 1. **Problem Statement**  
Given an `employees` table with:  
- `id`  
- `name`  
- `email` (format: `username@domain.com`)  

We need to:  
1. Extract the domain name from email addresses.  
2. Return results with `id`, `name`, and `domain`.  
3. **Bonus**: Also extract the username portion.  

---

## 2. **Solution for Extracting Domains**  
```sql
SELECT
    id,
    name,
    SUBSTRING(email FROM POSITION('@' IN email) + 1) AS domain
FROM
    employees;
```  

> [!TIP]  
> This solution works in databases like PostgreSQL and MySQL that support `SUBSTRING` and `POSITION`.  

---

## 3. **Solution for Extracting Usernames**  
```sql
SELECT
    id,
    name,
    SUBSTRING(email FROM 1 FOR POSITION('@' IN email) - 1) AS username
FROM
    employees;
```  

> [!NOTE]  
> The `FOR` clause in `SUBSTRING` ensures we only extract characters up to (but not including) the `@` symbol.  

---

## 4. **Explanation**  

### 4.1 **Domain Extraction**  
1. `POSITION('@' IN email)` finds the index of the `@` character.  
2. Adding `1` (`+ 1`) starts extraction from the character after `@`.  
3. Returns everything after `@` (the domain).  

### 4.2 **Username Extraction**  
1. Starts from position `1` (beginning of the string).  
2. Extracts up to but not including `@` (`FOR POSITION('@' IN email) - 1`).  
3. Returns everything before `@` (the username).  

---

## 5. **Key Concepts**  

### 5.1 **SUBSTRING() Function**  
- Extracts portions of strings.  
- Syntax: `SUBSTRING(string FROM start [FOR length])`.  

### 5.2 **POSITION() Function**  
- Returns the index of a substring.  
- Alternative: `STRPOS(email, '@')` in some databases.  

### 5.3 **String Manipulation**  
- Common requirement for data cleaning.  
- Useful for parsing standardized formats like emails.  

> [!IMPORTANT]  
> Understanding string functions is crucial for handling text data in SQL, especially in data preprocessing tasks.  

---

## 6. **Alternative Solutions**  

### 6.1 **Using SPLIT_PART (PostgreSQL)**  
```sql
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

> [!TIP]  
> `SPLIT_PART` is a PostgreSQL-specific function that simplifies splitting strings by a delimiter.  

### 6.2 **Using RIGHT/LEFT (SQL Server)**  
```sql
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

> [!NOTE]  
> `RIGHT` and `LEFT` combined with `CHARINDEX` are commonly used in SQL Server for string manipulation.  

---

These solutions demonstrate essential string manipulation techniques commonly tested in SQL interviews.  

> [!WARNING]  
> Be aware of database-specific functions and ensure compatibility with the system you're using.

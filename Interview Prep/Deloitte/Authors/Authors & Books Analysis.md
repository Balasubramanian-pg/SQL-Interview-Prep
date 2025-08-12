# Authors and Books Analysis  

### 1.List All Authors and Number of Books Written**  

### 1.1 **Solution**  
```sql
SELECT
    a.author_name,
    COUNT(b.book_id) AS number_of_books
FROM
    authors a
LEFT JOIN
    books b ON a.author_id = b.author_id
GROUP BY
    a.author_name;
```  

### 1.2 **Explanation**  
1. **LEFT JOIN**: Ensures all authors are included, even if they haven't written any books.  
2. **COUNT(b.book_id)**: Counts the books for each author.  
3. **GROUP BY author_name**: Groups the results by each author.  
4. Returns author names with their respective book counts.  

> [!TIP]  
> Using `LEFT JOIN` ensures no authors are excluded, even if they have zero books.  

---

## 2. **Problem 2: Find Books Published Last Year with Authors**  

### 2.1 **Solution**  
```sql
SELECT
    b.book_title,
    a.author_name
FROM
    books b
JOIN
    authors a ON b.author_id = a.author_id
WHERE
    EXTRACT(YEAR FROM b.publication_year) = EXTRACT(YEAR FROM CURRENT_DATE) - 1;
```  

### 2.2 **Explanation**  
1. **JOIN**: Connects books with their authors.  
2. **EXTRACT(YEAR FROM publication_year)**: Gets the publication year.  
3. **EXTRACT(YEAR FROM CURRENT_DATE) - 1**: Calculates last year.  
4. Returns book titles and author names for books published last year.  

> [!NOTE]  
> This query uses date functions to filter records based on the publication year relative to the current date.  

---

## 3. **Key Concepts Used**  

### 3.1 **JOIN Operations**  
- **LEFT JOIN**: To include all authors, even those without books.  
- **INNER JOIN** (default JOIN): To match books with authors.  

### 3.2 **Aggregation**  
- **COUNT()** with **GROUP BY**: To calculate book counts per author.  

### 3.3 **Date Functions**  
- **EXTRACT()**: To get year components.  
- **CURRENT_DATE**: To reference the current date.  

### 3.4 **Table Aliases**  
- Using `'a'` for `authors` and `'b'` for `books` for cleaner queries.  

> [!IMPORTANT]  
> Table aliases make queries more readable and concise, especially when working with multiple tables.  

---

These queries demonstrate common patterns for reporting on relational data in SQL interviews, particularly when working with author-book relationships and temporal filtering.  

> [!TIP]  
> Practice writing queries that combine joins, aggregation, and date functions to solve real-world data analysis problems.  

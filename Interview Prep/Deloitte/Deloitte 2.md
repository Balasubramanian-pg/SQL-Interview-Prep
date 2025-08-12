---
Difficulty: Easy
---
# SQL Interview Questions: Authors and Books Analysis

## Problem 1: List All Authors and Number of Books Written

### Solution

```SQL
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

### Explanation

1. **LEFT JOIN** ensures all authors are included, even if they haven't written any books
2. **COUNT(b.book_id)** counts the books for each author
3. **GROUP BY author_name** groups the results by each author
4. Returns author names with their respective book counts

## Problem 2: Find Books Published Last Year with Authors

### Solution

```SQL
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

### Explanation

1. **JOIN** connects books with their authors
2. **EXTRACT(YEAR FROM publication_year)** gets the publication year
3. **EXTRACT(YEAR FROM CURRENT_DATE) - 1** calculates last year
4. Returns book titles and author names for books published last year

## Key Concepts Used

1. **JOIN Operations**:
    - LEFT JOIN to include all authors
    - INNER JOIN (default JOIN) to match books with authors
2. **Aggregation**:
    - COUNT() with GROUP BY to calculate book counts per author
3. **Date Functions**:
    - EXTRACT() to get year components
    - CURRENT_DATE to reference the current date
4. **Table Aliases**:
    - Using 'a' for authors and 'b' for books for cleaner queries

These queries demonstrate common patterns for reporting on relational data in SQL interviews, particularly when working with author-book relationships and temporal filtering.
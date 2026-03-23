Here's the complete `INSERT INTO` query to populate the `authors` and `books` tables for testing author-book analysis scenarios:

```sql
-- Create authors table
CREATE TABLE IF NOT EXISTS authors (
    author_id INT PRIMARY KEY,
    author_name VARCHAR(100),
    country VARCHAR(50)
);

-- Create books table
CREATE TABLE IF NOT EXISTS books (
    book_id INT PRIMARY KEY,
    book_title VARCHAR(200),
    author_id INT,
    publication_year DATE,
    genre VARCHAR(50),
    FOREIGN KEY (author_id) REFERENCES authors(author_id)
);

-- Clear existing data
TRUNCATE TABLE authors;
TRUNCATE TABLE books;

-- Insert sample authors
INSERT INTO authors (author_id, author_name, country) VALUES
(1, 'J.K. Rowling', 'UK'),
(2, 'George R.R. Martin', 'USA'),
(3, 'Agatha Christie', 'UK'),
(4, 'Haruki Murakami', 'Japan'),
(5, 'Stephen King', 'USA'),
(6, 'Jane Austen', 'UK'),
(7, 'Ernest Hemingway', 'USA'),
(8, 'New Author', 'Canada');  -- Author with no books

-- Insert sample books with various publication years
INSERT INTO books (book_id, book_title, author_id, publication_year, genre) VALUES
-- J.K. Rowling books
(101, 'Harry Potter and the Philosopher''s Stone', 1, '1997-06-26', 'Fantasy'),
(102, 'Harry Potter and the Chamber of Secrets', 1, '1998-07-02', 'Fantasy'),
(103, 'The Casual Vacancy', 1, '2012-09-27', 'Fiction'),

-- George R.R. Martin books
(201, 'A Game of Thrones', 2, '1996-08-01', 'Fantasy'),
(202, 'A Clash of Kings', 2, '1998-11-16', 'Fantasy'),

-- Agatha Christie books
(301, 'Murder on the Orient Express', 3, '1934-01-01', 'Mystery'),
(302, 'Death on the Nile', 3, '1937-11-01', 'Mystery'),

-- Haruki Murakami books
(401, 'Norwegian Wood', 4, '1987-09-04', 'Fiction'),
(402, 'Kafka on the Shore', 4, '2002-09-12', 'Magical Realism'),

-- Stephen King books (including last year's publications)
(501, 'The Shining', 5, '1977-01-28', 'Horror'),
(502, 'It', 5, '1986-09-15', 'Horror'),
(503, 'Fairy Tale', 5, DATEADD(YEAR, -1, CURRENT_DATE), 'Fantasy'),  -- Published last year
(504, 'Billy Summers', 5, DATEADD(YEAR, -1, CURRENT_DATE), 'Thriller'),  -- Published last year

-- Jane Austen books (older publications)
(601, 'Pride and Prejudice', 6, '1813-01-28', 'Romance'),
(602, 'Emma', 6, '1815-12-23', 'Romance'),

-- Ernest Hemingway books
(701, 'The Old Man and the Sea', 7, '1952-09-01', 'Literary Fiction');
```

To test the author-book count query:

```sql
-- Problem 1: List all authors and number of books written
SELECT
    a.author_name,
    a.country,
    COUNT(b.book_id) AS number_of_books,
    CASE 
        WHEN COUNT(b.book_id) = 0 THEN 'No books'
        WHEN COUNT(b.book_id) = 1 THEN '1 book'
        ELSE CONCAT(COUNT(b.book_id), ' books')
    END AS book_count_description
FROM
    authors a
LEFT JOIN
    books b ON a.author_id = b.author_id
GROUP BY
    a.author_name, a.country
ORDER BY
    number_of_books DESC, a.author_name;
```

To test the books published last year query:

```sql
-- Problem 2: Find books published last year with authors
SELECT
    b.book_title,
    a.author_name,
    b.genre,
    YEAR(b.publication_year) AS publication_year
FROM
    books b
JOIN
    authors a ON b.author_id = a.author_id
WHERE
    YEAR(b.publication_year) = YEAR(CURRENT_DATE) - 1
ORDER BY
    a.author_name, b.book_title;
```

Key features of this test data:
1. **Authors with varying book counts** (from 0 to multiple books)
2. **Books published in different years** (historical and recent)
3. **Special case for last year's publications** (using DATEADD for dynamic calculation)
4. **Multiple genres** to test additional filtering capabilities
5. **International authors** from different countries
6. **One author with no books** to test LEFT JOIN behavior

The solutions should:
- Correctly count books per author (including zero for authors with no books)
- Properly identify books published in the previous calendar year
- Handle the dynamic nature of "last year" (will automatically adjust each year)
- Maintain proper relationships between authors and books
- Present results in a readable, organized format

Additional queries you might want to test:

```sql
-- Find authors who haven't published in the last 5 years
SELECT 
    a.author_name,
    MAX(YEAR(b.publication_year)) AS last_published_year
FROM 
    authors a
LEFT JOIN 
    books b ON a.author_id = b.author_id
GROUP BY 
    a.author_name
HAVING 
    MAX(YEAR(b.publication_year)) < YEAR(CURRENT_DATE) - 5
    OR MAX(b.publication_year) IS NULL
ORDER BY 
    last_published_year;

-- Count books by genre and country
SELECT 
    b.genre,
    a.country,
    COUNT(*) AS book_count
FROM 
    books b
JOIN 
    authors a ON b.author_id = a.author_id
GROUP BY 
    b.genre, a.country
ORDER BY 
    book_count DESC;
```

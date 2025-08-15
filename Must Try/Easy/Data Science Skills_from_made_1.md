---
Company:
  - Amazon
Created:
Difficulty:
Status:
Category:
Sub category:
Question Link:
---

Of course! This query is a very interesting way to solve a common "checklist" problem. It's a perfect candidate for analysis, and the alternative method is quite a bit more intuitive.

Here is the complete breakdown.

***

### 1. The Question

The business or data question being asked by this SQL query is:

**"Which candidates have all three of the required skills: 'Python', 'Tableau', and 'PostgreSQL'? List their candidate IDs."**

---

### 2. Table Schema

The query references a single table, `candidates`, which appears to store skills in a "long" or "normalized" format (one row per skill per candidate). A plausible schema would be:

```sql
-- Table: candidates
-- Stores a list of skills for each candidate.
CREATE TABLE candidates (
    candidate_id INT NOT NULL,       -- The ID of the candidate
    skill        VARCHAR(100) NOT NULL -- The name of a skill they possess
);

-- A composite primary key would be appropriate to ensure a candidate doesn't have the same skill listed twice.
ALTER TABLE candidates ADD PRIMARY KEY (candidate_id, skill);

-- An index on the skill column would help performance.
CREATE INDEX idx_candidates_skill ON candidates (skill);
```

---

### 3. Structured SQL Query (Method 1: `SUM` of `CASE` Statements)

This is your original query, formatted for clarity. It's a clever but somewhat unconventional approach.

```sql
SELECT
    candidate_id
FROM
    candidates
GROUP BY
    candidate_id
HAVING
    SUM(
        (CASE WHEN skill = 'Python' THEN 1 ELSE 0 END) +
        (CASE WHEN skill = 'Tableau' THEN 1 ELSE 0 END) +
        (CASE WHEN skill = 'PostgreSQL' THEN 1 ELSE 0 END)
    ) = 3
ORDER BY
    candidate_id ASC;
```

---

### 4. Explanation of the Query

This query works by assigning a "point" for each desired skill and then summing those points for each candidate.

1.  **`FROM candidates`**: The query starts by reading from the `candidates` table.
2.  **`GROUP BY candidate_id`**: This is a crucial step that groups all rows belonging to the same `candidate_id` together. The subsequent `HAVING` clause will operate on each of these groups.
3.  **`HAVING SUM(...) = 3`**: This is the filtering logic that is applied to each candidate's group of skills.
    *   **The Inner Expression**: `(CASE WHEN...) + (CASE WHEN...) + (CASE WHEN...)` is evaluated for **every single row** within a candidate's group.
        *   If a row has the skill 'Python', the expression becomes `1 + 0 + 0`, which equals `1`.
        *   If a row has the skill 'Tableau', it becomes `0 + 1 + 0`, which equals `1`.
        *   If a row has 'PostgreSQL', it's `0 + 0 + 1`, which equals `1`.
        *   If a row has an irrelevant skill like 'R', it becomes `0 + 0 + 0`, which equals `0`.
    *   **`SUM(...)`**: The `SUM` function then adds up these results (mostly `1`s and `0`s) for all rows in the candidate's group.
    *   **`= 3`**: For the total sum to be exactly `3`, the candidate must have had one row that produced a `1` for Python, one row that produced a `1` for Tableau, and one row that produced a `1` for PostgreSQL.

4.  **`SELECT candidate_id` and `ORDER BY`**: Finally, it selects the IDs of the candidates whose groups passed the `HAVING` clause filter and sorts them.

---

### 5. Another SQL Method (Method 2: Using `COUNT(DISTINCT)` with a `WHERE` Clause)

A much more common, readable, and often more performant way to solve this problem is to filter for the relevant skills first and then count them.

```sql
SELECT
    candidate_id
FROM
    candidates
WHERE
    skill IN ('Python', 'Tableau', 'PostgreSQL')
GROUP BY
    candidate_id
HAVING
    COUNT(DISTINCT skill) = 3
ORDER BY
    candidate_id ASC;
```

---

### 6. Explanation of the Alternative Query

This method is more direct and easier to understand.

1.  **`FROM candidates`**: Starts with the same table.
2.  **`WHERE skill IN ('Python', 'Tableau', 'PostgreSQL')`**: This is a key difference. The query first **filters the entire dataset**, throwing away all rows that don't contain one of our three target skills. This is efficient because the subsequent grouping operation has much less data to process.
3.  **`GROUP BY candidate_id`**: It then groups the remaining rows (only those with the three relevant skills) by candidate.
4.  **`HAVING COUNT(DISTINCT skill) = 3`**: This is a very clear and elegant condition.
    *   For each candidate's group, `COUNT(DISTINCT skill)` counts the number of unique skills.
    *   If a candidate has 'Python', 'Tableau', and 'PostgreSQL' in their filtered list of skills, the count of distinct skills will be exactly `3`.
    *   If a candidate only has 'Python' and 'Tableau' from the list, their count will be `2`, and they will be filtered out.
5.  **`SELECT` and `ORDER BY`**: These clauses work the same as before, selecting and ordering the final list of qualifying candidate IDs.

This second method is generally preferred for its readability and performance benefits from early filtering.
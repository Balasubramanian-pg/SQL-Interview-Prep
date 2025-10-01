# Top Two Subjects by Student  

## 1. **Problem Statement**  
Given a `students` table with:  
- `student_name`  
- `subject`  
- `marks`  

We need to:  
1. For each student, identify their top two highest-scoring subjects.  
2. Calculate the total marks from these top two subjects.  
3. Return results with student names and their total marks.  

> [!NOTE]  
> This problem tests your ability to use window functions, filtering, and aggregation in SQL.  

## 2. **Solution 1: Using Subquery with ROW_NUMBER()**  
```sql
SELECT
    student_name,
    SUM(marks) AS total_marks
FROM (
    SELECT
        student_name,
        marks,
        ROW_NUMBER() OVER (
            PARTITION BY student_name
            ORDER BY marks DESC
        ) AS row_num
    FROM students
) ranked_students
WHERE row_num <= 2
GROUP BY student_name;
```  

> [!TIP]  
> The subquery uses `ROW_NUMBER()` to rank subjects within each student's partition.  

> [!IMPORTANT]  
> `PARTITION BY student_name` ensures ranking is done separately for each student.  

> [!NOTE]  
> `ORDER BY marks DESC` sorts subjects in descending order of marks.  

> [!TIP]  
> `WHERE row_num <= 2` filters to keep only the top two subjects per student.  

> [!IMPORTANT]  
> `SUM(marks)` aggregates the marks of the top two subjects for each student.  

## 3. **Solution 2: Using Common Table Expression (CTE)**  
```sql
WITH ranked_students AS (
    SELECT
        student_name,
        marks,
        ROW_NUMBER() OVER (
            PARTITION BY student_name
            ORDER BY marks DESC
        ) AS row_num
    FROM students
)
SELECT
    student_name,
    SUM(marks) AS total_marks
FROM ranked_students
WHERE row_num <= 2
GROUP BY student_name;
```  

> [!TIP]  
> The CTE (`ranked_students`) encapsulates the ranking logic, making the main query cleaner.  

> [!IMPORTANT]  
> `ROW_NUMBER()` works the same way as in Solution 1, ranking subjects per student.  

> [!NOTE]  
> `WHERE row_num <= 2` filters the top two subjects per student.  

> [!TIP]  
> `SUM(marks)` calculates the total marks of the top two subjects.  

## 4. **Key Concepts Explained**  

### 4.1 **ROW_NUMBER() Window Function**  
- **Purpose**: Assigns sequential numbers to rows within each partition.  
- **Key Components**:  
  - `PARTITION BY student_name`: Creates groups for each student.  
  - `ORDER BY marks DESC`: Sorts marks in descending order within each group.  

> [!IMPORTANT]  
> `ROW_NUMBER()` is essential for ranking data within groups.  

### 4.2 **Filtering Top Two Rows**  
- **`WHERE row_num <= 2`**: Keeps only the top two subjects per student.  
- **Edge Case**: For students with fewer than two subjects (like Daniel), it returns all available.  

> [!NOTE]  
> This filter ensures only the top two rows are considered for aggregation.  

### 4.3 **Aggregation**  
- **`SUM(marks)`**: Calculates the total of the top two marks.  
- **`GROUP BY student_name`**: Ensures we get one row per student.  

> [!TIP]  
> Aggregation is used to combine the marks of the top two subjects.  

### 4.4 **Approach Comparison**  
- **Both Solutions**: Produce identical results.  
- **CTE Approach**: Often more readable for complex queries.  
- **Subquery Approach**: May be preferred for simpler cases.  

> [!IMPORTANT]  
> Choose the approach based on query complexity and readability.  

## 5. **Additional Notes**  
- **Handling Ties**: If two subjects have the same marks, `ROW_NUMBER()` will assign different ranks based on the order of rows in the table.  
- **Performance**: Both solutions are efficient for moderate-sized datasets. For very large tables, consider indexing the `student_name` and `marks` columns.  

> [!WARNING]  
> Ensure the `marks` column is properly indexed if the table is large to improve performance.  

This solution demonstrates common SQL patterns for:  
- Ranking data within groups  
- Filtering based on rankings  
- Aggregating filtered results  
- Handling edge cases (students with fewer than two subjects)  

> [!TIP]  
> Practice this pattern with different datasets to reinforce your understanding of window functions and aggregation in SQL.  

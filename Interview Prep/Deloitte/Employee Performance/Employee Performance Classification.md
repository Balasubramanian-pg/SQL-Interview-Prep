# Employee Performance Classification  

## 1. **Problem Statement**  
Given an `employees` table with:  
- `employee_id`  
- `name`  
- `performance_rating` (values: "excellent", "good", "average", etc.)  
- `salary`  

We need to classify employees into three performance levels:  
1. **Top Performer**: "excellent" rating **AND** salary > 7000  
2. **Good Performer**: "good" rating **AND** salary > 5000  
3. **Others**: All remaining cases  

> [!NOTE]  
> This problem tests your ability to use conditional logic in SQL for data categorization.  

---

## 2. **Solution Code**  
```sql
SELECT
    employee_id,
    name,
    performance_rating,
    salary,
    CASE
        WHEN performance_rating = 'excellent' AND salary > 7000 THEN 'Top Performer'
        WHEN performance_rating = 'good' AND salary > 5000 THEN 'Good Performer'
        ELSE 'Others'
    END AS performance_level
FROM employees;
```  

---

## 3. **Explanation**  

### 3.1 **CASE Statement Structure**  
- **Evaluates conditions in order**: Checks each `WHEN` condition sequentially.  
- **Returns the first matching result**: Stops evaluation after the first `TRUE` condition.  
- **`ELSE` catches all remaining cases**: Ensures every row gets a classification.  

> [!TIP]  
> The order of `WHEN` clauses matters. Place more specific conditions first.  

### 3.2 **Classification Logic**  
- **Top Performer**: Must meet **BOTH** conditions (`excellent` rating **AND** salary > 7000).  
- **Good Performer**: Must meet **BOTH** conditions (`good` rating **AND** salary > 5000).  
- **Others**: All remaining employees fall into this category.  

> [!IMPORTANT]  
> The `AND` operator ensures both criteria are met for top and good performers.  

### 3.3 **Result Column**  
- **Creates a new column** `performance_level`.  
- **Contains the classification string**: Based on the evaluated conditions.  

> [!NOTE]  
> The original columns (`employee_id`, `name`, etc.) are retained alongside the new `performance_level`.  


## 4. **Key Concepts Demonstrated**  

### 4.1 **Conditional Logic in SQL**  
- **Using `CASE WHEN`**: For row-by-row evaluation.  
- **Handling multiple conditions**: With `AND` to combine criteria.  

> [!TIP]  
> `CASE` statements are ideal for creating custom categories based on complex conditions.  

### 4.2 **Data Categorization**  
- **Creating meaningful business categories**: Based on performance and salary.  
- **Combining multiple criteria**: For precise classification.  

> [!IMPORTANT]  
> This approach is widely used in HR analytics for performance segmentation.  

### 4.3 **Result Presentation**  
- **Adding a derived column**: To the output for clarity.  
- **Maintaining all original columns**: Alongside the new classification.  

> [!NOTE]  
> Derived columns like `performance_level` enhance the usability of query results.  


## 5. **Additional Notes**  
- **Scalability**: This solution works efficiently for large datasets as `CASE` statements are optimized in most SQL databases.  
- **Flexibility**: Easily adaptable to different classification criteria by modifying `WHEN` conditions.  

> [!WARNING]  
> Ensure the `performance_rating` values are consistent (e.g., all lowercase or uppercase) to avoid mismatches.  


This solution efficiently categorizes employees based on multiple criteria, a common requirement for HR analytics and performance reporting systems. The `CASE` statement is a fundamental SQL tool for such conditional transformations.  

> [!TIP]  
> Practice modifying the conditions in the `CASE` statement to handle different business rules and improve your SQL logic skills.  

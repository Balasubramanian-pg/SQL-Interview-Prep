# SQL Problem: Create Performance Category Column

**Question:**
>[!Tip]
> How do I create a column called *performance* in my sales table that shows "High" if `sales_amount > 20` and "Low" otherwise?



## Solution Using CASE Expression

*   **SELECT Query Approach**
> [!TIP]
> Use CASE expression to create a computed column in your query results.
```sql
SELECT 
    sk_id,
    cust_id, 
    sales_amount,
    CASE 
        WHEN sales_amount > 20 THEN 'High'
        ELSE 'Low'
    END AS performance
FROM sales_table;
```

*   **Permanent Column Approach**
> [!NOTE]
> To add this as a permanent computed column to your table structure:
```sql
ALTER TABLE sales_table
ADD COLUMN performance VARCHAR(10)
GENERATED ALWAYS AS (
    CASE 
        WHEN sales_amount > 20 THEN 'High'
        ELSE 'Low'
    END
) STORED;
```

---

## Key Differences & Use Cases

*   **SELECT Query Method**
> [!TIP]
> Best for reports and temporary analysis - doesn't modify table structure.
*   Returns the performance column only in query results
*   No permanent changes to database schema
*   Flexible for one-time analysis and reporting
*   Can be used in views

*   **ALTER TABLE Method**
> [!CAUTION]
> Permanently modifies table structure - use when you need persistent categorization.
*   Adds physical column to table definition
*   Automatically updates when sales_amount changes
*   Available to all queries without rewriting CASE logic
*   Better for frequently accessed business rules

---

## Advanced CASE Expression Variations

*   **Multiple Conditions**
```sql
SELECT 
    sales_amount,
    CASE 
        WHEN sales_amount > 50 THEN 'Excellent'
        WHEN sales_amount > 20 THEN 'High' 
        WHEN sales_amount > 10 THEN 'Medium'
        ELSE 'Low'
    END AS performance_tier
FROM sales_table;
```

*   **Using BETWEEN for Ranges**
```sql
SELECT 
    sales_amount,
    CASE 
        WHEN sales_amount BETWEEN 21 AND 50 THEN 'High'
        WHEN sales_amount BETWEEN 11 AND 20 THEN 'Medium'
        ELSE 'Low'
    END AS performance
FROM sales_table;
```

---

## Performance Considerations

*   **Indexing Computed Columns**
> [!IMPORTANT]
> For better performance with permanent computed columns, consider adding indexes.
```sql
-- Add index for faster filtering on performance column
CREATE INDEX idx_sales_performance ON sales_table(performance);

-- Now this query will be faster:
SELECT * FROM sales_table WHERE performance = 'High';
```

*   **NULL Handling**
```sql
-- Explicit NULL handling
SELECT 
    sales_amount,
    CASE 
        WHEN sales_amount > 20 THEN 'High'
        WHEN sales_amount IS NULL THEN 'Unknown'
        ELSE 'Low'
    END AS performance
FROM sales_table;
```

---

## Common Mistakes to Avoid

*   **Missing ELSE Clause**
> [!WARNING]
> Without ELSE, unmatched conditions return NULL instead of 'Low'.
```sql
-- ❌ RISKY: Returns NULL for sales_amount <= 20
CASE 
    WHEN sales_amount > 20 THEN 'High'
END AS performance

-- ✅ BETTER: Explicitly handle all cases  
CASE 
    WHEN sales_amount > 20 THEN 'High'
    ELSE 'Low'
END AS performance
```

*   **Incorrect Operator Precedence**
```sql
-- ❌ CONFUSING: Hard to read complex conditions
CASE WHEN sales_amount > 20 AND department = 'Sales' OR region = 'North' THEN 'High' ELSE 'Low' END

-- ✅ CLEAR: Use parentheses for complex logic
CASE 
    WHEN (sales_amount > 20 AND department = 'Sales') OR region = 'North' 
    THEN 'High' 
    ELSE 'Low'
END
```

---

## Real-World Application

*   **Business Reporting Example**
```sql
-- Complete sales performance report
SELECT 
    sales_rep,
    region,
    sales_amount,
    CASE 
        WHEN sales_amount > 20 THEN 'High'
        ELSE 'Low'
    END AS performance,
    CASE 
        WHEN sales_amount > 20 THEN 'Eligible for Bonus'
        ELSE 'Standard Commission'
    END AS compensation_tier
FROM sales_table
WHERE sales_date >= '2024-01-01'
ORDER BY performance, sales_amount DESC;
```

Pivot-style transformations, often referred to as **conditional aggregation**, use the **`CASE`** expression inside an aggregate function (usually `SUM` or `COUNT`) to rotate data. This transforms unique values from a single categorical column (e.g., 'Status') into multiple output columns, summarizing the data for each category.

This technique is superior to the dedicated `PIVOT` operator in many SQL dialects because it is more portable and allows for flexible, multi-level aggregation in a single pass.

## Pivot Transformation Technique

The core pattern involves three steps:

1.  **Define the Categories:** Use the `CASE` statement to check the value of the categorical column.
2.  **Aggregate Conditionally:** Use `SUM` or `COUNT` around the `CASE` statement. When the condition is met (`THEN`), return the value to be summed/counted; otherwise, return `0` or `NULL`.
3.  **Group:** Use `GROUP BY` on the remaining descriptive columns.

### Core Pattern

| Goal | Pattern |
| :--- | :--- |
| **Pivoting Amounts (SUM)** | `SUM(CASE WHEN [Category] = 'Value_A' THEN [Amount_Column] ELSE 0 END) AS [Column_A]` |
| **Pivoting Counts (COUNT)** | `SUM(CASE WHEN [Category] = 'Value_B' THEN 1 ELSE 0 END) AS [Count_Column_B]` |

-----

## Pivot-Style Mini Playbook (Realistic Query)

### Scenario: Pivoting Order Statuses

Assume an `Orders` table with columns: `region`, `order_amount`, and `status` ('Pending', 'Shipped', 'Cancelled').

**Goal:** For each geographical **region**, show the total sales amount split into three new columns based on the order status.

```sql
SELECT
    region,
    
    -- 1. Total sales for PENDING orders
    SUM(
        CASE WHEN status = 'Pending' THEN order_amount ELSE 0 END
    ) AS Pending_Sales,
    
    -- 2. Total sales for SHIPPED orders
    SUM(
        CASE WHEN status = 'Shipped' THEN order_amount ELSE 0 END
    ) AS Shipped_Sales,
    
    -- 3. Total sales for CANCELLED orders
    SUM(
        CASE WHEN status = 'Cancelled' THEN order_amount ELSE 0 END
    ) AS Cancelled_Sales,
    
    -- 4. Count of Shipped Orders (Conditional COUNT)
    SUM(
        CASE WHEN status = 'Shipped' THEN 1 ELSE 0 END
    ) AS Shipped_Order_Count
    
FROM
    Orders
GROUP BY
    region
ORDER BY
    region;
```

### Result (Conceptual)

| region | Pending\_Sales | Shipped\_Sales | Cancelled\_Sales | Shipped\_Order\_Count |
| :--- | :--- | :--- | :--- | :--- |
| East | 500.00 | 8000.00 | 120.00 | 45 |
| West | 0.00 | 9500.00 | 0.00 | 60 |
| Central | 100.00 | 4500.00 | 50.00 | 30 |

-----

## Key Considerations

  * **NULL vs. 0:** In the `SUM` pattern, using `ELSE 0` ensures that the `SUM` function adds zero for non-matching rows. If you use `ELSE NULL` (which is the default when `ELSE` is omitted), the `SUM` function simply ignores the row, but the result remains the same since $X + \text{NULL} = X$. Using `ELSE 0` is generally clearer.
  * **Aliasing is Required:** Every conditional aggregation column must be given a unique, descriptive alias (`Pending_Sales`, `Shipped_Sales`, etc.).
  * **Flexibility:** This technique allows you to use complex conditions beyond simple equality, such as ranges (`WHEN amount > 100 THEN ...`), making it far more flexible than built-in `PIVOT` functions in some databases.

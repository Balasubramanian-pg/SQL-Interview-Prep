A manual pivot in SQL is achieved through **conditional aggregation**, which uses the **`CASE` expression inside an aggregate function** (usually `SUM` or `COUNT`). This technique rotates unique values from a single categorical column into multiple output columns, summarizing the data for each category.

This is often preferred over database-specific `PIVOT` operators because it is portable across all major SQL databases and allows for more flexible logic.

## Manual Pivot with CASE: The Technique

The core pattern involves defining the new columns using a `CASE` statement within an aggregate function, and then grouping the data by the remaining descriptive columns.

### Core Pivot Pattern

| Goal | Pattern |
| :--- | :--- |
| **Pivoting Amounts (SUM)** | `SUM(CASE WHEN [Category] = 'Value_A' THEN [Amount_Column] ELSE 0 END) AS [Column_A]` |
| **Pivoting Counts (COUNT)** | `SUM(CASE WHEN [Category] = 'Value_B' THEN 1 ELSE 0 END) AS [Count_Column_B]` |

### Example Scenario: Pivoting Sales Statuses

Assume a table, `DailySales`, with columns: `sales_rep`, `transaction_amount`, and `status` ('Paid', 'Pending', 'Cancelled').

**Goal:** For each **Sales Rep**, show the total amount of sales for each status ('Paid', 'Pending', 'Cancelled') as separate columns.

```sql
SELECT
    sales_rep,
    
    -- 1. Total amount of PAID sales
    SUM(
        CASE WHEN status = 'Paid' THEN transaction_amount ELSE 0 END
    ) AS Paid_Revenue,
    
    -- 2. Total amount of PENDING sales
    SUM(
        CASE WHEN status = 'Pending' THEN transaction_amount ELSE 0 END
    ) AS Pending_Revenue,
    
    -- 3. Total amount of CANCELLED sales
    SUM(
        CASE WHEN status = 'Cancelled' THEN transaction_amount ELSE 0 END
    ) AS Cancelled_Revenue,
    
    -- 4. Conditional COUNT (Count of Paid Transactions)
    SUM(
        CASE WHEN status = 'Paid' THEN 1 ELSE 0 END
    ) AS Paid_Transaction_Count
    
FROM
    DailySales
GROUP BY
    sales_rep
ORDER BY
    sales_rep;
```

## Key Considerations

  * **Group By:** You **must** use `GROUP BY` on the non-pivoted columns (`sales_rep` in the example) to aggregate the results correctly.
  * **SUM vs. COUNT:**
      * To get a sum of values (like money), use `SUM(CASE WHEN ... THEN amount ELSE 0 END)`.
      * To get a count of occurrences, use `SUM(CASE WHEN ... THEN 1 ELSE 0 END)`.
  * **NULL vs. 0:** Using `ELSE 0` ensures the `SUM` function adds a zero for non-matching rows. If you omit the `ELSE 0` (or use `ELSE NULL`), `SUM` correctly ignores the `NULL`, but `ELSE 0` is often clearer and more explicit for pivot transformations.

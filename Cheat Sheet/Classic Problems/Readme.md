# Cheat Sheet: Classic SQL Interview Problems

| Problem Type | What They're Testing | Solution Pattern | Key Technique |
|---|---|---|---|
| **Find duplicate records** | Basic GROUP BY + HAVING | `GROUP BY columns HAVING COUNT(*) > 1` | Count occurrences |
| **Remove duplicates, keep one** | ROW_NUMBER with DELETE/WHERE | `ROW_NUMBER() OVER(PARTITION BY dup_cols ORDER BY id)` then filter `rn = 1` | Window function deduplication |
| **Find gaps in sequences** | Self-join or NOT EXISTS | `WHERE NOT EXISTS (SELECT 1 WHERE id = t1.id + 1)` | Check for missing next value |
| **Find consecutive events** | LAG/LEAD or self-join | `WHERE date = LAG(date) OVER(ORDER BY date) + 1` | Compare adjacent rows |
| **Top N per group** | ROW_NUMBER + PARTITION BY | `ROW_NUMBER() OVER(PARTITION BY group_col ORDER BY value DESC)` filter `<= N` | Ranking within partitions |
| **Running totals** | SUM() OVER with frame | `SUM(amount) OVER(ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)` | Cumulative window function |
| **Moving averages** | AVG() OVER with frame | `AVG(value) OVER(ORDER BY date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)` | Sliding window calculation |
| **First/Last in group** | FIRST_VALUE / LAST_VALUE | `FIRST_VALUE(col) OVER(PARTITION BY group ORDER BY date)` | Get boundary values |
| **Compare to previous row** | LAG function | `value - LAG(value) OVER(ORDER BY date)` | Access previous row |
| **Employee-Manager hierarchy** | Self-join | `FROM employees e LEFT JOIN employees m ON e.manager_id = m.id` | Join table to itself |
| **N-level recursion** (org chart depth) | Recursive CTE | `WITH RECURSIVE cte AS (base UNION ALL recursive)` | Build hierarchy levels |
| **Find records with no match** | LEFT JOIN + IS NULL or NOT EXISTS | `LEFT JOIN table2 ON ... WHERE table2.id IS NULL` | Anti-join pattern |
| **Pivot rows to columns** | CASE + GROUP BY or PIVOT | `SUM(CASE WHEN category = 'X' THEN amount END)` | Conditional aggregation |
| **Unpivot columns to rows** | UNION ALL or UNPIVOT | `SELECT 'col1', col1 UNION ALL SELECT 'col2', col2` | Stack columns as rows |
| **Date range overlaps** | WHERE with date comparisons | `WHERE start1 <= end2 AND end1 >= start2` | Interval intersection logic |
| **Same value in ALL related records** | NOT EXISTS with negation | `WHERE NOT EXISTS (SELECT 1 WHERE condition != expected)` | Check universal condition |
| **Cumulative distinct count** | COUNT(DISTINCT) with window | Complex—usually requires self-join or specialized function | Running unique count |


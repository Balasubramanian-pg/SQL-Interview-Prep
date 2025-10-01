# Cheat Sheet: JOIN Patterns in Interviews

| Interview Question Pattern | What They're Really Testing | Best JOIN to Use | Why |
|---|---|---|---|
| **Find all customers who made purchases** | Do you understand basic matching | `INNER JOIN` | Only returns rows where match exists in both tables |
| **Find all customers, show purchases if they exist** | Do you know how to keep all left-side rows | `LEFT JOIN` | Keeps all customers, NULLs for those without purchases |
| **Find customers who have NEVER made a purchase** | Can you spot the NULL pattern | `LEFT JOIN` + `WHERE right.id IS NULL` | The "anti-join" pattern—finds non-matches |
| **Find all records from both tables, even if no match** | Do you know the rarest join type | `FULL OUTER JOIN` | Union of LEFT and RIGHT join results |
| **Generate all possible combinations** (e.g., every product with every store) | Understanding Cartesian products | `CROSS JOIN` | Creates every possible pair (m × n rows) |
| **Compare rows within the same table** (e.g., find employees earning more than their manager) | Self-join knowledge | `INNER JOIN table t1 ON table t2` (same table, different aliases) | Treats one table as two separate entities |
| **Find employees and their managers' names** | Practical self-join usage | `LEFT JOIN table t1 ON table t2` | LEFT handles CEO (no manager = NULL) |
| **Find records in Table A not in Table B** | Set difference logic | `LEFT JOIN` + `WHERE B.id IS NULL` or `NOT EXISTS` | Anti-join pattern or subquery |
| **Performance trap: Filter before or after join?** | Do you understand query execution | `INNER JOIN` with `WHERE` on **left table before join** if possible | Reduces rows before expensive join operation |
| **Multiple join conditions** (e.g., match on date range, not just ID) | Complex join logic | `JOIN ON t1.id = t2.id AND t1.date BETWEEN t2.start AND t2.end` | Multiple conditions in ON clause |
| **Find customers with purchases in ALL categories** | Relational division problem | `INNER JOIN` + `GROUP BY` + `HAVING COUNT(DISTINCT category) = (SELECT COUNT(*) FROM categories)` | Ensures complete set membership |

---

## Quick Decision Tree

```
Do both sides MUST have a match?
├─ YES → INNER JOIN
└─ NO → Need one side's all rows?
    ├─ Keep all LEFT side rows → LEFT JOIN
    ├─ Keep all RIGHT side rows → RIGHT JOIN
    └─ Keep all rows from BOTH → FULL OUTER JOIN

Need every combination?
└─ YES → CROSS JOIN

Comparing table to itself?
└─ YES → SELF JOIN (use aliases!)
```

---

## Critical Traps & Tricks

### **Trap 1: LEFT JOIN but filtering in WHERE kills the "LEFT" part**
```sql
-- ❌ WRONG: This becomes an INNER JOIN!
SELECT * 
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.order_date > '2024-01-01'  -- Filters out NULLs!

-- ✅ CORRECT: Put filter in ON clause or use IS NULL pattern
SELECT * 
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id 
    AND o.order_date > '2024-01-01'
```

**Interview Trick:** If they ask "find customers WITHOUT orders after 2024" and you write the first query, you fail!

---

### **Trap 2: JOIN vs WHERE for filtering**
```sql
-- Both produce same result, but semantic difference:
SELECT * FROM employees e
INNER JOIN departments d ON e.dept_id = d.id
WHERE d.name = 'Sales'

-- vs
SELECT * FROM employees e
INNER JOIN departments d ON e.dept_id = d.id AND d.name = 'Sales'
```

**Best Practice:** 
- Put **join conditions** in `ON`
- Put **filters** in `WHERE`
- Exception: LEFT JOIN filters on right table must go in `ON`

---

### **Trap 3: Multiple LEFT JOINs can multiply rows**
```sql
-- If one customer has 3 orders and 2 addresses:
SELECT c.name, o.order_id, a.address
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
LEFT JOIN addresses a ON c.id = a.customer_id
-- Result: 3 × 2 = 6 rows for that customer!
```

**Solution:** Use subqueries or aggregate before joining

---

### **Trick: The "NOT EXISTS" Alternative**
```sql
-- Finding customers without orders (3 ways):

-- Method 1: LEFT JOIN + IS NULL (most common)
SELECT c.* FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.id IS NULL

-- Method 2: NOT IN (dangerous if NULLs exist!)
SELECT * FROM customers
WHERE id NOT IN (SELECT customer_id FROM orders)

-- Method 3: NOT EXISTS (usually fastest)
SELECT * FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.id
)
```

**Interview Gold:** Mention that `NOT EXISTS` often performs better than `LEFT JOIN` for large datasets!

---

## Self-Join Pattern Recognition

| Problem Type | Self-Join Pattern |
|---|---|
| Employee-Manager hierarchy | `LEFT JOIN employees mgr ON emp.manager_id = mgr.id` |
| Compare consecutive rows | `JOIN table t1 ON table t2 ON t2.id = t1.id + 1` |
| Find duplicates | `JOIN table t1 ON table t2 ON t1.email = t2.email AND t1.id < t2.id` |
| Compare same entity at different times | `JOIN table current ON table previous ON current.id = previous.id AND current.date > previous.date` |

---

## When They Ask "Why Not Use...?"

| Question | Answer |
|---|---|
| **"Why not always use LEFT JOIN to be safe?"** | Performance cost! More rows to process. INNER JOIN is faster and clearer intent. |
| **"Why not use RIGHT JOIN?"** | You can, but LEFT is convention (left table = "main" table). RIGHT JOIN is just LEFT flipped. |
| **"CROSS JOIN vs comma syntax?"** | `FROM t1, t2` is implicit CROSS JOIN (old style). Explicit `CROSS JOIN` is clearer. |
| **"When would you actually use FULL OUTER JOIN?"** | Rare! Maybe comparing two data sources to find discrepancies on both sides. |

---

## Performance Quick Hits

✅ **DO:**
- Filter LEFT table before join when possible
- Use EXISTS instead of IN for subqueries with large result sets
- Index columns used in ON clauses
- Use INNER JOIN when you don't need unmatched rows

❌ **DON'T:**
- Use SELECT * with multiple joins (explodes column count)
- Put WHERE filters on right table of LEFT JOIN
- Assume NOT IN handles NULLs correctly (it doesn't!)
- Chain multiple LEFT JOINs without understanding row multiplication

---

## Interview Power Move

When asked about JOIN performance, mention:
1. **"Join order matters in some databases"** (though optimizers usually handle it)
2. **"EXISTS vs IN vs JOIN"** - know when each is best
3. **"LEFT JOIN with WHERE IS NULL is called an anti-join"** (shows you know the terminology)
4. **"Indexes on join keys are critical"** (shows you think about performance)

This demonstrates you understand not just syntax, but database internals!

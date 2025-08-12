Here’s the structured README document based on your provided content, formatted according to your requirements:

---

# SQL Joins Interview Question Summary  

## 1. **Question**  
Given two tables:  
- **Table1**: `ID` column with values `(1, 1, 1, 2, 2, 3, 4, null, null)` - 9 rows  
- **Table2**: `ID` column with values `(1, 1, 1, 2, 2, 3, null)` - 7 rows  

Determine how many records each type of join will produce when joining on the `ID` column:  
1. INNER JOIN  
2. LEFT JOIN  
3. RIGHT JOIN  
4. FULL JOIN  
5. CROSS JOIN  

---

## 2. **Easiest Solution**  

### 2.1 **INNER JOIN (14 records)**  
```sql
SELECT COUNT(*)
FROM Table1
INNER JOIN Table2 ON Table1.ID = Table2.ID;
```  

### 2.2 **LEFT JOIN (17 records)**  
```sql
SELECT COUNT(*)
FROM Table1
LEFT JOIN Table2 ON Table1.ID = Table2.ID;
```  

### 2.3 **RIGHT JOIN (15 records)**  
```sql
SELECT COUNT(*)
FROM Table1
RIGHT JOIN Table2 ON Table1.ID = Table2.ID;
```  

### 2.4 **FULL JOIN (18 records)**  
```sql
SELECT COUNT(*)
FROM Table1
FULL JOIN Table2 ON Table1.ID = Table2.ID;
```  

### 2.5 **CROSS JOIN (63 records)**  
```sql
SELECT COUNT(*)
FROM Table1
CROSS JOIN Table2;
```  

---

## 3. **Code Explanation**  

### 3.1 **INNER JOIN**  
- **Returns**: Only matching non-null records from both tables.  
- **Calculation**: Counts all pairs where `Table1.ID = Table2.ID` (and neither is `null`).  
- **Result**: 14 records.  

> [!TIP]  
> INNER JOIN excludes rows where there is no match in either table.  

### 3.2 **LEFT JOIN**  
- **Returns**: All records from `Table1` plus matching records from `Table2`.  
- **Calculation**: Includes all 14 matches from INNER JOIN plus 3 unmatched records from `Table1` (`4` and two `nulls`).  
- **Result**: 17 records (14 + 3).  

> [!NOTE]  
> LEFT JOIN retains all rows from the left table, even if there’s no match in the right table.  

### 3.3 **RIGHT JOIN**  
- **Returns**: All records from `Table2` plus matching records from `Table1`.  
- **Calculation**: Includes all 14 matches from INNER JOIN plus 1 unmatched record from `Table2` (`null`).  
- **Result**: 15 records (14 + 1).  

> [!NOTE]  
> RIGHT JOIN retains all rows from the right table, even if there’s no match in the left table.  

### 3.4 **FULL JOIN**  
- **Returns**: All records when there's a match in either table.  
- **Calculation**: Includes all 14 matches from INNER JOIN + 3 unmatched from LEFT + 1 unmatched from RIGHT.  
- **Result**: 18 records (14 + 3 + 1).  

> [!IMPORTANT]  
> FULL JOIN retains all rows from both tables, combining the effects of LEFT and RIGHT JOINs.  

### 3.5 **CROSS JOIN**  
- **Returns**: Cartesian product (every row from `Table1` paired with every row from `Table2`).  
- **Calculation**: Simple multiplication: 9 rows × 7 rows = 63 records.  
- **Result**: 63 records.  

> [!WARNING]  
> CROSS JOIN can produce very large result sets, especially with big tables. Use with caution.  

---

**Note**: NULL values don't match with each other in joins (`NULL ≠ NULL` in SQL).  

> [!TIP]  
> Understanding how each join type handles matches and non-matches is key to solving join-related interview questions.  

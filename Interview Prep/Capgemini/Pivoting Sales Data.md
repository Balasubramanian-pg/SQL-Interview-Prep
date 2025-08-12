# Pivoting Sales Data  

## 1. **Problem Statement**  
Given a `sales_data` table with columns:  
- `month`  
- `category` (electronics/clothing)  
- `amount`  

We need to:  
1. Pivot the data to show months as rows.  
2. Transform categories into columns.  
3. Display amounts in the corresponding cells.  

---

## 2. **PostgreSQL Solution (Using CROSSTAB)**  
```sql
SELECT * FROM CROSSTAB(
    'SELECT
         month,
         category,
         amount
      FROM sales_data
      ORDER BY 1, 2'
) AS ct (
    month VARCHAR,
    clothing NUMERIC,
    electronics NUMERIC
);
```  

---

## 3. **Key Components Explained**  
### 3.1 **CROSSTAB Function**  
- PostgreSQL's pivot function.  
- Requires a query returning exactly 3 columns:  
  - Row identifier (`month`).  
  - Category (values to become columns).  
  - Value (`amounts` to display).  

### 3.2 **Output Definition**  
- `AS ct` defines the result table alias.  
- Column definitions specify data types:  
  - `month VARCHAR` (text column).  
  - `clothing NUMERIC` (first category).  
  - `electronics NUMERIC` (second category).  

### 3.3 **Ordering**  
- `ORDER BY 1, 2` ensures proper grouping.  
- Sorts by `month` then `category`.  

> [!NOTE]  
> The `CROSSTAB` function requires the `tablefunc` extension to be enabled in PostgreSQL.  

---

## 4. **Alternative Solutions**  
### 4.1 **SQL Server (Using PIVOT)**  
```sql
SELECT
    month,
    [clothing] AS clothing,
    [electronics] AS electronics
FROM
    (SELECT month, category, amount FROM sales_data) AS SourceTable
PIVOT
(
    SUM(amount)
    FOR category IN ([clothing], [electronics])
) AS PivotTable;
```  

### 4.2 **Standard SQL (Using CASE)**  
```sql
SELECT
    month,
    SUM(CASE WHEN category = 'clothing' THEN amount END) AS clothing,
    SUM(CASE WHEN category = 'electronics' THEN amount END) AS electronics
FROM
    sales_data
GROUP BY
    month;
```  

## 5. **Key Concepts**  
### 5.1 **Pivoting Data**  
- Transforming rows to columns.  
- Common requirement for reports and dashboards.  

### 5.2 **Database-Specific Functions**  
- **PostgreSQL**: `CROSSTAB` (requires `tablefunc` extension).  
- **SQL Server**: `PIVOT`.  
- **Oracle**: `PIVOT`.  

### 5.3 **Performance Considerations**  
- Pivot operations can be resource-intensive.  
- Filter source data first if possible.  
- Consider materialized views for frequent pivots.  

> [!TIP]  
> Always test pivot queries with a small dataset first to ensure correctness and performance.  

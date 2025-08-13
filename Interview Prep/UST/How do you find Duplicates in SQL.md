Here’s a quick SQL snippet for inserting 20 rows into a sample table, with a few intentional duplicates. I’ll assume a simple table structure:

```sql
CREATE TABLE Customers (
    CustomerID INT,
    CustomerName VARCHAR(50),
    City VARCHAR(50)
);

INSERT INTO Customers (CustomerID, CustomerName, City)
VALUES
(1, 'Alice', 'New York'),
(2, 'Bob', 'Chicago'),
(3, 'Charlie', 'Houston'),
(4, 'David', 'Seattle'),
(5, 'Eva', 'Miami'),
(6, 'Frank', 'Boston'),
(7, 'Grace', 'Dallas'),
(8, 'Hannah', 'Denver'),
(9, 'Ivan', 'Atlanta'),
(10, 'Jack', 'Austin'),
(11, 'Alice', 'New York'),    -- duplicate
(12, 'Bob', 'Chicago'),       -- duplicate
(13, 'Kevin', 'Orlando'),
(14, 'Liam', 'San Francisco'),
(15, 'Mia', 'Portland'),
(16, 'Nora', 'Las Vegas'),
(17, 'Olivia', 'Phoenix'),
(18, 'Paul', 'Philadelphia'),
(19, 'Quinn', 'San Diego'),
(20, 'Eva', 'Miami');          -- duplicate
```

So the easiest way to find duplicates is to use the row_number() function

```sql
WITH CTE AS(
ROW_NUMBER() OVER(PARTITION BY CustomerName, City ORDER BY City) AS rn
FROM Customers
)
DELETE FROM CTE WHERE rn>1;
```

> What are we doing here? We are assigning a row number to each individual values so that if there are duplicates we can assign the same value instead of a tie breaker.
> Then using this duplicate row number we take the count if it more than 1, we delete.


Your output will be something like this
<img width="628" height="717" alt="image" src="https://github.com/user-attachments/assets/038adce9-3540-4215-9ed9-27ceaafd374a" />


The second method is use Group BY and HAVING Clause
So essentially you can group by the needed columns and take a count of these columns,

```SQL
SELECT Customername, City, COUNT(*) AS D
FROM Customers
GROUP BY Customername, City
HAVING COUNT(*) >1;
```
your output would call out the duplicate values for further investigation
<img width="510" height="142" alt="image" src="https://github.com/user-attachments/assets/8bcc6fe9-3a67-4d17-86a1-982fb0ac90d8" />


## Takeaway
Alright, let’s keep it practical. Three common ways to find and delete duplicates in SQL:

**1. Using `ROW_NUMBER()`**

```sql
WITH CTE AS (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY column1, column2 ORDER BY id) AS rn
    FROM your_table
)
DELETE FROM CTE WHERE rn > 1;
```

* Keeps the first record per duplicate group, deletes the rest.

**2. Using `GROUP BY` with `HAVING`**

```sql
SELECT column1, column2, COUNT(*)
FROM your_table
GROUP BY column1, column2
HAVING COUNT(*) > 1;
```

* This finds duplicates; combine with a `DELETE` using a subquery to remove them.

**3. Using `DISTINCT` into a new table**

```sql
CREATE TABLE temp_table AS
SELECT DISTINCT * FROM your_table;

DROP TABLE your_table;
ALTER TABLE temp_table RENAME TO your_table;
```

* Quick and dirty: creates a duplicate-free copy.


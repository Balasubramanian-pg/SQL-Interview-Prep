I'll help you create INSERT INTO statements for sales and quantity tables. Here are a few examples with different scenarios:

## Basic Sales Table Structure

```sql
-- Sample table structure
CREATE TABLE sales (
    sale_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    quantity INT,
    unit_price DECIMAL(10,2),
    sale_date DATE,
    customer_id INT
);

CREATE TABLE inventory (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    quantity_in_stock INT,
    reorder_level INT
);
```

## INSERT Statements for Sales Table

```sql
-- Insert single record
INSERT INTO sales (sale_id, product_name, quantity, unit_price, sale_date, customer_id)
VALUES (1, 'Laptop', 1, 999.99, '2024-01-15', 101);

-- Insert multiple records
INSERT INTO sales (sale_id, product_name, quantity, unit_price, sale_date, customer_id)
VALUES 
(2, 'Mouse', 3, 25.50, '2024-01-15', 102),
(3, 'Keyboard', 2, 75.00, '2024-01-16', 103),
(4, 'Monitor', 1, 299.99, '2024-01-17', 101),
(5, 'Headphones', 5, 49.99, '2024-01-18', 104);
```

## INSERT Statements for Quantity/Inventory Table

```sql
-- Insert inventory data
INSERT INTO inventory (product_id, product_name, quantity_in_stock, reorder_level)
VALUES 
(1, 'Laptop', 15, 5),
(2, 'Mouse', 100, 20),
(3, 'Keyboard', 50, 10),
(4, 'Monitor', 25, 5),
(5, 'Headphones', 75, 15);
```

## More Comprehensive Example

```sql
-- Extended sales table with more details
CREATE TABLE sales_detailed (
    sale_id INT PRIMARY KEY,
    product_id INT,
    product_name VARCHAR(100),
    category VARCHAR(50),
    quantity_sold INT,
    price_per_unit DECIMAL(10,2),
    total_amount DECIMAL(10,2),
    sale_date DATE,
    salesperson VARCHAR(50),
    region VARCHAR(50)
);

-- Insert data
INSERT INTO sales_detailed 
(sale_id, product_id, product_name, category, quantity_sold, price_per_unit, total_amount, sale_date, salesperson, region)
VALUES
(1, 101, 'iPhone 14', 'Electronics', 2, 799.00, 1598.00, '2024-01-10', 'John Doe', 'North'),
(2, 102, 'Samsung TV', 'Electronics', 1, 1299.99, 1299.99, '2024-01-11', 'Jane Smith', 'South'),
(3, 201, 'Office Chair', 'Furniture', 5, 149.50, 747.50, '2024-01-12', 'Mike Johnson', 'East'),
(4, 202, 'Desk Lamp', 'Home Goods', 10, 29.99, 299.90, '2024-01-13', 'Sarah Wilson', 'West'),
(5, 103, 'MacBook Pro', 'Electronics', 1, 1999.00, 1999.00, '2024-01-14', 'John Doe', 'North');
```

## Daily Sales Quantity Tracking

```sql
CREATE TABLE daily_sales_qty (
    record_id INT PRIMARY KEY,
    sale_date DATE,
    product_id INT,
    product_name VARCHAR(100),
    daily_quantity INT,
    warehouse_location VARCHAR(50)
);

INSERT INTO daily_sales_qty 
(record_id, sale_date, product_id, product_name, daily_quantity, warehouse_location)
VALUES
(1, '2024-01-15', 1, 'Laptop', 3, 'Warehouse A'),
(2, '2024-01-15', 2, 'Mouse', 15, 'Warehouse B'),
(3, '2024-01-15', 3, 'Keyboard', 8, 'Warehouse A'),
(4, '2024-01-16', 1, 'Laptop', 2, 'Warehouse A'),
(5, '2024-01-16', 4, 'Monitor', 4, 'Warehouse C');
```

## Quick Bulk Insert (10 records)

```sql
-- Quick sales data insertion
INSERT INTO sales (sale_id, product_name, quantity, unit_price, sale_date, customer_id)
VALUES 
(1, 'Product A', 10, 15.99, '2024-01-01', 1001),
(2, 'Product B', 5, 29.99, '2024-01-02', 1002),
(3, 'Product C', 8, 12.50, '2024-01-02', 1003),
(4, 'Product A', 3, 15.99, '2024-01-03', 1001),
(5, 'Product D', 12, 8.75, '2024-01-04', 1004),
(6, 'Product B', 2, 29.99, '2024-01-05', 1005),
(7, 'Product E', 6, 45.00, '2024-01-06', 1002),
(8, 'Product C', 15, 12.50, '2024-01-07', 1006),
(9, 'Product A', 7, 15.99, '2024-01-08', 1003),
(10, 'Product D', 4, 8.75, '2024-01-09', 1007);
```

Choose the example that best fits your table structure! Let me know if you need any modifications or have specific column requirements.

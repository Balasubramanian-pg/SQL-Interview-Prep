Here's the complete `INSERT INTO` query to populate the `products` and `orders` tables for testing both revenue calculation and unordered product scenarios:

```sql
-- Create products table
CREATE TABLE IF NOT EXISTS products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    product_price DECIMAL(10,2)
);

-- Create orders table
CREATE TABLE IF NOT EXISTS orders (
    order_id INT PRIMARY KEY,
    product_id INT,
    quantity INT,
    order_date DATE,
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- Clear existing data
TRUNCATE TABLE products;
TRUNCATE TABLE orders;

-- Insert sample products
INSERT INTO products (product_id, product_name, product_price) VALUES
(1, 'Laptop', 999.99),
(2, 'Smartphone', 699.99),
(3, 'Headphones', 149.99),
(4, 'Tablet', 399.99),
(5, 'Smartwatch', 199.99),  -- Will not be ordered
(6, 'Camera', 499.99),      -- Will not be ordered
(7, 'Keyboard', 79.99),
(8, 'Mouse', 29.99),
(9, 'Monitor', 249.99),
(10, 'Printer', 179.99);    -- Will not be ordered

-- Insert sample orders
INSERT INTO orders (order_id, product_id, quantity, order_date) VALUES
-- January orders
(101, 1, 2, '2023-01-15'),  -- Laptop
(102, 2, 3, '2023-01-16'),  -- Smartphone
(103, 3, 5, '2023-01-17'),  -- Headphones
(104, 1, 1, '2023-01-18'),  -- Laptop (multiple orders)
(105, 4, 2, '2023-01-19'),  -- Tablet

-- February orders
(201, 2, 1, '2023-02-05'),  -- Smartphone
(202, 7, 4, '2023-02-10'),  -- Keyboard
(203, 8, 10, '2023-02-15'), -- Mouse
(204, 9, 2, '2023-02-20'),  -- Monitor
(205, 3, 3, '2023-02-25'),  -- Headphones

-- March orders (some products ordered multiple times)
(301, 1, 1, '2023-03-01'),  -- Laptop
(302, 2, 2, '2023-03-05'),  -- Smartphone
(303, 4, 1, '2023-03-10'),  -- Tablet
(304, 7, 2, '2023-03-15'),  -- Keyboard
(305, 8, 5, '2023-03-20'); -- Mouse
```

To test the revenue calculation:

```sql
-- Problem 1: Retrieve Total Revenue from Each Product
SELECT
    p.product_name,
    SUM(o.quantity * p.product_price) AS total_revenue,
    COUNT(o.order_id) AS order_count,
    SUM(o.quantity) AS total_units_sold
FROM
    products p
JOIN
    orders o ON p.product_id = o.product_id
GROUP BY
    p.product_name
ORDER BY
    total_revenue DESC;
```

To test for unordered products:

```sql
-- Problem 2: Find Products Not Ordered At All
SELECT
    p.product_id,
    p.product_name,
    p.product_price
FROM
    products p
LEFT JOIN
    orders o ON p.product_id = o.product_id
WHERE
    o.order_id IS NULL
ORDER BY
    p.product_name;
```

Key features of this test data:
1. **Products with varying prices** (from $29.99 to $999.99)
2. **Different order quantities** (1-10 units per order)
3. **Multiple orders for some products** (like Laptop ordered 3 times)
4. **Three products never ordered** (Smartwatch, Camera, Printer)
5. **Temporal distribution** (orders across 3 months)

The revenue query should:
- Correctly calculate revenue (price × quantity)
- Show products in descending revenue order
- Include order count and total units sold

The unordered products query should:
- Only return the 3 products with no orders
- Include all product details
- Not show any products that appear in orders

This dataset helps verify:
- JOIN operations work correctly
- Aggregation functions handle multiple orders
- NULL checking properly identifies unordered products
- Calculations are mathematically accurate

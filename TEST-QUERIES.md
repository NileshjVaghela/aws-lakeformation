# Lake Formation Workshop - Test Queries

## Quick Start Queries (Copy & Paste into Athena Console)

### 1. Basic Counts
```sql
-- Count customers
SELECT COUNT(*) as total_customers FROM customers;

-- Count orders  
SELECT COUNT(*) as total_orders FROM orders;

-- Count products
SELECT COUNT(*) as total_products FROM products;
```

### 2. Sample Data
```sql
-- View customers
SELECT * FROM customers LIMIT 10;

-- View orders
SELECT * FROM orders LIMIT 10;

-- View products
SELECT * FROM products LIMIT 10;
```

### 3. Simple Aggregations
```sql
-- Customers by state
SELECT state, COUNT(*) as count
FROM customers
GROUP BY state
ORDER BY count DESC;

-- Orders by status
SELECT status, COUNT(*) as count
FROM orders
GROUP BY status;

-- Products by category
SELECT category, COUNT(*) as count, 
       ROUND(AVG(price), 2) as avg_price
FROM products
GROUP BY category
ORDER BY count DESC;
```

### 4. Simple Joins
```sql
-- Orders with customer info
SELECT o.order_id, o.order_date, o.status,
       c.first_name, c.last_name, c.city, c.state
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
LIMIT 20;

-- Orders with product info
SELECT o.order_id, o.quantity, o.status,
       p.product_name, p.category, p.price
FROM orders o
JOIN products p ON o.product_id = p.product_id
LIMIT 20;
```

### 5. Top Customers
```sql
-- Top 10 customers by order count
SELECT c.customer_id,
       c.first_name || ' ' || c.last_name as name,
       c.city, c.state,
       COUNT(o.order_id) as orders
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name, c.city, c.state
ORDER BY orders DESC
LIMIT 10;
```

### 6. Revenue Analysis
```sql
-- Revenue by category (completed orders only)
SELECT p.category,
       COUNT(o.order_id) as orders,
       SUM(o.quantity) as items_sold,
       ROUND(SUM(o.quantity * p.price), 2) as revenue
FROM orders o
JOIN products p ON o.product_id = p.product_id
WHERE o.status = 'Completed'
GROUP BY p.category
ORDER BY revenue DESC;
```

### 7. Top Products
```sql
-- Top 10 products by revenue
SELECT p.product_name, p.category, p.price,
       COUNT(o.order_id) as times_ordered,
       SUM(o.quantity) as quantity_sold,
       ROUND(SUM(o.quantity * p.price), 2) as revenue
FROM products p
JOIN orders o ON p.product_id = o.product_id
WHERE o.status = 'Completed'
GROUP BY p.product_id, p.product_name, p.category, p.price
ORDER BY revenue DESC
LIMIT 10;
```

### 8. Customer Lifetime Value
```sql
-- Top 20 customers by total spend
SELECT c.customer_id,
       c.first_name || ' ' || c.last_name as name,
       c.email, c.city, c.state,
       COUNT(o.order_id) as orders,
       ROUND(SUM(o.quantity * p.price), 2) as lifetime_value
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN products p ON o.product_id = p.product_id
WHERE o.status = 'Completed'
GROUP BY c.customer_id, c.first_name, c.last_name, c.email, c.city, c.state
ORDER BY lifetime_value DESC
LIMIT 20;
```

### 9. Geographic Analysis
```sql
-- Revenue by state
SELECT c.state,
       COUNT(DISTINCT c.customer_id) as customers,
       COUNT(o.order_id) as orders,
       ROUND(SUM(o.quantity * p.price), 2) as revenue
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = p.product_id
WHERE o.status = 'Completed'
GROUP BY c.state
ORDER BY revenue DESC;
```

### 10. Inventory Check
```sql
-- Low stock products (less than 50 units)
SELECT product_name, category, price, stock_quantity
FROM products
WHERE stock_quantity < 50
ORDER BY stock_quantity ASC
LIMIT 20;
```

---

## Dataset Summary

- **Customers**: 5,000 records (361 KB)
- **Products**: 500 records (19 KB)
- **Orders**: 20,000 records (723 KB)
- **Total**: 1.1 MB

## Tips

1. Always specify the database: `lf_workshop_student`
2. Set S3 output location: `s3://lf-workshop-data-ACCOUNT_ID/athena-results/`
3. Queries are case-insensitive
4. Use LIMIT to test queries before running on full dataset
5. Check "Query history" tab to see execution time and data scanned

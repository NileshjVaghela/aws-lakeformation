# Sample Data Files

This directory contains pre-generated CSV files for the Lake Formation workshop.

## Files

### 1. customers.csv (362 KB)
- **Records:** 5,000 customers
- **Columns:** 9
  - customer_id (integer)
  - first_name (string)
  - last_name (string)
  - email (string)
  - phone (string)
  - city (string)
  - state (string)
  - country (string)
  - registration_date (date)

### 2. orders.csv (723 KB)
- **Records:** 20,000 orders
- **Columns:** 6
  - order_id (integer)
  - customer_id (integer)
  - product_id (integer)
  - quantity (integer)
  - order_date (date)
  - status (string: Completed, Pending, Shipped, Cancelled)

### 3. products.csv (20 KB)
- **Records:** 500 products
- **Columns:** 5
  - product_id (integer)
  - product_name (string)
  - category (string)
  - price (decimal)
  - stock_quantity (integer)

## Usage

### Option 1: Upload via AWS Console
1. Download these files to your local computer
2. In S3 Console, upload to:
   - `customers.csv` → `s3://lf-workshop-data-ACCOUNT_ID/customers/`
   - `orders.csv` → `s3://lf-workshop-data-ACCOUNT_ID/orders/`
   - `products.csv` → `s3://lf-workshop-data-ACCOUNT_ID/products/`

### Option 2: Upload via AWS CLI
```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

aws s3 cp customers.csv s3://lf-workshop-data-${ACCOUNT_ID}/customers/
aws s3 cp orders.csv s3://lf-workshop-data-${ACCOUNT_ID}/orders/
aws s3 cp products.csv s3://lf-workshop-data-${ACCOUNT_ID}/products/
```

### Option 3: Upload via CloudShell
1. Open AWS CloudShell
2. Upload these files using the "Actions" → "Upload file" menu
3. Run the AWS CLI commands above

## Data Relationships

```
customers (1) ──── (N) orders (N) ──── (1) products
   5,000                20,000              500
```

- Each customer can have multiple orders
- Each order references one customer and one product
- Products can appear in multiple orders

## Sample Queries

After uploading to S3 and running Glue crawler, you can query:

```sql
-- Total customers
SELECT COUNT(*) FROM customers;  -- Result: 5000

-- Total orders
SELECT COUNT(*) FROM orders;  -- Result: 20000

-- Total products
SELECT COUNT(*) FROM products;  -- Result: 500

-- Join example
SELECT c.first_name, c.last_name, COUNT(o.order_id) as order_count
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY order_count DESC
LIMIT 10;
```

## File Generation

These files were generated using Python with realistic data patterns:
- Customer names from common US names
- Cities from top 15 US metropolitan areas
- Product categories across 10 different types
- Order dates within the last 365 days
- Random but realistic prices and quantities

## Total Dataset Size

- **Total records:** 25,500
- **Total size:** 1.1 MB
- **Estimated Athena scan cost:** $0.000005 per query (negligible)
- **Estimated Glue crawler cost:** $0.03 per run

---

**Note:** These are sample files for educational purposes. Data is randomly generated and does not represent real customers or transactions.

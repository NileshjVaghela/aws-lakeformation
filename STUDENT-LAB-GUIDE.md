# AWS Lake Formation Workshop - Student Lab Guide

## Lab Overview

In this hands-on lab, you will learn how to use AWS Lake Formation to build a data lake, catalog data using AWS Glue, and query data using Amazon Athena.

**Duration:** 60-90 minutes  
**Difficulty:** Beginner to Intermediate  

---

## Lab Objectives

By the end of this lab, you will be able to:
- Create S3 buckets for data lake storage
- Upload sample datasets to S3
- Create a Glue database for data cataloging
- Configure and run Glue crawlers to discover data schemas
- Register S3 locations with Lake Formation
- Query data using Amazon Athena
- Perform analytics on customer, order, and product data

---

## Prerequisites

- AWS Account access with student credentials
- Basic understanding of SQL
- Familiarity with AWS Console

---

## Lab Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         Data Lake                           │
│                                                             │
│  S3 Buckets                                                 │
│  ├── customers/     (5,000 records)                         │
│  ├── orders/        (20,000 records)                        │
│  └── products/      (500 records)                           │
│                                                             │
│  ↓                                                          │
│                                                             │
│  AWS Glue Crawler                                           │
│  └── Discovers schema and creates tables                   │
│                                                             │
│  ↓                                                          │
│                                                             │
│  AWS Glue Data Catalog                                      │
│  └── Stores metadata (tables, columns, data types)         │
│                                                             │
│  ↓                                                          │
│                                                             │
│  Amazon Athena                                              │
│  └── SQL queries on data lake                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Part 1: Setup S3 Data Lake (15 minutes)

### Step 1.1: Get Your Account ID

1. Open **AWS Console**
2. Click on your username in the top-right corner
3. Note your **Account ID** (12 digits)
4. Example: `016534441838`

### Step 1.2: Create S3 Bucket

1. Go to **S3** service (search "S3" in top search bar)
2. Click **"Create bucket"**
3. Bucket name: `lf-workshop-data-YOUR_ACCOUNT_ID`
   - Example: `lf-workshop-data-016534441838`
4. Region: **US East (N. Virginia) us-east-1**
5. Leave all other settings as default
6. Click **"Create bucket"**

### Step 1.3: Create Folder Structure

1. Click on your newly created bucket
2. Click **"Create folder"**
3. Create three folders:
   - `customers`
   - `orders`
   - `products`

### Step 1.4: Upload Sample Data

You will upload three pre-populated CSV files to your S3 bucket.

**Option A: Using AWS CloudShell (Recommended)**

1. Open **CloudShell** (icon in top-right corner of AWS Console)
2. Download the sample data files:
   - Get the files from your instructor OR
   - Download from the workshop materials folder
3. Upload files to CloudShell:
   - Click **"Actions"** → **"Upload file"**
   - Upload: `customers.csv`, `orders.csv`, `products.csv`
4. Run these commands to upload to S3:

```bash
# Set your account ID
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Upload files to S3
aws s3 cp customers.csv s3://lf-workshop-data-${ACCOUNT_ID}/customers/
aws s3 cp orders.csv s3://lf-workshop-data-${ACCOUNT_ID}/orders/
aws s3 cp products.csv s3://lf-workshop-data-${ACCOUNT_ID}/products/

echo "✅ All files uploaded to S3!"
```

**Option B: Using AWS Console (Manual Upload)**

1. Download the three CSV files from your instructor:
   - `customers.csv` (362 KB, 5,000 records)
   - `orders.csv` (723 KB, 20,000 records)
   - `products.csv` (20 KB, 500 records)
2. In S3 Console, navigate to your bucket: `lf-workshop-data-YOUR_ACCOUNT_ID`
3. Upload each file to its corresponding folder:
   - Click on `customers/` folder → **"Upload"** → Select `customers.csv` → **"Upload"**
   - Click on `orders/` folder → **"Upload"** → Select `orders.csv` → **"Upload"**
   - Click on `products/` folder → **"Upload"** → Select `products.csv` → **"Upload"**

**Option C: Using AWS CLI (From Your Computer)**

If you have AWS CLI installed locally:

```bash
# Configure your credentials first
aws configure

# Set your account ID
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Upload files (run from the directory containing the CSV files)
aws s3 cp customers.csv s3://lf-workshop-data-${ACCOUNT_ID}/customers/
aws s3 cp orders.csv s3://lf-workshop-data-${ACCOUNT_ID}/orders/
aws s3 cp products.csv s3://lf-workshop-data-${ACCOUNT_ID}/products/
```

### Step 1.5: Verify Data Upload

1. In S3 Console, navigate to your bucket
2. Verify you see:
   - `customers/customers.csv` (~361 KB)
   - `orders/orders.csv` (~723 KB)
   - `products/products.csv` (~19 KB)

---

## Part 2: Create Glue Database (5 minutes)

### Step 2.1: Open AWS Glue Console

1. Search for **"Glue"** in AWS Console
2. Click **AWS Glue**
3. In the left menu, click **"Databases"** (under Data Catalog)

### Step 2.2: Create Database

1. Click **"Add database"**
2. Database name: `lf_workshop_student`
3. Description: `Lake Formation workshop database`
4. Click **"Create database"**

---

## Part 3: Create and Run Glue Crawler (15 minutes)

### Step 3.1: Create Crawler

1. In AWS Glue Console, click **"Crawlers"** in left menu
2. Click **"Create crawler"**

**Step 1: Set crawler properties**
- Name: `lf-workshop-student-crawler`
- Click **"Next"**

**Step 2: Choose data sources**
- Click **"Add a data source"**
- Data source: **S3**
- S3 path: `s3://lf-workshop-data-YOUR_ACCOUNT_ID/customers/`
- Click **"Add an S3 data source"**
- Repeat for:
  - `s3://lf-workshop-data-YOUR_ACCOUNT_ID/orders/`
  - `s3://lf-workshop-data-YOUR_ACCOUNT_ID/products/`
- Click **"Next"**

**Step 3: Configure security settings**
- IAM role: Select **"LFWorkshopGlueRole"** (pre-created)
- Click **"Next"**

**Step 4: Set output and scheduling**
- Target database: Select **"lf_workshop_student"**
- Crawler schedule: **On demand**
- Click **"Next"**

**Step 5: Review and create**
- Review settings
- Click **"Create crawler"**

### Step 3.2: Run Crawler

1. Select your crawler: `lf-workshop-student-crawler`
2. Click **"Run"** button
3. Wait 1-2 minutes for crawler to complete
4. Status should change to: **Ready** with **Succeeded**

### Step 3.3: Verify Tables Created

1. In left menu, click **"Tables"** (under Data Catalog)
2. You should see 3 tables:
   - `customers` (9 columns)
   - `orders` (6 columns)
   - `products` (5 columns)
3. Click on each table to view schema

---

## Part 4: Query Data with Amazon Athena (30 minutes)

### Step 4.1: Open Athena Console

1. Search for **"Athena"** in AWS Console
2. Click **Amazon Athena**
3. Click **"Explore the query editor"**

### Step 4.2: Configure Query Result Location

1. Click **"Settings"** tab
2. Click **"Manage"**
3. Query result location: `s3://lf-workshop-data-YOUR_ACCOUNT_ID/athena-results/`
4. Click **"Save"**

### Step 4.3: Select Database

1. Go back to **"Editor"** tab
2. In left panel, under **"Database"**, select: `lf_workshop_student`
3. You should see 3 tables listed below

### Step 4.4: Run Basic Queries

**Query 1: Count customers**
```sql
SELECT COUNT(*) as total_customers FROM customers;
```
Expected result: **5000**

**Query 2: Count orders**
```sql
SELECT COUNT(*) as total_orders FROM orders;
```
Expected result: **20000**

**Query 3: Count products**
```sql
SELECT COUNT(*) as total_products FROM products;
```
Expected result: **500**

**Query 4: View sample customer data**
```sql
SELECT * FROM customers LIMIT 10;
```

**Query 5: View sample order data**
```sql
SELECT * FROM orders LIMIT 10;
```

### Step 4.5: Aggregation Queries

**Query 6: Customers by state**
```sql
SELECT state, COUNT(*) as customer_count
FROM customers
GROUP BY state
ORDER BY customer_count DESC;
```

**Query 7: Orders by status**
```sql
SELECT status, COUNT(*) as order_count
FROM orders
GROUP BY status
ORDER BY order_count DESC;
```

**Query 8: Products by category**
```sql
SELECT category, 
       COUNT(*) as product_count,
       ROUND(AVG(price), 2) as avg_price
FROM products
GROUP BY category
ORDER BY product_count DESC;
```

### Step 4.6: Join Queries

**Query 9: Orders with customer details**
```sql
SELECT o.order_id, 
       o.order_date,
       o.status,
       c.first_name,
       c.last_name,
       c.city,
       c.state
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
LIMIT 20;
```

**Query 10: Orders with product details**
```sql
SELECT o.order_id,
       o.order_date,
       o.quantity,
       o.status,
       p.product_name,
       p.category,
       p.price,
       ROUND(o.quantity * p.price, 2) as order_total
FROM orders o
JOIN products p ON o.product_id = p.product_id
LIMIT 20;
```

### Step 4.7: Advanced Analytics

**Query 11: Top 10 customers by order count**
```sql
SELECT c.customer_id,
       c.first_name || ' ' || c.last_name as customer_name,
       c.city,
       c.state,
       COUNT(o.order_id) as total_orders
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name, c.city, c.state
ORDER BY total_orders DESC
LIMIT 10;
```

**Query 12: Revenue by product category**
```sql
SELECT p.category,
       COUNT(DISTINCT o.order_id) as total_orders,
       SUM(o.quantity) as total_quantity,
       ROUND(SUM(o.quantity * p.price), 2) as total_revenue
FROM orders o
JOIN products p ON o.product_id = p.product_id
WHERE o.status = 'Completed'
GROUP BY p.category
ORDER BY total_revenue DESC;
```

**Query 13: Top 10 products by revenue**
```sql
SELECT p.product_name,
       p.category,
       p.price,
       COUNT(o.order_id) as times_ordered,
       SUM(o.quantity) as total_quantity_sold,
       ROUND(SUM(o.quantity * p.price), 2) as total_revenue
FROM products p
JOIN orders o ON p.product_id = o.product_id
WHERE o.status = 'Completed'
GROUP BY p.product_id, p.product_name, p.category, p.price
ORDER BY total_revenue DESC
LIMIT 10;
```

**Query 14: Customer lifetime value (top 20)**
```sql
SELECT c.customer_id,
       c.first_name || ' ' || c.last_name as customer_name,
       c.email,
       c.city,
       c.state,
       COUNT(o.order_id) as total_orders,
       ROUND(SUM(o.quantity * p.price), 2) as lifetime_value
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN products p ON o.product_id = p.product_id
WHERE o.status = 'Completed'
GROUP BY c.customer_id, c.first_name, c.last_name, c.email, c.city, c.state
ORDER BY lifetime_value DESC
LIMIT 20;
```

**Query 15: Revenue by state**
```sql
SELECT c.state,
       COUNT(DISTINCT o.order_id) as total_orders,
       COUNT(DISTINCT c.customer_id) as unique_customers,
       ROUND(SUM(o.quantity * p.price), 2) as total_revenue
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = p.product_id
WHERE o.status = 'Completed'
GROUP BY c.state
ORDER BY total_revenue DESC;
```

---

## Part 5: Lake Formation (Optional - 15 minutes)

### Step 5.1: Register S3 Location

1. Go to **AWS Lake Formation** console
2. In left menu, click **"Data lake locations"**
3. Click **"Register location"**
4. S3 path: `s3://lf-workshop-data-YOUR_ACCOUNT_ID/`
5. IAM role: **LFWorkshopGlueRole**
6. Click **"Register location"**

### Step 5.2: Grant Permissions

1. In Lake Formation, click **"Databases"** in left menu
2. Select `lf_workshop_student`
3. Click **"Actions"** → **"Grant"**
4. IAM users and roles: Select your student user
5. Database permissions: **Describe**
6. Table permissions: **Select**, **Describe**
7. Click **"Grant"**

---

## Lab Validation

### Checklist

- [ ] S3 bucket created with 3 folders
- [ ] 3 CSV files uploaded (customers, orders, products)
- [ ] Glue database created: `lf_workshop_student`
- [ ] Glue crawler created and run successfully
- [ ] 3 tables visible in Glue Data Catalog
- [ ] Athena query result location configured
- [ ] Successfully ran at least 5 Athena queries
- [ ] Able to join tables and perform analytics

### Expected Results

| Query | Expected Result |
|-------|----------------|
| `SELECT COUNT(*) FROM customers` | 5000 |
| `SELECT COUNT(*) FROM orders` | 20000 |
| `SELECT COUNT(*) FROM products` | 500 |
| Top state by customers | CA or NY (varies) |
| Most common order status | ~5000 per status |

---

## Troubleshooting

### Issue: Athena "Run" button is greyed out
**Solution:** Set query result location in Settings

### Issue: Tables not showing in Athena
**Solution:** 
1. Verify database is selected: `lf_workshop_student`
2. Refresh the page
3. Check Glue crawler completed successfully

### Issue: Query returns 0 results
**Solution:**
1. Verify S3 files are in correct folders
2. Check table locations in Glue (should point to directories, not files)
3. Re-run crawler

### Issue: Access Denied errors
**Solution:**
1. Verify IAM role `LFWorkshopGlueRole` has S3 permissions
2. Check Lake Formation permissions

---

## Cleanup (After Lab Completion)

### Step 1: Delete Athena Query Results
```bash
aws s3 rm s3://lf-workshop-data-YOUR_ACCOUNT_ID/athena-results/ --recursive
```

### Step 2: Delete Glue Resources
1. Delete crawler: `lf-workshop-student-crawler`
2. Delete tables: `customers`, `orders`, `products`
3. Delete database: `lf_workshop_student`

### Step 3: Delete S3 Bucket
1. Empty bucket first (delete all objects)
2. Delete bucket: `lf-workshop-data-YOUR_ACCOUNT_ID`

---

## Key Takeaways

✅ **AWS Glue** automatically discovers data schemas using crawlers  
✅ **AWS Glue Data Catalog** stores metadata for easy data discovery  
✅ **Amazon Athena** enables SQL queries on S3 data without servers  
✅ **AWS Lake Formation** provides centralized data lake governance  
✅ **S3** serves as scalable, cost-effective data lake storage  

---

## Additional Resources

- [AWS Glue Documentation](https://docs.aws.amazon.com/glue/)
- [Amazon Athena Documentation](https://docs.aws.amazon.com/athena/)
- [AWS Lake Formation Documentation](https://docs.aws.amazon.com/lake-formation/)
- [Athena SQL Reference](https://docs.aws.amazon.com/athena/latest/ug/ddl-sql-reference.html)

---

## Lab Complete! 🎉

You have successfully:
- Built a data lake on S3
- Cataloged data using AWS Glue
- Queried data using Amazon Athena
- Performed analytics on 25,000+ records

**Estimated Cost:** $0.03 - $0.10 per student

---

**Questions or Issues?**  
Contact your instructor or refer to the troubleshooting section above.

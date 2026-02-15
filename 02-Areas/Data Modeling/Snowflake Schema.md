---
tags: [data-modeling, snowflake, data-warehouse, dimensional-modeling, cloud-warehouse]
date: 2026-02-15
status: learning
---

# Snowflake Schema

## Overview

Snowflake Schema is a dimensional modeling technique used in Snowflake cloud data warehouse. It's an extension of traditional star schema with Snowflake-specific features like clustering keys, variant columns, and zero-copy time travel.

**Key Insight**: Snowflake schema = Star schema + automatic optimization

---

## Core Concepts

### Star Schema Fundamentals

Snowflake uses traditional star schema principles:
- **Fact Table**: Central table containing measures and foreign keys
- **Dimension Tables**: Descriptive tables (who, what, where, when)
- **Kimball Methodology**: Dimension design approach (conformed dimensions, bus matrices)

### Snowflake Extensions

| Feature | Traditional Star Schema | Snowflake Schema | Benefit |
|---------|---------------------|------------------|----------|
| **Micro-partitioning** | Manual partitioning | Automatic clustering keys | Automatic data pruning |
| **Zero-Copy Cloning** | Duplicate tables | Zero-copy cloning | Instant table copies |
| **Time Travel** | Slow query | Instant | Query historical data |
| **Variant Columns** | Single data type | VARIANT (JSON, XML, etc.) | Semi-structured data |
| **Result Caching** | No caching | Automatic | Query performance boost |

---

## Snowflake vs. Traditional Star Schema

```sql
-- Traditional Star Schema (Redshift, BigQuery)
CREATE TABLE fact_sales (
    sales_id BIGINT PRIMARY KEY,
    date_key INT,
    product_key INT,
    customer_key INT,
    sales_amount DECIMAL(10,2),
    quantity INT,
    discount_percent DECIMAL(5,2)
);

-- Snowflake Schema (with clustering key)
CREATE OR REPLACE TABLE fact_sales_snowflake (
    sales_id BIGINT PRIMARY KEY,
    date_key INT,
    product_key INT,
    customer_key INT,
    sales_amount DECIMAL(10,2),
    quantity INT,
    discount_percent DECIMAL(5,2),
    -- Snowflake clustering key for micro-partitioning
    CLUSTERING KEY (date_key, product_key, store_key)
);
```

---

## Snowflake-Specific Features

### 1. Clustering Keys

**Definition**: Cluster key automatically partitions data for better performance.

**Benefits**:
- Automatic micro-partitioning
- Data pruning (skip irrelevant partitions)
- Improved query performance

**SQL Example**:
```sql
-- Create table with clustering key
CREATE OR REPLACE TABLE fact_orders (
    order_id BIGINT PRIMARY KEY,
    order_date DATE,
    customer_id INT,
    product_id INT,
    amount DECIMAL(10,2),
    store_id INT
    CLUSTER BY (order_date, customer_id)  -- Clustering key
);

-- Query with clustering (uses micro-partitions)
SELECT
    DATE_TRUNC('MONTH', order_date) AS order_month,
    customer_id,
    SUM(amount) AS total_sales,
    COUNT(*) AS order_count
FROM fact_orders
WHERE order_date >= '2024-01-01'
GROUP BY order_month, customer_id;

-- Snowflake automatically uses clustering to prune partitions
```

### 2. Variant Columns

**Definition**: Store semi-structured data (JSON, XML, arrays).

**SQL Example**:
```sql
-- Create table with VARIANT column
CREATE OR REPLACE TABLE orders (
    order_id BIGINT PRIMARY KEY,
    customer_info VARIANT,  -- Semi-structured (JSON)
    order_items VARIANT,     -- Array
    metadata VARIANT
);

-- Insert JSON data
INSERT INTO orders (order_id, customer_info)
VALUES (
    123,
    PARSE_JSON('{"name": "Alice", "email": "alice@example.com", "preferences": {"theme": "dark"}}')
);

-- Flatten and query VARIANT
SELECT
    order_id,
    customer_info:name AS customer_name,
    customer_info:email AS customer_email,
    customer_info:preferences:theme AS theme
FROM orders;

-- Query array elements
SELECT
    order_id,
    VALUE AS item_value,
    INDEX AS item_index
FROM orders,
    LATERAL FLATTEN(input => VALUE) AS items
WHERE order_id = 123;
```

### 3. Zero-Copy Cloning

**Definition**: Instant table duplication without data copying.

**SQL Example**:
```sql
-- Original table
CREATE OR REPLACE TABLE orders_current (
    order_id BIGINT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    amount DECIMAL(10,2),
    status VARCHAR(20)
);

-- Zero-copy clone (instant)
CREATE OR REPLACE TABLE orders_archive CLONE orders_current;

-- Clone with modifications
CREATE OR REPLACE TABLE orders_backup CLONE orders_current
WITH (
    -- Add computed column
    total_amount DECIMAL(10,2) AS (amount * 1.10) AS tax_amount
);

-- Query from any of them (instant switch)
SELECT * FROM orders_current WHERE order_date = '2024-01-15';
SELECT * FROM orders_archive WHERE order_date = '2024-01-15';
SELECT * FROM orders_backup WHERE order_date = '2024-01-15';
```

### 4. Time Travel

**Definition**: Query historical data at any point in time.

**SQL Example**:
```sql
-- Query what the order looked like 7 days ago
SELECT * FROM orders AT(TIMESTAMP => TO_TIMESTAMP('2024-01-08')) AS orders_7_days_ago;

-- Query what the order will be in 7 days
SELECT * FROM orders AT(TIMESTAMP => TO_TIMESTAMP(FORMADD(DAYS, CURRENT_DATE(), 7))) AS orders_7_days_future;

-- Compare current state with historical state
SELECT
    current.id AS order_id,
    current.amount AS current_amount,
    historical.amount AS historical_amount,
    (current.amount - historical.amount) AS difference
FROM orders current
INNER JOIN orders_historical AS historical
    ON current.id = historical.id
WHERE historical.created_at = TIMESTAMP('2024-01-01 00:00:00');
```

---

## Snowflake Schema Architecture

```
                    ┌──────────────────────────────────┐
                    │  Snowflake Data Warehouse      │
                    └────────┬───────────────────────┘
                               │
                ┌──────────────┴──────────────┐
                │                              │
        ┌──────▼──────┐        ┌─────────────┴───────────────┐
        │  Fact Tables │        │   Dimension Tables        │
        │  (Sales, Orders)│        │   (Customer, Product, Date)│
        └──────┬───────┘        └──────┬────────────────────┘
                │                              │
        ┌───────────────────────────────────────┴──────────────┐
        │         Data Sharing (Snowflake)               │
        └──────────────────────────────────────────────────┘
```

---

## Snowflake-Specific Data Types

### 1. GEOGRAPHY

**Definition**: Stores geospatial data (points, polygons, linestrings).

**SQL Example**:
```sql
-- Create table with GEOGRAPHY column
CREATE OR REPLACE TABLE stores (
    store_id BIGINT PRIMARY KEY,
    store_name VARCHAR(100),
    location GEOGRAPHY,  -- Snowflake geospatial type
    region VARCHAR(50),
    country VARCHAR(50)
);

-- Insert geospatial data (WKT format)
INSERT INTO stores (store_id, store_name, location)
VALUES (
    123,
    'Store A',
    TO_GEOGRAPHY('POINT(-122.4194, 37.7749)')  -- San Francisco coordinates
);

-- Query nearby stores
SELECT
    s.store_name,
    ST_DWITHIN(s.location, ST_MAKEPOINT(-122.4219, 37.7759), 10000)  -- 10km radius
    ST_DISTANCE(s.location, ST_MAKEPOINT(-122.4219, 37.7759)) AS distance_km
FROM stores s
WHERE s.region = 'CA';
```

### 2. VARIANT (Semi-Structured)

**Definition**: Flexible column type for JSON, XML, arrays.

**SQL Example**:
```sql
-- Insert and query JSON
CREATE OR REPLACE TABLE events (
    event_id BIGINT PRIMARY KEY,
    event_data VARIANT,  -- JSON payload
    event_timestamp TIMESTAMP_LTZ(9)
);

-- Insert JSON event
INSERT INTO events (event_id, event_data, event_timestamp)
VALUES (
    456,
    PARSE_JSON('{"type": "click", "user_id": 101, "properties": {"source": "mobile", "browser": "chrome"}}'),
    TIMESTAMP_FROM_PARTS(UNIX_TIMESTAMP(1705251200000), 3)
);

-- Query JSON properties
SELECT
    event_id,
    event_data:type AS event_type,
    event_data:user_id AS user_id,
    event_data:properties:source AS traffic_source,
    event_data:properties:browser AS user_browser
FROM events
WHERE event_timestamp > TIMESTAMP('2024-01-15 00:00:00');
```

### 3. ARRAY

**Definition**: Store repeated elements efficiently.

**SQL Example**:
```sql
-- Insert array data
CREATE OR REPLACE TABLE customer_orders (
    customer_id BIGINT PRIMARY KEY,
    order_ids ARRAY  -- Snowflake array type
);

INSERT INTO customer_orders (customer_id, order_ids)
VALUES (101, [123, 124, 125, 128]);

-- Query array elements (ARRAY_CONTAINS)
SELECT
    customer_id,
    INDEX(order_ids) AS order_index,
    VALUE AS order_id
FROM customer_orders
WHERE ARRAY_CONTAINS(order_ids, 127) AND customer_id = 101;

-- Flatten array to rows (LATERAL FLATTEN)
SELECT
    customer_id,
    VALUE AS order_id
FROM customer_orders,
    LATERAL FLATTEN(input => order_ids) AS flat
WHERE customer_id = 101;
```

---

## Snowflake vs. Star Schema Comparison

| Feature | Traditional Star Schema | Snowflake Schema |
|---------|---------------------|------------------|
| **Partitioning** | Manual (range/hash) | Automatic (cluster keys) |
| **Cloning** | Duplicate data (slow) | Zero-copy (instant) |
| **Time Travel** | Slow query | Instant |
| **Semi-Structured Data** | String columns | VARIANT columns (JSON, XML) |
| **Geospatial** | Latitude/longitude strings | GEOGRAPHY type (ST_DWITHIN, ST_DISTANCE) |
| **Optimization** | Manual tuning | Automatic (query cache, result cache) |
| **Performance** | Good | Better (auto-scaling) |

---

## Python: Snowflake Connector

### Installing Snowflake Python

```bash
# Install Snowflake Python connector
pip install snowflake-connector-python
pip install snowflake-sqlalchemy
```

### Python: Load to Snowflake

```python
import snowflake.connector
import pandas as pd
from datetime import datetime

# Snowflake connection
conn = snowflake.connector.connect(
    user='your_user',
    password='your_password',
    account='your_account',
    warehouse='your_warehouse',
    role='your_role',
    database='your_database',
    schema='public'
)

# Upload DataFrame to Snowflake
df_orders = pd.read_csv('sales_data.csv')

# Write to Snowflake table (auto-creates or replaces)
success, nchunks, nrows, _ = conn.write_pandas(
    df_orders,
    'orders',  # Table name
    quote_identifiers=False  # Don't quote column names
)

print(f"Uploaded {nrows} rows to Snowflake. Success: {success}")
```

### Python: Query Snowflake

```python
# Execute query
cur = conn.cursor()

# Simple query
cur.execute("SELECT * FROM orders WHERE order_date >= '2024-01-01' LIMIT 10")

results = cur.fetchall()
for row in results:
    print(f"Order {row['ORDER_ID']}: ${row['AMOUNT']}")

# Fetch DataFrame
df_results = cur.fetch_pandas_all()
print(f"Total: {len(df_results)} orders")
```

### Python: Stored Procedures

```python
# Call Snowflake stored procedure
cur = conn.cursor()

# Call procedure with parameters
cur.callproc(
    'calculate_monthly_metrics',
    ('2024-01-01', '2024-01-31', 'sales')
)

# Get results
results = cur.fetchall()
print(f"Monthly metrics calculated: {len(results)} rows")
```

---

## SQL Advanced Patterns

### 1. Multi-Table Joins with Clustering

```sql
-- Join multiple fact tables (each with clustering key)
SELECT
    DATE_TRUNC('MONTH', s.order_date) AS month,
    s.customer_id,
    SUM(s.amount) AS sales_total
FROM fact_sales s
WHERE s.order_date >= '2024-01-01'
GROUP BY month, s.customer_id;

JOIN (SELECT
    r.customer_id,
    SUM(r.amount) AS returns_total
FROM fact_returns r
WHERE r.order_date >= '2024-01-01'
GROUP BY r.customer_id) ON s.customer_id = r.customer_id
HAVING sales_total < returns_total;
```

### 2. Hierarchical Data (HIERARCHY)

**Definition**: Store and query hierarchical data (parent-child relationships).

**SQL Example**:
```sql
-- Create table with HIERARCHY column
CREATE OR REPLACE TABLE organization_structure (
    employee_id BIGINT PRIMARY KEY,
    employee_name VARCHAR(100),
    manager_id BIGINT,    -- Self-reference
    department_id BIGINT,
    HIERARCHY (manager_id)  -- Snowflake hierarchical function
);

-- Insert data (manager_id references employee_id for CEO)
INSERT INTO organization_structure (employee_id, employee_name, manager_id, department_id)
VALUES
    (1, 'CEO', NULL, 1),  -- CEO has no manager
    (2, 'VP Sales', 1, 1),
    (3, 'Manager Sales', 2, 1),
    (4, 'Sales Associate 1', 3, 1);

-- Query hierarchical data
-- Get all descendants of VP Sales (recursive)
SELECT
    employee_id,
    employee_name,
    level
FROM organization_structure
WHERE HIERARCHY(2) AND department_id = 1;  -- All under VP Sales

-- Get immediate children of Manager Sales (single level)
SELECT
    employee_id,
    employee_name,
    level
FROM organization_structure
WHERE manager_id = 3 AND level = 1;
```

### 3. Merge with Matching (MATCH_RECOGNIZE)

**Definition**: Merge data and identify duplicates.

**SQL Example**:
```sql
-- Upsert orders (insert new, update existing)
MERGE INTO target_orders AS target
USING staging_orders AS source
ON target.order_id = source.order_id
WHEN MATCHED THEN
    UPDATE SET
        amount = source.amount,
        updated_at = CURRENT_TIMESTAMP()
WHEN NOT MATCHED THEN
    INSERT (order_id, customer_id, amount, created_at)
    VALUES (source.order_id, source.customer_id, source.amount, CURRENT_TIMESTAMP());
```

---

## Performance Optimization

### 1. Automatic Clustering

**Benefits**:
- Snowflake automatically creates micro-partitions
- Data pruning: Skip irrelevant partitions during queries
- Reduced I/O: Scan only relevant data

### 2. Result Caching

**How It Works**:
- Query results cached for 24 hours
- Same query: instant (from cache)
- Schema change: cache invalidated

**SQL Example**:
```sql
-- First run: Cache miss (scan table)
SELECT * FROM large_fact_table WHERE date >= '2024-01-01';

-- Second run (within 24h): Cache hit (no scan)
SELECT * FROM large_fact_table WHERE date >= '2024-01-01';

-- Force cache refresh (if schema changed)
ALTER TABLE large_fact_table CLUSTER BY (date_key, customer_key);
```

### 3. Materialized Views

**Benefits**:
- Pre-computed results
- Faster queries
- Reduced complexity

**SQL Example**:
```sql
-- Create materialized view (Snowflake MV)
CREATE OR REPLACE MATERIALIZED VIEW monthly_sales_mv AS
SELECT
    DATE_TRUNC('MONTH', order_date) AS month,
    customer_id,
    SUM(amount) AS total_sales,
    COUNT(*) AS order_count
FROM fact_orders
GROUP BY DATE_TRUNC('MONTH', order_date), customer_id;

-- Snowflake automatically refreshes MV (or manual)
-- Query MV (instant)
SELECT * FROM TABLE(monthly_sales_mv) WHERE month = '2024-01';
```

---

## Security & Governance

### 1. Role-Based Access Control (RBAC)

**SQL Example**:
```sql
-- Grant access to specific roles
GRANT SELECT ON fact_sales TO ROLE analysts;
GRANT INSERT ON fact_sales TO ROLE etl_users;

-- Grant access on specific schema
GRANT ALL ON SCHEMA sales TO ROLE sales_manager;

-- Grant access on specific table
GRANT SELECT ON dim_customer TO ROLE support_team (email LIKE '%@company.com');
```

### 2. Row Access Policies

**Definition**: Fine-grained control over which rows a user can access.

**SQL Example**:
```sql
-- Create row access policy
CREATE OR REPLACE ROW ACCESS POLICY customer_data_policy
ON dim_customer
AS (email_column) CASE
    -- Only allow access to customer data where email matches
    WHEN current_email() LIKE email_column THEN 'ALLOW_ROW_ACCESS'
    ELSE 'DENY_ROW_ACCESS';

-- Apply policy to roles
GRANT ROW ACCESS POLICY customer_data_policy ON dim_customer TO ROLE analyst_role;
GRANT ROW ACCESS POLICY customer_data_policy ON dim_customer TO ROLE manager_role;

-- Remove policy
DROP ROW ACCESS POLICY customer_data_policy ON dim_customer;
```

---

## Data Sharing (Snowflake)

### 1. Create Share

**SQL Example**:
```sql
-- Create share (database level)
CREATE OR REPLACE SHARE sales_analytics
COMMENT = 'Share sales data with external partners';

-- Grant access to share
GRANT USAGE ON SHARE sales_analytics TO ACCOUNT partner_account;

-- Grant access to specific databases/tables
GRANT SELECT ON SCHEMA public TO SHARE sales_analytics;
GRANT SELECT ON TABLE fact_sales TO SHARE sales_analytics;

-- Grant read-only on specific table
GRANT SELECT ON COLUMN fact_sales.sales_amount TO SHARE sales_analytics;
```

### 2. Consume from Share

**SQL Example** (Partner Account):
```sql
-- Use shared data
SELECT
    s.order_id,
    s.customer_id,
    s.amount
FROM partner_db.public.fact_sales s;
```

### 3. Secure Data Exchange

**SQL Example**:
```sql
-- List shares
SHOW SHARES;

-- Show access granted
SHOW GRANTS ON SHARE sales_analytics;

-- Revoke access
REVOKE SELECT ON SCHEMA public FROM SHARE sales_analytics FROM ACCOUNT partner_account;
```

---

## Interview Questions

**Q1: What's the difference between Snowflake schema and traditional star schema?**

**A1**:
- **Traditional Star Schema**: Standard dimensional modeling with fact and dimension tables. Requires manual partitioning, tuning, and maintenance.
- **Snowflake Schema**: Extended star schema with Snowflake-specific features:
  - **Clustering Keys**: Automatic micro-partitioning for performance
  - **Zero-Copy Cloning**: Instant table duplication
  - **Variant Columns**: Support for JSON, XML, arrays
  - **Time Travel**: Instant historical data queries
  - **Result Caching**: Automatic query performance boost
  - **Automatic Scaling**: Compute and storage auto-scale

**Q2: What is a clustering key in Snowflake?**

**A2**: A clustering key is a subset of table columns (usually 1-3) that Snowflake uses to automatically co-locate and sort data in micro-partitions. This improves query performance by:
- Pruning irrelevant micro-partitions
- Reducing I/O
- Improving cache hit rate

**Q3: What's the VARIANT data type in Snowflake?**

**A3**: VARIANT is a flexible data type in Snowflake that can store semi-structured data (JSON, XML, Avro) and arrays. Unlike traditional data types (VARCHAR, INT), VARIANT allows:
- **Flexible Schema**: Different rows can have different structures
- **Native Parsing**: Snowflake has built-in functions (PARSE_JSON) to extract data
- **Array Support**: Efficient storage and querying of repeated elements
- **Column Variation**: Easier schema evolution

**Q4: What's zero-copy cloning in Snowflake?**

**A4**: Zero-copy cloning is a Snowflake feature that creates an instant copy of a table by copying only metadata (pointers to micro-partitions) without duplicating the actual data files. This is extremely fast (seconds for any table size) and requires no additional storage space. Traditional cloning would copy both data and metadata, which is slow and doubles storage.

**Q5: How does Snowflake handle slowly changing dimensions (SCDs)?**

**A5**: Snowflake handles SCDs through:
- **Time Travel**: Query data at any point in time (what was true on 2024-01-15?) without keeping full history
- **Zero-Copy Cloning**: Create snapshots of dimension tables
- **Merged Views**: Materialize dimension snapshots (current, historical)
- **Streaming Ingestion**: Use Snowpipe or Snowflake Streams for real-time updates
- **Update Logic**: `MERGE` statements (upsert) for handling changes
- **Partitioning**: Use clustering keys to separate current from historical data

**Q6: What's data sharing in Snowflake?**

**A6**: Data sharing in Snowflake is a feature that allows you to securely share selected data objects (databases, schemas, tables, or specific columns) with other Snowflake accounts without moving the data. It's used for:
- **Multi-Tenant Collaboration**: Share data with partners, customers
- **Data Marketplace**: Sell data products
- **B2B Integration**: Share data with external platforms
- **Cross-Cloud Collaboration**: Share data between different cloud accounts

---

## Related Notes

- [[Star Schema]] - Traditional dimensional modeling
- [[Data Modeling]] - Core dimensional modeling concepts
- [[Slowly Changing Dimensions SCDs]] - Handling historical changes
- [[Data Warehousing]] - Warehouse architecture context
- [[System Design]] - Scalability, partitioning, caching

---

## Resources

- [Snowflake Schema Overview - Databricks](https://www.databricks.com/glossary/snowflake-schema)
- [Snowflake Schema in Data Modeling - OWOX](https://www.owox.com/blog/articles/snowflake-schema-data-modeling)
- [Snowflake Schema Use Cases - Microsoft Fabric](https://community.fabric.microsoft.com/t5/Desktop/snowflake-schema-use-cases/td-p/3988503)
- [Modernizing Data Warehousing with Snowflake and Hybrid Data Vault](https://www.snowflake.com/en/blog/modernizing-data-warehousing-snowflake-and-hybrid-data-vault)
- [hierarchical modeling data warehouse snowflake or star - Stack Overflow](https://stackoverflow.com/questions/27653047/hierarchical-modeling-data-warehouse-snowflake-or-star)

---

**Progress**: 🟡 Learning (concepts understood, need hands-on practice)

**Next Steps**:
- [ ] Practice Snowflake Python connector (install, connect, upload)
- [ ] Implement clustering keys on fact tables
- [ ] Use VARIANT columns for semi-structured data
- [ ] Practice time travel queries (historical data)
- [ ] Create materialized views for frequent aggregations
- [ ] Set up data sharing with external accounts
- [ ] Implement row access policies for fine-grained security

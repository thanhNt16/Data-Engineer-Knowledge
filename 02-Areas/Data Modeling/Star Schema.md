---
tags: [data-modeling, star-schema, dimensional-modeling, warehouse, snowflake]
date: 2026-02-15
status: learning
---

# Star Schema

## Overview

The Star Schema is a dimensional modeling technique used in data warehouses to optimize query performance and simplify business intelligence (BI) reporting.

**Key Insight**: Star schema separates business processes (fact tables) from descriptive attributes (dimension tables).

---

## Core Components

### Fact Table

**Definition**: Central table containing quantitative, transactional data (events).

**Characteristics**:
- Contains foreign keys to dimension tables
- Contains numeric measures (facts)
- Large (millions/billions of rows)
- Contains additive measures (can be summed)

**Example**:
```sql
CREATE TABLE fact_sales (
    sales_id BIGINT PRIMARY KEY,
    date_key INT REFERENCES dim_date(date_key),
    product_key INT REFERENCES dim_product(product_key),
    customer_key INT REFERENCES dim_customer(customer_key),
    store_key INT REFERENCES dim_store(store_key),
    sales_amount DECIMAL(10,2) NOT NULL,  -- Measure
    quantity INT NOT NULL,                     -- Measure
    discount_percent DECIMAL(5,2)           -- Measure
    transaction_count INT                    -- Measure
);
```

### Dimension Tables

**Definition**: Descriptive tables containing textual attributes (who, what, where, when).

**Characteristics**:
- Contains descriptive attributes (name, address, category)
- Smaller (thousands/hundreds of rows)
- Contains slowly changing data (type 2 SCDs)
- Joined to fact tables via foreign keys

**Common Dimensions**:

| Dimension | Description | Example Attributes |
|-----------|-------------|---------------------|
| **Customer** | Who purchased | name, email, address, tier |
| **Product** | What was purchased | name, category, price, brand |
| **Date** | When it occurred | date, month, quarter, year |
| **Store** | Where it was purchased | location, region, store_type |
| **Time** | When (time-of-day) | hour, minute, second |

**Example**:
```sql
CREATE TABLE dim_customer (
    customer_key BIGINT PRIMARY KEY,
    customer_name VARCHAR(100),
    email VARCHAR(255),
    phone VARCHAR(20),
    city VARCHAR(50),
    state VARCHAR(50),
    country VARCHAR(50),
    postal_code VARCHAR(10)
);

CREATE TABLE dim_product (
    product_key BIGINT PRIMARY KEY,
    product_name VARCHAR(100),
    brand_name VARCHAR(100),
    category VARCHAR(50),
    subcategory VARCHAR(50),
    unit_price DECIMAL(10,2),
    product_description TEXT
);

CREATE TABLE dim_date (
    date_key INT PRIMARY KEY,
    full_date DATE NOT NULL,
    day_of_week INT,                -- 1-7
    month INT,                     -- 1-12
    quarter INT,                   -- 1-4
    year INT,                      -- 4-digit
    is_holiday BOOLEAN
);
```

---

## Star Schema Structure

```
            ┌───────────────────────┐
            │     dim_product     │
            │  (Product Dimension) │
            └───────────┬─────────┘
                        │
                        │
            ┌───────────▼──────────────┐
            │       dim_customer      │
            │   (Customer Dimension) │
            └───────────┬────────────┘
                        │
                        │
          ┌─────────────▼──────────────┐
          │      dim_store           │
          │   (Store Dimension)    │
          └─────────────┬─────────────┘
                        │
                        │
                      ┌─────────▼───────────┐
                      │  fact_sales           │
                      │ (Central Fact Table) │
                      └─────────────┬───────────┘
                                    │
                        │
                ┌───────────▼──────────────┐
                │    dim_date            │
                │ (Date Dimension)     │
                └──────────────────────┘
```

---

## Star Schema Advantages

| Advantage | Description |
|-----------|-------------|
| **Simplified Queries** | BI tools generate simple, fast SQL |
| **Performance** | Central fact table, minimal joins |
| **Maintenance** | Dimensions updated independently (SCD Type 2) |
| **Business Understanding** | Intuitive for business users |
| **Consistency** | Single source of truth for dimensions |
| **Flexibility** | Easy to add new dimensions |

---

## Star Schema Disadvantages

| Disadvantage | Description | Solution |
|--------------|-------------|-----------|
| **Data Redundancy** | Dimensions repeated across fact tables | Normalize to dimensions, accept denormalization |
| **Data Integrity** | No referential integrity across dimensions | Use foreign keys, constraints |
| **Load Complexity** | ETL requires joining multiple sources | Use staging tables, incremental loads |
| **Analysis Complexity** | Complex aggregations across stars | Use BI tools, star joins |
| **Changing Dimensions** | SCD management becomes complex | Implement Type 2 (Update existing) |

---

## Python Implementation: ETL

### Building Star Schema

```python
import pandas as pd
import sqlalchemy
from datetime import datetime
from typing import Dict, Any

class StarSchemaBuilder:
    def __init__(self, engine):
        self.engine = engine

    def load_raw_sales(self, file_path: str) -> pd.DataFrame:
        """Load raw sales data"""
        return pd.read_csv(file_path)

    def create_dimension_tables(self, df: pd.DataFrame) -> Dict[str, Any]:
        """Create and populate dimension tables"""
        dimensions = {}
        
        # Customer Dimension
        customers = df[['customer_name', 'email', 'phone', 'city', 'state', 'country', 'postal_code']].drop_duplicates()
        customers['customer_key'] = range(1, len(customers) + 1)
        dimensions['dim_customer'] = customers
        
        # Product Dimension
        products = df[['product_name', 'brand_name', 'category', 'subcategory', 'unit_price', 'product_description']].drop_duplicates()
        products['product_key'] = range(1, len(products) + 1)
        dimensions['dim_product'] = products
        
        # Date Dimension
        df['order_date'] = pd.to_datetime(df['order_date'])
        dates = df[['order_date']].drop_duplicates()
        dates['date_key'] = range(1, len(dates) + 1)
        dates['day_of_week'] = dates['order_date'].dt.dayofweek
        dates['month'] = dates['order_date'].dt.month
        dates['quarter'] = dates['order_date'].dt.quarter
        dates['year'] = dates['order_date'].dt.year
        dimensions['dim_date'] = dates
        
        # Store Dimension
        stores = df[['store_name', 'store_type', 'city', 'state', 'country']].drop_duplicates()
        stores['store_key'] = range(1, len(stores) + 1)
        dimensions['dim_store'] = stores
        
        return dimensions

    def create_fact_table(self, df: pd.DataFrame, dimensions: Dict[str, Any]) -> pd.DataFrame:
        """Create fact table by joining raw data with dimension keys"""
        # Add dimension keys to fact data
        df = df.merge(
            dimensions['dim_customer'][['customer_name', 'customer_key']],
            on='customer_name',
            how='left'
        )
        
        df = df.merge(
            dimensions['dim_product'][['product_name', 'product_key']],
            on='product_name',
            how='left'
        )
        
        df = df.merge(
            dimensions['dim_store'][['store_name', 'store_key']],
            on='store_name',
            how='left'
        )
        
        df = df.merge(
            dimensions['dim_date'][['order_date', 'date_key']],
            on='order_date',
            how='left'
        )
        
        # Select fact columns (measures + keys)
        fact_data = df[[
            'sales_id',
            'date_key',
            'product_key',
            'customer_key',
            'store_key',
            'sales_amount',
            'quantity',
            'discount_percent',
            'transaction_count'
        ]
        
        return fact_data

    def load_to_warehouse(self, df: pd.DataFrame, table_name: str) -> None:
        """Load data into warehouse"""
        df.to_sql(
            table_name,
            con=self.engine,
            if_exists='replace',
            index=False
        )
        print(f"Loaded {len(df)} rows into {table_name}")

    def run_etl(self, raw_file: str):
        """Complete ETL: Extract → Transform → Load"""
        # 1. Extract
        raw_data = self.load_raw_sales(raw_file)
        
        # 2. Transform (Create dimensions)
        dimensions = self.create_dimension_tables(raw_data)
        
        # 3. Transform (Create fact)
        fact_data = self.create_fact_table(raw_data, dimensions)
        
        # 4. Load
        self.load_to_warehouse(fact_data, 'fact_sales')
        for name, df in dimensions.items():
            self.load_to_warehouse(df, name)

# Usage
engine = sqlalchemy.create_engine('postgresql://user:password@localhost:5432/mydb')
etl = StarSchemaBuilder(engine)

etl.run_etl('sales_202401.csv')
print("Star Schema ETL completed")
```

---

## SQL Queries for Star Schema

### 1. Simple Star Join

```sql
-- Join fact table with dimensions
SELECT
    d.product_name,
    d.customer_name,
    d.store_name,
    d.full_date,
    f.sales_amount,
    f.quantity
FROM fact_sales f
JOIN dim_product d ON f.product_key = d.product_key
JOIN dim_customer c ON f.customer_key = c.customer_key
JOIN dim_store s ON f.store_key = s.store_key
JOIN dim_date dt ON f.date_key = dt.date_key
WHERE d.full_date BETWEEN '2024-01-01' AND '2024-01-31';
```

### 2. Aggregation with Star Schema

```sql
-- Revenue by product category (star join + aggregation)
SELECT
    dp.category,
    SUM(f.sales_amount) AS total_revenue,
    COUNT(*) AS order_count,
    AVG(f.sales_amount) AS avg_order_value
FROM fact_sales f
JOIN dim_product dp ON f.product_key = dp.product_key
JOIN dim_date dt ON f.date_key = dt.date_key
WHERE dt.year = 2024 AND dt.month = 1
GROUP BY dp.category
ORDER BY total_revenue DESC;
```

### 3. Time-Based Analysis

```sql
-- Monthly revenue trend (star schema + time dimension)
SELECT
    dt.year,
    dt.month,
    SUM(f.sales_amount) AS monthly_revenue,
    COUNT(*) AS monthly_orders
FROM fact_sales f
JOIN dim_date dt ON f.date_key = dt.date_key
GROUP BY dt.year, dt.month
ORDER BY dt.year, dt.month;

-- Quarter-over-quarter comparison
SELECT
    dt.quarter,
    SUM(f.sales_amount) AS quarterly_revenue
FROM fact_sales f
JOIN dim_date dt ON f.date_key = dt.date_key
GROUP BY dt.year, dt.quarter
ORDER BY dt.year, dt.quarter;
```

### 4. Top N Analysis

```sql
-- Top 10 customers by revenue
SELECT
    dc.customer_name,
    dc.email,
    SUM(f.sales_amount) AS total_spent
    COUNT(*) AS order_count
FROM fact_sales f
JOIN dim_customer dc ON f.customer_key = dc.customer_key
GROUP BY dc.customer_key, dc.customer_name, dc.email
ORDER BY total_spent DESC
LIMIT 10;
```

---

## Dimensional Modeling: Kimball Method

### 1. Identify Business Processes

**Processes**:
- Sales (order processing)
- Inventory (product tracking)
- Shipping (delivery)
- Returns (refund)

**Fact Tables**:
- fact_sales (sales transactions)
- fact_inventory (stock movements)
- fact_shipping (delivery events)
- fact_returns (refund events)

### 2. Identify Dimensions

**Dimensions**:
- Customer, Product, Store, Date, Time
- Salesperson, Promotion, Payment Method

### 3. Determine Grain

**Grain**: Lowest level of detail in the data warehouse.

**Example**:
- **Grain**: Order + Product + Store + Date
- **Not**: Product + Store (multiple orders across time)

### 4. Bus Matrix

**Bus**: Collection of related fact tables sharing dimensions.

**Example Bus Matrix**:

| Bus | Fact Tables | Shared Dimensions |
|-----|-------------|------------------|
| **Sales** | fact_sales, fact_returns | Customer, Product, Store, Date, Time |
| **Inventory** | fact_inventory | Product, Store, Date, Time |
| **Shipping** | fact_shipping | Store, Date, Time |

**Bus Matrix Diagram**:
```
Sales Bus: [Customer, Product, Store, Date, Time]
                ↓
        [fact_sales, fact_returns]

Inventory Bus: [Product, Store, Date, Time]
                ↓
        [fact_inventory]

Shipping Bus: [Store, Date, Time]
                ↓
        [fact_shipping]
```

---

## Performance Optimization

### 1. Partitioning

```sql
-- Partition fact table by date range
CREATE TABLE fact_sales_partitioned (
    sales_id BIGINT,
    date_key INT,
    sales_amount DECIMAL(10,2),
    quantity INT
) PARTITION BY RANGE (date_key);

-- Partition by year
CREATE TABLE fact_sales_2024 PARTITION OF fact_sales_partitioned
FOR VALUES FROM ('2024-01-01') TO ('2024-12-31');

-- Query specific partition (fast!)
SELECT * FROM fact_sales_2024
WHERE date_key BETWEEN 20240101 AND 20240131;
```

### 2. Indexing

```sql
-- Foreign key indexes (fact table)
CREATE INDEX idx_fact_sales_customer ON fact_sales(customer_key);
CREATE INDEX idx_fact_sales_product ON fact_sales(product_key);
CREATE INDEX idx_fact_sales_date ON fact_sales(date_key);
CREATE INDEX idx_fact_sales_store ON fact_sales(store_key);

-- Dimension table indexes (frequent queries)
CREATE INDEX idx_dim_product_name ON dim_product(product_name);
CREATE INDEX idx_dim_customer_email ON dim_customer(email);
CREATE INDEX idx_dim_store_location ON dim_store(city, state);
```

### 3. Denormalization Strategies

```sql
-- Option 1: Include frequently joined columns in fact
CREATE TABLE fact_sales_denormalized AS
SELECT
    f.sales_id,
    f.date_key,
    f.product_key,
    dp.product_name,          -- Denormalized from dimension
    dp.category,
    dp.brand_name,
    f.customer_key,
    dc.customer_name,         -- Denormalized from dimension
    dc.email,
    f.store_key,
    ds.store_name,
    ds.store_type,           -- Denormalized from dimension
    f.sales_amount,
    f.quantity
FROM fact_sales f
JOIN dim_product dp ON f.product_key = dp.product_key
JOIN dim_customer dc ON f.customer_key = dc.customer_key
JOIN dim_store ds ON f.store_key = ds.store_key;

-- Option 2: Materialized view (snapshot)
CREATE MATERIALIZED VIEW mv_sales_product_category AS
SELECT
    f.date_key,
    dp.category,
    SUM(f.sales_amount) AS category_revenue
FROM fact_sales f
JOIN dim_product dp ON f.product_key = dp.product_key
GROUP BY f.date_key, dp.category;

-- Query materialized view (very fast)
SELECT * FROM mv_sales_product_category
WHERE date_key BETWEEN 20240101 AND 20240131;
```

---

## Snowflake vs. Traditional Star Schema

### Snowflake Extensions

| Feature | Traditional Star Schema | Snowflake Schema |
|---------|---------------------|------------------|
| **Clustering Keys** | None | Primary key + cluster key (automatic micro-partitioning) |
| **Variant Columns** | No | Semi-structured data (JSON, XML) |
| **Time Travel** | Manual SCD | Automatic time travel with zero-copy cloning |
| **Performance** | Standard | Automatic optimization (result caching) |
| **Data Types** | Structured only | Flexible (VARIANT, OBJECT, ARRAY) |

### Snowflake Example

```sql
-- Snowflake star schema with clustering key
CREATE OR REPLACE TABLE fact_orders_snowflake (
    order_id BIGINT,
    order_date DATE NOT NULL,
    product_key INT,
    customer_key INT,
    store_key INT,
    amount DECIMAL(10,2),
    quantity INT,
    CLUSTER BY (order_date, store_key, product_key)  -- Snowflake clustering
);

-- Load data (Snowflake COPY)
COPY INTO fact_orders_snowflake
FROM @my_stage/orders.csv;

-- Time travel (what was true on 2024-01-15?)
SELECT * FROM fact_orders_snowflake
AT(TIMESTAMP => TO_TIMESTAMP('2024-01-15 00:00:00'));

-- Compare historical with current
SELECT
    h.order_date AS historical_date,
    h.amount AS historical_amount,
    c.order_date AS current_date,
    c.amount AS current_amount
FROM fact_orders_snowflake h
JOIN fact_orders_snowflake c ON h.order_id = c.order_id
WHERE h.order_date BETWEEN '2024-01-10' AND '2024-01-20';
```

---

## Interview Questions

**Q1: What is a star schema and when would you use it?**

**A1**: A star schema is a dimensional modeling technique used in data warehouses where a central fact table is connected to multiple dimension tables. It resembles a star when visualized, hence the name. I would use it when building a data warehouse for BI and reporting, especially when query performance is critical and you need to analyze data across multiple dimensions (who, what, where, when).

**Q2: What's the difference between a fact table and a dimension table?**

**A2**:
- **Fact Table**: Contains quantitative, transactional data (what happened). Usually large (millions/billions of rows), contains measures (sales amount, quantity, discount), and foreign keys to dimensions.
- **Dimension Table**: Contains descriptive, qualitative attributes (who, what, where). Usually smaller (thousands/hundreds of rows), contains textual data (name, email, address), and contains slowly changing data (SCD Type 2).

**Q3: What is a conformed dimension?**

**A3**: A conformed dimension is a dimension that has the same meaning across multiple fact tables in a data warehouse. For example, a "date" dimension used in "sales" fact table and "returns" fact table should have the same structure (columns, granularity). Conformed dimensions enable consistent analysis across fact tables and business processes.

**Q4: What's grain and why is it important?**

**A4**: Grain is the lowest level of detail that can be defined in the data warehouse. It's important because it determines the uniqueness of records in fact tables. For example, if the grain is "order_id", then each order is a single row. If the grain is "customer + date", then each customer can have only one order per day. Grain is critical for accurate aggregations and to avoid double-counting.

**Q5: What's the difference between star schema and snowflake schema?**

**A5**:
- **Star Schema**: Traditional dimensional modeling with separate fact and dimension tables. Joining fact with dimensions forms a star.
- **Snowflake Schema**: An extension of star schema for Snowflake cloud data warehouse. Key differences: Snowflake supports clustering keys (automatic micro-partitioning), variant columns (semi-structured data), automatic time travel, and built-in query optimization. Traditional star schemas don't have these features.

**Q6: How would you handle slowly changing dimensions (SCD Type 2) in a star schema?**

**A6**: I would implement SCD Type 2 (Update existing) for slowly changing dimensions. When a dimension attribute changes (e.g., customer moves, product price changes), I would update the dimension table row (set the old row's valid_to timestamp to NOW) and insert a new row with the new values as the current row. This preserves historical changes in the dimension table while keeping the fact table unchanged. The fact table would always reference the current row (is_current = TRUE).

**Q7: What's a bus matrix and how does it help in dimensional modeling?**

**A7**: A bus matrix is a tool that identifies which fact tables share dimensions. It's represented as a matrix where rows are business processes (sales, inventory, shipping) and columns are dimensions (customer, product, store, date). Bus matrices help organize the data warehouse by grouping related fact tables around shared dimensions. This promotes reusability (dimensions shared across fact tables) and consistency (common dimensions ensure accurate analysis). It prevents duplicate dimension tables (one "product" dimension used by all fact tables that reference products).

---

## Related Notes

- [[Data Modeling]] - Core dimensional modeling concepts
- [[Slowly Changing Dimensions SCDs]] - SCD implementation for dimensions
- [[Data Warehousing]] - Star schema in warehouse context
- [[Snowflake Schema]] - Cloud data warehouse star schema

---

## Resources

- [Star Schema - Wikipedia](https://en.wikipedia.org/wiki/Star_schema)
- [Designing the Star Schema in Data Warehousing - GeeksforGeeks](https://www.geeksforgeeks.org/dbms/designing-the-star-schema-in-data-warehouse-modeling/)
- [Dimensional Modeling Guide: Star Schema & Kimball Method](https://datadef.io/guides/dimensional-modeling/)
- [Snowflake Schema Overview - Databricks](https://www.databricks.com/glossary/snowflake-schema)

---

**Progress**: 🟡 Learning (concepts understood, need implementation practice)

**Next Steps**:
- [ ] Practice building star schema ETL pipelines
- [ ] Implement bus matrix for complex warehouse
- [ ] Create conformed dimensions across fact tables
- [ ] Optimize star schema queries (indexing, partitioning)
- [ ] Learn Snowflake schema extensions
- [ ] Practice grain identification for datasets

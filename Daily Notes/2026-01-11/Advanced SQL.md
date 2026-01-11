# Daily Note - 2026-01-11

## 🔍 Focus / Goal
Understand what "Advanced SQL" really means and learn the key techniques for data engineering interviews and real-world applications.

## 📚 Learning Notes

### Topics Covered
- What Advanced SQL actually encompasses (Data Modeling + SQL Techniques + Query Optimization)
- Six essential SQL techniques
- Query optimization strategies
- Five key table types for data modeling
- The 3-hop architecture pattern

### Key Takeaways

1. **Advanced SQL = Data Modeling + Effective Retrieval**
   - Not just knowing complex functions
   - Understanding how to structure data for efficient access
   - Combining modeling knowledge with SQL techniques

2. **Six Essential SQL Techniques**
   - Window Functions: Calculations across related rows
   - CTEs: Modular, readable query structure
   - JOIN Types: Inner, Left, Right, Full, Cross, Semi/Anti
   - MERGE INTO: Upsert operations (insert or update)
   - EXPLODE/CROSS JOIN: Working with nested arrays
   - Advanced Aggregates: Percentiles, hypothetical sets, statistical functions

3. **Query Optimization Core Principles**
   - **Reduce data processed**: Filter early, select only needed columns
   - **Increase resource utilization**: Understand partitioning and clustering
   - **Narrow vs Wide transformations**: Narrow = no shuffle (fast), Wide = shuffle (expensive)
   - **Avoid data skew**: Uneven distribution causes stragglers

4. **Partitioning & Clustering Strategy**
   - Partition on: High-cardinality, frequently filtered columns (dates, regions)
   - Avoid over-partitioning (too many small files)
   - Cluster within partitions on columns used in JOINs/WHERE clauses
   - Use query planner (`EXPLAIN`) to understand execution

5. **Five Key Table Types**
   - **Dimension**: Descriptive data (customers, products) - the "who" and "what"
   - **Fact**: Interactions between dimensions with metrics (orders, transactions)
   - **Bridge**: Many-to-many relationships (account-customer mappings)
   - **One Big Table (OBT)**: Fact + all dimensions joined - simple but expensive
   - **Summary/Aggregate**: Pre-calculated rollups for performance

6. **The 3-Hop Architecture**
   - Staging → Core → Serving
   - Staging: Raw data from sources
   - Core: Cleaned, modeled (facts/dimensions)
   - Serving: Optimized for consumption (OBTs, aggregates, marts)

## 💡 Insights & Connections

- **Related to**: [Advanced SQL](../Technical%20Skills/Advanced%20SQL.md), [Data Modeling](../Data%20Architecture%20Patterns/Data%20Modeling.md), [Data Warehouse](../Data%20Architecture%20Patterns/Data%20Warehouse.md)
- **Connection to yesterday's learning**: Data modeling principles (insert-only facts, snapshot dimensions) work hand-in-hand with advanced SQL techniques
- **Realization**: "Advanced SQL" job requirements are really testing for understanding of the full stack - from data structure to efficient retrieval
- **Questions raised**:
  - How do window functions perform on large datasets vs. self-joins?
  - When is OBT worth the storage cost vs. maintaining separate dimensions?
  - How does 3-hop architecture differ from medallion architecture (bronze/silver/gold)?
- **Ideas to explore**:
  - Learn more about handling data skew in distributed systems
  - Study cost-based optimizer internals
  - Compare nested data (arrays/structs) vs. normalized schema approaches

## 🔗 Resources
- [x] https://www.startdataengineering.com/post/advanced-sql/ - Advanced SQL: Data Modeling & Effective Retrieval
- [ ] https://www.startdataengineering.com/post/25-sql-techniques/ - 25 SQL Techniques
- [ ] https://www.startdataengineering.com/post/data-flow/ - Data Flow Best Practices

## 🛠️ Hands-on Practice

```sql
-- Window Function: Calculate running total by department
SELECT
    employee_id,
    department,
    salary,
    SUM(salary) OVER (
        PARTITION BY department
        ORDER BY hire_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as running_total,
    ROW_NUMBER() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) as salary_rank_in_dept
FROM employees;

-- CTE Chain: Find high-value customers with their orders
WITH customer_order_counts AS (
    -- Step 1: Aggregate order data
    SELECT
        customer_id,
        COUNT(*) as total_orders,
        SUM(amount) as lifetime_value
    FROM orders
    WHERE order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 12 MONTH)
    GROUP BY customer_id
),
high_value_customers AS (
    -- Step 2: Filter high-value segment
    SELECT customer_id, lifetime_value
    FROM customer_order_counts
    WHERE total_orders > 10 AND lifetime_value > 5000
)
-- Step 3: Join with customer details
SELECT
    c.customer_id,
    c.name,
    c.email,
    hvc.lifetime_value,
    hvc.total_orders
FROM customers c
INNER JOIN high_value_customers hvc ON c.customer_id = hvc.customer_id
ORDER BY hvc.lifetime_value DESC;

-- MERGE INTO: Upsert customer data
MERGE INTO customers_target t
USING customers_source s
ON t.customer_id = s.customer_id
WHEN MATCHED AND s.updated_at > t.updated_at THEN
    UPDATE SET
        name = s.name,
        email = s.email,
        updated_at = s.updated_at
WHEN NOT MATCHED THEN
    INSERT (customer_id, name, email, created_at, updated_at)
    VALUES (s.customer_id, s.name, s.email, s.created_at, s.updated_at);

-- Explode Array: Extract order items (Snowflake/BigQuery syntax)
SELECT
    order_id,
    ORDER_DATE,
    item.value:product_id::INT as product_id,
    item.value:quantity::INT as quantity,
    item.value:unit_price::DECIMAL(10,2) as unit_price
FROM orders
CROSS JOIN LATERAL FLATTEN(items) as item;

-- Advanced Aggregates: Percentiles and Statistics
SELECT
    department,
    COUNT(*) as employee_count,
    AVG(salary) as avg_salary,
    MEDIAN(salary) as median_salary,
    STDDEV(salary) as salary_stddev,
    PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY salary) as p95_salary,
    MIN(salary) as min_salary,
    MAX(salary) as max_salary
FROM employees
GROUP BY department
ORDER BY avg_salary DESC;

-- Handling Data Skew: Salting technique for skewed joins
-- When joining on a key with uneven distribution (e.g., nulls or frequent values)
SELECT
    COALESCE(o.customer_id, -1) % 100 as salt_key,  -- Distribute NULLs
    o.order_id,
    c.customer_name,
    o.order_amount
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE COALESCE(o.customer_id, -1) % 100 = 0  -- Process one partition at a time
UNION ALL
-- Repeat for other salt values (0-99) and combine results
```

## 📝 Tomorrow's Plan
- Practice window functions with real datasets
- Learn about cost-based query optimization
- Explore nested data structures (arrays/structs) in different SQL dialects
- Compare performance: CTEs vs. subqueries vs. temporary tables

---

### Tags
`#daily-note` `#sql` `#advanced-sql` `#query-optimization` `#data-modeling` `#window-functions` `#cte` `#performance`

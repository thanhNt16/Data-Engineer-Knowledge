# SQL Interview Questions

> Practice problems and patterns for data engineering SQL interviews

## 📊 Question Categories

1. **Window Functions** - Ranking, offsets, analytics
2. **Joins & Unions** - Combining data
3. **Aggregations** - Grouping & filtering
4. **Data Manipulation** - String, date, conditional logic
5. **Performance** - Optimization, indexing
6. **Real-World Scenarios** - Business problems

---

## 1. Window Functions

### Q1: Find the Top N Per Group

**Problem**: Find the top 3 highest-paid employees in each department.

```sql
WITH ranked_employees AS (
    SELECT
        employee_name,
        department,
        salary,
        ROW_NUMBER() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) as salary_rank
    FROM employees
)
SELECT
    employee_name,
    department,
    salary
FROM ranked_employees
WHERE salary_rank <= 3
ORDER BY department, salary DESC;
```

**Variations**:
- Use `RANK()` for ties
- Use `DENSE_RANK()` for no gaps in ranking
- Find top 10% instead of top N

---

### Q2: Running Total & Moving Average

**Problem**: Calculate running total and 7-day moving average of daily sales.

```sql
SELECT
    order_date,
    daily_sales,
    SUM(daily_sales) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as running_total,
    AVG(daily_sales) OVER (
        ORDER BY order_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as moving_avg_7day
FROM daily_sales_summary
ORDER BY order_date;
```

---

### Q3: Year-Over-Year Comparison

**Problem**: Compare this year's sales to last year's sales by month.

```sql
WITH monthly_sales AS (
    SELECT
        DATE_TRUNC('month', order_date) as month,
        EXTRACT(YEAR FROM order_date) as year,
        SUM(amount) as monthly_sales
    FROM orders
    GROUP BY 1, 2
),
year_comparison AS (
    SELECT
        month,
        year,
        monthly_sales,
        LAG(monthly_sales, 12) OVER (
            ORDER BY month
        ) as same_month_last_year
    FROM monthly_sales
)
SELECT
    month,
    year,
    monthly_sales,
    same_month_last_year,
    ROUND(
        (monthly_sales - same_month_last_year) * 100.0 /
        NULLIF(same_month_last_year, 0),
        2
    ) as yoy_growth_pct
FROM year_comparison
WHERE same_month_last_year IS NOT NULL
ORDER BY month DESC;
```

---

### Q4: Find Gaps in Sequences

**Problem**: Find missing order IDs in a sequence.

```sql
WITH number_series AS (
    SELECT GENERATE_SERIES(
        (SELECT MIN(order_id) FROM orders),
        (SELECT MAX(order_id) FROM orders)
    ) as order_id
)
SELECT ns.order_id as missing_id
FROM number_series ns
LEFT JOIN orders o ON ns.order_id = o.order_id
WHERE o.order_id IS NULL
ORDER BY ns.order_id;
```

---

## 2. Joins & Unions

### Q5: Find Customers Without Orders

**Problem**: List all customers who haven't placed an order in the last 90 days.

```sql
SELECT
    c.customer_id,
    c.customer_name,
    c.email,
    MAX(o.order_date) as last_order_date
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name, c.email
HAVING MAX(o.order_date) IS NULL
   OR MAX(o.order_date) < CURRENT_DATE - INTERVAL '90 days'
ORDER BY last_order_date DESC NULLS LAST;
```

---

### Q6: Anti-Join Pattern

**Problem**: Find products that have never been ordered.

```sql
-- Method 1: LEFT JOIN
SELECT p.product_id, p.product_name
FROM products p
LEFT JOIN order_items oi ON p.product_id = oi.product_id
WHERE oi.product_id IS NULL;

-- Method 2: NOT EXISTS (often more efficient)
SELECT p.product_id, p.product_name
FROM products p
WHERE NOT EXISTS (
    SELECT 1
    FROM order_items oi
    WHERE oi.product_id = p.product_id
);

-- Method 3: NOT IN (be careful with NULLs!)
SELECT p.product_id, p.product_name
FROM products p
WHERE p.product_id NOT IN (
    SELECT DISTINCT product_id
    FROM order_items
    WHERE product_id IS NOT NULL
);
```

---

### Q7: Full Outer Join for Mismatched Data

**Problem**: Combine customer data from two systems, showing matches and mismatches.

```sql
SELECT
    COALESCE(c1.customer_id, c2.customer_id) as customer_id,
    c1.name as name_system_a,
    c2.name as name_system_b,
    c1.email as email_system_a,
    c2.email as email_system_b,
    CASE
        WHEN c1.customer_id IS NULL THEN 'Only in B'
        WHEN c2.customer_id IS NULL THEN 'Only in A'
        WHEN c1.email != c2.email THEN 'Email mismatch'
        ELSE 'Match'
    END as status
FROM customers_system_a c1
FULL OUTER JOIN customers_system_b c2
    ON c1.customer_id = c2.customer_id;
```

---

## 3. Aggregations

### Q8: Percentile Calculation

**Problem**: Find the 95th percentile of order values.

```sql
SELECT
    PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY order_amount) as p95_order_amount,
    PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY order_amount) as median_order_amount,
    AVG(order_amount) as avg_order_amount
FROM orders;
```

---

### Q9: Histogram/Bucket Analysis

**Problem**: Create a histogram of customer spending ranges.

```sql
WITH customer_spending AS (
    SELECT
        customer_id,
        SUM(order_amount) as total_spent
    FROM orders
    GROUP BY customer_id
),
spending_buckets AS (
    SELECT
        customer_id,
        total_spent,
        CASE
            WHEN total_spent < 100 THEN '0-100'
            WHEN total_spent < 500 THEN '100-500'
            WHEN total_spent < 1000 THEN '500-1000'
            WHEN total_spent < 5000 THEN '1000-5000'
            ELSE '5000+'
        END as spending_range
    FROM customer_spending
)
SELECT
    spending_range,
    COUNT(*) as customer_count,
    MIN(total_spent) as min_amount,
    MAX(total_spent) as max_amount,
    AVG(total_spent) as avg_amount
FROM spending_buckets
GROUP BY spending_range
ORDER BY MIN(total_spent);
```

---

### Q10: Cohort Analysis

**Problem**: Calculate retention by customer cohort.

```sql
WITH customer_cohorts AS (
    SELECT
        customer_id,
        DATE_TRUNC('month', MIN(order_date)) as cohort_month
    FROM orders
    GROUP BY customer_id
),
monthly_activity AS (
    SELECT
        c.customer_id,
        c.cohort_month,
        DATE_TRUNC('month', o.order_date) as activity_month,
        EXTRACT(YEAR FROM AGE(o.order_date, c.cohort_month)) * 12 +
            EXTRACT(MONTH FROM AGE(o.order_date, c.cohort_month)) as month_number
    FROM customer_cohorts c
    JOIN orders o ON c.customer_id = o.customer_id
    WHERE o.order_date >= c.cohort_month
),
cohort_sizes AS (
    SELECT
        cohort_month,
        COUNT(DISTINCT customer_id) as cohort_size
    FROM customer_cohorts
    GROUP BY cohort_month
)
SELECT
    a.cohort_month,
    a.month_number,
    COUNT(DISTINCT a.customer_id) as active_customers,
    c.cohort_size,
    ROUND(
        COUNT(DISTINCT a.customer_id) * 100.0 / c.cohort_size,
        2
    ) as retention_pct
FROM monthly_activity a
JOIN cohort_sizes c ON a.cohort_month = c.cohort_month
GROUP BY a.cohort_month, a.month_number, c.cohort_size
ORDER BY a.cohort_month, a.month_number;
```

---

## 4. Data Manipulation

### Q11: Pivot (Rows to Columns)

**Problem**: Pivot monthly sales data by product category.

```sql
WITH monthly_sales AS (
    SELECT
        DATE_TRUNC('month', order_date) as month,
        category,
        SUM(amount) as monthly_sales
    FROM orders o
    JOIN products p ON o.product_id = p.product_id
    GROUP BY 1, 2
)
SELECT
    month,
    COALESCE(SUM(CASE WHEN category = 'Electronics' THEN monthly_sales END), 0) as electronics_sales,
    COALESCE(SUM(CASE WHEN category = 'Clothing' THEN monthly_sales END), 0) as clothing_sales,
    COALESCE(SUM(CASE WHEN category = 'Books' THEN monthly_sales END), 0) as books_sales,
    COALESCE(SUM(CASE WHEN category = 'Home' THEN monthly_sales END), 0) as home_sales
FROM monthly_sales
GROUP BY month
ORDER BY month;
```

---

### Q12: Unpivot (Columns to Rows)

**Problem**: Unpivot quarterly sales columns into rows.

```sql
SELECT
    product_id,
    'Q1' as quarter,
    q1_sales as sales_amount
FROM product_sales
UNION ALL
SELECT
    product_id,
    'Q2' as quarter,
    q2_sales as sales_amount
FROM product_sales
UNION ALL
SELECT
    product_id,
    'Q3' as quarter,
    q3_sales as sales_amount
FROM product_sales
UNION ALL
SELECT
    product_id,
    'Q4' as quarter,
    q4_sales as sales_amount
FROM product_sales
ORDER BY product_id, quarter;
```

---

### Q13: String Parsing

**Problem**: Parse email addresses to extract domain and count by domain.

```sql
SELECT
    SUBSTRING(email FROM POSITION('@' IN email) + 1) as domain,
    COUNT(*) as customer_count
FROM customers
WHERE email IS NOT NULL AND email LIKE '%@%'
GROUP BY domain
ORDER BY customer_count DESC;
```

---

## 5. Performance

### Q14: Optimize Slow Query

**Problem**: This query is slow. Optimize it.

```sql
-- BEFORE: Slow
SELECT
    c.customer_name,
    o.order_date,
    SUM(oi.quantity * oi.unit_price) as order_total
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.order_date >= '2024-01-01'
GROUP BY c.customer_name, o.order_date
ORDER BY o.order_date DESC;

-- AFTER: Optimized
-- 1. Add filter before JOIN
-- 2. Use subquery to reduce data
WITH recent_orders AS (
    SELECT
        order_id,
        customer_id,
        order_date
    FROM orders
    WHERE order_date >= '2024-01-01'
),
order_totals AS (
    SELECT
        order_id,
        SUM(quantity * unit_price) as order_total
    FROM order_items
    GROUP BY order_id
)
SELECT
    c.customer_name,
    ro.order_date,
    ot.order_total
FROM recent_orders ro
JOIN order_totals ot ON ro.order_id = ot.order_id
JOIN customers c ON ro.customer_id = c.customer_id
ORDER BY ro.order_date DESC;
```

---

### Q15: Eliminate Duplicate Rows

**Problem**: Find and remove duplicate records.

```sql
-- Find duplicates
SELECT
    customer_id,
    email,
    COUNT(*) as duplicate_count
FROM customers
GROUP BY customer_id, email
HAVING COUNT(*) > 1;

-- Keep only the most recent record
WITH ranked_customers AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id, email
            ORDER BY created_at DESC
        ) as rank_num
    FROM customers
)
DELETE FROM customers
WHERE id IN (
    SELECT id FROM ranked_customers WHERE rank_num > 1
);
```

---

## 6. Real-World Scenarios

### Q16: Calculate Customer Lifetime Value (CLV)

```sql
WITH customer_metrics AS (
    SELECT
        customer_id,
        MIN(order_date) as first_order_date,
        MAX(order_date) as last_order_date,
        COUNT(DISTINCT order_id) as total_orders,
        SUM(amount) as total_spent,
        AVG(amount) as avg_order_value
    FROM orders
    GROUP BY customer_id
),
customer_ltv AS (
    SELECT
        customer_id,
        total_spent as lifetime_value,
        total_orders,
        avg_order_value,
        AGE(last_order_date, first_order_date) as customer_age,
        ROUND(
            total_spent /
                NULLIF(
                    EXTRACT(DAY FROM AGE(last_order_date, first_order_date)) / 30.0,
                    0
                ),
        2
        ) as monthly_spend
    FROM customer_metrics
)
SELECT *
FROM customer_ltv
ORDER BY lifetime_value DESC
LIMIT 10;
```

---

### Q17: Detect Anomalous Activity

**Problem**: Find customers with unusual order patterns.

```sql
WITH customer_stats AS (
    SELECT
        customer_id,
        COUNT(*) as order_count,
        AVG(amount) as avg_order_amount,
        STDDEV(amount) as stddev_amount
    FROM orders
    WHERE order_date >= CURRENT_DATE - INTERVAL '90 days'
    GROUP BY customer_id
),
anomalies AS (
    SELECT
        o.order_id,
        o.customer_id,
        o.amount,
        cs.avg_order_amount,
        cs.stddev_amount,
        ABS(o.amount - cs.avg_order_amount) / NULLIF(cs.stddev_amount, 0) as z_score
    FROM orders o
    JOIN customer_stats cs ON o.customer_id = cs.customer_id
    WHERE o.order_date >= CURRENT_DATE - INTERVAL '7 days'
)
SELECT *
FROM anomalies
WHERE z_score > 3  -- 3 standard deviations from mean
ORDER BY z_score DESC;
```

---

### Q18: Churn Prediction Features

**Problem**: Create features for predicting customer churn.

```sql
WITH customer_features AS (
    SELECT
        c.customer_id,
        -- Recency: Days since last order
        EXTRACT(DAY FROM AGE(CURRENT_DATE, MAX(o.order_date))) as days_since_last_order,

        -- Frequency: Number of orders
        COUNT(DISTINCT o.order_id) as total_orders,

        -- Monetary: Total spent
        SUM(o.amount) as total_spent,

        -- Average order value
        AVG(o.amount) as avg_order_value,

        -- Order frequency trend (orders in last 30 days vs previous 30 days)
        COUNT(DISTINCT CASE
            WHEN o.order_date >= CURRENT_DATE - INTERVAL '30 days'
            THEN o.order_id
        END) as orders_last_30_days,
        COUNT(DISTINCT CASE
            WHEN o.order_date >= CURRENT_DATE - INTERVAL '60 days'
            AND o.order_date < CURRENT_DATE - INTERVAL '30 days'
            THEN o.order_id
        END) as orders_previous_30_days,

        -- Category diversity
        COUNT(DISTINCT p.category) as unique_categories_purchased,

        -- Days since first order (tenure)
        EXTRACT(DAY FROM AGE(CURRENT_DATE, MIN(o.order_date))) as customer_tenure_days

    FROM customers c
    LEFT JOIN orders o ON c.customer_id = o.customer_id
    LEFT JOIN order_items oi ON o.order_id = oi.order_id
    LEFT JOIN products p ON oi.product_id = p.product_id
    GROUP BY c.customer_id
)
SELECT *
FROM customer_features
ORDER BY days_since_last_order DESC NULLS LAST;
```

---

## 📋 Practice Checklist

### Window Functions
- [ ] ROW_NUMBER, RANK, DENSE_RANK
- [ ] LAG, LEAD
- [ ] FIRST_VALUE, LAST_VALUE
- [ ] Running totals
- [ ] Moving averages
- [ ] Year-over-year comparisons

### Joins
- [ ] INNER, LEFT, RIGHT, FULL
- [ ] CROSS JOIN
- [ ] SELF JOIN
- [ ] ANTI JOIN (NOT EXISTS)
- [ ] SEMI JOIN (EXISTS)

### Aggregations
- [ ] GROUP BY, HAVING
- [ ] Multiple grouping sets
- [ ] ROLLUP, CUBE
- [ ] Percentiles
- [ ] Histograms
- [ ] Cohort analysis

### Performance
- [ ] EXPLAIN ANALYZE
- [ ] Indexing strategies
- [ ] Query optimization
- [ ] Subquery vs CTE
- [ ] Materialization

---

## 🔗 Related Topics

- [[02-Areas/SQL/Advanced SQL]]
- [[05-Interview Prep/Data Modeling]]
- [[Skills Tracker/Interview Roadmap]]

---

**Practice Platforms**:
- [LeetCode SQL](https://leetcode.com/problemset/all/?difficulty=Easy&page=1&topicSlugs=Database)
- [HackerRank SQL](https://www.hackerrank.com/domains/sql)
- [SQLZoo](https://sqlzoo.net/)
- [Interview Query](https://www.interviewquery.com/)

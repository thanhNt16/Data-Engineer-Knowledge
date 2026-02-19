# SQL Coding Problems

> Comprehensive collection of SQL interview coding problems with solution links

## Practice Platforms

| Platform | URL | Description |
|----------|-----|-------------|
| **LeetCode SQL 50** | [leetcode.com/problem-list/top-sql-50](https://leetcode.com/problem-list/top-sql-50/) | Curated list of 50 essential SQL problems |
| **DataLemur** | [datalemur.com/sql-interview-questions](https://datalemur.com/sql-interview-questions) | Real interview questions from FAANG+ companies |
| **StrataScratch** | [platform.stratascratch.com/coding](https://platform.stratascratch.com/coding?codeType=1) | 1000+ SQL coding questions from real interviews |
| **HackerRank SQL** | [hackerrank.com/domains/sql](https://www.hackerrank.com/domains/sql) | SQL challenges from basic to advanced |

---

## YouTube Solution Playlists

| Channel | Playlist/Video | Focus |
|---------|---------------|-------|
| **Ankit Bansal** | [Complex SQL Questions](https://www.youtube.com/playlist?list=PLBTZqjSKn0IeKBQDjLmzisazhqQy4iGkb) | 91+ scenario-based problems |
| **GeeksforGeeks** | [Top 20 SQL Interview](https://www.youtube.com/playlist?list=PLqM7alHXFySGweLxxAdBDK1CcDEgF-Kwx) | Classic interview problems |
| **techTFQ** | [Top 10 SQL Queries](https://www.youtube.com/watch?v=ZML_EJrBhnY) | Common interview queries |
| **Build with Akshit** | [LeetCode SQL 50 Solutions](https://www.youtube.com/watch?v=wjGbdg0eHwQ) | LeetCode walkthrough |
| **Alex The Analyst** | [Analyst Builder Solutions](https://www.youtube.com/watch?v=ZHaYOC0H5KE) | Easy SQL problems |
| **kudvenkat** | [SQL Server Interview Q&A](https://www.youtube.com/playlist?list=PL6n9fhu94yhXcztdLO7i6mdyaegC8CJwR) | 24 comprehensive videos |

---

## Easy Problems

### Bikes Last Used
| Resource | Link |
|----------|------|
| **StrataScratch** | [stratascratch.com](https://platform.stratascratch.com/coding?codeType=1) - Search "Bikes Last Used" |
| **Concept** | Window functions, MAX() with GROUP BY |

---

### Highest Grade for Each Student
| Resource | Link |
|----------|------|
| **LeetCode** | [1112. Highest Grade For Each Student](https://leetcode.com/problems/highest-grade-for-each-student/) |
| **Concept** | GROUP BY with MAX(), JOINs |

---

### Workers With The Highest Salaries (PySpark)
| Resource | Link |
|----------|------|
| **Concept** | DataFrame API, orderBy, first(), window functions |
| **PySpark Equivalent** | `df.orderBy(col("salary").desc()).first()` |

---

## Medium Problems

### Get Highest Answer Rate Question (LeetCode 578)
| Resource | Link |
|----------|------|
| **LeetCode** | [578. Get Highest Answer Rate Question](https://leetcode.com/problems/get-highest-answer-rate-question/) |
| **YouTube** | Search "LeetCode 578 SQL solution" |
| **Concept** | Aggregation, JOINs, rate calculation |

---

### Pizza Toppings Cost Analysis (LeetCode 3050)
| Resource | Link |
|----------|------|
| **LeetCode** | [3050. Pizza Toppings Cost Analysis](https://leetcode.com/problems/pizza-toppings-cost-analysis/) |
| **Concept** | GROUP BY, SUM(), average calculations |

---

### Titanic Survivors by Class (SQL version)
| Resource | Link |
|----------|------|
| **Kaggle** | [Titanic Dataset](https://www.kaggle.com/c/titanic) |
| **Concept** | GROUP BY, COUNT(), CASE WHEN, survival rate calculation |

```sql
-- Sample Solution
SELECT
    pclass,
    COUNT(*) as total_passengers,
    SUM(CASE WHEN survived = 1 THEN 1 ELSE 0 END) as survivors,
    ROUND(AVG(survived) * 100, 2) as survival_rate
FROM titanic
GROUP BY pclass
ORDER BY pclass;
```

---

### Users By Average Session Time
| Resource | Link |
|----------|------|
| **StrataScratch** | Search "Average Session Time" |
| **Concept** | AVG(), TIMESTAMPDIFF, GROUP BY |

---

### Election Results
| Resource          | Link                                                 |
| ----------------- | ---------------------------------------------------- |
| **Concept**       | Self JOIN, aggregation, finding winners              |
| **Typical Query** | COUNT votes, GROUP BY candidate, ORDER BY votes DESC |

---

### User with Most Approved Flags
| Resource | Link |
|----------|------|
| **Concept** | COUNT with FILTER, subqueries, LIMIT 1 |

---

### Monthly Percentage Difference
| Resource | Link |
|----------|------|
| **StrataScratch** | [Monthly Percentage Difference](https://platform.stratascratch.com/coding?codeType=1) |
| **YouTube** | Ankit Bansal playlist covers similar problems |
| **Concept** | LAG() window function, percentage calculation |

```sql
-- Sample Solution
SELECT
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) as prev_month,
    ROUND((revenue - LAG(revenue) OVER (ORDER BY month)) * 100.0 /
          LAG(revenue) OVER (ORDER BY month), 2) as pct_diff
FROM monthly_revenue;
```

---

### Workers With The Highest Salaries (SQL version)
| Resource | Link |
|----------|------|
| **LeetCode** | Similar to "Department Highest Salary" (185) |
| **Concept** | Subquery with MAX, or DENSE_RANK() window function |

```sql
-- Using Subquery
SELECT * FROM workers
WHERE salary = (SELECT MAX(salary) FROM workers);

-- Using Window Function
WITH ranked AS (
    SELECT *, DENSE_RANK() OVER (ORDER BY salary DESC) as rnk
    FROM workers
)
SELECT * FROM ranked WHERE rnk = 1;
```

---

## Hard Problems

### Department Top Three Salaries (LeetCode 185)
| Resource | Link |
|----------|------|
| **LeetCode** | [185. Department Top Three Salaries](https://leetcode.com/problems/department-top-three-salaries/) |
| **YouTube** | [GeeksforGeeks Solution](https://www.youtube.com/watch?v=-6v7ctxC7yk) |
| **Concept** | DENSE_RANK() window function, JOIN |

```sql
-- Sample Solution
WITH RankedSalaries AS (
    SELECT
        e.name as Employee,
        e.salary as Salary,
        d.name as Department,
        DENSE_RANK() OVER (PARTITION BY d.id ORDER BY e.salary DESC) as rnk
    FROM Employee e
    JOIN Department d ON e.departmentId = d.id
)
SELECT Department, Employee, Salary
FROM RankedSalaries
WHERE rnk <= 3;
```

---

### Find Top Scoring Student II (LeetCode 3188)
| Resource | Link |
|----------|------|
| **LeetCode** | [3188. Find Top Scoring Student II](https://leetcode.com/problems/find-top-scoring-student-ii/) |
| **Concept** | Complex JOINs, aggregation, subqueries |

---

### CEO Subordinate Hierarchy (LeetCode 3236)
| Resource | Link |
|----------|------|
| **LeetCode** | [3236. CEO Subordinate Hierarchy](https://leetcode.com/problems/ceo-subordinate-hierarchy/) |
| **Concept** | Recursive CTE, hierarchical data |

```sql
-- Recursive CTE Pattern
WITH RECURSIVE Hierarchy AS (
    -- Base case: CEO
    SELECT id, name, manager_id, 1 as level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive case: subordinates
    SELECT e.id, e.name, e.manager_id, h.level + 1
    FROM employees e
    JOIN Hierarchy h ON e.manager_id = h.id
)
SELECT * FROM Hierarchy ORDER BY level;
```

---

### Popularity Percentage
| Resource | Link |
|----------|------|
| **Concept** | Self JOIN, COUNT DISTINCT, percentage calculation |

---

### Retention Rate
| Resource | Link |
|----------|------|
| **DataLemur** | [Active User Retention](https://datalemur.com/questions/user-retention) |
| **YouTube** | techTFQ covers retention queries |
| **Concept** | Self JOIN on user_id with date conditions |

```sql
-- Retention Rate Pattern
WITH first_month AS (
    SELECT user_id, MIN(DATE_TRUNC('month', event_date)) as first_month
    FROM user_events
    GROUP BY user_id
),
retained AS (
    SELECT f.user_id
    FROM first_month f
    JOIN user_events u ON f.user_id = u.user_id
        AND DATE_TRUNC('month', u.event_date) = f.first_month + INTERVAL '1 month'
)
SELECT
    ROUND(COUNT(DISTINCT r.user_id) * 100.0 / COUNT(DISTINCT f.user_id), 2) as retention_rate
FROM first_month f
LEFT JOIN retained r ON f.user_id = r.user_id;
```

---

### Titanic Survivors by Class (PySpark version)
| Resource | Link |
|----------|------|
| **Concept** | PySpark DataFrame API, groupBy, agg, withColumn |

```python
# PySpark Solution
from pyspark.sql import functions as F
from pyspark.sql.window import Window

result = df.groupBy("pclass") \
    .agg(
        F.count("*").alias("total"),
        F.sum(F.col("survived")).alias("survivors"),
        F.round(F.avg("survived") * 100, 2).alias("survival_rate")
    ) \
    .orderBy("pclass")
```

---

## Scenario-Based / Company-Specific Problems

### Accenture: City & Pin Code Split
| Resource | Link |
|----------|------|
| **Concept** | String manipulation, SUBSTRING, SPLIT_PART |
| **Keywords** | `SUBSTRING()`, `SPLIT_PART()`, `REGEXP` |

```sql
-- Split city and pincode
SELECT
    SUBSTRING(address, 1, POSITION(',' IN address) - 1) as city,
    SUBSTRING(address, POSITION(',' IN address) + 2, LENGTH(address)) as pin_code
FROM locations;
```

---

### Adobe Interview: Recursive CTE
| Resource | Link |
|----------|------|
| **YouTube** | [Recursive CTE Explained](https://www.youtube.com/watch?v=Kd3HTph0Mds) |
| **Concept** | WITH RECURSIVE, hierarchical queries |

---

### Amazon: Reverse Product IDs
| Resource | Link |
|----------|------|
| **Concept** | REVERSE() function, string manipulation |

```sql
SELECT id, REVERSE(product_id) as reversed_id
FROM products;
```

---

### American Express: Pages Liked by Friends
| Resource | Link |
|----------|------|
| **Concept** | Multi-table JOINs, finding mutual interests |

---

### Bank Account Summary
| Resource | Link |
|----------|------|
| **LeetCode** | [1581. Customer Who Visited but Did Not Make Any Transactions](https://leetcode.com/problems/customer-who-visited-but-did-not-make-any-transactions/) |
| **Concept** | LEFT JOIN with NULL check, aggregation |

---

### Calculate Percentage Increase (Covid Cases)
| Resource | Link |
|----------|------|
| **Concept** | LAG() window function, percentage calculation |

```sql
SELECT
    date,
    cases,
    LAG(cases) OVER (ORDER BY date) as prev_day_cases,
    ROUND((cases - LAG(cases) OVER (ORDER BY date)) * 100.0 /
          LAG(cases) OVER (ORDER BY date), 2) as pct_increase
FROM covid_data;
```

---

### CDC Implementation (Manual Approach)
| Resource | Link |
|----------|------|
| **Concept** | Change Data Capture patterns |
| **Keywords** | MERGE, UPSERT, timestamps, hash comparison |

```sql
-- CDC Pattern
INSERT INTO target (id, name, updated_at)
SELECT s.id, s.name, s.updated_at
FROM source s
WHERE NOT EXISTS (
    SELECT 1 FROM target t
    WHERE t.id = s.id AND t.updated_at >= s.updated_at
);
```

---

### Celebrity Name Split
| Resource | Link |
|----------|------|
| **Concept** | SPLIT_PART, SUBSTRING, string functions |

```sql
SELECT
    name,
    SPLIT_PART(name, ' ', 1) as first_name,
    SPLIT_PART(name, ' ', 2) as last_name
FROM celebrities;
```

---

### Consecutive Days Login (5+ Days)
| Resource | Link |
|----------|------|
| **YouTube** | [Ankit Bansal - Consecutive Days](https://www.youtube.com/watch?v=qyAgWL066Vo) |
| **Concept** | ROW_NUMBER() with date arithmetic, grouping consecutive values |

```sql
-- Consecutive Days Pattern
WITH date_diff AS (
    SELECT
        user_id,
        login_date,
        login_date - ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date)::int as grp
    FROM logins
)
SELECT user_id, COUNT(*) as consecutive_days
FROM date_diff
GROUP BY user_id, grp
HAVING COUNT(*) >= 5;
```

---

### Count Consonants
| Resource | Link |
|----------|------|
| **Concept** | REGEXP_REPLACE, LENGTH, string manipulation |

```sql
SELECT
    word,
    LENGTH(REGEXP_REPLACE(LOWER(word), '[aeiou]', '', 'g')) as consonant_count
FROM words;
```

---

### Deloitte: Sales Increase Every Year
| Resource | Link |
|----------|------|
| **YouTube** | [Deloitte SQL Interview](https://www.youtube.com/watch?v=iQeasQnNsvk) |
| **Concept** | LAG() comparison, year-over-year growth |

```sql
WITH yearly_sales AS (
    SELECT
        EXTRACT(YEAR FROM sale_date) as year,
        SUM(amount) as total
    FROM sales
    GROUP BY EXTRACT(YEAR FROM sale_date)
)
SELECT year, total,
    CASE WHEN total > LAG(total) OVER (ORDER BY year) THEN 'Increased' ELSE 'Not Increased' END as status
FROM yearly_sales;
```

---

### Distinct Routes (Flight Routes)
| Resource | Link |
|----------|------|
| **Concept** | DISTINCT with LEAST/GREATEST for bidirectional pairs |

```sql
SELECT DISTINCT
    LEAST(origin, destination) as city1,
    GREATEST(origin, destination) as city2
FROM flights;
```

---

### Drop Duplicates from Dataframe
| Resource | Link |
|----------|------|
| **SQL** | `DISTINCT` or `ROW_NUMBER()` |
| **PySpark** | `df.dropDuplicates()` |

---

### Exchange Seats
| Resource | Link |
|----------|------|
| **LeetCode** | [626. Exchange Seats](https://leetcode.com/problems/exchange-seats/) |
| **Concept** | CASE WHEN with MOD, swapping adjacent rows |

```sql
SELECT
    CASE
        WHEN id = (SELECT MAX(id) FROM seat) AND MOD(id, 2) = 1 THEN id
        WHEN MOD(id, 2) = 1 THEN id + 1
        ELSE id - 1
    END as id,
    student
FROM seat
ORDER BY id;
```

---

### EY: Consecutive Job Status
| Resource | Link |
|----------|------|
| **Concept** | Islands and gaps problem, grouping consecutive values |

---

### File Contents / Word Count > 1
| Resource | Link |
|----------|------|
| **Concept** | String splitting, aggregation, HAVING clause |

```sql
SELECT word, COUNT(*) as count
FROM words
GROUP BY word
HAVING COUNT(*) > 1;
```

---

### Fill Null Values (Forward Fill)
| Resource | Link |
|----------|------|
| **Concept** | Last observation carried forward (LOCF) |

```sql
-- Forward Fill Pattern
SELECT
    id,
    value,
    LAST_VALUE(value IGNORE NULLS) OVER (ORDER BY id) as filled_value
FROM data;

-- Alternative using subquery
SELECT
    id,
    COALESCE(value, (
        SELECT value FROM data d2
        WHERE d2.id < d1.id AND d2.value IS NOT NULL
        ORDER BY d2.id DESC LIMIT 1
    )) as filled_value
FROM data d1;
```

---

### Find Missing IDs
| Resource | Link |
|----------|------|
| **Concept** | GENERATE_SERIES, LEFT JOIN with NULL |

```sql
SELECT s.id
FROM GENERATE_SERIES(1, (SELECT MAX(id) FROM table)) s(id)
LEFT JOIN table t ON s.id = t.id
WHERE t.id IS NULL;
```

---

### Flatten JSON
| Resource | Link |
|----------|------|
| **PostgreSQL** | `jsonb_array_elements()`, `jsonb_path_query()` |
| **BigQuery** | `UNNEST(JSON_EXTRACT_ARRAY())` |

```sql
SELECT
    id,
    jsonb_array_elements(data->'items') as item
FROM orders;
```

---

### Fractal: Game of Thrones (Max Wins per Region)
| Resource | Link |
|----------|------|
| **Concept** | GROUP BY with MAX, subqueries |

---

### Freshworks: Price Start of Month & Diff
| Resource | Link |
|----------|------|
| **Concept** | FIRST_VALUE, monthly aggregation |

---

### Infosys: IPL Team Combinations
| Resource | Link |
|----------|------|
| **Concept** | Self JOIN for combinations, COMBINATIONS |

```sql
-- All team pair combinations
SELECT
    t1.team as team1,
    t2.team as team2
FROM teams t1
JOIN teams t2 ON t1.team < t2.team;
```

---

### IPL Winning Streak
| Resource | Link |
|----------|------|
| **Concept** | Consecutive wins, islands problem |

```sql
WITH streaks AS (
    SELECT
        team,
        match_date,
        result,
        match_date - ROW_NUMBER() OVER (PARTITION BY team ORDER BY match_date)::int as grp
    FROM matches
    WHERE result = 'W'
)
SELECT team, MAX(cnt) as longest_streak
FROM (
    SELECT team, grp, COUNT(*) as cnt
    FROM streaks
    GROUP BY team, grp
) t
GROUP BY team;
```

---

### Join Output Counts (Inner, Left, Right, Full)
| Resource | Link |
|----------|------|
| **Concept** | Understanding JOIN types and row counts |

| Join Type | Returns |
|-----------|---------|
| INNER | Matching rows only |
| LEFT | All left + matching right |
| RIGHT | All right + matching left |
| FULL | All rows from both |

---

### Left Anti Join / Records in A not B
| Resource | Link |
|----------|------|
| **SQL** | LEFT JOIN with NULL check or NOT EXISTS |

```sql
-- Method 1: LEFT JOIN
SELECT a.*
FROM table_a a
LEFT JOIN table_b b ON a.id = b.id
WHERE b.id IS NULL;

-- Method 2: NOT EXISTS
SELECT * FROM table_a a
WHERE NOT EXISTS (SELECT 1 FROM table_b b WHERE b.id = a.id);

-- PySpark
df_a.join(df_b, on="id", how="left_anti")
```

---

### Managers with 5 Direct Reports
| Resource | Link |
|----------|------|
| **LeetCode** | [570. Managers with at Least 5 Direct Reports](https://leetcode.com/problems/managers-with-at-least-5-direct-reports/) |

```sql
SELECT e.name
FROM Employee e
JOIN Employee sub ON e.id = sub.managerId
GROUP BY e.id, e.name
HAVING COUNT(sub.id) >= 5;
```

---

### NTT Data: Strictly Decreasing Quantity
| Resource | Link |
|----------|------|
| **Concept** | LAG() comparison, sequential validation |

---

### Pivot in PySpark
| Resource | Link |
|----------|------|
| **Concept** | `pivot()` function |

```python
# PySpark Pivot
df.groupBy("category") \
    .pivot("month") \
    .sum("sales")
```

---

### Premier League Stats
| Resource | Link |
|----------|------|
| **Concept** | Complex aggregation, win/loss/draw calculation |

---

### Price as of Date (Big 4)
| Resource | Link |
|----------|------|
| **Concept** | Point-in-time lookup, CORRELATED SUBQUERY |

```sql
SELECT
    t.transaction_date,
    t.quantity,
    (SELECT p.price
     FROM prices p
     WHERE p.product_id = t.product_id
       AND p.effective_date <= t.transaction_date
     ORDER BY p.effective_date DESC
     LIMIT 1) as price
FROM transactions t;
```

---

### PwC: Consecutive Login
| Resource | Link |
|----------|------|
| **Concept** | Similar to consecutive days problem above |

---

### PwC: Source vs Target (Full Join)
| Resource | Link |
|----------|------|
| **Concept** | FULL OUTER JOIN for data comparison |

```sql
SELECT
    COALESCE(s.id, t.id) as id,
    CASE
        WHEN s.id IS NULL THEN 'Missing in Source'
        WHEN t.id IS NULL THEN 'Missing in Target'
        WHEN s.value != t.value THEN 'Different'
        ELSE 'Same'
    END as status
FROM source s
FULL OUTER JOIN target t ON s.id = t.id;
```

---

### Rank vs Dense Rank vs Row Number
| Resource | Link |
|----------|------|
| **YouTube** | [techTFQ - Ranking Functions](https://www.youtube.com/watch?v=ZML_EJrBhnY) |

| Function | Behavior | Example (10, 10, 20) |
|----------|----------|----------------------|
| ROW_NUMBER | Unique sequential | 1, 2, 3 |
| RANK | Same rank, skip | 1, 1, 3 |
| DENSE_RANK | Same rank, no skip | 1, 1, 2 |

---

### Read Third Quarter of Table
| Resource | Link |
|----------|------|
| **Concept** | OFFSET with LIMIT, NTILE |

```sql
-- Method 1: OFFSET
SELECT * FROM table
ORDER BY id
LIMIT 5 OFFSET 10;  -- Adjust based on table size

-- Method 2: NTILE
WITH quarters AS (
    SELECT *, NTILE(4) OVER (ORDER BY id) as q
    FROM table
)
SELECT * FROM quarters WHERE q = 3;
```

---

### Remove Redundant Pairs
| Resource | Link |
|----------|------|
| **Concept** | LEAST/GREATEST for deduplication |

---

### Report Contiguous Dates
| Resource | Link |
|----------|------|
| **LeetCode** | [1454. Active Users](https://leetcode.com/problems/active-users/) |
| **Concept** | Islands problem, date grouping |

---

### Returning Active Users
| Resource | Link |
|----------|------|
| **DataLemur** | [Reactivated Users](https://datalemur.com/questions/reactivated-users) |
| **Concept** | Finding users who returned after inactivity |

---

### Salary Report / Employee Transaction
| Resource | Link |
|----------|------|
| **Concept** | Complex JOINs, aggregation |

---

### Strong Friendship
| Resource | Link |
|----------|------|
| **LeetCode** | [1949. Strong Friendship](https://leetcode.com/problems/strong-friendship/) |
| **Concept** | Mutual relationships, self-joins |

---

### Student Report Card
| Resource | Link |
|----------|------|
| **Concept** | CASE WHEN for grades, aggregation |

```sql
SELECT
    student_id,
    AVG(score) as avg_score,
    CASE
        WHEN AVG(score) >= 90 THEN 'A'
        WHEN AVG(score) >= 80 THEN 'B'
        WHEN AVG(score) >= 70 THEN 'C'
        WHEN AVG(score) >= 60 THEN 'D'
        ELSE 'F'
    END as grade
FROM scores
GROUP BY student_id;
```

---

### Swap ID
| Resource | Link |
|----------|------|
| **Concept** | UPDATE with CASE, swapping values |

```sql
-- Swap odd/even IDs
UPDATE seat
SET id = CASE
    WHEN MOD(id, 2) = 1 THEN id + 1
    ELSE id - 1
END;
```

---

### TCS: Employees Logged In 3+ Times
| Resource | Link |
|----------|------|
| **YouTube** | [TCS SQL Interview](https://www.youtube.com/watch?v=4XVd0de9t-0) |
| **Concept** | GROUP BY with HAVING COUNT >= 3 |

```sql
SELECT employee_id, COUNT(*) as login_count
FROM logins
GROUP BY employee_id
HAVING COUNT(*) >= 3;
```

---

### TCS: Rolling 30 Day Revenue
| Resource | Link |
|----------|------|
| **Concept** | Window functions with RANGE |

```sql
SELECT
    date,
    SUM(revenue) OVER (
        ORDER BY date
        RANGE BETWEEN INTERVAL '30 days' PRECEDING AND CURRENT ROW
    ) as rolling_30d_revenue
FROM daily_revenue;
```

---

### Total Time Spent in Office
| Resource | Link |
|----------|------|
| **Concept** | TIMESTAMPDIFF, entry/exit pairs |

```sql
SELECT
    employee_id,
    SUM(EXTRACT(EPOCH FROM (exit_time - entry_time)) / 3600) as total_hours
FROM office_visits
GROUP BY employee_id;
```

---

### Transactions Per Visit
| Resource | Link |
|----------|------|
| **Concept** | COUNT per GROUP, average calculation |

---

### Union vs Union All
| Resource | Link |
|----------|------|
| **Concept** | Set operations |

| Operator | Removes Duplicates | Performance |
|----------|-------------------|-------------|
| UNION | Yes | Slower |
| UNION ALL | No | Faster |

---

### Unpivot in PySpark
| Resource | Link |
|----------|------|
| **Concept** | Converting columns to rows |

```python
# PySpark Unpivot (stack)
from pyspark.sql import functions as F

unpivoted = df.select(
    "id",
    F.expr("stack(3, 'Jan', jan, 'Feb', feb, 'Mar', mar) as (month, value)")
)
```

---

## Key Concepts Summary

### Window Functions
```sql
ROW_NUMBER() OVER (PARTITION BY col ORDER BY col)
RANK() OVER (PARTITION BY col ORDER BY col)
DENSE_RANK() OVER (PARTITION BY col ORDER BY col)
LAG(col, n) OVER (ORDER BY col)
LEAD(col, n) OVER (ORDER BY col)
SUM(col) OVER (PARTITION BY col ORDER BY col ROWS BETWEEN...)
FIRST_VALUE(col) OVER (PARTITION BY col ORDER BY col)
```

### Common Patterns
1. **Nth Highest/Lowest**: DENSE_RANK() or LIMIT/OFFSET
2. **Consecutive Values**: Date - ROW_NUMBER() grouping
3. **Running Total**: SUM() OVER with ROWS
4. **YoY/MoM Growth**: LAG() comparison
5. **Islands/Gaps**: ROW_NUMBER() difference grouping
6. **Forward Fill**: LAST_VALUE() IGNORE NULLS
7. **Deduplication**: ROW_NUMBER() with DELETE
8. **Pivot**: CASE WHEN with aggregation

---

## Related Topics

- [[05-Interview Prep/SQL Interview Questions]]
- [[02-Areas/SQL/Advanced SQL]]
- [[05-Interview Prep/System Design Framework]]

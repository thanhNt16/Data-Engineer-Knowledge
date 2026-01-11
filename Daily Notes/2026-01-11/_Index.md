# Learning Summary - 2026-01-11

## Topics Covered

### 1. Data Modeling Techniques
Six techniques for building production-ready tables fast and consistently:
- Bus Matrix - Demonstrate value to stakeholders
- Data Contracts - Set end-user expectations
- Insert-only Fact Tables + Snapshot Dimensions
- Data Quality Checks (4 high-ROI checks)
- Data Lineage - Debug data issues
- Centralized Metric Definitions

**Note**: [Data Modeling.md](Data%20Modeling.md)

### 2. Advanced SQL
Understanding what "Advanced SQL" really means:
- SQL Techniques (Window Functions, CTEs, JOINs, MERGE, EXPLODE, Aggregates)
- Query Optimization (Partitioning, Clustering, Data Skew)
- Data Modeling (5 table types: Dimensions, Facts, Bridges, OBT, Summaries)
- 3-Hop Architecture pattern

**Note**: [Advanced SQL.md](Advanced%20SQL.md)

## Key Insights

- Data modeling and SQL are interconnected - you need both to be effective
- "Advanced SQL" is not just complex functions, but understanding how to structure data and retrieve it efficiently
- The 3-hop architecture (Staging → Core → Serving) aligns with the data modeling principles learned

## Resources Used

- https://www.startdataengineering.com/post/fast-consistent-data-model/
- https://www.startdataengineering.com/post/advanced-sql/

## Tomorrow's Plan

- Practice window functions with real datasets
- Learn about WAP (Write-Audit-Publish) pattern implementation
- Explore data quality testing frameworks (Great Expectations, Soda, etc.)
- Study cost-based query optimization

---

**Tags**: `#data-modeling` `#sql` `#advanced-sql` `#query-optimization` `#data-quality`

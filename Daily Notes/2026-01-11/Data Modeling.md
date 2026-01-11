# Daily Note - 2026-01-11

## 🔍 Focus / Goal
Learn six data modeling techniques for building production-ready tables fast and consistently.

## 📚 Learning Notes

### Topics Covered
- Bus Matrix for demonstrating value
- Data Contracts for setting expectations
- Insert-only fact tables and snapshot dimensions
- Data quality checks with high ROI
- Data lineage for debugging
- Centralized metric definitions

### Key Takeaways

1. **Bus Matrix connects technical work to business value**
   - Rows = business processes
   - Columns = business dimensions
   - Helps stakeholders understand impact and demonstrates progress

2. **Data Contracts prevent misuse**
   - Table grain, SLA, schema, owner, DQ checks
   - Should be easily discoverable by end users
   - Sets clear expectations between data team and consumers

3. **Insert-only + Snapshot approach is simpler than SCD2**
   - Fact tables: insert only (no updates)
   - Dimensions: complete snapshot each run
   - One grain per table
   - Use views as interface to protect users from schema changes

4. **Four essential data quality checks**
   - Table constraints (uniqueness, not null, allowed values)
   - Referential integrity (orphan records detection)
   - Reconciliation (compare to source)
   - Metric variance (detect anomalies)

5. **Data lineage is non-negotiable**
   - Critical for debugging
   - Required for audits in regulated industries
   - Tools: SQL Mesh, dbt

6. **Centralize metrics or suffer**
   - Scattered metrics = impossible debugging
   - Use semantic layer or data marts
   - Clear ownership required

## 💡 Insights & Connections

- **Related to**: [Data Modeling](../Data%20Architecture%20Patterns/Data%20Modeling.md), [Data Warehouse](../Data%20Architecture%20Patterns/Data%20Warehouse.md)
- **Questions raised**:
  - How do we implement WAP (Write-Audit-Publish) pattern in practice?
  - What's the trade-off between snapshot dimensions and SCD2 for slowly changing attributes?
  - Which semantic layer tools are most popular in 2026?
- **Ideas to explore**:
  - Compare dbt vs SQL Mesh for lineage capabilities
  - Learn more about Kimball naming conventions
  - Study real-world bus matrix examples

## 🔗 Resources
- [x] https://www.startdataengineering.com/post/fast-consistent-data-model/ - Six Data Modeling Techniques For Building Production-Ready Tables Fast
- [ ] https://dataengineering.wiki/ - Data Engineering Wiki (for further exploration)

## 🛠️ Hands-on Practice

```sql
-- Example: Referential Integrity Check
-- Find orphaned records in fact table

SELECT
    'orders' as table_name,
    COUNT(*) as orphaned_count
FROM fact_orders f
LEFT JOIN dim_customer c ON f.customer_key = c.customer_key
WHERE c.customer_key IS NULL;

-- Example: Reconciliation Check
-- Compare row counts with source
SELECT
    'row_count_reconciliation' as check_type,
    (SELECT COUNT(*) FROM fact_orders) as warehouse_count,
    (SELECT COUNT(*) FROM source_orders) as source_count,
    ABS((SELECT COUNT(*) FROM fact_orders) -
        (SELECT COUNT(*) FROM source_orders)) as difference;
```

## 📝 Tomorrow's Plan
- Learn about WAP (Write-Audit-Publish) pattern implementation
- Explore data quality testing frameworks (Great Expectations, Soda, etc.)
- Study different semantic layer tools (LookML, MetricFlow, dbt Semantic Layer)

---

### Tags
`#daily-note` `#data-modeling` `#data-quality` `#data-warehouse` `#best-practices`

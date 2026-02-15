---
tags: [data-quality, metrics, testing, great-expectations, soda, monte-carlo]
date: 2026-02-15
status: learning
---

# Data Quality

## Overview

Data quality ensures that data is fit for its intended purpose. It's critical for AI/ML models, business decisions, and regulatory compliance.

**Key Insight**: "Data quality is a business problem, not a technical problem."

---

## 6 Data Quality Dimensions

### 1. Completeness

**Definition**: Data includes all required attributes and records.

| Metric | Description | Measurement | Example |
|---------|-------------|------------|----------|
| **Record Completeness** | % of records with no nulls | 95% have all fields |
| **Attribute Completeness** | % of non-null values | Email: 98%, Phone: 85% |
| **Population Completeness** | % of expected records | 90% of expected users present |

**SQL Implementation**:
```sql
-- Measure record completeness
SELECT
    table_name,
    COUNT(*) AS total_records,
    COUNT(*) - COUNT(*) AS complete_records,
    COUNT(*) / COUNT(*)::FLOAT AS completeness_rate
FROM information_schema.tables
WHERE table_schema = 'public';

-- Measure attribute completeness
SELECT
    table_name,
    COUNT(*) AS total_records,
    COUNT(column_name) AS non_null_count,
    (COUNT(*) - COUNT(column_name))::FLOAT / COUNT(*) AS completeness_rate
FROM information_schema.columns
WHERE table_schema = 'public'
GROUP BY table_name, column_name
ORDER BY completeness_rate ASC;
```

### 2. Accuracy

**Definition**: Data conforms to true values or business rules.

| Type | Description | Example | Validation |
|------|-------------|----------|-------------|
| **Data Validity** | Correct format, type, range | Email has @, Phone is digits |
| **Business Validity** | Follows business rules | Order amount > 0, Status valid |
| **Statistical Accuracy** | Matches ground truth | Model predictions vs actuals |

**Python Implementation**:
```python
from great_expectations import Dataset, ExpectationSuite, ExpectColumnValuesToBeInSet

# Load dataset
df = pd.read_csv("orders.csv")

# Create expectation suite
suite = ExpectationSuite(df)

# Data validity: Order status must be valid
suite.add_expectation(
    ExpectColumnValuesToBeInSet(
        column="status",
        value_set=["pending", "completed", "cancelled"]
    )
)

# Business validity: Order amount must be positive
suite.add_expectation(
    ExpectColumnValuesToBeBetween(
        column="amount",
        min_value=0,
        max_value=100000
    )
)

# Validate
results = suite.validate()
print(f"Passed: {results['statistics']['passed']}")
print(f"Failed: {results['statistics']['failed']}")
```

### 3. Consistency

**Definition**: Data is uniform and compatible across systems.

| Type | Description | Example | SQL Check |
|------|-------------|----------|------------|
| **Referential Integrity** | Foreign key constraints exist | ON DELETE CASCADE |
| **Domain Integrity** | Unique constraints on business keys | UNIQUE(user_id, email) |
| **Format Consistency** | Consistent data formats | YYYY-MM-DD HH:MM:SS |

**SQL Implementation**:
```sql
-- Check referential integrity
SELECT
    'Orders with missing users' AS issue,
    COUNT(*) AS count
FROM orders o
LEFT JOIN users u ON o.user_id = u.user_id
WHERE u.user_id IS NULL;

-- Check format consistency (date formats)
SELECT
    'Invalid date formats' AS issue,
    COUNT(*) AS count
FROM events
WHERE date_column !~ '^\d{4}-\d{2}-\d{2} \d{2}:\d{2}$'::timestamp;
```

### 4. Timeliness

**Definition**: Data is available when expected.

| Metric | Description | Target | Example |
|---------|-------------|--------|----------|
| **Data Freshness** | Time since last update | Orders updated within 1 hour |
| **SLA Compliance** | Service level agreement | 99% of queries < 100ms |
| **Lag** | Delay between data generation and availability | < 5 minutes |

**Python Implementation**:
```python
from great_expectations import Dataset, ExpectationColumnValuesToBeInSet

df = pd.read_csv("orders.csv")
df['order_date'] = pd.to_datetime(df['created_at'])

suite = ExpectationSuite(df)

# Data freshness: Orders from last 7 days
seven_days_ago = datetime.now() - timedelta(days=7)
suite.add_expectation(
    ExpectColumnValuesToBeInSet(
        column="order_date",
        value_set=lambda x: x >= seven_days_ago
    )
)

# Lag: Average delay between order creation and processing
df['processing_delay_hours'] = (
    pd.to_datetime(df['processed_at']) - df['order_date']
).dt.total_seconds() / 3600

suite.add_expectation(
    ExpectColumnMeanToBeBetween(
        column="processing_delay_hours",
        min_value=0,
        max_value=1  # Within 1 hour
    )
)

results = suite.validate()
```

### 5. Uniqueness

**Definition**: No duplicate records within same entity.

| Level | Description | Example | Implementation |
|--------|-------------|----------|-------------|
| **Record Uniqueness** | Primary key unique | AUTO_INCREMENT, UNIQUE(id) |
| **Attribute Uniqueness** | No duplicate values for attribute | UNIQUE(email) |
| **Composite Uniqueness** | No duplicate combinations | UNIQUE(email, account_id) |

**SQL Implementation**:
```sql
-- Find duplicate orders
SELECT
    order_id,
    user_id,
    order_date,
    COUNT(*) AS occurrence_count
FROM orders
GROUP BY user_id, order_date
HAVING COUNT(*) > 1
ORDER BY occurrence_count DESC;

-- Add unique constraint
ALTER TABLE orders ADD CONSTRAINT uq_user_order_date UNIQUE (user_id, order_date);
```

### 6. Validity

**Definition**: Data conforms to defined constraints and rules.

| Validation Type | Example | Python Check |
|----------------|----------|--------------|--------------|
| **Type Validation** | Correct data types | isinstance(value, int) |
| **Range Validation** | Within min/max bounds | 0 <= amount <= 100000 |
| **Pattern Validation** | Matches regex pattern | email regex |
| **Enum Validation** | In allowed set | status IN (a, b, c) |

**Python Implementation**:
```python
import pandas as pd
import re

class DataValidator:
    def __init__(self):
        self.email_pattern = re.compile(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$')
        self.phone_pattern = re.compile(r'^\+?\d{1,3}[- ]?\(?\d{3}\)?')

    def validate_orders(self, df):
        issues = []
        
        # Type validation
        if not pd.api.types.is_numeric_dtype(df['order_id']):
            issues.append("order_id must be integer")
        
        # Range validation
        if (df['amount'] < 0).any():
            issues.append("amount must be positive")
        
        # Pattern validation
        invalid_emails = df[~df['email'].str.match(self.email_pattern)]
        if len(invalid_emails) > 0:
            issues.append(f"{len(invalid_emails)} invalid emails")
        
        return {
            'valid': len(issues) == 0,
            'issues': issues
        }

validator = DataValidator()
result = validator.validate_orders(orders_df)
print(f"Valid: {result['valid']}, Issues: {result['issues']}")
```

---

## Data Quality Tools

### Great Expectations

**Description**: Python-based testing framework for data pipelines.

**Features**:
- Expectation library (60+ types)
- Data Docs (HTML documentation)
- Profiling (column statistics, distributions)
- Validation at any scale

**Example**:
```python
from great_expectations import DataContext
from great_expectations.expectations import ExpectColumnToBeUnique

context = DataContext(base_directory='./data')

# Expectation: Order IDs are unique
batch = context.get_batch("orders")
batch.expect_column_to_be_unique("order_id")

# Save to Data Docs
context.build_data_docs()
```

### Soda

**Description**: SQL-first testing tool with cloud warehouse integrations.

**Features**:
- SQL-based checks
- Cloud warehouse support (Snowflake, BigQuery, Redshift)
- Automated scans
- Monitoring dashboard

**Example**:
```sql
-- Soda check: Orders table freshness
SELECT
    COUNT(*) AS total_orders,
    MAX(order_date) AS latest_order_date,
    CURRENT_DATE - MAX(order_date) AS data_age_days
FROM public.orders;

-- Fail if data_age_days > 1
```

```yaml
# checks/soda/orders_freshness.yml
checks for orders_freshness:
  - freshness:
      name: orders_freshness
      warn_after:
        hours: 1
      fail_after:
        hours: 24
      query: |
        SELECT MAX(order_date) FROM public.orders
```

### Monte Carlo

**Description**: ML-based data observability platform.

**Features**:
- Automated anomaly detection
- Incident management
- Root cause analysis
- Data lineage visualization
- Table-level monitoring

**Use Case**:
- ML feature drift detection
- Unexpected data pattern alerts
- SLA monitoring

### Datafold

**Description**: Data diff and quality monitoring tool.

**Features**:
- Column-level diff (before/after)
- Row-level diff
- CI/CD integration
- Pandas support

**Example**:
```python
import datafold as dlf

# Diff two DataFrames
diff_result = dlf.diff(
    df_old,
    df_new,
    key_columns=['order_id'],
    ignore_columns=['updated_at']
)

print(f"Changed rows: {len(diff_result.changed_rows)}")
print(f"Added rows: {len(diff_result.added_rows)}")
```

---

## Data Testing Strategies

### 1. Unit Testing

**Goal**: Validate individual data components.

```python
import pytest
import pandas as pd

def test_order_amount_positive():
    """Test that order amounts are always positive"""
    # Mock data
    order = {'order_id': 1, 'amount': 100.00}
    
    # Test
    assert order['amount'] > 0, "Order amount must be positive"

def test_user_email_format():
    """Test email format validation"""
    email = "user@example.com"
    assert '@' in email, "Email must contain @"
    assert email.count('@') == 1, "Email must have only one @"
```

### 2. Integration Testing

**Goal**: Validate data flow between pipeline components.

```python
import pytest
from fastapi.testclient import TestClient
from main import app

def test_post_order_endpoint():
    """Test API endpoint with valid order data"""
    client = TestClient(app)
    
    valid_order = {
        "user_id": 1,
        "product_id": 101,
        "amount": 50.00
    }
    
    response = client.post("/api/orders", json=valid_order)
    
    assert response.status_code == 200
    assert response.json()['status'] == 'created'

def test_post_order_invalid_amount():
    """Test API endpoint with invalid order amount"""
    invalid_order = {
        "user_id": 1,
        "product_id": 101,
        "amount": -100.00  # Invalid
    }
    
    response = client.post("/api/orders", json=invalid_order)
    
    # Should be rejected (422 or 400)
    assert response.status_code in [400, 422]
```

### 3. Snapshot Testing

**Goal**: Validate database state at specific point in time.

```python
import pytest
import pandas as pd

def test_orders_snapshot():
    """Test that orders match expected snapshot"""
    # Load snapshot (expected state)
    expected = pd.read_csv("tests/snapshots/orders_snapshot.csv")
    
    # Load current state
    actual = pd.read_sql("SELECT * FROM orders", conn)
    
    # Compare
    pd.testing.assert_frame_equal(expected, actual)

def test_data_quality_snapshot():
    """Test data quality metrics match expectations"""
    df = pd.read_sql("SELECT * FROM orders", conn)
    
    # Completeness: No nulls in critical columns
    assert df['order_id'].notna().all()
    assert df['user_id'].notna().all()
    assert df['amount'].notna().all()
    
    # Accuracy: Amounts are positive
    assert (df['amount'] > 0).all()
```

---

## Data Contracts

### Contract Definition

A data contract is a formal agreement between data producer and consumer, specifying schema, quality, and SLA.

**Contract Elements**:
```yaml
# contracts/orders_data_contract.yml
name: orders_data_contract
version: 1.0
owner: data_team
consumer: analytics_platform

schema:
  table: orders
  columns:
    - name: order_id
      type: BIGINT
      nullable: false
    - name: user_id
      type: BIGINT
      nullable: false
    - name: amount
      type: DECIMAL(10,2)
      nullable: false
    - name: order_date
      type: TIMESTAMP
      nullable: false

quality_expectations:
  - completeness:
      column: order_id
      min_percentage: 100
  - accuracy:
      column: amount
      min_value: 0
      max_value: 100000
  - timeliness:
    column: order_date
    max_data_age_days: 1

sla:
  response_time_ms_p99: 500
  uptime_percentage: 99.9
```

### Contract Enforcement

```python
from datacontract import DataContract

# Load contract
orders_contract = DataContract('contracts/orders_data_contract.yml')

# Validate current data against contract
validation_result = orders_contract.validate(df=orders_df)

if not validation_result.passed:
    print(f"Contract violation: {validation_result.failed_reason}")
    # Take action: Reject data, notify team, fix pipeline
else:
    print("Data contract satisfied")
```

---

## Data Quality Metrics Dashboard

### Key Metrics to Track

| Metric | Formula | Target | Alert Threshold |
|---------|---------|--------|-----------------|
| **Freshness** | CURRENT_TIMESTAMP - MAX(timestamp) | < 1 hour | > 6 hours |
| **Completeness** | (non_null_rows / total_rows) | > 95% | < 90% |
| **Accuracy** | (correct_values / total_values) | > 99% | < 95% |
| **Duplicate Rate** | (duplicate_rows / total_rows) | < 1% | > 5% |
| **Volume** | COUNT(*) per day | Within 10% of normal | > 20% |

### Python: Metrics Calculation

```python
import pandas as pd
from datetime import datetime

def calculate_dq_metrics(df):
    """Calculate data quality metrics for a DataFrame"""
    metrics = {}
    
    # Completeness
    total_rows = len(df)
    null_counts = df.isnull().sum()
    metrics['completeness'] = {
        col: (total_rows - null_counts[col]) / total_rows
        for col in df.columns
    }
    
    # Accuracy (for numerical columns)
    if 'amount' in df.columns:
        positive_count = (df['amount'] > 0).sum()
        total_count = len(df['amount'])
        metrics['accuracy_positive_amount'] = positive_count / total_count
    
    # Duplicates
    metrics['duplicate_rows'] = df.duplicated().shape[0]
    metrics['duplicate_rate'] = metrics['duplicate_rows'] / total_rows
    
    # Freshness
    if 'order_date' in df.columns:
        df['order_date'] = pd.to_datetime(df['order_date'])
        max_date = df['order_date'].max()
        now = datetime.now()
        age_days = (now - max_date).days
        metrics['data_age_days'] = age_days
    
    return metrics

metrics = calculate_dq_metrics(orders_df)
print(f"Completeness: {metrics['completeness']['order_id']:.2%}")
print(f"Duplicate rate: {metrics['duplicate_rate']:.2%}")
print(f"Data age: {metrics['data_age_days']} days")
```

---

## Interview Questions

**Q1: What's the difference between data validity and data accuracy?**

**A1**:
- **Validity**: Data conforms to defined rules (format, type, range). Example: Email has @, Amount is positive.
- **Accuracy**: Data matches ground truth or actual values. Example: Predicted value matches actual value.

**Q2: How would you handle data quality issues in a production data pipeline?**

**A2**:
1. **Detect**: Implement automated monitoring (Soda, Great Expectations) to catch issues early
2. **Alert**: Send alerts to data engineers and stakeholders when issues detected
3. **Quarantine**: Route bad data to a quarantine table for review
4. **Repair**: Implement data cleaning jobs to fix common issues
5. **Root Cause Analysis**: Investigate source of issues (upstream system, ETL logic)
6. **Prevent**: Add data contracts and validation rules at pipeline entry

**Q3: What's data profiling and how is it different from data testing?**

**A3**:
- **Data Profiling**: Exploratory analysis of data to understand its structure, patterns, and statistics. Uses techniques like histograms, scatter plots, summary statistics. Goal: Understand data characteristics.
- **Data Testing**: Verifying data against expectations and rules. Uses test cases, validation rules, and quality metrics. Goal: Ensure data meets requirements.

**Q4: How would you implement exactly-once data ingestion to avoid duplicates?**

**A4**:
1. **Deduplication**: Use fuzzy matching to identify duplicate records before insertion
2. **Idempotent Operations**: Design operations (UPSERT) that produce same result if run multiple times
3. **Database Constraints**: Add UNIQUE constraints on natural keys
4. **Upsert Logic**: `INSERT ... ON CONFLICT DO NOTHING` (PostgreSQL) or `MERGE` (BigQuery)
5. **Watermark Tracking**: Keep track of last processed offset in streaming systems
6. **Hash-based Sharding**: Ensure same key goes to same shard to prevent cross-shard duplicates

**Q5: What's the cost of poor data quality?**

**A5**:
- **Financial**: Incorrect decisions from bad data (e.g., marketing to wrong segment)
- **Operational**: Time spent debugging issues, re-running failed jobs
- **Customer Trust**: Damaged reputation, customer churn
- **Compliance**: Regulatory fines for data accuracy (GDPR, CCPA)
- **ML Model Performance**: Models trained on poor data make poor predictions

---

## Related Notes

- [[Database Fundamentals]] - Transactions, isolation, consistency
- [[System Design]] - Data quality patterns, CAP theorem
- [[Apache Flink - Real-Time Analytics]] - Stateful stream processing
- [[Apache Kafka - Event Streaming]] - Event streaming for data ingestion

---

## Resources

- [Great Expectations Documentation](https://docs.greatexpectations.io/)
- [Soda Documentation](https://docs.soda.io/)
- [Monte Carlo Documentation](https://www.montecarlodata.com/docs/)
- [Datafold Documentation](https://www.datafold.com/)
- [Data Quality Metrics: A Complete Guide with Examples and Measurement Methods - IcedQ](https://icedq.com/6-data-quality-dimensions-complete-guide-examples-measurement-methods/)
- [dbt Docs: Data Contracts - dbt Labs](https://docs.getdbt.com/guides/data-contracts/)

---

**Progress**: 🟢 Learning (concepts understood, need implementation practice)

**Next Steps**:
- [ ] Implement Great Expectations for current pipeline
- [ ] Add Soda checks for warehouse tables
- [ ] Create data contract for critical tables
- [ ] Build data quality dashboard
- [ ] Implement automated alerting
- [ ] Practice unit testing with pytest

---
tags: [data-modeling, scd, dimensional-modeling, data-warehouse]
date: 2026-02-15
status: learning
---

# Slowly Changing Dimensions (SCDs)

## Overview

Slowly Changing Dimensions (SCDs) are a dimension design technique used in data warehouses to track the history of changes to dimensional attributes over time.

**Key Insight**: SCDs enable you to:
- Track historical changes to dimension attributes
- Analyze trends and patterns over time
- Support time-travel queries (what was true at point X)
- Meet business reporting requirements (current vs. historical)

---

## SCD Types

### Type 0 - Overwrite

**Description**: Every time a dimension attribute changes, overwrite the existing record.

**When to Use**:
- History not required
- Only current state matters
- Simplicity over completeness

**Example**:
```
User table at T1:
| user_id | name | email | valid_from |
|---------|------|-------|-----------|
| 101 | Alice | alice@old.com | 2024-01-01 |

User table at T2 (email updated):
| user_id | name | email | valid_from |
|---------|------|-------|-----------|
| 101 | Alice | alice@new.com | 2024-01-02 |

Old record (alice@old.com) is OVERWRITTEN - history lost!
```

**SQL Implementation**:

```sql
-- Overwrite on every change (SCD Type 0)
MERGE INTO target_users AS target
USING staging_users AS source
ON target.user_id = source.user_id
WHEN MATCHED THEN
    UPDATE SET
        name = source.name,
        email = source.email,
        updated_at = CURRENT_TIMESTAMP
WHEN NOT MATCHED THEN
    INSERT (user_id, name, email, created_at, updated_at)
    VALUES (source.user_id, source.name, source.email, CURRENT_TIMESTAMP, CURRENT_TIMESTAMP);
```

---

### Type 1 - Add New Row

**Description**: Create a new row every time an attribute changes. Keep all historical versions.

**When to Use**:
- Need complete history
- Want to track when each attribute changed
- Can handle multiple versions per entity

**Example**:
```
User table:
| user_id | name | email | valid_from | valid_to | is_current |
|---------|------|-------|-------------|-----------|-------------|
| 101 | Alice | alice@old.com | 2024-01-01 | 2024-01-02 | FALSE |
| 101 | Alice | alice@new.com | 2024-01-02 | NULL | TRUE |
```

**SQL Implementation**:

```sql
-- SCD Type 1: Add new row on change
INSERT INTO target_users (user_id, name, email, valid_from, valid_to, is_current)
SELECT
    user_id,
    name,
    email,
    CURRENT_TIMESTAMP AS valid_from,
    NULL AS valid_to,  -- NULL means current
    CASE
        WHEN NOT EXISTS (
            SELECT 1 FROM target_users
            WHERE user_id = source.user_id AND is_current = TRUE
        ) THEN TRUE
        ELSE FALSE
    END AS is_current
FROM staging_users source;
```

**Query History**:

```sql
-- Query user's email history
SELECT user_id, name, email, valid_from, valid_to
FROM target_users
WHERE user_id = 101
ORDER BY valid_from;
```

---

### Type 2 - Update Existing Record

**Description**: Add new row on first change, then update the latest record (is_current flag) on subsequent changes. No overwrite.

**When to Use**:
- Want history of first change
- Save storage (only keep + current)
- Easier queries (only look at current flag)

**Example**:
```
User table:
| user_id | name | email | valid_from | valid_to | is_current |
|---------|------|-------|-------------|-----------|-------------|
| 101 | Alice | alice@old.com | 2024-01-01 | 2024-01-02 | FALSE |
| 101 | Alice | alice@new.com | 2024-01-02 | NULL | TRUE |

After second change:
| user_id | name | email | valid_from | valid_to | is_current |
|---------|------|-------|-------------|-----------|-------------|
| 101 | Alice | alice@old.com | 2024-01-01 | 2024-01-02 | FALSE |
| 101 | Alice | alice@new.com | 2024-01-02 | 2024-01-05 | FALSE |
| 101 | Alice | alice@new2.com | 2024-01-05 | NULL | TRUE |
```

**SQL Implementation**:

```sql
-- SCD Type 2: Update existing record
-- First change: INSERT
INSERT INTO target_users (user_id, name, email, valid_from, valid_to, is_current)
SELECT
    user_id, name, email, CURRENT_TIMESTAMP AS valid_from,
    NULL AS valid_to,
    CASE
        WHEN NOT EXISTS (
            SELECT 1 FROM target_users
            WHERE user_id = source.user_id AND is_current = TRUE
        ) THEN TRUE
        ELSE FALSE
    END AS is_current
FROM staging_users source
WHERE NOT EXISTS (
    SELECT 1 FROM target_users WHERE user_id = source.user_id
);

-- Subsequent changes: UPDATE current, INSERT new
-- Update existing current to set valid_to
UPDATE target_users
SET valid_to = CURRENT_TIMESTAMP, is_current = FALSE
WHERE user_id = source.user_id AND is_current = TRUE;

-- Insert new record as current
INSERT INTO target_users (user_id, name, email, valid_from, valid_to, is_current)
SELECT user_id, name, email, CURRENT_TIMESTAMP, NULL, TRUE
FROM staging_users source
WHERE NOT EXISTS (
    SELECT 1 FROM target_users WHERE user_id = source.user_id AND valid_to IS NULL
);
```

---

### Type 3 - Add New and Expire Old

**Description**: Track history for a fixed time window (e.g., 30 days). Create new row on change, expire old records after window.

**When to Use**:
- Want limited history
- Storage is concern
- Need historical for reporting period only
- Data privacy requirements (keep only N days)

**Example**:
```
User table (30-day window):
| user_id | name | email | valid_from | valid_to | is_current |
|---------|------|-------|-------------|-----------|-------------|
| 101 | Alice | alice@old.com | 2024-01-01 | 2024-01-02 | FALSE |
| 101 | Alice | alice@new.com | 2024-01-02 | 2024-01-05 | FALSE |
| 101 | Alice | alice@new2.com | 2024-01-05 | 2024-02-04 | TRUE |

After 30 days, alice@old.com record expires (valid_to = NOW - 30 days)
```

**SQL Implementation**:

```sql
-- SCD Type 3: Add new and expire old records (30-day window)
WITH expiration_cutoff AS (
    SELECT CURRENT_TIMESTAMP - INTERVAL '30 days' AS cutoff_date
)
-- Expire old records
UPDATE target_users
SET valid_to = expiration_cutoff.cutoff_date, is_current = FALSE
WHERE is_current = TRUE
  AND valid_from < expiration_cutoff.cutoff_date;

-- Insert new records
INSERT INTO target_users (user_id, name, email, valid_from, valid_to, is_current)
SELECT
    user_id, name, email, CURRENT_TIMESTAMP AS valid_from,
    NULL AS valid_to,
    CASE
        WHEN NOT EXISTS (
            SELECT 1 FROM target_users
            WHERE user_id = source.user_id AND is_current = TRUE
        ) THEN TRUE
        ELSE FALSE
    END AS is_current
FROM staging_users source
WHERE NOT EXISTS (
    SELECT 1 FROM target_users
    WHERE user_id = source.user_id AND valid_from > CURRENT_TIMESTAMP - INTERVAL '30 days'
);

-- Clean up fully expired records
DELETE FROM target_users
WHERE valid_to < CURRENT_TIMESTAMP - INTERVAL '30 days';
```

---

### Type 4 - Add New and Close Old

**Description**: Like Type 2, but explicitly set valid_to for old records when inserting new.

**When to Use**:
- Want explicit history tracking
- Clear audit trail of when records closed
- Easier data governance

**SQL Implementation**:

```sql
-- SCD Type 4: Add new and close old
-- Close old current record
UPDATE target_users
SET valid_to = CURRENT_TIMESTAMP, is_current = FALSE
WHERE user_id = source.user_id AND is_current = TRUE;

-- Insert new record
INSERT INTO target_users (user_id, name, email, valid_from, valid_to, is_current)
SELECT
    user_id, name, email, CURRENT_TIMESTAMP, NULL, TRUE
FROM staging_users source;
```

---

### Type 6 / Type 7 - Hybrid (Multiple Attributes)

**Description**: Track different attributes with different SCD types. For example, name changes frequently (SCD Type 1), address changes rarely (SCD Type 2).

**When to Use**:
- Some attributes change frequently, others rarely
- Want optimal storage for each attribute type
- Mixed reporting requirements

**Example**:
```
User table:
| user_id | name (SCD 1) | address (SCD 2) | email (SCD 0) |
|---------|-----------------|-----------------|---------------|
| 101 | Alice, 2024-01-01 | NULL, 2024-01-01 | alice@old.com |
| 101 | Alice, 2024-01-02 | 123 Main St, 2024-01-05 | alice@old.com |

Note: address has history (multiple records), name is current (one record)
```

**SQL Implementation**:

```sql
-- Hybrid: Name SCD 1 (overwrite), Address SCD 2 (add new)

-- Name (SCD Type 1): Overwrite
MERGE INTO target_users AS target
USING staging_users AS source
ON target.user_id = source.user_id
WHEN MATCHED THEN
    UPDATE SET
        name = source.name,
        name_updated_at = CURRENT_TIMESTAMP
WHEN NOT MATCHED THEN
    INSERT (user_id, name, name_updated_at)
    VALUES (source.user_id, source.name, CURRENT_TIMESTAMP);

-- Address (SCD Type 2): Add new row on change
INSERT INTO target_users_addresses (user_id, address, valid_from, is_current)
SELECT
    user_id, address, CURRENT_TIMESTAMP,
    CASE
        WHEN NOT EXISTS (
            SELECT 1 FROM target_users_addresses
            WHERE user_id = source.user_id AND is_current = TRUE
        ) THEN TRUE
        ELSE FALSE
    END AS is_current
FROM staging_users_addresses source;

-- Email (SCD Type 0): Simple overwrite
UPDATE target_users
SET email = source.email, email_updated_at = CURRENT_TIMESTAMP
WHERE user_id = source.user_id;
```

---

## SCD Implementation Best Practices

### 1. Surrogate Key

**Always use a surrogate key** (auto-increment ID) as the primary key, not the natural key.

```sql
CREATE TABLE target_users (
    surrogate_key BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    user_id BIGINT NOT NULL,  -- Natural business key
    name VARCHAR(100),
    email VARCHAR(255),
    valid_from TIMESTAMP,
    valid_to TIMESTAMP,
    is_current BOOLEAN
);
```

**Why**:
- Natural keys can change (e.g., email changes)
- Surrogate key never changes
- Enables tracking history even when business key changes

### 2. Effective Dates (valid_from, valid_to)

Always include `valid_from` and `valid_to` timestamps.

```sql
-- Query current state
SELECT * FROM target_users WHERE is_current = TRUE;

-- Query history as of specific time
SELECT * FROM target_users
WHERE valid_from <= '2024-01-15' AND
      (valid_to IS NULL OR valid_to > '2024-01-15')
ORDER BY valid_from DESC;
```

### 3. Flags and Metadata

Add metadata to support queries and data quality.

```sql
CREATE TABLE target_users (
    surrogate_key BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    name VARCHAR(100),
    email VARCHAR(255),
    valid_from TIMESTAMP,
    valid_to TIMESTAMP,
    is_current BOOLEAN,
    scd_type SMALLINT,  -- Which SCD type (0, 1, 2, 3, 4)
    updated_at TIMESTAMP,
    source_system VARCHAR(50)  -- Where data came from
    row_hash VARCHAR(64)  -- For deduplication
);
```

### 4. Performance Optimization

```sql
-- Indexes for SCD queries
CREATE INDEX idx_target_users_user_id ON target_users(user_id);
CREATE INDEX idx_target_users_current ON target_users(is_current) WHERE is_current = TRUE;
CREATE INDEX idx_target_users_validity ON target_users(valid_from, valid_to);

-- Partition by date range for large tables
CREATE TABLE target_users_partitioned (
    surrogate_key BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    name VARCHAR(100),
    email VARCHAR(255),
    valid_from TIMESTAMP,
    valid_to TIMESTAMP,
    is_current BOOLEAN
) PARTITION BY RANGE (valid_from);
```

---

## SCD vs. Event Sourcing

| Aspect | SCD | Event Sourcing |
|---------|-------|----------------|
| **History** | Snapshots at specific times | Complete event log |
| **Storage** | More efficient (current state) | Larger (all events) |
| **Queries** | Point-in-time easy | Requires re-computation |
| **Updates** | In-place updates | Append-only |
| **Use Case** | Data warehouse | Event-driven systems |

---

## Common Interview Questions

**Q1: What are the different SCD types?**

**A1**:
- **Type 0**: Overwrite - No history, just current state
- **Type 1**: Add new - Full history, growing storage
- **Type 2**: Update existing - First change adds row, rest update current
- **Type 3**: Add and expire - Fixed window of history
- **Type 4**: Add and close - Explicit history tracking
- **Type 6/7**: Hybrid - Different types for different attributes

**Q2: When would you use SCD Type 0 vs Type 1?**

**A2**:
- **Type 0 (Overwrite)**: When history doesn't matter, storage is cheap, simplicity is priority
- **Type 1 (Add new)**: When you need complete history for audit, trend analysis, or compliance

**Q3: What's a surrogate key and why use it?**

**A3**: A surrogate key is an auto-generated ID (like SERIAL, IDENTITY). Use it because:
- Natural keys can change (email, phone number)
- Surrogate key never changes
- Ensures referential integrity even when business keys change
- Supports tracking multiple versions of the same entity

**Q4: How do you handle slowly changing dimensions in a data warehouse?**

**A4**:
1. **Identify**: Determine which attributes change slowly
2. **Choose SCD Type**: Based on requirements (history needed, storage constraints)
3. **Implement**: Use MERGE or INSERT/UPDATE logic
4. **Query**: Use `is_current` flag for current state, time filters for history
5. **Maintain**: Regularly clean up expired data (Type 3)

**Q5: What's the difference between SCD Type 1 and Type 2?**

**A5**:
- **Type 1**: Adds new row for EVERY change. History grows unbounded.
- **Type 2**: Adds new row for FIRST change, then UPDATES the current record. More storage-efficient.

**Example**:
```
SCD Type 1: 10 changes = 10 rows
SCD Type 2: 10 changes = 3 rows (first insert + 9 updates)
```

---

## Related Notes

- [[Data Modeling]] - Core dimensional modeling concepts
- [[Data Warehousing]] - SCDs in warehouse context
- [[Star Schema]] - Dimensional modeling pattern
- [[Snowflake Schema]] - Alternative dimensional pattern

---

## Resources

- [ThoughtSpot: Slowly Changing Dimensions](https://www.thoughtspot.com/data-trends/data-modeling/slowly-changing-dimensions)
- [DataCamp: Mastering SCDs](https://www.datacamp.com/tutorial/mastering-slowly-changing-dimensions)
- [Express Analytics: SCD Implementation Logic](https://www.expressanalytics.com/blog/what-is-a-slowly-changing-dimension-and-the-logic-in-implementation)
- [Oracle: Understanding SCDs](https://docs.oracle.com/cd/E41507_01/epm91pbr3/eng/epm/phcw/concept_UnderstandingSlowlyChangingDimensions-405719.html)

---

## Practical Example: Complete SCD Type 2 Implementation

### Python: SCD Type 2 ETL Pipeline

```python
import psycopg2
from datetime import datetime

def scd_type_2_upsert(staging_data, conn):
    """SCD Type 2: Update existing, insert new"""
    cursor = conn.cursor()
    
    for record in staging_data:
        user_id = record['user_id']
        name = record['name']
        email = record['email']
        
        # Check if user exists and is current
        cursor.execute("""
            SELECT surrogate_key, name, email
            FROM target_users
            WHERE user_id = %s AND is_current = TRUE
        """, (user_id,))
        
        existing = cursor.fetchone()
        
        if existing:
            existing_name = existing[1]
            existing_email = existing[2]
            
            # Check if any field changed
            if existing_name != name or existing_email != email:
                # Update existing to set valid_to and is_current = FALSE
                cursor.execute("""
                    UPDATE target_users
                    SET valid_to = CURRENT_TIMESTAMP, is_current = FALSE
                    WHERE surrogate_key = %s
                """, (existing[0],))
                
                # Insert new record as current
                cursor.execute("""
                    INSERT INTO target_users
                        (user_id, name, email, valid_from, valid_to, is_current)
                    VALUES (%s, %s, %s, CURRENT_TIMESTAMP, NULL, TRUE)
                """, (user_id, name, email))
            # No change - do nothing
        else:
            # First record - insert as current
            cursor.execute("""
                INSERT INTO target_users
                        (user_id, name, email, valid_from, valid_to, is_current)
                    VALUES (%s, %s, %s, CURRENT_TIMESTAMP, NULL, TRUE)
            """, (user_id, name, email))
    
    conn.commit()
    cursor.close()
```

### dbt Model: SCD Type 2

```sql
-- models/dim_users_scd2.sql
{{
    "materialized": "table",
    "unique_key": "user_id",
    "incremental_strategy": "insert_overwrite",
    
    "config": {
        "tags": ["dim", "users", "scd2"]
    }
}}

-- SELECT to detect changes and implement SCD Type 2
{% set is_first_load = flags.get('dim_users_scd2', False) %}

WITH staged_users AS (
    SELECT
        user_id,
        name,
        email,
        '{{ run_started_at }}' AS valid_from,
        NULL AS valid_to,
        TRUE AS is_current
    FROM {{ source('raw_users') }}
    WHERE {{ is_first_load }}
),

changes AS (
    SELECT
        s.user_id,
        s.name,
        s.email,
        CURRENT_TIMESTAMP AS valid_from,
        NULL AS valid_to,
        TRUE AS is_current
    FROM {{ source('raw_users') }} s
    JOIN {{ target('dim_users_scd2') }} t
        ON s.user_id = t.user_id
    WHERE t.is_current AND (s.name != t.name OR s.email != t.email)
),

merged_changes AS (
    SELECT * FROM staged_users
    UNION ALL
    SELECT * FROM changes
),

-- Mark old current records as expired
expired_current AS (
    SELECT
        t.surrogate_key,
        t.user_id,
        t.name,
        t.email,
        t.valid_from,
        CURRENT_TIMESTAMP AS valid_to,
        FALSE AS is_current
    FROM {{ target('dim_users_scd2') }} t
    WHERE t.is_current AND EXISTS (
        SELECT 1 FROM merged_changes c WHERE c.user_id = t.user_id
    )
)

-- Merge: Update existing or insert new
SELECT
    m.user_id,
    m.name,
    m.email,
    m.valid_from,
    m.valid_to,
    m.is_current
FROM merged_changes m
```

---

## Summary

**Key Takeaways**:
1. SCDs enable time-travel and historical tracking
2. Type 0: Fast, no history - Use for reference data only
3. Type 1: Complete history - Use when audit is required
4. Type 2: Storage-efficient - Use when history needed but storage is concern
5. Type 3: Time-windowed - Use for compliance (data retention)
6. Always use surrogate keys (auto-increment) as primary key
7. Include `is_current` flag for easy "current state" queries
8. Index on user_id and is_current for performance

---

**Progress**: 🟢 Learning (concepts understood, need implementation practice)

**Next Steps**:
- [ ] Implement SCD Type 2 with dbt
- [ ] Create SCD Type 1 for reference data
- [ ] Add SCD metrics (change rate, history depth)
- [ ] Practice querying historical data points

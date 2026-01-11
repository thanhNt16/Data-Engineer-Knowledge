# System Design Framework

> A structured approach to data engineering system design interviews

## 🎯 6-Layer Framework for Data System Design

Based on industry best practices, use this 6-layer framework to structure your system design answers:

```
┌─────────────────────────────────────────────────────────┐
│  1. Requirements Clarification                          │
├─────────────────────────────────────────────────────────┤
│  2. Conceptual Design                                   │
├─────────────────────────────────────────────────────────┤
│  3. Data Modeling                                       │
├─────────────────────────────────────────────────────────┤
│  4. Technology Selection                                │
├─────────────────────────────────────────────────────────┤
│  5. Architecture Design                                 │
├─────────────────────────────────────────────────────────┤
│  6. Scalability & Trade-offs                            │
└─────────────────────────────────────────────────────────┘
```

---

## Layer 1: Requirements Clarification

### Key Questions to Ask

**Functional Requirements**:
- What problem are we solving?
- What are the use cases?
- Who are the users?
- What data do we need to collect?
- What questions do we need to answer?

**Non-Functional Requirements**:
- **Scale**: What's the data volume? (GB, TB, PB)
- **Velocity**: Batch or real-time? (events/day, events/second)
- **Latency**: How fresh does data need to be? (minutes, seconds, real-time)
- **Consistency**: Strong or eventual consistency acceptable?
- **Availability**: Uptime requirements? (99.9%, 99.99%)
- **Cost**: Budget constraints?

### Example Use Cases

| Use Case | Volume | Velocity | Latency | Complexity |
|----------|--------|----------|---------|------------|
| Clickstream Analytics | TB/day | Millions/sec | Real-time | High |
| Daily Reporting | GB/day | Batch | Hours | Low |
| Real-time Recommendations | PB/day | Millions/sec | Sub-second | Very High |
| Data Warehouse | TB/month | Batch | Daily | Medium |

---

## Layer 2: Conceptual Design

### High-Level Architecture

```
Sources → Ingestion → Processing → Storage → Serving
```

### Key Decisions

1. **Processing Model**: Batch vs Stream vs Hybrid
2. **Storage Pattern**: Data Lake, Warehouse, Lakehouse
3. **Architecture Pattern**: Lambda, Kappa, or Modern
4. **Serving Layer**: OBT, Aggregate Tables, or Query Federation

### Architecture Patterns

#### Lambda Architecture (Classic)
```
                    ┌──────────────┐
                    │   Serving    │
                    └──────┬───────┘
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
     ┌─────────┐    ┌─────────┐    ┌─────────┐
     │  Batch  │    │ Speed   │    │  Real-  │
     │  Layer  │    │  Layer  │    │  time   │
     └────┬────┘    └────┬────┘    └────┬────┘
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                    ┌─────────┐
                    │ Ingest  │
                    └─────────┘
```

#### Kappa Architecture (Modern)
```
┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐
│ Source  │ → │ Ingest  │ → │ Process │ → │ Serving │
└─────────┘   └─────────┘   └─────────┘   └─────────┘
                                    │
                                    ▼
                              ┌─────────┐
                              │ Replay  │
                              │ Stream │
                              └─────────┘
```

---

## Layer 3: Data Modeling

### 3-Hop Architecture (Multi-hop)

```
┌──────────┐    ┌──────────┐    ┌──────────┐
│ Staging  │ → │  Core    │ → │ Serving  │
│  (Raw)   │    │ (Clean)  │    │ (Mart)   │
└──────────┘    └──────────┘    └──────────┘
```

**1. Staging Layer (Bronze)**
- Raw data from sources
- Minimal transformation
- Schema-on-read
- Purpose: Data lake, audit trail

**2. Core Layer (Silver)**
- Cleaned & validated
- Fact & Dimension tables
- Enforced data types
- Purpose: Reusable data assets

**3. Serving Layer (Gold)**
- Business-ready
- One Big Table (OBT) or Aggregates
- Optimized for queries
- Purpose: End-user consumption

### Table Types

| Type | Purpose | Example |
|------|---------|---------|
| **Dimension** | Descriptive data | Customers, Products |
| **Fact** | Events/transactions | Orders, Clicks |
| **Bridge** | Many-to-many | Account-Customer |
| **OBT** | Simplified access | Sales + all dims |
| **Aggregate** | Performance | Monthly summaries |

---

## Layer 4: Technology Selection

### Decision Matrix

| Scenario | Recommended Stack |
|----------|------------------|
| **Small-Medium (<10TB)** | BigQuery, Snowflake, dbt |
| **Large (>100TB)** | Spark + S3 + Redshift |
| **Real-time** | Kafka + Kinesis + Flink |
| **Cost-Optimized** | Athena + S3 + Glue |
| **Analytics-Focused** | Snowflake + dbt + Looker |
| **Engineering-Focused** | Airflow + Spark + Iceberg |

### Database Selection

| Use Case | SQL | NoSQL | Warehouse | Lake |
|----------|-----|-------|-----------|------|
| Transactional | ✅ PostgreSQL | ✅ DynamoDB | ❌ | ❌ |
| Analytics | ❌ | ❌ | ✅ BigQuery | ✅ Athena |
| Real-time | ❌ | ❌ | ❌ | ✅ Delta |
| Mixed | ❌ | ❌ | ✅ Lakehouse | ✅ |

---

## Layer 5: Architecture Design

### Pipeline Components

```
┌──────────────┐
│   Sources    │
│  (API, DB)   │
└───────┬──────┘
        │
        ▼
┌──────────────┐
│   Ingestion  │  ← Kafka, Kinesis
│  (Buffer)    │
└───────┬──────┘
        │
        ▼
┌──────────────┐
│ Processing   │  ← Spark, Flink, dbt
└───────┬──────┘
        │
        ▼
┌──────────────┐
│   Storage    │  ← S3, Redshift
│  (Serving)   │
└──────────────┘
```

### Key Components

**1. Ingestion**
- Batch: Scheduled jobs, API calls
- Stream: Kafka, Kinesis, Pub/Sub
- CDC: Debezium, Airbyte

**2. Processing**
- Batch: Spark, Hive, dbt
- Stream: Flink, Spark Streaming, Kinesis Data Analytics
- Transformation: SQL, Python, Scala

**3. Storage**
- Hot: Redshift, BigQuery, Snowflake
- Warm: S3, ADLS, GCS
- Cold: Glacier, Archive

**4. Orchestration**
- Airflow, Dagster, Prefect
- Scheduling, monitoring, retries

**5. Quality**
- Great Expectations, Soda, dbt tests
- Data validation, anomaly detection

---

## Layer 6: Scalability & Trade-offs

### Scalability Strategies

**Vertical Scaling (Scale Up)**
- Larger instance
- More memory/CPU
- Simpler architecture
- Eventual limit

**Horizontal Scaling (Scale Out)**
- More instances
- Distributed processing
- Complex coordination
- Nearly unlimited

### Performance Optimization

| Technique | Use Case | Trade-off |
|-----------|----------|-----------|
| Partitioning | Large table scans | More files |
| Clustering | Filter optimization | Write overhead |
| Caching | Frequent queries | Staleness risk |
| Materialization | Pre-computed aggregations | Storage cost |
| Denormalization | Query performance | Data redundancy |

### Common Trade-offs

| Trade-off | Option A | Option B | Choose A when... |
|-----------|----------|----------|------------------|
| **Latency vs Cost** | Real-time | Batch | Real-time is critical |
| **Consistency vs Availability** | Strong | Eventual | Accuracy matters |
| **Schema** | Schema-on-write | Schema-on-read | Structure is stable |
| **Processing** | Batch | Stream | Latency > minutes OK |
| **Storage** | Hot (Warehouse) | Cold (Lake) | Frequent queries |

### Anti-Patterns to Avoid

1. **Premature Optimization** - Don't optimize before you have a problem
2. **Over-engineering** - Keep it simple initially
3. **Ignoring Data Quality** - Build it in from day one
4. **No Monitoring** - You can't improve what you don't measure
5. **Tight Coupling** - Design for change

---

## 🎯 Example: Design a Real-Time Analytics System

### Requirements
- 1M events/second
- Sub-second query latency
- 100 TB/day
- Real-time dashboards

### Solution

```
┌──────────┐
│  Mobile  │
│   Apps   │
└────┬─────┘
     │
     ▼
┌──────────────┐
│    Kafka     │  ← Ingestion buffer
│   (100 TB)   │
└──────┬───────┘
       │
       ├─────────────────┐
       │                 │
       ▼                 ▼
┌──────────────┐  ┌──────────────┐
│   Flink      │  │   S3 Raw     │
│  (Process)   │  │   (Backup)   │
└──────┬───────┘  └──────────────┘
       │
       ▼
┌──────────────┐
│  Redshift    │  ← Hot storage for queries
│  (Serving)   │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│   Dashboard  │
│    (BI)      │
└──────────────┘
```

### Key Decisions
- **Kafka**: Handle 1M events/sec burst
- **Flink**: Real-time processing, windowing
- **Redshift**: Fast queries with clustering
- **S3**: Cost-effective raw storage

### Trade-offs
- ✅ Real-time: Sub-second latency
- ❌ Cost: Redshift is expensive
- ✅ Scalable: Can add more partitions
- ❌ Complexity: Multiple systems to maintain

---

## 📋 System Design Checklist

### Requirements
- [ ] Clarified functional requirements
- [ ] Identified non-functional requirements
- [ ] Understood scale & constraints
- [ ] Defined success metrics

### Architecture
- [ ] Chose processing model (batch/stream)
- [ ] Designed high-level architecture
- [ ] Identified key components
- [ ] Defined data flow

### Data Modeling
- [ ] Created fact/dimension model
- [ ] Defined table types
- [ ] Specified grain for each table
- [ ] Designed partitioning strategy

### Technology
- [ ] Selected storage solution
- [ ] Chose processing framework
- [ ] Defined orchestration
- [ ] Planned monitoring

### Trade-offs
- [ ] Explicitly stated trade-offs
- [ ] Justified key decisions
- [ ] Identified risks
- [ ] Proposed mitigation

---

## 🔗 Related Topics

- [[02-Areas/System Design]]
- [[02-Areas/Data Modeling]]
- [[05-Interview Prep/System Design Cases]]

---

**Resources**:
- [System Design Interview Cheat Sheet](https://blog.surfalytics.com/p/ultimate-cheatsheet-for-data-engineering)
- [Data Engineering System Design](https://seattledataguy.substack.com/p/how-i-run-system-design-interviews)

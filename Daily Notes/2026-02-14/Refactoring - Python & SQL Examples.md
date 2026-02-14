---
tags: [refactoring, python, sql, examples, streaming, 2026-02-14]
date: 2026-02-14
status: learned
---

# Refactoring: Python & SQL Examples (2026-02-14)

## What Was Refactored

Added comprehensive Python and SQL examples to all streaming and trends notes for hands-on learning.

### Files Updated

1. **Apache Flink - Real-Time Analytics**
   - Added Flink Python API examples (DataStream API, Table API)
   - Added Flink SQL examples (tumbling, sliding, session windows)
   - Added complex event processing (CEP) patterns
   - Added stateful processing with Python
   - Added performance optimization examples

2. **Apache Kafka - Event Streaming**
   - Added advanced producer patterns (batch, streaming)
   - Added advanced consumer patterns (multi-topic, metrics)
   - Added kSQL/Streaming SQL examples
   - Added end-to-end dashboard example (producer + consumer + kSQL)
   - Added SQL integration examples (ETL, analytics queries)
   - Added performance optimization tips

3. **Data Engineering Trends 2026**
   - Added CDC Python implementation examples
   - Added streaming-first lakehouse Python/SQL examples
   - Added data quality checks (Great Expectations + SQL)
   - Added AI-ready pipelines (Feature Store with Feast)
   - Added platformization (Data Contracts with Python + SQL)
   - Added Dagster vs Airflow comparison with code
   - Added traditional vs 2026 approach comparison table

---

## Python Examples Added

### Flink
- Word count with DataStream API
- Windowed aggregations with Table API
- Stream joins with SQL
- Stateful processing (KeyedProcessFunction)
- Checkpointing configuration

### Kafka
- Batch producer with error handling
- Real-time streaming producer
- Multi-topic consumer
- Consumer with metrics tracking
- End-to-end dashboard (producer + consumer)

### Trends
- Debezium CDC event handling
- Kafka to Iceberg pipeline
- Great Expectations validation
- Feature Store integration (Feast)
- Data contract enforcement (Pydantic)
- Dagster software-defined assets
- Airflow traditional DAGs

---

## SQL Examples Added

### Flink SQL
- Tumbling window aggregations
- Sliding window with late data
- Session windows (user activity)
- Complex event processing (price surge detection)
- Stream-stream joins

### kSQL (Kafka Streaming SQL)
- Create stream from topic
- Windowed aggregations (5-minute windows)
- Filter and transform operations
- Stream-stream joins (orders + users)
- Pattern matching (suspicious users)
- Real-time dashboard metrics

### Traditional SQL (for comparison)
- CDC merge operations (PostgreSQL, Snowflake)
- Iceberg lakehouse queries (time travel, partitioning)
- Data quality checks (nulls, ranges, duplicates, referential integrity)
- Analytics queries (daily revenue, top products, RFM segmentation)
- Data contract validations (constraints, quality scores)

---

## Key Learnings

### Python Patterns

| Pattern | Use Case | Example |
|----------|-------------|----------|
| **Batch processing** | High-throughput ingestion | `for batch in chunks(records): producer.send_batch()` |
| **Streaming** | Real-time processing | `while True: producer.send(topic, value)` |
| **Stateful** | Remember context across events | `self.state_state.update(count)` |
| **Error handling** | Robust production code | `future.add_errback(self.error_handler)` |
| **Idempotency** | Exactly-once guarantees | `enable_idempotence=True` |

### SQL Patterns

| Pattern | Use Case | Example |
|----------|-------------|----------|
| **Windowing** | Time-based aggregations | `TUMBLE(time, INTERVAL '5 MINUTE')` |
| **Merging** | Upsert for CDC | `MERGE INTO target USING source ON id` |
| **Time travel** | Query historical state | `AS OF SYSTEM TIME 'timestamp'` |
| **CEP** | Pattern matching | `MATCH_RECOGNIZE(PATTERN (A B+))` |
| **Data quality** | Validation checks | `WHERE column IS NULL OR column < 0` |

---

## Next Steps

Based on refactored notes:
1. **Practice Flink locally** - Run Python examples with Docker
2. **Build Kafka dashboard** - End-to-end producer/consumer/kSQL
3. **Implement CDC** - Try Debezium with Python handler
4. **Setup Great Expectations** - Add data quality checks
5. **Try Dagster** - Compare with Airflow workflow
6. **Build streaming lakehouse** - Kafka → Iceberg → queries

---

## Related Notes

- [[02-Areas/Streaming/Apache Flink - Real-Time Analytics]] - Now with Python + SQL
- [[02-Areas/Streaming/Apache Kafka - Event Streaming]] - Now with Python + SQL
- [[03-Resources/Data Engineering Trends 2026]] - Now with implementation examples

---

**Progress**: 🟢 All refactored with practical code examples

**Impact**: Notes now include 50+ Python + 50+ SQL examples for hands-on learning

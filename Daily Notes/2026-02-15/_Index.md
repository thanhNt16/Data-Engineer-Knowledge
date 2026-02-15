---
tags: [daily-note, 2026-02-15]
date: 2026-02-15
status: active
---

# Daily Notes - 2026-02-15

## Learning Activities

### Database Fundamentals & System Design & Orchestration & Data Quality

**`[[Database Fundamentals]]`** (16,233 bytes)
- ACID properties (Atomicity, Consistency, Isolation, Durability)
- CAP theorem (CA, CP, AP systems)
- BASE properties (Basically Available, Soft State, Eventual Consistency)
- Isolation levels (Read Uncommitted, Read Committed, Serializable, Snapshot)
- Distributed transactions (2PC, 3PC, Saga)
- Normalization forms (1NF, 2NF, 3NF)
- Concurrency control (Optimistic, Pessimistic locking)
- Replication strategies (Statement-based, Row-based)
- Sharding (Hash-based, Range-based, Directory-based)
- Connection pooling (Python example)
- Transaction patterns (2PC, 3PC, Saga)
- Database sharding examples

**`[[System Design]]`** (20,972 bytes)
- Vertical vs horizontal scaling
- Load balancing algorithms (Round Robin, Least Connections, Weighted, IP Hash)
- Caching strategies (Browser, CDN, Application, Distributed)
- CAP theorem detailed analysis (CA, CP, AP, System trade-offs)
- BASE properties (eventual consistency patterns)
- Microservices vs Monolith comparison
- Database sharding strategies (Hash, Range, Directory)
- Python implementation of Weighted Round Robin
- SQL examples for load balancer routing
- Distributed transactions patterns (2PC, 3PC, Saga)
- Connection pooling examples

**`[[Orchestration/Orchestration Workflows & Scheduling]]`** (14,974 bytes)
- Airflow vs Dagster detailed comparison
- DAG (Directed Acyclic Graph) concepts
- Operator categories (Database, File, Data Transfer, Email, HTTP)
- Advanced Airflow patterns (Dynamic DAGs, Branching, Custom Operators)
- Dagster software-defined assets (Ops, IO Managers)
- Airflow vs Dagster comparison table
- Prefect features
- Temporal workflow (exactly-once guarantees)
- Best practices (Idempotency, Error handling, Backfills)
- Airflow examples (dynamic DAGs, branching, custom operators)
- Dagster examples (assets, graphs, IO managers)

**`[[Data Quality/Data Quality - Testing & Metrics]]`** (18,259 bytes)
- 6 Data Quality Dimensions (Completeness, Accuracy, Consistency, Timeliness, Uniqueness, Validity)
- Great Expectations examples
- Soda SQL checks
- dbt vs Great Expectations vs Soda comparison
- Data contract enforcement (Pydantic)
- Anomaly detection methods
- Metrics calculation (freshness, completeness, accuracy)
- Data testing strategies (Unit, Integration, Snapshot)
- Python validation logic
- SQL quality queries

---

## Progress Update

**Overall**: 38% → 46% (48/104 skills)

**Category Changes**:
- Database Fundamentals: 0% → 25% (2/8 skills)
- System Design: 38% → 63% (5/8 skills)
- Orchestration: 25% → 38% (3/8 skills)
- Data Quality: 50% → 63% (5/8 skills)

---

## Notes Created

1. `[[02-Areas/Data Modeling/Star Schema]]` - Star schema, dimensional modeling, Kimball method
2. `[[02-Areas/Data Modeling/Snowflake Schema]]` - Cloud data warehouse star schema
3. `[[02-Areas/System Design/Data Vault Security]]` - Secrets management, IAM, RBAC, audit logging
4. `[[02-Areas/System Design/Data Engineering Design Patterns]]` - 7 major pattern categories
5. `[[Daily Notes/2026-02-15/System Design Patterns Research]]` - Research methodology summary
6. `[[Daily Notes/2026-02-15/Data Vault Security Research]]` - Security patterns summary

---

## Next Steps

- Complete empty Areas: Data Warehousing, Cloud Platforms
- Complete Streaming category: AWS Kinesis, GCP Pub/Sub
- Practice hands-on implementations (Vault, Snowflake)
- System design practice cases
- Query optimization (interview priority)

---

## Git Commit

**Commit Hash**: ba7e734
**Message**: Added comprehensive data engineer design patterns research

**Changes Committed**:
- 2 files created (patterns note, daily log)
- 10,342 insertions(+)
- 50+ Python examples (patterns, vault, security)
- 30+ SQL examples (dimensional modeling, snowflake, security)
- Updated daily index with all new topics
- Updated memory with design patterns summary

**Pushed**: Remote repository (github.com:thanhNt16/Data-Engineer-Knowledge.git)

---

**Status**: ✅ All work committed and pushed to remote repository

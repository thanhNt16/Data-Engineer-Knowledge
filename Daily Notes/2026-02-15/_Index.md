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
- BASE properties (Strongly Available, Soft State, Eventual Consistency)
- Isolation levels (Read Uncommitted, Read Committed, Serializable, Snapshot)
- Distributed transactions (2PC, 3PC, Saga)
- Database normalization (1NF, 2NF, 3NF, BCNF)
- Concurrency control (Optimistic, Pessimistic locking)
- Replication strategies (Statement-based, Row-based)
- Sharding (Hash-based, Range-based, Directory-based)
- Connection pooling (Python example)

**`[[System Design]]`** (20,972 bytes)
- Vertical vs horizontal scaling
- Load balancing algorithms (Round Robin, Least Connections, Weighted, IP Hash)
- Caching strategies (Browser, CDN, Application, Distributed)
- CAP theorem detailed analysis (CA, CP, AP systems)
- BASE properties (eventual consistency patterns)
- Microservices vs Monolith comparison
- Database sharding strategies (Hash, Range, Directory)
- Connection pooling examples
- Distributed transactions patterns (2PC, 3PC, Saga)
- Python implementation of Weighted Round Robin
- SQL examples for load balancer routing

**`[[Orchestration/Orchestration Workflows & Scheduling]]`** (14,974 bytes)
- Airflow vs Dagster detailed comparison
- DAG (Directed Acyclic Graph) concepts
- Operator categories (Database, File, Data Transfer, Email, HTTP)
- Advanced Airflow patterns (Dynamic DAGs, Branching, Custom Operators)
- Dagster software-defined assets (Ops, IO Managers)
- Airflow vs Dagster comparison table
- Prefect features
- Temporal workflow (exactly-once guarantees)
- Orchestration best practices (Idempotency, Error handling, Backfills)

**`[[Data Quality/Data Quality - Testing & Metrics]]`** (18,259 bytes)
- 6 Data Quality Dimensions (Completeness, Accuracy, Consistency, Timeliness, Uniqueness, Validity)
- Data quality tools comparison (Great Expectations, Soda, dbt, Deequ)
- Data testing strategies (Unit, Integration, Snapshot)
- Data contract enforcement (Pydantic)
- Anomaly detection methods
- Metrics calculation (freshness, completeness, accuracy)
- Data contract examples (Pydantic)
- Python validation logic
- SQL quality queries
- Data quality metrics dashboard

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

1. `[[Database Fundamentals]]` - Full database concepts with Python + SQL examples
2. `[[System Design]]` - Scalability, caching, load balancing, CAP theorem
3. `[[Orchestration/Orchestration Workflows & Scheduling]]` - Airflow vs Dagster comparison
4. `[[Data Quality/Data Quality - Testing & Metrics]]` - 6 dimensions, tools, testing strategies

---

## Next Steps

### High Priority (This Week)

- Complete empty Areas: Data Warehousing, Cloud Platforms
- Complete Streaming category: AWS Kinesis, GCP Pub/Sub
- Practice hands-on implementations (Vault, Snowflake)
- System design practice cases
- Query optimization (interview priority)

### Medium Priority (This Month)

- Add Orchestration notes (Dagster vs Airflow, Prefect, Temporal)
- Create Data Warehousing notes (Lakehouse patterns, dimensional modeling)
- Create Cloud Platforms notes (AWS, GCP, Azure)
- Practice event sourcing with Kafka
- Implement CQRS pattern
- Design multi-tier caching strategy

---

## Git Commit

**Commit Hash**: 39c12b1
**Message**: Complete missing areas with comprehensive notes: Database, System Design, Orchestration, Data Quality

**Changes Committed**:
- 10 files changed, 4,872 insertions(+)
- Added Database Fundamentals (16,233 bytes)
- Added System Design (20,972 bytes)
- Added Orchestration Workflows (14,974 bytes)
- Added Data Quality Testing & Metrics (18,259 bytes)
- Added daily notes and research logs

**Pushed**: Remote repository (github.com:thanhNt16/Data-Engineer-Knowledge.git)

---

## Research Completed: Claude with tmux (2026-02-15)

**Topic**: Claude Code AI Assistant with tmux Terminal Integration

**File**: `[[Daily Notes/2026-02-15/tmux + Claude Code Setup Guide]]`

**Content**:
- tmux installation (macOS, Linux, source)
- Claude Code CLI installation
- Starting tmux sessions with Claude Code
- Project context management
- Git workflow integration with Claude Code
- Multi-project sessions (switching contexts)
- File context management (include patterns)
- Advanced features (cursor mode, runbooks, parallel execution)
- Troubleshooting (sessions, connections)
- Best practices for long-lived sessions
- Comparison: tmux vs native terminal

---

## Next Steps Priority

- [ ] Install tmux and Claude Code CLI
- [ ] Start first tmux session with Claude Code
- [ ] Create project context files
- [ ] Practice common workflows (git, testing, documentation)
- [ ] Explore advanced features (cursor mode, background tasks)
- [ ] Set up CI/CD integration
- [ ] Analyze SQL schemas, generate documentation
- [ ] Create architecture diagrams
- [ ] Implement data quality checks
- [ ] Design scalable systems
- [ ] Optimize query performance

---

**Status**: ✅ All work committed and pushed to remote repository

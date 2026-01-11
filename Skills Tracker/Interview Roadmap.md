# Data Engineering Interview Roadmap 2026

> Comprehensive guide to prepare for data engineering interviews at top companies

## 📋 Interview Format Overview

Most companies follow this interview structure:
1. **Recruiter Screen** (30 min) - Background, interests, basic fit
2. **Technical Screen** (45-60 min) - SQL, coding, or system design
3. **On-site Rounds** (4-6 rounds)
   - SQL & Data Modeling
   - Coding/DSA
   - System Design
   - Behavioral
   - Role-specific (Data Engineering deep-dive)

---

## 🎯 Phase 1: Fundamentals (Weeks 1-4)

### SQL Mastery
**Status**: 🟡 In Progress

**Topics to Cover**:
- [x] Window Functions (ROW_NUMBER, RANK, LAG/LEAD)
- [x] CTEs (Common Table Expressions)
- [x] Advanced JOINs (INNER, LEFT, RIGHT, FULL, CROSS, SEMI/ANTI)
- [ ] Query Optimization (EXPLAIN, indexing, partitioning)
- [ ] Aggregate Functions (GROUP BY, HAVING,ROLLUP, CUBE)
- [ ] String Manipulation & Regex
- [ ] Date/Time Operations
- [ ] Performance Tuning

**Practice**: [[05-Interview Prep/SQL Questions]]

---

### Data Modeling
**Status**: 🟡 In Progress

**Topics to Cover**:
- [x] Dimensional Modeling (Star vs Snowflake)
- [x] Fact & Dimension Tables
- [x] SCD (Slowly Changing Dimensions)
- [ ] Normalization vs Denormalization
- [ ] Data Vault Modeling
- [ ] Bus Matrix
- [ ] One Big Table (OBT) Pattern
- [ ] Bridge Tables (Many-to-Many)

**Practice**: [[05-Interview Prep/Data Modeling]]

---

### Database Fundamentals
**Status**: 🟢 Completed

**Topics to Cover**:
- [x] ACID Properties
- [x] CAP Theorem
- [x] BASE Properties
- [ ] Indexing Strategies (B-Tree, Hash, Bitmap)
- [ ] Concurrency Control
- [ ] Transaction Isolation Levels
- [ ] Partitioning & Sharding
- [ ] Replication

---

## 🚀 Phase 2: Core Skills (Weeks 5-8)

### ETL/ELT Pipeline Design
**Status**: 🟢 Proficient

**Topics to Cover**:
- [x] Batch vs Stream Processing
- [x] Pipeline Patterns (Multi-hop, Medallion)
- [x] Data Validation & Quality Checks
- [ ] Error Handling & Retry Logic
- [ ] Idempotency
- [ ] Backfill Strategies
- [ ] Incremental Loading
- [ ] Data Lineage

**Key Questions**:
- How would you design a pipeline that processes 1TB daily?
- How do you handle late-arriving data?
- How do you ensure data quality in production?

---

### Data Structures & Algorithms
**Status**: 🟡 Learning

**Topics to Cover**:
- [ ] Arrays, Strings, Hash Maps
- [ ] Linked Lists, Trees, Graphs
- [ ] Sorting & Searching
- [ ] Time & Space Complexity
- [ ] Hash Tables & Collision Resolution
- [ ] Heaps & Priority Queues
- [ ] Graph Algorithms (BFS, DFS)
- [ ] Dynamic Programming Basics

**Practice**: [[05-Interview Prep/Coding-DSA]]

**LeetCode Focus**: Easy-Medium, SQL-heavy

---

### Python for Data Engineering
**Status**: 🟢 Proficient

**Topics to Cover**:
- [x] List/Dict/Set Comprehensions
- [x] File I/O & JSON Processing
- [x] Error Handling & Logging
- [ ] Decorators & Context Managers
- [ ] Generators & Iterators
- [ ] Virtual Environments
- [ ] Testing (pytest, unittest)
- [ ] Type Hints

---

## 🏗️ Phase 3: Advanced Topics (Weeks 9-12)

### System Design
**Status**: 🟡 Learning

**Topics to Cover**:
- [ ] Scalability Principles
- [ ] Caching Strategies
- [ ] Load Balancing
- [ ] Database Selection (SQL vs NoSQL)
- [ ] Message Queues (Kafka, Kinesis)
- [ ] Data Warehouse Design
- [ ] Real-time vs Batch Trade-offs
- [ ] Cost Optimization

**Practice**: [[05-Interview Prep/System Design]]

**Key Questions**:
- Design a real-time analytics system
- Design a data warehouse for an e-commerce company
- Design a pipeline to handle 100M events/day

---

### Streaming & Real-time
**Status**: 🔴 Not Started

**Topics to Cover**:
- [ ] Stream Processing Concepts
- [ ] Apache Kafka
- [ ] Kinesis vs Pub/Sub
- [ ] Windowing (Tumbling, Sliding, Session)
- [ ] Watermarks & Late Data
- [ ] Exactly-Once Semantics
- [ ] Backpressure Handling

---

### Orchestration
**Status**: 🟡 Learning

**Topics to Cover**:
- [ ] Apache Airflow
- [ ] DAG Design Patterns
- [ ] Task Dependencies
- [ ] Scheduling Strategies
- [ ] Monitoring & Alerting
- [ ] Backfills
- [ ] XCom & TaskFlow
- [ ] Operators (TransferOperator, Sensor)

---

## ☁️ Phase 4: Cloud Platforms (Weeks 13-16)

### AWS
**Status**: 🟡 Learning

**Services to Know**:
- [ ] Redshift (Data Warehouse)
- [ ] RDS/DynamoDB (Databases)
- [ ] S3 (Object Storage)
- [ ] Glue (ETL)
- [ ] Kinesis (Streaming)
- [ ] Lambda (Serverless)
- [ ] EMR (Big Data)
- [ ] Step Functions (Orchestration)

---

### GCP
**Status**: 🔴 Not Started

**Services to Know**:
- [ ] BigQuery (Data Warehouse)
- [ ] Bigtable (NoSQL)
- [ ] Pub/Sub (Messaging)
- [ ] Dataflow (Stream & Batch)
- [ ] Cloud Functions (Serverless)
- [ ] Dataproc (Big Data)
- [ ] Composer (Orchestration)

---

### Azure
**Status**: 🔴 Not Started

**Services to Know**:
- [ ] Synapse Analytics (Data Warehouse)
- [ ] SQL Database / Cosmos DB
- [ ] Event Hubs (Streaming)
- [ ] Data Factory (ETL)
- [ ] Databricks (Analytics)
- [ ] Functions (Serverless)

---

## 💬 Phase 5: Behavioral (Ongoing)

**Status**: 🟡 Learning

**Topics to Cover**:
- [ ] STAR Method (Situation, Task, Action, Result)
- [ ] Leadership Principles
- [ ] Conflict Resolution
- [ ] Project Experiences
- [ ] Failures & Learnings
- [ ] Team Collaboration
- [ ] Technical Communication

**Practice**: [[05-Interview Prep/Behavioral]]

**Common Questions**:
- Tell me about a challenging data pipeline you built
- Describe a time you had to make a trade-off decision
- How do you handle data quality issues in production?

---

## 📊 Progress Matrix

See detailed progress: [[Skills Tracker/Progress Matrix]]

| Category | Topics Completed | Total Topics | Progress |
|----------|-----------------|--------------|----------|
| SQL | 6/8 | 8 | 75% |
| Data Modeling | 6/8 | 8 | 75% |
| Database Fundamentals | 3/8 | 8 | 38% |
| ETL/ELT | 4/8 | 8 | 50% |
| DSA | 2/8 | 8 | 25% |
| Python | 5/8 | 8 | 63% |
| System Design | 3/8 | 8 | 38% |
| Streaming | 0/7 | 7 | 0% |
| Orchestration | 2/8 | 8 | 25% |
| AWS | 1/8 | 8 | 13% |
| GCP | 0/7 | 7 | 0% |
| Azure | 0/7 | 7 | 0% |
| **Overall** | **37/98** | **98** | **38%** |

---

## 🗓️ 16-Week Study Plan

### Weeks 1-4: Fundamentals
- Week 1: Advanced SQL (Window functions, CTEs)
- Week 2: Data Modeling (Star schema, SCD)
- Week 3: Database concepts & Query optimization
- Week 4: Practice problems & review

### Weeks 5-8: Core Skills
- Week 5: ETL/ELT patterns & Pipeline design
- Week 6: Data Structures & Algorithms (Basics)
- Week 7: Python for Data Engineering
- Week 8: Mock interviews & practice

### Weeks 9-12: Advanced Topics
- Week 9: System Design fundamentals
- Week 10: System Design practice cases
- Week 11: Streaming concepts (Kafka)
- Week 12: Orchestration (Airflow)

### Weeks 13-16: Cloud & Practice
- Week 13: AWS services
- Week 14: GCP/Azure services
- Week 15: Mock interviews (all topics)
- Week 16: Final review & prep

---

## 📚 Recommended Resources

### Books
- "Data Warehousing in the Real World" - Ralph Kimball
- "Designing Data-Intensive Applications" - Martin Kleppmann
- "Streaming Systems" - Tyler Akidau

### Courses
- [Udemy - Data Engineering Interview Preparation](https://www.udemy.com/course/interview-preparation-data-engineering-sep-2025-edition/)
- [Coursera - Data Engineering with Google Cloud](https://www.coursera.org/professional-certificates/google-data-engineering)

### Practice Platforms
- LeetCode (SQL focus)
- Interview Query
- Exponent
- Pramp

---

## 🎯 Company-Specific Prep

### FAANG+
- Heavy focus on System Design
- Strong SQL & Optimization
- DSA (Medium level)
- Behavioral (Leadership Principles)

### Startups
- Practical Pipeline Design
- Tool-specific knowledge (dbt, Airflow)
- End-to-end problem solving
- Culture fit

### Enterprise
- Data Modeling & Governance
- ETL/ELT patterns
- Cloud platform expertise
- Communication skills

---

**Next Steps**:
1. Update progress in [[Skills Tracker/Progress Matrix]]
2. Practice SQL questions: [[05-Interview Prep/SQL Questions]]
3. Study System Design: [[05-Interview Prep/System Design]]

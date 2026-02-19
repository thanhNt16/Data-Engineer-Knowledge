# GCP Interview Questions

> Google Cloud Platform interview questions for data engineers

## Core GCP Data Services

### Q1: What are the key GCP services for data engineering?

**Answer**: Essential GCP data services include:

| Service | Purpose |
|---------|---------|
| **Cloud Storage** | Object storage for data lakes |
| **BigQuery** | Serverless data warehouse |
| **Dataflow** | Stream and batch processing |
| **Dataproc** | Managed Spark and Hadoop |
| **Pub/Sub** | Messaging and streaming |
| **Data Fusion** | Visual ETL |
| **Cloud SQL/Spanner** | Relational databases |
| **Bigtable** | NoSQL wide-column database |

---

### Q2: What is BigQuery and what makes it unique?

**Answer**: BigQuery is a serverless, highly scalable data warehouse:

**Key Features**:
- **Serverless**: No infrastructure to manage
- **Columnar storage**: Optimized for analytics
- **Standard SQL**: Familiar query language
- **Federated queries**: Query external data sources
- **BI Engine**: In-memory analysis engine
- **ML integration**: BigQuery ML for machine learning

**Pricing Model**:
- On-demand: Pay per TB of data scanned
- Flat-rate: Reserved slots for consistent workloads

---

### Q3: Explain BigQuery partitioning and clustering.

**Answer**:

| Feature | Description | Benefit |
|---------|-------------|---------|
| **Partitioning** | Divide table by column (date, integer, ingestion time) | Reduces query cost by scanning less data |
| **Clustering** | Sort data within partitions by columns | Further improves query performance |

**Best Practices**:
- Partition by date for time-series data
- Cluster on frequently filtered/joined columns
- Limit clustering columns to 4

---

## Dataflow

### Q4: What is Cloud Dataflow and how does it work?

**Answer**: Dataflow is a managed service for stream and batch processing:

**Features**:
- Apache Beam SDK support
- Auto-scaling workers
- Unified batch/stream processing
- Built-in templates for common patterns

**Use Cases**:
- ETL pipelines
- Real-time analytics
- Data ingestion and processing

---

### Q5: Compare Dataflow with Dataproc.

**Answer**:

| Aspect | Dataflow | Dataproc |
|--------|----------|----------|
| **Engine** | Apache Beam | Spark/Hadoop |
| **Model** | Fully managed | Managed clusters |
| **Scaling** | Automatic | Manual/Auto-scaling |
| **Best For** | Stream processing, ETL | Batch processing, Spark jobs |
| **Pricing** | Per job | Per cluster hour |

---

## Streaming with Pub/Sub

### Q6: What is Cloud Pub/Sub?

**Answer**: Pub/Sub is an asynchronous messaging service:

**Components**:
| Component | Purpose |
|-----------|---------|
| **Topics** | Message channels |
| **Subscriptions** | Message delivery configuration |
| **Publishers** | Message producers |
| **Subscribers** | Message consumers |

**Features**:
- At-least-once delivery
- Message ordering
- Dead-letter topics
- Exactly-once processing (preview)

---

## Cloud Storage

### Q7: Explain Cloud Storage classes.

**Answer**:

| Class | Use Case | Availability |
|-------|----------|--------------|
| **Standard** | Frequently accessed data | 99.99% |
| **Nearline** | Accessed < once/month | 99.9% |
| **Coldline** | Accessed < once/year | 99.9% |
| **Archive** | Long-term archival | 99.9% |

**Lifecycle Management**:
- Auto-transition between classes
- Delete old objects
- Reduce costs automatically

---

## Data Orchestration

### Q8: What orchestration options exist on GCP?

**Answer**:

| Service | Use Case |
|---------|----------|
| **Cloud Composer** | Managed Apache Airflow |
| **Workflows** | Serverless orchestration |
| **Cloud Scheduler** | Cron-based scheduling |
| **Cloud Functions** | Event-driven triggers |

---

## GCP vs AWS vs Azure

### Q9: Compare BigQuery with Redshift and Synapse.

**Answer**:

| Feature | BigQuery | Redshift | Synapse |
|---------|----------|----------|---------|
| **Model** | Serverless | Clusters | Both |
| **Scaling** | Automatic | Manual | Automatic |
| **Storage** | Separate from compute | Attached | Separated |
| **Pricing** | Query-based | Instance-based | Mixed |

---

## Best Practices

### Q10: What are GCP best practices for data engineering?

**Answer**:

- **Cost Optimization**: Use slots reservation, partition tables
- **Performance**: Denormalize for BigQuery, use clustered tables
- **Security**: IAM roles, VPC Service Controls
- **Data Quality**: Cloud DLP for PII detection
- **Monitoring**: Cloud Monitoring and Logging

---

## Related Topics

- [[05-Interview Prep/Cloud/AWS Interview Questions]]
- [[05-Interview Prep/Cloud/Azure Interview Questions]]
- [[02-Areas/Data Warehousing/Data Warehouse]]
- [[02-Areas/Orchestration/Orchestration Workflows & Scheduling]]

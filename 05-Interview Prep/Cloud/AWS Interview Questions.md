# AWS Interview Questions

> Amazon Web Services interview questions for data engineers

## Core AWS Services

### Q1: What are the key AWS services for data engineering?

**Answer**: Essential AWS data services include:

| Service | Purpose |
|---------|---------|
| **S3** | Object storage for data lakes |
| **Redshift** | Data warehousing |
| **Glue** | ETL and data catalog |
| **Athena** | Serverless SQL queries on S3 |
| **EMR** | Big data processing (Spark, Hadoop) |
| **Kinesis** | Real-time data streaming |
| **DynamoDB** | NoSQL database |
| **Lambda** | Serverless compute |

---

### Q2: Explain S3 storage classes and when to use each.

**Answer**:

| Storage Class | Use Case |
|---------------|----------|
| **Standard** | Frequently accessed data |
| **Intelligent-Tiering** | Unknown access patterns |
| **Standard-IA** | Infrequently accessed data |
| **One Zone-IA** | Infrequently accessed, non-critical data |
| **Glacier Instant** | Archive with millisecond retrieval |
| **Glacier Flexible** | Archive with hours retrieval |
| **Glacier Deep Archive** | Long-term archive (12+ hours retrieval) |

---

### Q3: How does AWS Glue work for ETL pipelines?

**Answer**: AWS Glue provides:

- **Crawlers**: Automatically discover and catalog data
- **Data Catalog**: Central metadata repository
- **ETL Jobs**: Serverless Spark-based transformations
- **Workflows**: Orchestrate multiple jobs

**Key Benefits**:
- Serverless (no infrastructure management)
- Automatic schema inference
- Integration with other AWS services
- Built-in job bookmarks for incremental processing

---

## Redshift

### Q4: What is Amazon Redshift and when would you use it?

**Answer**: Redshift is a fully managed data warehouse:

**Features**:
- Columnar storage for analytical queries
- Massively Parallel Processing (MPP)
- Petabyte-scale storage
- Redshift Spectrum for querying S3 data

**Use Cases**:
- Business intelligence and reporting
- Large-scale data analytics
- Data warehouse consolidation

---

### Q5: Explain Redshift distribution styles.

**Answer**:

| Style | Description | Use Case |
|-------|-------------|----------|
| **KEY** | Rows distributed by column value | Frequently joined tables |
| **ALL** | Copy to every node | Small, slowly changing dimensions |
| **AUTO** | Let Redshift decide | Uncertain access patterns |

---

## Streaming with Kinesis

### Q6: What is Amazon Kinesis and its components?

**Answer**: Kinesis is a platform for streaming data:

| Component | Purpose |
|-----------|---------|
| **Kinesis Data Streams** | Real-time data streaming |
| **Kinesis Data Firehose** | Load streaming data to destinations |
| **Kinesis Data Analytics** | SQL-based stream processing |
| **Kinesis Video Streams** | Video stream processing |

---

## Serverless Data Processing

### Q7: How would you build a serverless data pipeline on AWS?

**Answer**: A typical serverless pipeline:

```
S3 (raw data) → Lambda (trigger) → Glue (ETL) → Redshift (warehouse)
```

**Alternative with streaming**:
```
Kinesis → Lambda → S3/DynamoDB → Athena/Redshift
```

**Benefits**:
- No server management
- Auto-scaling
- Pay-per-use
- High availability

---

## AWS vs Other Clouds

### Q8: Compare AWS data services with Azure and GCP.

**Answer**:

| Service Type | AWS | Azure | GCP |
|--------------|-----|-------|-----|
| Object Storage | S3 | Blob Storage | Cloud Storage |
| Data Warehouse | Redshift | Synapse | BigQuery |
| ETL | Glue | Data Factory | Dataflow |
| Streaming | Kinesis | Event Hubs | Pub/Sub |
| NoSQL | DynamoDB | Cosmos DB | Firestore |

---

## Best Practices

### Q9: What are AWS best practices for data engineering?

**Answer**:

- **Cost Optimization**: Use S3 lifecycle policies, Reserved Instances
- **Security**: Enable encryption, use IAM roles, VPC endpoints
- **Performance**: Partition data, use columnar formats (Parquet)
- **Reliability**: Multi-AZ deployments, backup strategies
- **Monitoring**: CloudWatch metrics and alarms

---

## Related Topics

- [[05-Interview Prep/Cloud/Azure Interview Questions]]
- [[05-Interview Prep/Cloud/GCP Interview Questions]]
- [[02-Areas/Data Warehousing/Data Warehouse]]
- [[02-Areas/Streaming/Apache Kafka - Event Streaming]]

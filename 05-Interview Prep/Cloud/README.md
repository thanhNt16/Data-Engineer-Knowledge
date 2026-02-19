# Cloud Interview Questions

> Cloud platform interview questions for data engineers

## Overview

Cloud computing is essential for modern data engineering. This section covers the three major cloud providers with dedicated interview question sets.

## Provider-Specific Questions

| Provider | File | Focus Areas |
|----------|------|-------------|
| [[AWS Interview Questions]] | Amazon Web Services | S3, Redshift, Glue, Kinesis, Lambda |
| [[Azure Interview Questions]] | Microsoft Azure | Synapse, Data Factory, Databricks, Event Hubs |
| [[GCP Interview Questions]] | Google Cloud Platform | BigQuery, Dataflow, Pub/Sub, Cloud Storage |

## General Cloud Concepts

### Multi-Cloud Strategy

- **Vendor Lock-in**: Risk of being tied to one provider
- **Best-of-Breed**: Using best services from each provider
- **Complexity**: Increased operational overhead
- **Data Transfer**: Egress costs between providers

### Cloud Data Architecture Patterns

```
Raw Data Layer (Landing Zone)
    ↓
Processed Layer (Clean/Transformed)
    ↓
Curated Layer (Business-Ready)
    ↓
Analytics Layer (Serving/Reporting)
```

### Common Cloud Data Services

| Service Type | AWS | Azure | GCP |
|--------------|-----|-------|-----|
| Object Storage | S3 | Blob Storage | Cloud Storage |
| Data Warehouse | Redshift | Synapse | BigQuery |
| ETL/Orchestration | Glue | Data Factory | Dataflow |
| Streaming | Kinesis | Event Hubs | Pub/Sub |
| NoSQL | DynamoDB | Cosmos DB | Firestore |
| Spark | EMR | Databricks | Dataproc |
| Serverless | Lambda | Functions | Cloud Functions |

## Key Interview Topics

### 1. Cost Optimization
- Storage tiering strategies
- Compute right-sizing
- Reserved vs on-demand pricing
- Query optimization

### 2. Security & Compliance
- IAM and access control
- Encryption (at-rest and in-transit)
- VPC/VNet configuration
- Data governance

### 3. Performance
- Partitioning strategies
- Caching mechanisms
- Data format selection (Parquet, Avro)
- Query optimization

### 4. Reliability
- Multi-region deployments
- Backup and recovery
- Disaster recovery planning
- SLAs and SLOs

## Related Topics

- [[05-Interview Prep/System Design Framework]]
- [[02-Areas/Data Warehousing/Data Warehouse]]
- [[02-Areas/Streaming/Apache Kafka - Event Streaming]]

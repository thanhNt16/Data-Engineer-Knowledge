# Azure Interview Questions

> Microsoft Azure interview questions for data engineers

## Core Azure Data Services

### Q1: What are the key Azure services for data engineering?

**Answer**: Essential Azure data services include:

| Service | Purpose |
|---------|---------|
| **Blob Storage** | Object storage for data lakes |
| **Azure Synapse Analytics** | Unified analytics service |
| **Data Factory** | ETL and data integration |
| **Databricks** | Apache Spark-based analytics |
| **Event Hubs** | Real-time data streaming |
| **Cosmos DB** | Globally distributed NoSQL |
| **Azure Data Lake Storage** | Enterprise data lake |
| **Stream Analytics** | Real-time analytics |

---

### Q2: Explain Azure Synapse Analytics architecture.

**Answer**: Synapse provides:

- **Synapse SQL**: Dedicated and serverless SQL pools
- **Spark Pools**: Big data processing
- **Data Explorer**: Log and telemetry analytics
- **Integration Runtime**: Data movement and transformation
- **Pipelines**: Workflow orchestration

**Key Features**:
- Unified experience for data warehousing and big data
- Integrated security with Azure Active Directory
- Built-in connectors to Azure services

---

### Q3: How does Azure Data Factory work?

**Answer**: Data Factory is Azure's ETL service:

**Components**:
| Component | Purpose |
|-----------|---------|
| **Pipelines** | Logical grouping of activities |
| **Activities** | Data movement and transformation |
| **Datasets** | Data structures references |
| **Linked Services** | Connection information |
| **Integration Runtimes** | Compute infrastructure |

**Key Features**:
- Code-free UI or code-based (ARM templates, SDK)
- 90+ built-in connectors
- Mapping Data Flows for transformations
- Triggers for scheduling and events

---

## Azure Storage

### Q4: Compare Azure Blob Storage tiers.

**Answer**:

| Tier | Use Case | Storage Cost | Access Cost |
|------|----------|--------------|-------------|
| **Hot** | Frequently accessed | Higher | Lower |
| **Cool** | Infrequently accessed (30+ days) | Lower | Higher |
| **Cold** | Rarely accessed (90+ days) | Lowest | Highest |
| **Archive** | Long-term retention (180+ days) | Very low | Very high |

---

### Q5: What is ADLS Gen2 and how does it differ from Blob Storage?

**Answer**: Azure Data Lake Storage Gen2:

**Features**:
- Hierarchical namespace (directory structure)
- Optimized for analytics workloads
- Compatible with Hadoop/Spark
- ACL-based security

**Differences from Blob Storage**:
- Blob: Flat namespace, object storage focus
- ADLS Gen2: Hierarchical, analytics-optimized

---

## Streaming with Event Hubs

### Q6: What is Azure Event Hubs?

**Answer**: Event Hubs is a big data streaming platform:

**Features**:
- Millions of events per second
- Real-time and batch processing
- Kafka-compatible API
- Capture to Azure Storage/Data Lake

**Use Cases**:
- Application telemetry
- Log aggregation
- Real-time analytics
- IoT data ingestion

---

## Azure Databricks

### Q7: How does Azure Databricks integrate with Azure services?

**Answer**: Databricks provides:

- **Unity Catalog**: Unified governance
- **MLflow Integration**: ML lifecycle management
- **Auto Loader**: Incremental data loading
- **Delta Lake**: ACID transactions on data lake

**Integration Points**:
- Native connectivity to ADLS, Blob, Synapse
- Azure Active Directory authentication
- Azure Monitor for logging

---

## Security & Governance

### Q8: What are Azure security best practices for data engineering?

**Answer**:

- **Identity**: Azure AD integration, Managed Identities
- **Encryption**: At-rest and in-transit encryption
- **Network**: VNet integration, Private Endpoints
- **Data Protection**: Soft delete, versioning
- **Governance**: Azure Purview for data catalog

---

## Related Topics

- [[05-Interview Prep/Cloud/AWS Interview Questions]]
- [[05-Interview Prep/Cloud/GCP Interview Questions]]
- [[02-Areas/Data Warehousing/Data Warehouse]]

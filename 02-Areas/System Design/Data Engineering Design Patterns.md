---
tags: [system-design, patterns, lambda, kappa, cqrs, data-lakehouse, event-driven, data-pipeline]
date: 2026-02-15
status: learning
---

# Data Engineering Design Patterns

## Overview

Data engineering design patterns are reusable solutions to common problems in building data pipelines, platforms, and architectures. These patterns ensure scalability, reliability, and maintainability.

**Key Insight**: "Patterns are distilled wisdom from decades of data engineering experience."

---

## 1. Pipeline Design Patterns

### 1.1 ELT vs. ETL vs. ELT (Extract-Load-Transform)

| Pattern | Process Flow | Use Case | Latency | Cost | Complexity |
|---------|--------------|----------|---------|------|------------|
| **ETL** | Extract → Transform → Load | Data warehouse | High (overnight) | Medium |
| **ELT** | Extract → Load → Transform | Modern data platforms | Low (minutes) | Low |
| **ELT** | Extract → Load → Transform | Cloud-native (Snowflake, BigQuery) | Very Low | Low |
| **LT** | Load → Transform | Event streaming | Very Low | Low |

### ELT vs. ETL: Python Example

```python
import pandas as pd
from sqlalchemy import create_engine

# ETL Approach (Traditional)
def etl_pipeline():
    # 1. Extract
    raw_data = pd.read_csv("source.csv")
    
    # 2. Transform (heavy processing)
    cleaned = raw_data.dropna()
    transformed = cleaned.groupby('category')['amount'].sum()
    
    # 3. Load (final result)
    engine = create_engine("postgresql://user:pass@db")
    transformed.to_sql("etl_table", engine, if_exists='replace')
    print("ETL: Extract → Transform → Load")
```

```python
# ELT Approach (Modern)
def elt_pipeline():
    # 1. Extract
    raw_data = pd.read_csv("source.csv")
    
    # 2. Load (raw data)
    engine = create_engine("snowflake://user:pass@account/db")
    raw_data.to_sql("stg_raw_orders", engine, if_exists='append')
    print("ELT: Extract → Load (staging)")
    
    # 3. Transform (in warehouse using SQL)
    # Snowflake executes: CREATE TABLE AS SELECT FROM stg_raw_orders
    transformation_sql = """
    CREATE OR REPLACE TABLE etl_orders AS
    SELECT
        category,
        SUM(amount) AS total_amount,
        COUNT(*) AS order_count
    FROM stg_raw_orders
    GROUP BY category
    """
    with engine.connect() as conn:
        conn.execute(transformation_sql)
    print("ELT: Transform in warehouse (SQL)")
```

**Advantages**:
- **ETL**: Simpler, good for small data, transformations happen before load
- **ELT**: Better for large data, leverages warehouse power, better compression
- **ELT**: Cloud-native, lowest latency, best performance

---

## 2. Architectural Patterns

### 2.1 Lambda Architecture

**Definition**: Batch layer + speed layer (streaming) combined in serving layer.

**Components**:
- **Batch Layer**: High-throughput, accurate, high latency
- **Speed Layer**: Low-latency, real-time, eventually consistent
- **Serving Layer**: Merged views from both layers

**Python Example: Lambda Orchestration**

```python
from airflow import DAG
from airflow.operators.postgres import PostgresOperator
from airflow.providers.kafka.operators import KafkaToPostgresOperator
from datetime import datetime, timedelta

dag = DAG(
    'lambda_architecture',
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily',
    catchup=False
)

# Batch Layer: Nightly aggregate
batch_agg = PostgresOperator(
    task_id='batch_aggregation',
    sql="""
        CREATE TABLE IF NOT EXISTS batch_orders AS
        SELECT
            DATE(order_time) AS order_date,
            product_id,
            SUM(amount) AS daily_revenue,
            COUNT(*) AS order_count
        FROM raw_orders
        WHERE DATE(order_time) = '{{ ds }}'
        GROUP BY DATE(order_time), product_id
    """,
    postgres_conn_id='warehouse'
)

# Speed Layer: Real-time streaming
stream_ingest = KafkaToPostgresOperator(
    task_id='stream_ingestion',
    kafka_config={'topic': 'orders', 'bootstrap.servers': 'kafka:9092'},
    postgres_conn_id='warehouse',
    table='stream_orders'
)

# Serving Layer: Merge batch + streaming
merge_views = PostgresOperator(
    task_id='merge_views',
    sql="""
        CREATE OR REPLACE VIEW orders_view AS
        SELECT * FROM batch_orders
        UNION ALL
        SELECT * FROM stream_orders
        WHERE created_at >= NOW() - INTERVAL '1 day'
        UNION ALL
        SELECT * FROM stream_orders
        WHERE DATE(created_at) = '{{ ds }}'
    """,
    postgres_conn_id='warehouse'
)

# Dependencies: Batch completes before merge
stream_ingest >> [batch_agg, merge_views]
```

**When to Use**:
- Need both real-time and accurate historical views
- Can tolerate slight inconsistency in speed layer
- Want to reduce load on streaming layer (batch pre-aggregates)

---

### 2.2 Kappa Architecture

**Definition**: Streaming-only architecture (no batch layer), processing engine directly on streams.

**Components**:
- **Source**: Event streams (Kafka, Kinesis)
- **Processing**: Stream processor (Flink, Spark Streaming, Samza)
- **Storage**: Output sink (database, lakehouse)
- **Serving**: Query output directly

**Python Example: Kappa with Flink**

```python
from pyflink.datastream import StreamExecutionEnvironment
from pyflink.table import StreamTableEnvironment

# Stream processing (Kappa)
env = StreamExecutionEnvironment.get_execution_environment()

t_env = StreamTableEnvironment.create(env)

# Define source stream
t_env.execute_sql("""
    CREATE TABLE orders_stream (
        order_id BIGINT,
        user_id BIGINT,
        amount DECIMAL(10, 2),
        event_time TIMESTAMP(3)
    ) WITH (
        'connector' = 'kafka',
        'topic' = 'orders',
        'properties.bootstrap.servers' = 'kafka:9092',
        'scan.startup.mode' = 'latest-offset'
    )
""")

# Real-time aggregation (no batch layer)
t_env.execute_sql("""
    CREATE TABLE user_stats AS
    SELECT
        user_id,
        TUMBLE_START(event_time, INTERVAL '1' HOUR) AS window_start,
        COUNT(*) AS order_count,
        SUM(amount) AS total_amount
    FROM orders_stream
    GROUP BY user_id, TUMBLE(event_time, INTERVAL '1' HOUR)
""")

# Materialize to database for querying
t_env.execute_sql("""
    INSERT INTO user_stats_query
    SELECT user_id, window_start, order_count, total_amount
    FROM user_stats
""")

env.execute("Kappa Architecture: Real-time processing only")
```

**When to Use**:
- Pure streaming use case (no need for batch)
- Real-time analytics, fraud detection, dashboards
- Simplified architecture (no batch/speed layer sync)

**Comparison**:

| Aspect | Lambda | Kappa |
|--------|---------|--------|
| **Latency** | Speed: ms, Batch: hours | Always ms |
| **Accuracy** | Speed: approximate, Batch: accurate | Accurate |
| **Complexity** | High (sync layers) | Low |
| **Cost** | Higher (dual processing) | Lower |
| **Use Case** | Mixed real-time/batch | Pure real-time |

---

### 2.3 Lakehouse Architecture

**Definition**: Single platform combining data lake flexibility with warehouse performance and ACID transactions.

**Components**:
- **Storage**: Cloud object storage (S3, GCS, ADLS) with table format
- **Metadata**: Catalog (Unity Catalog, Glue, Hive Metastore)
- **Format**: Iceberg, Delta Lake, Hudi (ACID, time travel)
- **Query Engine**: Spark, Trino, Databricks SQL, BigQuery
- **Compute**: Serverless, provisioned, auto-scaling

**Python Example: Iceberg Lakehouse**

```python
from pyspark.sql import SparkSession
from pyiceberg.catalog import load_catalog

# Iceberg catalog
catalog = load_catalog("default")

spark = SparkSession.builder \
    .appName("lakehouse_iceberg") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.iceberg.spark.SparkCatalog") \
    .config("spark.sql.catalog.default_catalog", "default") \
    .config("spark.sql.catalog.default.warehouse", "lakehouse") \
    .getOrCreate()

# Load raw data into Iceberg table
df = spark.read.json("s3://raw-data/orders/")
df.createOrReplaceTempView("raw_orders")

# Transform and write to Iceberg (Silver layer)
df_silver = df.selectExpr(
    "order_id",
    "user_id",
    "amount",
    "CAST(event_time AS TIMESTAMP) AS order_time"
).filter("amount > 0")

df_silver.writeToIceberg(
    "lakehouse.silver.orders",
    mode="overwrite"
)

# Aggregate to Gold layer
df_gold = df_silver.groupBy("user_id").agg({
    "total_spent": "SUM(amount)",
    "order_count": "COUNT(*)"
})

df_gold.writeToIceberg(
    "lakehouse.gold.user_stats",
    mode="append"  -- Time travel enabled
)

# Time travel query: Query user stats as of yesterday
yesterday_data = spark.sql("""
    SELECT * FROM lakehouse.gold.user_stats
    FOR SYSTEM_TIME AS OF TIMESTAMP('2024-01-13 23:59:59.999')
""")

yesterday_data.show()
```

**SQL Example: Iceberg Time Travel**

```sql
-- Query current state
SELECT user_id, total_spent, order_count
FROM lakehouse.gold.user_stats;

-- Query historical state (yesterday)
SELECT user_id, total_spent, order_count
FROM lakehouse.gold.user_stats
FOR SYSTEM_TIME AS OF TIMESTAMP('2024-01-13 23:59:59');

-- Query all changes in last 7 days
SELECT user_id, total_spent
FROM lakehouse.gold.user_stats
FOR SYSTEM_TIME BETWEEN FROM_TIMESTAMP('2024-01-06 00:00:00')
AND TO_TIMESTAMP('2024-01-13 00:00:00');
```

**Lakehouse Layers**:

| Layer | Purpose | Quality | Processing |
|--------|---------|----------|------------|
| **Bronze** | Raw data from sources | None |
| **Silver** | Cleaned, validated | Basic transformations |
| **Gold** | Aggregated, business-ready | Complex aggregations |

**When to Use**:
- Need both flexibility (lake) and performance (warehouse)
- Time travel queries (what was true at time X)
- Schema evolution (ACID transactions on diverse data)
- BI and ML workloads on same platform

---

## 3. Event-Driven Patterns

### 3.1 Event Sourcing

**Definition**: Capture all changes as immutable events, rebuild state by replaying events.

**Python Example: Event Sourcing**

```python
from datetime import datetime
import json
from typing import List, Dict

class EventStore:
    def __init__(self):
        self.events: List[Dict] = []

    def append_event(self, event_type: str, aggregate_id: str, data: dict):
        """Append event to log"""
        event = {
            "event_type": event_type,
            "aggregate_id": aggregate_id,
            "timestamp": datetime.now().isoformat(),
            "data": data
        }
        self.events.append(event)

    def get_state(self, aggregate_id: str) -> dict:
        """Rebuild state by replaying events"""
        state = {}
        for event in self.events:
            if event['aggregate_id'] == aggregate_id:
                if event['event_type'] == 'OrderCreated':
                    state['order'] = event['data']
                    state['status'] = 'created'
                    state['order_date'] = event['timestamp']
                elif event['event_type'] == 'OrderPaid':
                    state['status'] = 'paid'
                    state['paid_date'] = event['timestamp']
                    state['amount'] = event['data']['amount']
                elif event['event_type'] == 'OrderCancelled':
                    state['status'] = 'cancelled'
                    state['cancelled_date'] = event['timestamp']
        return state

# Usage
event_store = EventStore()

# Append events (immutable log)
event_store.append_event('OrderCreated', 'order_123', {
    'user_id': 101,
    'product_id': 456,
    'amount': 99.99
})

event_store.append_event('OrderPaid', 'order_123', {'amount': 99.99})
event_store.append_event('OrderCancelled', 'order_123', {})

# Rebuild state
current_state = event_store.get_state('order_123')
print(f"Current State: {current_state}")
```

**When to Use**:
- Audit trails required (who changed what and when)
- Event replay to rebuild state (debugging, testing)
- Microservices with eventual consistency
- CQRS (Command Query Responsibility Segregation)

---

### 3.2 CQRS (Command Query Responsibility Segregation)

**Definition**: Separate read and write models for scaling and flexibility.

**Components**:
- **Command Model**: Handle writes (event sourcing, task queue)
- **Query Model**: Optimized for reads (materialized views, caches)
- **Synchronization**: Update model to reflect writes

**Python Example: CQRS with Kafka**

```python
from kafka import KafkaProducer, KafkaConsumer
import json

# Command Model (Write - Event Sourcing)
command_producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

def handle_order_command(order_command):
    """Handle write via event sourcing"""
    event = {
        "event_type": order_command['type'],  # OrderCreated, OrderPaid, etc.
        "aggregate_id": order_command['order_id'],
        "data": order_command
    }

    # Write to command event stream
    command_producer.send(
        topic='order_commands',
        key=str(order_command['order_id']).encode('utf-8'),
        value=event
    )

# Query Model (Read - Materialized Views)
read_consumer = KafkaConsumer(
    'order_queries',
    bootstrap_servers=['localhost:9092'],
    value_deserializer=lambda m: json.loads(m.decode('utf-8'))
)

def update_read_model():
    """Process query model (materialized view)"""
    for message in read_consumer:
        query = message.value
        event_type = query['event_type']
        order_id = query['aggregate_id']

        # Update materialized view (read model)
        if event_type == 'OrderCreated':
            # INSERT INTO materialized_orders
            print(f"Created order {order_id} in read model")
        elif event_type == 'OrderPaid':
            # UPDATE materialized_orders SET status = 'paid'
            print(f"Updated order {order_id} to paid in read model")
        elif event_type == 'OrderCancelled':
            # UPDATE materialized_orders SET status = 'cancelled'
            print(f"Cancelled order {order_id} in read model")

# Separate optimized queries for reads
def get_user_orders(user_id: int):
    """Query read model (optimized for reads)"""
    # SELECT FROM materialized_orders (cached, indexed)
    return f"SELECT * FROM materialized_orders WHERE user_id = {user_id}"

# Usage
handle_order_command({"type": "OrderCreated", "order_id": 123})
handle_order_command({"type": "OrderPaid", "order_id": 123, "amount": 99.99})
update_read_model()
```

**When to Use**:
- Read and write patterns differ significantly
- Need read model optimized for fast queries
- Write model needs strong consistency (event sourcing)
- Scalable microservices

---

## 4. Performance Patterns

### 4.1 Partitioning Strategies

### 4.1.1 Key-Based Partitioning

**Definition**: Hash partition key to distribute data evenly.

**SQL Example**: Partitioning by user_id

```sql
-- Create partitioned table
CREATE TABLE orders_partitioned (
    order_id BIGINT,
    user_id BIGINT NOT NULL,
    amount DECIMAL(10, 2),
    order_date TIMESTAMP
) PARTITION BY HASH(user_id);

-- Query specific partition (fast)
SELECT * FROM orders_partitioned
WHERE user_id = 101;  -- Queries single partition

-- Insert distributes across partitions
INSERT INTO orders_partitioned (order_id, user_id, amount, order_date)
VALUES (123, 101, 99.99, NOW()),
       (124, 102, 149.99, NOW()),
       (125, 103, 199.99, NOW());
```

**Pros**:
- Even distribution across partitions
- Single user queries hit single partition
- Easy to scale (add partitions)

**Cons**:
- No range queries (must scan all partitions)
- Partition skew if some keys are hot

---

### 4.1.2 Range Partitioning

**Definition**: Partition by range of key (e.g., date ranges).

**SQL Example**: Partitioning by date

```sql
-- Create partitioned table by date
CREATE TABLE orders_range_partitioned (
    order_id BIGINT,
    user_id BIGINT,
    amount DECIMAL(10, 2),
    order_date TIMESTAMP NOT NULL
) PARTITION BY RANGE (order_date);

-- Create partitions
CREATE TABLE orders_2024_q1 PARTITION OF orders_range_partitioned
    FOR VALUES FROM ('2024-01-01') TO ('2024-03-31');

CREATE TABLE orders_2024_q2 PARTITION OF orders_range_partitioned
    FOR VALUES FROM ('2024-04-01') TO ('2024-06-30');

-- Query specific date range (very fast)
SELECT * FROM orders_range_partitioned
WHERE order_date BETWEEN '2024-01-15' AND '2024-01-31';

-- Insert routes to correct partition based on order_date
INSERT INTO orders_range_partitioned (...)
VALUES (123, 101, 99.99, '2024-01-20');  -- Goes to 2024_q1
INSERT INTO orders_range_partitioned (...)
VALUES (456, 202, 149.99, '2024-04-15');  -- Goes to 2024_q2
```

**Pros**:
- Range queries are very fast (prune partitions)
- Time-based queries optimal
- Easy to archive old data (drop old partitions)

**Cons**:
- Partition skew if some dates are hot
- Manual partition management (add new partitions)

---

### 4.2 Sharding (Horizontal Scaling)

**Definition**: Distribute data across multiple databases/clusters.

### 4.2.1 Hash Sharding

**Python Example**: Consistent Hash Sharding

```python
import hashlib
from typing import Dict, Any
import psycopg2

class ShardManager:
    def __init__(self, shard_configs: list):
        """
        shard_configs = [
            {'shard_id': 0, 'host': 'db-0.example.com', 'port': 5432},
            {'shard_id': 1, 'host': 'db-1.example.com', 'port': 5432},
            {'shard_id': 2, 'host': 'db-2.example.com', 'port': 5432},
            {'shard_id': 3, 'host': 'db-3.example.com', 'port': 5432},
        ]
        """

    def get_shard(self, key: str) -> Dict[str, Any]:
        """Consistent hash sharding"""
        hash_value = int(hashlib.md5(key.encode()).hexdigest(), 16)
        shard_index = hash_value % len(self.shard_configs)
        return self.shard_configs[shard_index]

    def execute_query(self, query: str, params: dict):
        """Route query to correct shard and execute"""
        shard = self.get_shard(str(params.get('user_id', '')))
        conn = psycopg2.connect(
            host=shard['host'],
            port=shard['port'],
            database='mydb'
        )
        cursor = conn.cursor()
        cursor.execute(query, params)
        result = cursor.fetchall()
        conn.close()
        return result

# Usage
shard_manager = ShardManager([
    {'shard_id': 0, 'host': 'db-0.example.com', 'port': 5432},
    {'shard_id': 1, 'host': 'db-1.example.com', 'port': 5432},
    {'shard_id': 2, 'host': 'db-2.example.com', 'port': 5432},
    {'shard_id': 3, 'host': 'db-3.example.com', 'port': 5432}
])

# Query routes to db-1 (user_id = 101 % 4 = 1)
result = shard_manager.execute_query(
    "SELECT * FROM users WHERE user_id = %(user_id)s",
    {'user_id': 101}
)

print(f"Query routed to shard {result[0]['shard_id']}: {len(result)} rows")
```

---

### 4.3 Caching Strategies

### 4.3.1 Multi-Level Caching

**Definition**: L1 (application) → L2 (distributed) → L3 (database)

**Python Example: Three-Tier Cache**

```python
import redis
import time
from typing import Optional
from functools import lru_cache

# L1: Application Cache (In-Memory)
@lru_cache(maxsize=1000)
def l1_get_user_profile(user_id: int) -> Optional[dict]:
    """Fast in-memory cache for hot data"""
    # Simulate database query
    time.sleep(0.001)  # Simulate 1ms DB latency
    return {'user_id': user_id, 'name': f'User {user_id}'}

# L2: Distributed Cache (Redis)
class RedisCache:
    def __init__(self, redis_client):
        self.redis = redis_client
        self.ttl = 3600  # 1 hour

    def get(self, key: str) -> Optional[dict]:
        """Get from Redis"""
        cached = self.redis.get(key)
        if cached:
            return eval(cached)  # Deserialize
        return None

    def set(self, key: str, value: dict) -> None:
        """Set in Redis with TTL"""
        self.redis.setex(key, str(value), self.ttl)

    def delete(self, key: str) -> None:
        """Invalidate cache"""
        self.redis.delete(key)

# L3: Database (Always check if L1/L2 miss)
def get_user_profile_l3(user_id: int) -> dict:
    """Fallback to database"""
    # Simulate database query
    time.sleep(0.010)  # Simulate 10ms DB latency
    return {
        'user_id': user_id,
        'name': f'User {user_id} (from DB)',
        'preferences': {'theme': 'dark', 'notifications': True}
    }

# Multi-Level Cache Logic
class MultiLevelCache:
    def __init__(self, redis_client):
        self.l2 = RedisCache(redis_client)

    def get_user_profile(self, user_id: int) -> dict:
        """L1 → L2 → L3 cache lookup"""
        # Try L1 (Memory)
        profile = l1_get_user_profile(user_id)
        if profile:
            return profile

        # Try L2 (Redis)
        cache_key = f"user_profile:{user_id}"
        profile = self.l2.get(cache_key)
        if profile:
            # Backfill L1
            l1_get_user_profile.cache_clear()
            return profile

        # Fallback to L3 (Database)
        profile = get_user_profile_l3(user_id)

        # Cache in L2 and L1
        self.l2.set(cache_key, profile)
        l1_get_user_profile.cache_set(user_id, profile)

        return profile

    def invalidate_user(self, user_id: int) -> None:
        """Invalidate all cache levels"""
        # Clear L1
        l1_get_user_profile.cache_clear()
        # Invalidate L2
        cache_key = f"user_profile:{user_id}"
        self.l2.delete(cache_key)
        print(f"Invalidated user {user_id} from all cache levels")

# Usage
import redis
redis_client = redis.StrictRedis(host='localhost', port=6379)

cache = MultiLevelCache(redis_client)

# First request: Cache miss (L1, L2, DB)
profile1 = cache.get_user_profile(101)
print(f"Request 1: {profile1['source']}")

# Second request: Cache hit (L1)
profile2 = cache.get_user_profile(101)
print(f"Request 2: {profile2['source']}")

# Invalidate and re-fetch
cache.invalidate_user(101)
profile3 = cache.get_user_profile(101)
print(f"Request 3 (after invalidate): {profile3['source']}")
```

---

### 4.4 Write-Through Optimization

### 4.4.1 Batch Writes

**Definition**: Group multiple writes into single batch operation.

**Python Example**: Batch Database Insert

```python
import psycopg2
from typing import List, Dict

def batch_insert_orders(orders: List[Dict]) -> None:
    """Batch insert for higher throughput"""
    conn = psycopg2.connect("dbname=mydb user=user password=pass")
    cursor = conn.cursor()

    # Execute batch insert (single round-trip)
    batch_query = """
        INSERT INTO orders (order_id, user_id, amount, order_date)
        VALUES (%s, %s, %s, %s)
    """

    # Prepare batch data
    values = [
        (order['order_id'], order['user_id'], order['amount'], order['order_date'])
        for order in orders
    ]

    cursor.executemany(batch_query, values)
    conn.commit()
    conn.close()
    print(f"Inserted {len(orders)} orders in batch")

# Usage
orders_to_insert = [
    {'order_id': 1001, 'user_id': 101, 'amount': 99.99, 'order_date': '2024-01-15'},
    {'order_id': 1002, 'user_id': 102, 'amount': 149.99, 'order_date': '2024-01-15'},
    {'order_id': 1003, 'user_id': 103, 'amount': 199.99, 'order_date': '2024-01-15'},
    # ... 1000 orders
]

batch_insert_orders(orders_to_insert)  # Single network round-trip
```

---

## 5. Data Integration Patterns

### 5.1 CDC (Change Data Capture)

**Definition**: Capture row-level changes from databases in real-time.

**Python Example**: Debezium CDC Handler

```python
from kafka import KafkaProducer
import json
from datetime import datetime

# Producer for CDC events
cdc_producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

def handle_cdc_event(event):
    """Process CDC event and route to lakehouse"""
    cdc_event = {
        "operation": event['op'],  # c, u, d, r (create, update, delete, read)
        "table": event['source']['table'],
        "before": event.get('before'),
        "after": event.get('after'),
        "timestamp": datetime.now().isoformat(),
        "source": event['source']
    }

    # Route to lakehouse (Iceberg/S3)
    topic_name = f"cdc_{cdc_event['table']}"
    cdc_producer.send(
        topic=topic_name,
        key=str(event.get('after', {}).get('id', '')),  # Key by ID
        value=cdc_event
    )

    print(f"Processed {cdc_event['operation']} on {cdc_event['table']}")
```

**SQL Example**: Upsert CDC Changes

```sql
-- Upsert CDC changes into lakehouse
MERGE INTO target_orders AS target
USING staging_cdc AS source
ON target.order_id = source.after_id
WHEN MATCHED THEN
    UPDATE SET
        user_id = source.after.user_id,
        amount = source.after.amount,
        updated_at = CURRENT_TIMESTAMP
WHEN NOT MATCHED THEN
    INSERT (order_id, user_id, amount, order_date, created_at, updated_at)
    VALUES (
        source.after.order_id,
        source.after.user_id,
        source.after.amount,
        source.after.order_date,
        source.after.ts_ms,
        CURRENT_TIMESTAMP
    );
```

---

## 6. Governance Patterns

### 6.1 Data Contracts

**Definition**: Formal agreement between data producers and consumers.

**Python Example**: Pydantic Data Contract

```python
from pydantic import BaseModel, validator, Field
from datetime import datetime
from enum import Enum

class OrderStatus(str, Enum):
    PENDING = "pending"
    COMPLETED = "completed"
    CANCELLED = "cancelled"

class OrderDataContract(BaseModel):
    order_id: int = Field(..., gt=0)
    user_id: int = Field(..., ge=1)
    amount: float = Field(..., gt=0)
    status: OrderStatus
    order_date: datetime

    @validator('status')
    def validate_order_date(cls, v, values):
        """Order date must be in past"""
        if v.order_date > datetime.now():
            raise ValueError("Order date cannot be in future")
        return v

# Validate incoming data against contract
def validate_incoming_order(data: dict) -> OrderDataContract:
    try:
        contract = OrderDataContract(**data)
        return {"valid": True, "data": contract}
    except ValueError as e:
        return {"valid": False, "error": str(e)}

# Usage
incoming_order = {
    "order_id": 123,
    "user_id": 101,
    "amount": 99.99,
    "status": "completed",
    "order_date": "2024-01-15 12:00:00"
}

validation_result = validate_incoming_order(incoming_order)
if validation_result['valid']:
    # Accept data
    print("Contract validated, accepting data")
else:
    # Reject data
    print(f"Contract validation failed: {validation_result['error']}")
```

---

## 7. Interview Questions

**Q1: What's the difference between Lambda and Kappa architecture?**

**A1**:
- **Lambda**: Dual-layer architecture (batch + speed layers). Batch layer provides accurate historical views, speed layer provides real-time updates. Merged views combine both.
- **Kappa**: Streaming-only architecture. No batch layer. Stream processor (Flink, Spark Streaming) processes events directly to output sink. Lower latency but no pre-computed aggregations.

**Q2: When would you use ELT instead of ETL?**

**A2**:
- **Data Volume**: Large datasets (TB scale) - ELT leverages warehouse compute power
- **Latency**: Need low-latency - ELT loads raw data quickly, transforms in warehouse
- **Cloud-Native**: Snowflake, BigQuery, Databricks - ELT is native and efficient
- **Cost**: Cloud warehouse compute is cheaper than custom ETL servers

**Q3: What's event sourcing and when to use it?**

**A3**: Event sourcing captures all changes as immutable events and rebuilds state by replaying them. Use when:
- Complete audit trail required (who changed what and when)
- Event replay needed (debugging, time travel, testing)
- Microservices with eventual consistency
- Complex business logic that depends on history
- CQRS architecture (separate read/write models)

**Q4: How does CQRS improve performance?**

**A4**: CQRS (Command Query Responsibility Segregation) separates read and write models:
- **Command Model**: Optimized for writes (event sourcing, task queues)
- **Query Model**: Optimized for reads (materialized views, caching, denormalized schema)
- **Benefit**: Read and write patterns can scale independently (read-heavy workloads need more read replicas)

**Q5: What's the benefit of multi-level caching (L1/L2/L3)?**

**A5**:
- **L1 (Application/Memory)**: Fastest access for hot data (sub-microsecond)
- **L2 (Distributed Cache - Redis)**: Shared cache for all instances, durable, larger than L1
- **L3 (Database)**: Source of truth, always available
- **Benefit**: 3-tier cache minimizes database load while maintaining consistency and availability

---

## Related Notes

- [[Lambda Architecture]] - Detailed lambda example
- [[Data Lakehouse]] - Lakehouse patterns and implementation
- [[Apache Flink - Real-Time Analytics]] - Stream processing for Kappa
- [[Apache Kafka - Event Streaming]] - Event sourcing implementation
- [[Database Fundamentals]] - CAP theorem, distributed transactions
- [[System Design]] - Scalability, caching, load balancing
- [[Orchestration]] - Pipeline orchestration patterns

---

## Resources

- [Data Pipeline Design Patterns - GeeksforGeeks](https://www.geeksforgeeks.org/system-design/data-pipeline-design-patterns/)
- [10 Pipeline Design Patterns for Data Engineers - Pipeline Insights](https://pipeline2insights.substack.com/p/10-pipeline-design-patterns-for-data-engineers)
- [Data Pipeline Architecture: 9 Patterns & Best Practices - Altation](https://www.alation.com/blog/data-pipeline-architecture-9-patterns-best-practices)
- [A data engineer's guide to pipeline frameworks - AI Accelerator Institute](https://www.aiacceleratorinstitute.com/7-must-know-frameworks-for-data-engineers-in-2026)

---

**Progress**: 🟡 Learning (patterns understood, need hands-on implementation)

**Next Steps**:
- [ ] Implement Lambda architecture with Flink
- [ ] Build event sourcing system with Kafka
- [ ] Design multi-tier caching strategy
- [ ] Implement CDC pipeline with Debezium
- [ ] Create data contract validation framework

---
tags: [streaming, kafka, messaging, producer, consumer, apache]
date: 2026-02-14
status: learning
---

# Apache Kafka - Distributed Event Streaming

## Overview

Apache Kafka is a distributed event streaming platform for **high-throughput, fault-tolerant, real-time** data pipelines. It's designed as a **commit log** that stores records in categories called topics.

**Key Characteristics**:
- **Distributed**: Runs across multiple servers (brokers)
- **Scalable**: Horizontal scaling by adding brokers
- **Fault-tolerant**: Data replication across brokers
- **High-throughput**: Millions of messages per second
- **Low-latency**: Milliseconds end-to-end

**Use Cases**:
- Real-time data pipelines
- Stream processing (Flink, Spark Streaming)
- Event-driven microservices
- Log aggregation
- Activity tracking

---

## Core Concepts

### 1. Topics

A **topic** is a category or feed name to which records are published.

```
Topic: user-clicks
├── Partition 0
├── Partition 1
├── Partition 2
└── Partition 3
```

**Topic Properties**:
- **Partitions**: Divide data for parallel processing
- **Replication Factor**: Copy factor for fault tolerance
- **Retention**: How long data is kept (time or size)

**Example**:
```bash
# Create topic with 3 partitions, replication factor 3
kafka-topics.sh --create \
  --bootstrap-server localhost:9092 \
  --topic user-clicks \
  --partitions 3 \
  --replication-factor 3
```

---

### 2. Partitions

A **partition** is an ordered, immutable sequence of records.

**Partitioning Keys**:
- **No key**: Round-robin distribution
- **With key**: Hash(key) % partitions (same key → same partition)

```
Key: user_123 → Partition 0
Key: user_456 → Partition 1
Key: user_789 → Partition 2
Key: user_123 → Partition 0 (same key = same partition)
```

**Offset**: Unique identifier per record within a partition
- Starts at 0, increments per message
- Consumers track their offset per partition
- Allows replay or resume

---

### 3. Producers

A **producer** publishes records to topics.

**Producer API Example (Python)**:
```python
from kafka import KafkaProducer
import json

# Configure producer
producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

# Send message with key
message = {
    'user_id': '123',
    'event': 'click',
    'timestamp': '2026-02-14T15:30:00Z'
}

# Send with key (ensures same partition for same user)
producer.send(
    topic='user-clicks',
    key=b'user_123',
    value=message
)

# Flush to ensure delivery
producer.flush()
```

**Producer Guarantees**:
- **acks=0**: Fire and forget (no confirmation)
- **acks=1**: Leader acks (default)
- **acks=all**: All replicas ack (strongest durability)

**Example**:
```python
# Strongest durability guarantee
producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    acks='all',                    # Wait for all replicas
    retries=3,                     # Retry on failure
    enable_idempotence=True,       # Exactly-once delivery
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)
```

---

### 4. Consumers

A **consumer** reads records from topics.

**Consumer API Example (Python)**:
```python
from kafka import KafkaConsumer
import json

# Configure consumer
consumer = KafkaConsumer(
    'user-clicks',
    bootstrap_servers=['localhost:9092'],
    auto_offset_reset='earliest',  # Read from beginning if no offset
    enable_auto_commit=False,      # Manual commit control
    group_id='analytics-service',
    value_deserializer=lambda m: json.loads(m.decode('utf-8'))
)

# Consume messages
for message in consumer:
    data = message.value
    partition = message.partition
    offset = message.offset
    
    print(f"Partition: {partition}, Offset: {offset}, Data: {data}")
    
    # Process data here
    process_click_event(data)
    
    # Manual commit after successful processing
    consumer.commit()
```

**Consumer Groups**:
- Multiple consumers share a topic
- Each partition consumed by one consumer in group
- **Max consumers = max partitions per topic**

```
Consumer Group: analytics-service
├── Consumer 1 → Partition 0, Partition 3
├── Consumer 2 → Partition 1
└── Consumer 3 → Partition 2

Total consumers: 3 (≤ 4 partitions)
```

---

### 5. Consumer Groups & Rebalancing

A **consumer group** is a set of consumers that coordinate to consume a topic.

**Rebalancing**: Occurs when consumers join/leave or partitions change

**Example Scenario**:
```
Initial State:
- 4 partitions, 2 consumers
- Consumer 1: Partitions 0, 1
- Consumer 2: Partitions 2, 3

Consumer 3 joins:
- Rebalance triggered
- New assignment:
  - Consumer 1: Partition 0
  - Consumer 2: Partition 1
  - Consumer 3: Partitions 2, 3
```

**Rebalance Strategies**:
- **Eager**: All consumers stop, reassign, resume
- **Cooperative**: Incremental rebalancing (newer Kafka versions)

---

## Architecture

### Kafka Cluster Components

```
┌─────────────────────────────────────────┐
│         Kafka Cluster                   │
│  ┌──────────┐  ┌──────────┐  ┌────────┐ │
│  │ Broker 1 │  │ Broker 2 │  │Broker 3│ │
│  │Leader: 0 │  │Leader: 1 │  │Leader:2│ │
│  │Replica:2 │  │Replica:0 │  │Replica:1│ │
│  └──────────┘  └──────────┘  └────────┘ │
└─────────────────────────────────────────┘
         ▲              ▲              ▲
         │              │              │
    ┌────┴────┐    ┌────┴────┐    ┌────┴────┐
    │Producer │    │Consumer │    │Consumer │
    └─────────┘    └─────────┘    └─────────┘
```

### Key Components

| Component | Purpose |
|-----------|---------|
| **Broker** | Kafka server, stores data |
| **Topic** | Logical channel for messages |
| **Partition** | Ordered sequence within topic |
| **Replica** | Copy of partition (leader + followers) |
| **Producer** | Writes messages to topics |
| **Consumer** | Reads messages from topics |
| **Consumer Group** | Set of consumers sharing work |
| **Controller** | Broker managing partition elections |

---

## End-to-End Example: Real-Time Analytics

### Scenario

Track user clicks, compute real-time metrics, store for analytics.

### 1. Producer: Click Events

```python
# producer.py
from kafka import KafkaProducer
import json
import time
import random

producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    key_serializer=lambda k: k.encode('utf-8'),
    value_serializer=lambda v: json.dumps(v).encode('utf-8'),
    acks='all'
)

# Simulate click events
events = ['click', 'view', 'add_to_cart', 'purchase']

for i in range(100):
    event = {
        'event_id': f"evt_{i}",
        'user_id': f"user_{random.randint(1, 10)}",
        'event_type': random.choice(events),
        'page': random.choice(['home', 'product', 'checkout']),
        'timestamp': time.time()
    }
    
    producer.send(
        topic='user-events',
        key=event['user_id'],  # Same user → same partition
        value=event
    )
    
    time.sleep(0.1)

producer.flush()
print("Sent 100 events")
```

### 2. Consumer: Real-Time Processing

```python
# consumer.py
from kafka import KafkaConsumer
import json
from collections import defaultdict

consumer = KafkaConsumer(
    'user-events',
    bootstrap_servers=['localhost:9092'],
    group_id='real-time-analytics',
    auto_offset_reset='latest',  # Start from new messages
    value_deserializer=lambda m: json.loads(m.decode('utf-8'))
)

# Metrics tracking
metrics = defaultdict(lambda: {
    'clicks': 0,
    'views': 0,
    'purchases': 0
})

for message in consumer:
    event = message.value
    user_id = event['user_id']
    event_type = event['event_type']
    
    # Update metrics
    metrics[user_id][event_type + 's'] += 1
    
    # Print real-time stats every 10 events
    if sum(v for u in metrics.values() for v in u.values()) % 10 == 0:
        print(f"\n--- Current Metrics ---")
        for uid, data in metrics.items():
            print(f"User {uid}: {data}")
```

### 3. Output

```
--- Current Metrics ---
User user_5: {'clicks': 3, 'views': 2, 'purchases': 1}
User user_2: {'clicks': 2, 'views': 4, 'purchases': 0}

--- Current Metrics ---
User user_5: {'clicks': 6, 'views': 4, 'purchases': 2}
User user_2: {'clicks': 4, 'views': 8, 'purchases': 1}
```

---

## Advanced Patterns

### 1. Exactly-Once Semantics

**Challenge**: Producer may duplicate messages (retries) or lose data (acks=0)

**Solution**: Enable idempotent producer
```python
producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    enable_idempotence=True,  # Ensures exactly-once per partition
    acks='all'
)
```

**Transactions**: For atomic writes across multiple partitions
```python
producer.init_transaction()
producer.begin_transaction()

try:
    producer.send('topic1', value='data1')
    producer.send('topic2', value='data2')
    producer.commit_transaction()
except Exception as e:
    producer.abort_transaction()
```

---

### 2. Compact Topic (Latest State)

Instead of time-based retention, keep only latest value per key.

```bash
# Create compacted topic
kafka-topics.sh --create \
  --bootstrap-server localhost:9092 \
  --topic user-profiles \
  --config cleanup.policy=compact
```

**Use Case**: User profile updates
```
Key: user_123 → Value: {name: "Alice", city: "NYC"}
Key: user_123 → Value: {name: "Alice", city: "LA"}  ← Latest kept
Key: user_456 → Value: {name: "Bob", city: "SFO"}
```

---

### 3. Time-Based Partitioning

Organize data by day/hour for efficient queries.

```python
from datetime import datetime

# Send to date-based topic
date_str = datetime.now().strftime('%Y-%m-%d')
topic_name = f"events-{date_str}"

producer.send(topic_name, value=event)
```

---

## Practical Examples: Python & SQL

### Advanced Producer Patterns

#### 1. Batch Producer with Error Handling

```python
from kafka import KafkaProducer
import json
import time
from concurrent.futures import ThreadPoolExecutor

class BatchProducer:
    def __init__(self, bootstrap_servers):
        self.producer = KafkaProducer(
            bootstrap_servers=bootstrap_servers,
            value_serializer=lambda v: json.dumps(v).encode('utf-8'),
            acks='all',
            retries=3,
            max_in_flight_requests_per_connection=1,
            enable_idempotence=True
        )

    def send_batch(self, records, topic):
        """Send batch of records with error handling"""
        futures = []
        for record in records:
            future = self.producer.send(
                topic=topic,
                key=str(record['user_id']).encode('utf-8'),
                value=record
            )
            future.add_errback(self.error_handler)
            futures.append(future)

        # Wait for all to complete
        for future in futures:
            future.get(timeout=30)

    def error_handler(self, exception):
        print(f"Error sending message: {exception}")

    def close(self):
        self.producer.flush()
        self.producer.close()

# Usage
producer = BatchProducer(['localhost:9092'])

# Simulate batch of events
events = [
    {'user_id': 1, 'event': 'click', 'timestamp': time.time()},
    {'user_id': 2, 'event': 'view', 'timestamp': time.time()},
    {'user_id': 3, 'event': 'purchase', 'timestamp': time.time()},
]

producer.send_batch(events, 'user-events')
producer.close()
```

#### 2. Streaming Producer (Real-Time)

```python
import time
import random
from kafka import KafkaProducer
import json

producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8'),
    key_serializer=lambda k: k.encode('utf-8'),
    compression_type='gzip'  # Compress messages
)

# Simulate real-time events
event_types = ['click', 'view', 'add_to_cart', 'purchase']

print("Starting real-time producer (Ctrl+C to stop)...")

try:
    while True:
        event = {
            'user_id': random.randint(1, 100),
            'event_type': random.choice(event_types),
            'page': random.choice(['home', 'product', 'checkout']),
            'timestamp': int(time.time())
        }

        # Send with key for partitioning
        producer.send(
            topic='realtime-events',
            key=f"user_{event['user_id']}",
            value=event
        )

        print(f"Sent: {event}")
        time.sleep(0.1)  # 10 events per second

except KeyboardInterrupt:
    print("\nStopping producer...")
    producer.flush()
    producer.close()
```

### Advanced Consumer Patterns

#### 3. Multi-Topic Consumer

```python
from kafka import KafkaConsumer
import json

consumer = KafkaConsumer(
    'user-events',      # Subscribe to multiple topics
    'realtime-events',
    'order-events',
    bootstrap_servers=['localhost:9092'],
    group_id='analytics-consumer',
    auto_offset_reset='earliest',
    enable_auto_commit=False,
    value_deserializer=lambda m: json.loads(m.decode('utf-8'))
)

print("Subscribed to:", consumer.subscription())

for message in consumer:
    topic = message.topic
    partition = message.partition
    offset = message.offset
    data = message.value

    print(f"Topic: {topic}, Partition: {partition}, Offset: {offset}")
    print(f"Data: {data}")

    # Route to different processing based on topic
    if topic == 'user-events':
        process_user_event(data)
    elif topic == 'realtime-events':
        process_realtime_event(data)
    elif topic == 'order-events':
        process_order_event(data)

    # Manual commit after processing
    consumer.commit()
```

#### 4. Consumer with Metrics

```python
from kafka import KafkaConsumer
import time
from collections import defaultdict

class MetricsConsumer:
    def __init__(self, bootstrap_servers):
        self.consumer = KafkaConsumer(
            'user-events',
            bootstrap_servers=bootstrap_servers,
            group_id='metrics-consumer',
            enable_auto_commit=False
        )
        self.metrics = defaultdict(int)

    def process_messages(self):
        start_time = time.time()
        message_count = 0

        for message in self.consumer:
            message_count += 1

            # Track metrics
            event_type = message.value.get('event_type', 'unknown')
            self.metrics[event_type] += 1

            # Print metrics every 1000 messages
            if message_count % 1000 == 0:
                self.print_metrics()

            self.consumer.commit()

    def print_metrics(self):
        print("\n=== Event Metrics ===")
        for event_type, count in sorted(self.metrics.items()):
            print(f"{event_type}: {count}")

# Usage
consumer = MetricsConsumer(['localhost:9092'])
consumer.process_messages()
```

### Kafka SQL (kSQL / Streaming SQL)

kSQL (now Confluent ksqlDB) allows SQL-like queries on Kafka topics.

#### 1. Create Stream from Topic

```sql
-- Create a stream from Kafka topic
CREATE STREAM orders (
    order_id BIGINT,
    user_id BIGINT,
    product_id INT,
    amount DECIMAL(10, 2),
    order_time TIMESTAMP
) WITH (
    KAFKA_TOPIC = 'orders',
    VALUE_FORMAT = 'JSON',
    TIMESTAMP = 'order_time'
);
```

#### 2. Windowed Aggregations

```sql
-- Revenue per product in 5-minute windows
CREATE STREAM product_revenue AS
SELECT
    product_id,
    TUMBLE_START(order_time, INTERVAL '5' MINUTES) AS window_start,
    SUM(amount) AS total_revenue,
    COUNT(*) AS order_count
FROM orders
GROUP BY
    product_id,
    TUMBLE(order_time, INTERVAL '5' MINUTES');
```

#### 3. Filter and Transform

```sql
-- Filter high-value orders (> $100)
CREATE STREAM high_value_orders AS
SELECT
    order_id,
    user_id,
    amount,
    order_time,
    'high_value' AS order_category
FROM orders
WHERE amount > 100
EMIT CHANGES;
```

#### 4. Stream-Stream Join

```sql
-- Join orders with user profiles
CREATE STREAM enriched_orders AS
SELECT
    o.order_id,
    o.user_id,
    u.name AS user_name,
    u.tier AS user_tier,
    o.amount,
    o.order_time
FROM orders o
LEFT JOIN users u
ON o.user_id = u.user_id
WITHIN 1 HOUR  -- Join events within 1 hour
EMIT CHANGES;
```

#### 5. Pattern Matching (Complex Event Processing)

```sql
-- Detect rapid purchases (3+ within 5 minutes)
CREATE TABLE suspicious_users AS
SELECT
    user_id,
    COUNT(*) AS purchase_count,
    COLLECT_LIST(amount) AS amounts
FROM orders
WHERE order_type = 'purchase'
WINDOW TUMBLING (SIZE 5 MINUTES)
GROUP BY user_id
HAVING COUNT(*) >= 3
EMIT CHANGES;
```

#### 6. Real-Time Aggregations with GROUP BY

```sql
-- Dashboard metrics: revenue, orders, avg order value
CREATE STREAM dashboard_metrics AS
SELECT
    TUMBLE_START(order_time, INTERVAL '1' MINUTE) AS window_start,
    SUM(amount) AS total_revenue,
    COUNT(*) AS total_orders,
    AVG(amount) AS avg_order_value,
    MIN(amount) AS min_order_value,
    MAX(amount) AS max_order_value
FROM orders
GROUP BY TUMBLE(order_time, INTERVAL '1' MINUTE')
EMIT CHANGES;
```

### End-to-End Example: Real-Time E-Commerce Dashboard

#### Python: Producer (Order Events)

```python
import time
import random
from kafka import KafkaProducer
import json

producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8'),
    key_serializer=lambda k: k.encode('utf-8'),
    acks='all',
    compression_type='gzip'
)

# Generate realistic order events
def generate_order():
    return {
        'order_id': random.randint(10000, 99999),
        'user_id': random.randint(1, 1000),
        'product_id': random.choice([101, 102, 103, 104]),
        'amount': round(random.uniform(10, 500), 2),
        'order_time': int(time.time() * 1000),
        'status': random.choice(['pending', 'completed', 'cancelled'])
    }

print("Generating order events...")

for i in range(100):
    order = generate_order()
    producer.send(
        topic='orders',
        key=f"user_{order['user_id']}",  # Partition by user
        value=order
    )
    print(f"Sent order {i+1}: {order['order_id']}")
    time.sleep(0.05)  # 20 orders/second

producer.flush()
producer.close()
```

#### Python: Consumer (Real-Time Analytics)

```python
from kafka import KafkaConsumer
from collections import defaultdict
import time

consumer = KafkaConsumer(
    'orders',
    bootstrap_servers=['localhost:9092'],
    group_id='analytics-consumer',
    auto_offset_reset='latest',
    enable_auto_commit=False,
    value_deserializer=lambda m: json.loads(m.decode('utf-8'))
)

# Real-time metrics
metrics = {
    'total_revenue': 0.0,
    'order_count': 0,
    'orders_per_minute': 0,
    'revenue_per_minute': 0.0,
    'top_products': defaultdict(float)
}

minute_start = time.time()

print("Starting real-time analytics...")

for message in consumer:
    order = message.value

    # Update metrics
    metrics['total_revenue'] += order['amount']
    metrics['order_count'] += 1
    metrics['top_products'][order['product_id']] += order['amount']

    # Print metrics every minute
    current_time = time.time()
    if current_time - minute_start >= 60:
        metrics['orders_per_minute'] = metrics['order_count']
        metrics['revenue_per_minute'] = metrics['total_revenue']

        print("\n=== Real-Time Dashboard ===")
        print(f"Total Revenue: ${metrics['total_revenue']:.2f}")
        print(f"Total Orders: {metrics['order_count']}")
        print(f"Orders/Minute: {metrics['orders_per_minute']}")
        print(f"Revenue/Minute: ${metrics['revenue_per_minute']:.2f}")

        print("\nTop Products:")
        sorted_products = sorted(
            metrics['top_products'].items(),
            key=lambda x: x[1],
            reverse=True
        )[:5]
        for product_id, revenue in sorted_products:
            print(f"  Product {product_id}: ${revenue:.2f}")

        # Reset minute metrics
        metrics['order_count'] = 0
        metrics['total_revenue'] = 0.0
        minute_start = current_time

    consumer.commit()
```

#### kSQL: Real-Time Queries

```sql
-- 1. Create orders stream
CREATE STREAM orders_stream (
    order_id BIGINT,
    user_id BIGINT,
    product_id INT,
    amount DOUBLE,
    order_time BIGINT,
    status VARCHAR
) WITH (
    KAFKA_TOPIC = 'orders',
    VALUE_FORMAT = 'JSON',
    TIMESTAMP = 'order_time'
);

-- 2. Create materialized view for dashboard metrics
CREATE TABLE dashboard_metrics AS
SELECT
    TIMESTAMPTOSTRING(TUMBLE_START(order_time, MINUTES(1)), 'yyyy-MM-dd HH:mm:ss') AS window_time,
    COUNT(*) AS total_orders,
    SUM(amount) AS total_revenue,
    AVG(amount) AS avg_order_value
FROM orders_stream
WHERE status = 'completed'
GROUP BY TUMBLE(order_time, MINUTES(1));

-- 3. Real-time top products
CREATE TABLE top_products AS
SELECT
    product_id,
    SUM(amount) AS revenue
FROM orders_stream
WHERE status = 'completed'
WINDOW TUMBLING (SIZE 5 MINUTES)
GROUP BY product_id
HAVING SUM(amount) > 0;

-- 4. User activity (orders per user)
CREATE TABLE user_activity AS
SELECT
    user_id,
    COUNT(*) AS order_count,
    SUM(amount) AS total_spent,
    AVG(amount) AS avg_order_value
FROM orders_stream
WHERE status = 'completed'
WINDOW TUMBLING (SIZE 1 HOUR)
GROUP BY user_id;
```

---

## SQL Examples: Kafka Integration

### 1. Reading Kafka into SQL Database (ETL)

```sql
-- Read from Kafka, write to PostgreSQL
-- Using Kafka Connect or custom consumer

-- PostgreSQL table to store processed orders
CREATE TABLE orders (
    order_id BIGINT PRIMARY KEY,
    user_id BIGINT,
    product_id INT,
    amount DECIMAL(10, 2),
    order_time TIMESTAMP,
    status VARCHAR(20),
    processed_at TIMESTAMP DEFAULT NOW()
);

-- Bulk insert from Kafka consumer
INSERT INTO orders (order_id, user_id, product_id, amount, order_time, status)
VALUES
    (12345, 67890, 101, 99.99, '2024-02-14 10:30:00', 'completed'),
    (12346, 67891, 102, 149.99, '2024-02-14 10:31:00', 'completed'),
    (12347, 67892, 103, 199.99, '2024-02-14 10:32:00', 'pending');
```

### 2. Analytics Queries on Processed Data

```sql
-- Daily revenue analysis
SELECT
    DATE(order_time) AS order_date,
    COUNT(*) AS order_count,
    SUM(amount) AS total_revenue,
    AVG(amount) AS avg_order_value,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY amount) AS median_order_value
FROM orders
WHERE order_time >= NOW() - INTERVAL '7 days'
GROUP BY DATE(order_time)
ORDER BY order_date DESC;

-- Top products by revenue (last 7 days)
SELECT
    product_id,
    COUNT(*) AS order_count,
    SUM(amount) AS total_revenue,
    AVG(amount) AS avg_price
FROM orders
WHERE order_time >= NOW() - INTERVAL '7 days'
GROUP BY product_id
ORDER BY total_revenue DESC
LIMIT 10;

-- User segmentation (RFM analysis)
WITH user_rfm AS (
    SELECT
        user_id,
        MAX(order_time) AS last_order_date,
        COUNT(*) AS frequency,
        SUM(amount) AS monetary_value
    FROM orders
    WHERE order_time >= NOW() - INTERVAL '90 days'
    GROUP BY user_id
),
user_segments AS (
    SELECT
        user_id,
        last_order_date,
        frequency,
        monetary_value,
        CASE
            WHEN last_order_date >= NOW() - INTERVAL '7 days' THEN 'active'
            WHEN last_order_date >= NOW() - INTERVAL '30 days' THEN 'at_risk'
            ELSE 'churned'
        END AS recency_segment,
        CASE
            WHEN frequency > 10 THEN 'vip'
            WHEN frequency > 5 THEN 'regular'
            ELSE 'occasional'
        END AS frequency_segment
    FROM user_rfm
)
SELECT
    recency_segment,
    frequency_segment,
    COUNT(*) AS user_count,
    AVG(monetary_value) AS avg_ltv
FROM user_segments
GROUP BY recency_segment, frequency_segment
ORDER BY
    CASE recency_segment
        WHEN 'active' THEN 1
        WHEN 'at_risk' THEN 2
        ELSE 3
    END;
```

---

## Python vs SQL Comparison

| Task | Python (Producer/Consumer) | SQL (kSQL / Database) |
|------|---------------------------|------------------------|
| **Send events** | `producer.send(topic, value)` | `INSERT INTO stream VALUES (...)` |
| **Consume events** | `for message in consumer:` | `SELECT * FROM stream EMIT CHANGES` |
| **Filter** | `if condition: process()` | `WHERE condition` |
| **Aggregate** | Manual counter/dict | `SUM(), COUNT(), AVG()` |
| **Window** | Python deque or custom | `TUMBLE(), HOP(), SESSION()` |
| **Join streams** | Manual dict lookup | `JOIN stream1 ON key` |

---

## Performance Tips

### Producer Optimization

```python
# Batch messages
for batch in chunks(records, size=1000):
    producer.send_batch(batch, topic)

# Compression
producer = KafkaProducer(compression_type='gzip')  # or 'snappy', 'lz4'

# Async sends
futures = [producer.send(topic, value) for value in values]
for future in futures:
    result = future.get(timeout=30)
```

### Consumer Optimization

```python
# Fetch more messages per poll
consumer = KafkaConsumer(
    max_poll_records=500,  # Larger batches
    fetch_max_wait_ms=500
)

# Process in parallel (thread pool)
from concurrent.futures import ThreadPoolExecutor

def process_message(message):
    process_event(message.value)

with ThreadPoolExecutor(max_workers=4) as executor:
    executor.map(process_message, consumer)
```

---

## Kafka vs Other Messaging Systems

| Feature | Kafka | RabbitMQ | AWS Kinesis | Google Pub/Sub |
|---------|-------|----------|-------------|----------------|
| **Throughput** | Very High | High | Very High | High |
| **Latency** | Low | Very Low | Low | Low |
| **Persistence** | Yes (disk) | Yes (disk/RAM) | Yes (disk) | No (in-memory) |
| **Ordering** | Per partition | Per queue | Per shard | Not guaranteed |
| **Scaling** | Horizontal | Vertical | Horizontal | Horizontal |
| **Use Case** | Analytics, streaming | Work queues | Real-time | Event-driven |

---

## Monitoring & Operations

### Key Metrics

| Metric | What to Monitor |
|--------|-----------------|
| **Messages/sec** | Throughput per topic |
| **Consumer Lag** | How far behind consumers are |
| **Disk Usage** | Broker disk space |
| **Under Replicated Partitions** | Replication health |
| **Request Latency** | Producer/consumer delays |

### Tools

- **Kafka Manager**: Web UI for monitoring
- **Burrow**: Consumer group lag monitoring
- **Prometheus + Grafana**: Metrics dashboards
- **Conduktor**: Kafka platform with UI

---

## Best Practices

### 1. Partitioning Strategy

| Strategy | When to Use | Example |
|----------|-------------|---------|
| **Hash by key** | Order per user/entity | `user_id` |
| **Round-robin** | No ordering needed | Logs |
| **Random** | Load balance | Analytics |
| **Custom** | Specific partitioning | Time-based |

### 2. Producer Configuration

```python
# Production-ready producer
producer = KafkaProducer(
    bootstrap_servers=['broker1:9092,broker2:9092'],
    acks='all',                    # Strongest durability
    retries=3,                     # Retry on failure
    max_in_flight_requests_per_connection=5,
    enable_idempotence=True,       # Exactly-once
    compression_type='gzip',       # Reduce network bandwidth
    batch_size=16384,              # Batch messages
    linger_ms=10,                  # Wait for batch
)
```

### 3. Consumer Configuration

```python
# Production-ready consumer
consumer = KafkaConsumer(
    'topic-name',
    bootstrap_servers=['broker1:9092'],
    group_id='my-consumer-group',
    auto_offset_reset='earliest',  # Start from beginning
    enable_auto_commit=False,      # Manual control
    max_poll_records=500,          # Batch processing
    session_timeout_ms=30000,      # Rebalance timeout
    heartbeat_interval_ms=3000,   # Heartbeat frequency
)
```

### 4. Error Handling

```python
try:
    producer.send('topic', value=data).add_callback(
        lambda metadata: print(f"Sent to {metadata.topic}")
    ).add_errback(
        lambda e: print(f"Failed: {e}")
    )
except Exception as e:
    # Retry logic
    send_with_backoff(data)
```

---

## Interview Questions

**Q1: How does Kafka ensure message ordering?**
- Ordering guaranteed only within a partition
- Same key → same partition via hash(key) % partitions
- Different partitions = no ordering guarantee

**Q2: What happens when a consumer fails?**
- Consumer stops sending heartbeats
- Group coordinator triggers rebalance
- Partitions reassigned to remaining consumers
- Failed consumer's offsets reset (configurable)

**Q3: What's the difference between acks=0, 1, all?**
- `acks=0`: No confirmation, fastest but may lose data
- `acks=1`: Leader confirms, default balance
- `acks=all`: All replicas confirm, strongest durability

**Q4: How do you handle consumer lag?**
- Monitor lag with `kafka-consumer-groups.sh`
- Add consumers to group (up to partition count)
- Scale up consumer resources
- Optimize processing logic

**Q5: What's a compacted topic and when to use?**
- Keeps only latest value per key, not time-based
- Use for state/data store (user profiles, configs)
- Not for event streams (need full history)

---

## Common Commands

```bash
# List topics
kafka-topics.sh --list --bootstrap-server localhost:9092

# Describe topic
kafka-topics.sh --describe --topic user-events --bootstrap-server localhost:9092

# Consume from CLI
kafka-console-consumer.sh --topic user-events \
  --bootstrap-server localhost:9092 --from-beginning

# Produce to CLI
kafka-console-producer.sh --topic user-events \
  --bootstrap-server localhost:9092

# Check consumer group lag
kafka-consumer-groups.sh --describe --group my-group \
  --bootstrap-server localhost:9092
```

---

## Related Concepts

- [[Apache Flink - Real-Time Analytics]] - Stream processing
- [[Kinesis]] - AWS streaming alternative
- [[Pub/Sub]] - GCP messaging
- [[Exactly-Once]] - Processing guarantees
- [[Windowing]] - Time-based aggregations

---

## Learning Resources

- [Kafka Documentation](https://kafka.apache.org/documentation/)
- [Confluent Kafka Courses](https://www.confluent.io/kafka-courses/)
- [Kafka: The Definitive Guide (Book)](https://www.confluent.io/resources/kafka-the-definitive-guide/)

---

**Progress**: 🟡 Learning (concepts understood, need hands-on practice)

**Next Steps**:
- [ ] Run local Kafka cluster (Docker)
- [ ] Build producer/consumer example
- [ ] Practice consumer rebalancing scenarios
- [ ] Compare Kafka vs Kinesis

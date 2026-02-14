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

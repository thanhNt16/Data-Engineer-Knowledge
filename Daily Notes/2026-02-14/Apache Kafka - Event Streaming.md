---
tags: [streaming, kafka, producer, consumer, learning, 2026-02-14]
date: 2026-02-14
status: learned
---

# Apache Kafka - Event Streaming (2026-02-14)

## What I Learned

Created comprehensive Apache Kafka reference with hands-on examples:

### Core Concepts
- **Topics**: Logical channels for messages
- **Partitions**: Ordered sequences, parallel processing
- **Producers**: Publish records with keys for partitioning
- **Consumers**: Read records, track offsets per partition
- **Consumer Groups**: Share topic consumption across consumers

### Hands-On Examples

**Python Producer**:
```python
producer.send(
    topic='user-clicks',
    key=b'user_123',
    value={'user_id': '123', 'event': 'click', 'timestamp': '...'}
)
```

**Python Consumer**:
```python
for message in consumer:
    data = message.value
    partition = message.partition
    offset = message.offset
    process_click_event(data)
    consumer.commit()
```

### Advanced Patterns
- **Exactly-once semantics**: `enable_idempotence=True`
- **Compacted topics**: Keep latest state per key
- **Time-based partitioning**: Organize data by date
- **Consumer rebalancing**: Dynamic group management

### Architecture
- **Broker**: Kafka server storing data
- **Replica**: Data copies (leader + followers)
- **Controller**: Broker managing elections
- **High-throughput**: Millions of messages/sec
- **Fault-tolerant**: Data replication across brokers

### Interview Prep
- 5 key interview questions with answers
- Focus on ordering guarantees, consumer failure handling
- Producer acks (0, 1, all) trade-offs
- Consumer lag monitoring and handling

---

## Key Insights

### Partitioning Strategy
| Strategy | Use Case |
|----------|----------|
| Hash by key | Order per user/entity |
| Round-robin | No ordering needed |
| Time-based | Efficient querying |

### Producer Configuration (Production)
```python
acks='all',                    # Strongest durability
enable_idempotence=True,       # Exactly-once
compression_type='gzip',       # Reduce bandwidth
batch_size=16384,              # Batch messages
linger_ms=10,                  # Wait for batch
```

### Consumer Configuration (Production)
```python
enable_auto_commit=False,      # Manual control
max_poll_records=500,          # Batch processing
session_timeout_ms=30000,      # Rebalance timeout
heartbeat_interval_ms=3000,    # Heartbeat frequency
```

---

## Comparisons

**Kafka vs RabbitMQ vs Kinesis vs Pub/Sub**:
- Kafka: Highest throughput, disk persistence
- RabbitMQ: Lowest latency, work queues
- Kinesis: AWS-native, auto-scaling
- Pub/Sub: GCP-native, no persistence

---

## Related Notes

- [[02-Areas/Streaming/Apache Kafka - Event Streaming]] - Full reference
- [[02-Areas/Streaming/Apache Flink - Real-Time Analytics]] - Stream processing
- [[Skills Tracker/Progress Matrix]] - Updated to 75% streaming

---

## Progress Impact

**Streaming Category**: 63% → 75% (6/8 skills)

New skills marked as learning:
- Apache Kafka (producers, consumers, partitions, consumer groups)

---

## Next Steps

- [ ] Run Kafka locally (Docker cluster)
- [ ] Build producer/consumer from examples
- [ ] Practice consumer rebalancing scenarios
- [ ] Learn AWS Kinesis
- [ ] Complete streaming category (Kinesis, Pub/Sub)

---

**Total Progress**: 30% → 31% (32/104 skills)

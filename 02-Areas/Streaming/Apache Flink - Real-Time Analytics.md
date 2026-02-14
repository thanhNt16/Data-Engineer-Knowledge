---
tags: [streaming, flink, real-time, analytics, apache]
date: 2026-02-14
status: learning
---

# Apache Flink - Real-Time Analytics

## Overview

Apache Flink is a distributed processing engine and stateful computations over bounded and unbounded data streams. It's designed for **stateful computations over data streams** with **exactly-once semantics**.

**Key Use Cases**:
- Real-time analytics (event-time processing)
- Complex event processing (CEP)
- Stream/batch unification
- Stateful stream processing

---

## Core Architecture

### Dataflow Graph

```
Sources → Transformations (Operators) → Sinks
```

**Components**:
- **JobManager**: Controls execution (scheduling, checkpoints, recovery)
- **TaskManager**: Executes tasks (slots for parallel tasks)
- **Slot**: Unit of resource in TaskManager (holds operator chains)

### Execution Model

| Mode | Description | Use Case |
|------|-------------|----------|
| **Streaming** | Unbounded, continuous | Real-time analytics |
| **Batch** | Bounded, finite | Historical processing |
| **Hybrid** | Unified API for both | Both scenarios |

---

## Key Concepts

### 1. Time Semantics

| Time Type | Description | When Used |
|-----------|-------------|-----------|
| **Event Time** | Time embedded in data | Correctness, late data |
| **Ingestion Time** | Time at source | When event time unavailable |
| **Processing Time** | Time at operator | Low-latency, approximations |

**Watermarks**: Timestamps that track event time progress
- Handle late-arriving data
- Trigger window computations
- Configurable latency trade-offs

### 2. Windowing

| Window Type | Description | Example |
|-------------|-------------|---------|
| **Tumbling** | Fixed-size, non-overlapping | 1-minute aggregates |
| **Sliding** | Fixed-size, overlapping | 1-min window every 30s |
| **Session** | Dynamic size, gap-based | User activity sessions |

**Window Functions**:
- `Reduce`: Aggregates incremental
- `Aggregate`: General aggregation
- `Process`: Full window access (stateful)

### 3. State Management

**State Types**:
- **Keyed State**: Partitioned by key (e.g., per user)
- **Operator State**: Per operator instance (e.g., Kafka offsets)

**State Backends**:
- **MemoryStateBackend**: Local memory (testing)
- **FsStateBackend**: Local memory + filesystem (production)
- **RocksDBStateBackend**: Local RocksDB (large state)

**Checkpointing**: Periodic, asynchronous state snapshots
- Enables exactly-once semantics
- Fault tolerance via recovery
- Configurable interval (default 500ms)

---

## Flink vs Kafka Streams vs Spark Streaming

| Feature | Flink | Kafka Streams | Spark Streaming |
|---------|-------|---------------|-----------------|
| **Processing Model** | Native streaming | Native streaming | Micro-batch |
| **Latency** | Milliseconds | Milliseconds | Seconds |
| **State Management** | Built-in | Built-in (RocksDB) | External |
| **Exactly-Once** | Yes | Yes | Yes (with sink support) |
| **Windowing** | Advanced | Basic | Basic |
| **Ecosystem** | Rich | Kafka-native | Spark ecosystem |
| **Use Case** | Complex streaming | Kafka-centric | Unified batch/stream |

---

## Common Operators

```java
// Example: Streaming word count with windowing
DataStream<Tuple2<String, Integer>> counts = text
    .flatMap(new Tokenizer())
    .keyBy(0)
    .window(TumblingEventTimeWindows.of(Time.minutes(1)))
    .sum(1);
```

**Key Operators**:
- `map`, `flatMap`: Transformations
- `keyBy`: Partitioning
- `window`, `windowAll`: Windowing
- `join`, `coGroup`: Stream joins
- `connect`: Union of different types
- `process`: Low-level operator access

---

## Fault Tolerance

**Checkpoint Barrier**: Marker that flows through dataflow
- Aligns state snapshots
- Exactly-once guarantee
- Recovery from last checkpoint

**Savepoint**: Manual, explicit checkpoints
- Application upgrades
- A/B testing
- Migration between clusters

---

## Integration Points

### Sources
- **Kafka**: `FlinkKafkaConsumer`
- **Kinesis**: `KinesisConsumer`
- **Files**: `FileSource`
- **Socket**: `SocketTextStreamFunction`
- **Custom**: Implement `SourceFunction`

### Sinks
- **Kafka**: `FlinkKafkaProducer`
- **Elasticsearch**: `ElasticsearchSink`
- **JDBC**: `JdbcSink`
- **File**: `FileSink`
- **Custom**: Implement `SinkFunction`

---

## Performance Optimization

### Parallelism
- **Operator chaining**: Fuses operators (fewer network hops)
- **Task slots**: Trade-off between parallelism and resources
- **Slot sharing**: Multiple operators share slot

### Backpressure Handling
- Automatic flow control
- TCP-based credit-based protocol
- Downstream slowdown propagates upstream

### State Backend Selection
| Backend | Size | Latency | Throughput | Use Case |
|---------|------|---------|------------|----------|
| Memory | < 100MB | Low | High | Testing |
| Fs | < 10GB | Medium | Medium | Small state |
| RocksDB | Unlimited | High | High | Production, large state |

---

## Monitoring & Metrics

**Built-in Metrics**:
- Operator-level: records in/out, latency
- TaskManager: CPU, memory, GC
- JobManager: checkpoint duration, recovery

**Integrations**:
- Prometheus, InfluxDB
- Grafana dashboards
- Kafka Lag monitoring

---

## Common Patterns

### 1. Late Data Handling
```java
.allowedLateness(Time.seconds(10))
.sideOutputLateData(lateTag)
```

### 2. Session Windows
```java
.window(EventTimeSessionWindows.withGap(Time.minutes(5)))
```

### 3. Watermark Strategies
```java
WatermarkStrategy
    .forBoundedOutOfOrderness(Duration.ofSeconds(5))
    .withTimestampAssigner((event, ts) -> event.timestamp)
```

### 4. State TTL
```java
StateTtlConfig ttlConfig = StateTtlConfig
    .newBuilder(Time.hours(24))
    .setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite)
    .build();
```

---

## Interview Questions

**Q1: How does Flink handle exactly-once semantics?**
- Checkpoint barriers align state snapshots
- Source offsets, operator state, sink commits atomic
- Recovery from last checkpoint

**Q2: What's the difference between event time and processing time?**
- Event time: embedded in data, correctness
- Processing time: wall-clock at operator, low latency
- Watermarks bridge gap, handle late data

**Q3: How do you handle late-arriving data?**
- Watermarks define lateness threshold
- `allowedLateness()` extends window lifetime
- Side outputs collect truly late data

**Q4: When would you choose Flink over Spark Streaming?**
- Flink: Low latency (<1s), complex stateful processing
- Spark: Micro-batch, unified batch/streaming, Spark ecosystem

**Q5: How does backpressure work in Flink?**
- Automatic TCP flow control
- Credit-based protocol
- Downstream slowdown propagates upstream

---

## Related Concepts

- [[Kafka]]: Message broker for streaming
- [[Windowing]]: Time-based aggregations
- [[Watermarks]]: Event time progress
- [[Exactly-Once]]: Processing guarantees
- [[State Management]]: Stateful processing

---

## Learning Resources

- [Flink Documentation](https://nightlies.apache.org/flink/flink-docs-release-1.20/)
- [Flink Training](https://flink.apache.org/training/)
- [Real-time Processing with Flink](https://www.ververica.com/real-time-apache-flink)

---

**Progress**: 🟡 Learning (fundamentals understood, need practice)

**Next Steps**:
- [ ] Run Flink locally (Docker or local cluster)
- [ ] Implement sliding window example
- [ ] Practice stateful word count
- [ ] Compare Flink vs Kafka Streams hands-on

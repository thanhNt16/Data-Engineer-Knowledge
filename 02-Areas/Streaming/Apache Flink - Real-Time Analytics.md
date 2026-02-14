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

---

## Practical Examples: Python & SQL

### Flink Python API Examples

#### 1. Word Count (DataStream API)

```python
from pyflink.datastream import StreamExecutionEnvironment
from pyflink.datastream.functions import FlatMapFunction, ReduceFunction
from pyflink.common.typeinfo import Types

env = StreamExecutionEnvironment.get_execution_environment()

# Create source (e.g., Kafka, socket)
data_stream = env.from_collection([
    {"text": "hello world"},
    {"text": "flink streaming"},
    {"text": "hello flink"}
])

# Transform: Split words
class Splitter(FlatMapFunction):
    def flat_map(self, value):
        words = value["text"].lower().split()
        for word in words:
            yield word, 1

word_stream = data_stream.flat_map(
    Splitter(),
    Types.TUPLE([Types.STRING(), Types.INT()])
)

# Count words (key by word, then reduce)
class Counter(ReduceFunction):
    def reduce(self, value1, value2):
        return value1[0], value1[1] + value2[1]

word_counts = word_stream.key_by(lambda x: x[0]).reduce(Counter)

# Print results
word_counts.print()

# Execute job
env.execute("Word Count")
```

#### 2. Windowed Aggregation (Python Table API)

```python
from pyflink.table import StreamTableEnvironment, DataTypes

# Create table environment
t_env = StreamTableEnvironment.create(env)

# Register source (e.g., Kafka orders stream)
t_env.execute_sql("""
    CREATE TABLE orders (
        order_id BIGINT,
        user_id BIGINT,
        amount DECIMAL(10, 2),
        order_time TIMESTAMP(3),
        WATERMARK FOR order_time AS order_time - INTERVAL '5' SECOND
    ) WITH (
        'connector' = 'kafka',
        'topic' = 'orders',
        'properties.bootstrap.servers' = 'kafka:9092',
        'format' = 'json'
    )
""")

# Query: Revenue per user in 1-minute tumbling windows
result = t_env.sql_query("""
    SELECT
        TUMBLE_START(order_time, INTERVAL '1' MINUTE) AS window_start,
        user_id,
        SUM(amount) AS total_revenue,
        COUNT(*) AS order_count
    FROM orders
    GROUP BY
        TUMBLE(order_time, INTERVAL '1' MINUTE),
        user_id
""")

# Print or sink
result.execute().print()
```

#### 3. Joining Streams (Python Table API)

```python
# Create second stream: user profiles
t_env.execute_sql("""
    CREATE TABLE users (
        user_id BIGINT,
        name STRING,
        tier STRING
    ) WITH (
        'connector' = 'kafka',
        'topic' = 'users',
        'properties.bootstrap.servers' = 'kafka:9092',
        'format' = 'json'
    )
""")

# Join orders with users to get tier information
enriched_orders = t_env.sql_query("""
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
    AND o.order_time BETWEEN u.user_id - INTERVAL '1' HOUR
                          AND u.user_id + INTERVAL '1' HOUR
""")

enriched_orders.execute().print()
```

### Flink SQL Examples

Flink SQL provides SQL-like syntax for stream processing with windowing, joins, and aggregations.

#### 1. Tumbling Window Aggregation

```sql
-- Total revenue every 5 minutes
SELECT
    TUMBLE_START(event_time, INTERVAL '5' MINUTE) AS window_start,
    TUMBLE_END(event_time, INTERVAL '5' MINUTE) AS window_end,
    product_category,
    SUM(revenue) AS total_revenue,
    COUNT(*) AS transaction_count
FROM transactions
GROUP BY
    TUMBLE(event_time, INTERVAL '5' MINUTE),
    product_category;
```

#### 2. Sliding Window with Late Data

```sql
-- Rolling 10-minute window, updated every 1 minute
SELECT
    HOP_START(event_time, INTERVAL '1' MINUTE, INTERVAL '10' MINUTE) AS window_start,
    HOP_END(event_time, INTERVAL '1' MINUTE, INTERVAL '10' MINUTE) AS window_end,
    user_id,
    SUM(amount) AS rolling_total,
    AVG(amount) AS rolling_avg
FROM events
GROUP BY
    HOP(event_time, INTERVAL '1' MINUTE, INTERVAL '10' MINUTE),
    user_id;
```

#### 3. Session Window (User Activity)

```sql
-- Group events by user with 30-minute session gap
SELECT
    SESSION_START(event_time, INTERVAL '30' MINUTE) AS session_start,
    SESSION_END(event_time, INTERVAL '30' MINUTE) AS session_end,
    user_id,
    COUNT(*) AS event_count,
    MAX(event_time) - MIN(event_time) AS session_duration
FROM user_events
GROUP BY
    SESSION(event_time, INTERVAL '30' MINUTE),
    user_id;
```

#### 4. Complex Event Processing (CEP)

```sql
-- Detect price surge: >20% increase within 5 minutes
SELECT *
FROM TABLE(
    MATCH_RECOGNIZE(
        PARTITION BY product_id
        ORDER BY event_time
        MEASURES
            A.price AS start_price,
            B.price AS end_price,
            (B.price - A.price) / A.price AS price_change
        ONE ROW PER MATCH
        AFTER MATCH SKIP PAST LAST ROW
        PATTERN (A B+) WITHIN INTERVAL '5' MINUTES
        DEFINE
            B AS (B.price - A.price) / A.price > 0.20
    )
)
WHERE price_change > 0.20;
```

#### 5. Stream-Stream Join

```sql
-- Join click stream with ad impression stream (within 1 hour)
SELECT
    c.click_time,
    c.user_id,
    i.campaign_id,
    i.ad_id
FROM clicks c
INNER JOIN impressions i
ON c.user_id = i.user_id
AND c.click_time BETWEEN
    i.impression_time - INTERVAL '1' HOUR
    AND i.impression_time + INTERVAL '1' HOUR;
```

### Advanced Python Examples

#### 6. Stateful Processing (Keyed State)

```python
from pyflink.datastream.functions import KeyedProcessFunction, RuntimeContext
from pyflink.common.state import ValueStateDescriptor

class CountAlert(KeyedProcessFunction):
    def __init__(self):
        self.state_desc = ValueStateDescriptor("count", Types.INT())

    def open(self, runtime_context: RuntimeContext):
        self.count_state = runtime_context.get_state(self.state_desc)

    def process_element(self, value, ctx):
        count = self.count_state.value() or 0
        count += 1

        # Alert if count exceeds threshold
        if count > 100:
            print(f"Alert! User {value['user_id']} exceeded threshold: {count}")

        # Update state
        self.count_state.update(count)

# Apply to keyed stream
alerts = data_stream.key_by(lambda x: x['user_id']).process(CountAlert())
```

#### 7. Checkpointing Configuration

```python
from pyflink.common import CheckpointingMode

# Enable exactly-once with checkpointing
env.enable_checkpointing(60000)  # 60 second intervals

# Configure checkpointing
env.get_checkpoint_config().set_checkpointing_mode(
    CheckpointingMode.EXACTLY_ONCE
)
env.get_checkpoint_config().set_min_pause_between_checkpoints(30000)
env.get_checkpoint_config().set_checkpoint_timeout(600000)
env.get_checkpoint_config().set_max_concurrent_checkpoints(1)
env.get_checkpoint_config().set_externalized_checkpoints(
    cleanup_mode='RETAIN_ON_CANCELLATION'
)
```

---

## Real-World Example: Real-Time Dashboard

### Use Case

Compute real-time metrics for an e-commerce dashboard: revenue, orders per minute, top products.

### Flink SQL Implementation

```sql
-- 1. Revenue per minute (tumbling window)
CREATE VIEW revenue_per_minute AS
SELECT
    TUMBLE_START(order_time, INTERVAL '1' MINUTE) AS window_start,
    SUM(amount) AS total_revenue,
    COUNT(*) AS order_count,
    AVG(amount) AS avg_order_value
FROM orders
GROUP BY TUMBLE(order_time, INTERVAL '1' MINUTE);

-- 2. Top 5 products by revenue (sliding window)
CREATE VIEW top_products AS
SELECT
    HOP_START(order_time, INTERVAL '1' MINUTE, INTERVAL '10' MINUTE) AS window_start,
    product_id,
    SUM(amount) AS revenue
FROM orders
GROUP BY
    HOP(order_time, INTERVAL '1' MINUTE', INTERVAL '10' MINUTE),
    product_id;

-- 3. User activity (session window)
CREATE VIEW user_sessions AS
SELECT
    SESSION_START(order_time, INTERVAL '30' MINUTE) AS session_start,
    SESSION_END(order_time, INTERVAL '30' MINUTE) AS session_end,
    user_id,
    COUNT(*) AS orders_in_session,
    SUM(amount) AS session_revenue
FROM orders
GROUP BY
    SESSION(order_time, INTERVAL '30' MINUTE),
    user_id;

-- Query all views for dashboard
SELECT
    'Revenue' AS metric_name,
    window_start,
    total_revenue AS value
FROM revenue_per_minute

UNION ALL

SELECT
    'Top Products' AS metric_name,
    window_start,
    revenue AS value
FROM top_products
QUALIFY ROW_NUMBER() OVER (PARTITION BY window_start ORDER BY revenue DESC) <= 5
ORDER BY window_start, revenue DESC;
```

### Python Implementation

```python
from pyflink.table import StreamTableEnvironment

t_env = StreamTableEnvironment.create(env)

# Define all queries
t_env.execute_sql("""
    -- Dashboard: Revenue per minute
    INSERT INTO dashboard_revenue
    SELECT
        TUMBLE_START(order_time, INTERVAL '1' MINUTE) AS window_start,
        'total_revenue' AS metric,
        CAST(SUM(amount) AS STRING) AS value
    FROM orders
    GROUP BY TUMBLE(order_time, INTERVAL '1' MINUTE)
""")

t_env.execute_sql("""
    -- Dashboard: Top products
    INSERT INTO dashboard_products
    SELECT
        HOP_START(order_time, INTERVAL '1' MINUTE, INTERVAL '10' MINUTE) AS window_start,
        'top_product' AS metric,
        product_id AS value
    FROM orders
    GROUP BY
        HOP(order_time, INTERVAL '1' MINUTE', INTERVAL '10' MINUTE),
        product_id
""")

# Execute all statements
t_env.execute("Real-Time Dashboard")
```

---

## SQL Cheat Sheet: Flink vs Traditional

| Operation | Traditional SQL | Flink SQL |
|-----------|----------------|------------|
| **Full table scan** | `SELECT * FROM table` | `SELECT * FROM stream` |
| **Aggregation** | `GROUP BY column` | `GROUP BY TUMBLE(time, interval)` |
| **Time filtering** | `WHERE time > '2024-01-01'` | `WHERE time BETWEEN start AND end` |
| **Windowing** | Window functions | `TUMBLE/HOP/SESSION()` |
| **Late data** | N/A | `ALLOWED_LATENESS()` |
| **Watermarks** | N/A | `WATERMARK FOR time AS time - interval` |

---

## Performance Optimization Examples

### 1. State Backend Selection (Python)

```python
from pyflink.common.state import StateBackend

# Use RocksDB for large state
env.set_state_backend(StateBackend.rocks_db())

# Or use in-memory for small state (faster)
# env.set_state_backend(StateBackend.memory())
```

### 2. Operator Chaining (Python)

```python
# Chain operators to reduce network hops
data_stream.flatMap(...).key_by(...).map(...).filter(...)

# Flink automatically chains these operators
```

### 3. Parallelism Configuration

```python
# Set parallelism per operator
data_stream = source_stream.map(...).set_parallelism(4)

# Or globally for the job
env.set_parallelism(8)
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

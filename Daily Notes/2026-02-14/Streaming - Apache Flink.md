---
tags: [streaming, flink, learning, 2026-02-14]
date: 2026-02-14
status: learned
---

# Streaming - Apache Flink (2026-02-14)

## What I Learned

Created comprehensive Apache Flink knowledge base covering:

### Core Concepts
- **Architecture**: JobManager, TaskManager, Slots, operator chains
- **Time Semantics**: Event time, ingestion time, processing time
- **Watermarks**: Event time progress tracking, late data handling
- **Windowing**: Tumbling, sliding, session windows
- **State Management**: Keyed state, operator state, state backends
- **Exactly-Once**: Checkpoint barriers, fault tolerance, recovery

### Technical Details
- Stream vs batch processing models
- Checkpointing and savepoints
- State backends (Memory, Fs, RocksDB)
- Backpressure handling (credit-based protocol)
- Operators: map, flatMap, keyBy, window, join, process

### Comparisons
- Flink vs Kafka Streams vs Spark Streaming
- When to use each based on latency, use case, ecosystem

### Interview Prep
- 5 key interview questions with detailed answers
- Focus areas: exactly-once semantics, late data handling, backpressure

---

## Next Steps

- [ ] Run Flink locally (Docker cluster)
- [ ] Implement sliding window example
- [ ] Practice stateful word count
- [ ] Compare Flink vs Kafka Streams hands-on
- [ ] Study Apache Kafka fundamentals
- [ ] Learn Kinesis (AWS streaming)

---

## Related Notes

- [[02-Areas/Streaming/Apache Flink - Real-Time Analytics]] - Full reference
- [[Skills Tracker/Progress Matrix]] - Updated to 63% streaming

---

## Progress Impact

**Streaming Category**: 0% → 63% (5/8 skills)

Skills marked as learning:
- Stream Processing Concepts
- Windowing
- Watermarks
- Exactly-Once
- Backpressure

---

**Total Progress**: 25% → 30% (31/104 skills)

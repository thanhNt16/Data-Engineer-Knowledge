---
tags: [database, fundamentals, acid, cap, base, transactions, consistency]
date: 2026-02-15
status: learning
---

# Database Fundamentals

## Overview

Database transactions and consistency properties are foundational concepts for data engineers. Understanding these ensures reliable, scalable data systems.

---

## ACID Properties

ACID is an acronym for the four key properties that guarantee database transactions are processed reliably.

| Property | Description | Example Violation |
|-----------|-------------|---------------------|
| **Atomicity** | Transaction is all-or-nothing | Account balance debits but credits don't match |
| **Consistency** | Transaction moves database from one valid state to another | Account has negative balance |
| **Isolation** | Concurrent transactions don't interfere | Two transfers see different balances |
| **Durability** | Committed transactions survive failures | System crashes before commit is lost |

### ACID Violations

```sql
-- Atomicity Violation: Partial update
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE user_id = 1;
-- System crashes here (rollback doesn't happen!)
UPDATE accounts SET balance = balance + 100 WHERE user_id = 2;
COMMIT;

-- Isolation Violation: Dirty read
-- Transaction 1 reads balance = 100
-- Transaction 2 updates balance = 80 before T1 commits
-- T1 now sees balance = 80 (wrong - should be 100)
```

---

## CAP Theorem

CAP states that in a distributed system, you can only simultaneously have **two** of the following three properties:

| Property | Description | Trade-off |
|-----------|-------------|-----------|
| **C**onsistency | All nodes see same data at the same time | Slower performance (synchronization) |
| **A**vailability | Every request receives a response | Potential inconsistency (no synchronization) |
| **P**artition Tolerance | System continues operating despite network partitions | Data may be stale or out of sync |

### CAP Combinations

| System | Type | Use Case | Example |
|---------|------|-----------|----------|
| **CA** | Strong consistency, available | Traditional RDBMS (PostgreSQL) |
| **CP** | Consistent, partition-tolerant | Distributed database with leader election |
| **AP** | Available, partition-tolerant | Caching layer, eventual consistency |
| **System** | Sacrificing one property | All distributed systems (must choose 2 of 3) |

### CAP in Data Engineering

```python
# Example: Eventual consistency (AP system)
import time
import random

# Data might be stale for a few seconds
def read_data(key):
    time.sleep(0.1)  # Simulate network latency
    return data_store.get(key, "default")

# Write happens, eventually propagates
def write_data(key, value):
    data_store[key] = value
```

---

## BASE Properties

BASE is an alternative to ACID designed for distributed systems, offering more availability at the cost of consistency.

| Property | BASE | ACID |
|-----------|------|------|
| **B**asically Available | Highly available | Consistent |
| **S**oft State | Eventual consistency | Strong consistency |
| **E**ventual consistency | Reads can return stale data | Writes immediately visible |
| **Y**es, Consistent (read/write) | Consistent (read/write) | Data is always consistent |

### BASE vs ACID

| Use Case | Preferred | Reason |
|-----------|-----------|--------|
| **Financial Systems** | ACID | Can't lose or duplicate transactions |
| **Social Media** | BASE | Availability > strong consistency |
| **E-Commerce** | BASE | High availability, users tolerate some inconsistency |
| **Caching Layer** | BASE | Performance over consistency |

### BASE Example: Eventual Consistency

```sql
-- User updates profile
UPDATE users SET email = 'new@email.com' WHERE user_id = 101;

-- Immediate consistency (ACID): All replicas see new email immediately
-- Eventual consistency (BASE): Replicas might see old email briefly
-- Write availability (BASE): Update succeeds even if some replicas are down
```

---

## Isolation Levels

Isolation determines how visible the changes made by one transaction are to other concurrent transactions.

| Level | Description | Dirty Reads | Non-repeatable Reads | Phantom Reads |
|--------|-------------|------------|----------------------|----------------|
| **Read Uncommitted** | ✅ Yes | ❌ No | ❌ No | ✅ Yes |
| **Read Committed** | ❌ No | ❌ No | ❌ No | ✅ Yes |
| **Repeatable Read** | ✅ Yes | ❌ No | ✅ Yes | ❌ No | ❌ No |
| **Serializable** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |
| **Snapshot** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |

### Isolation Levels in Practice

```sql
-- Read Committed (default in PostgreSQL)
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

BEGIN;
SELECT balance FROM accounts WHERE user_id = 101;
COMMIT;
```

---

## Distributed Transactions

### Two-Phase Commit (2PC)

**Problem**: Ensure atomicity across multiple resources (different databases or services).

**Participants**:
- **Coordinator**: Manages transaction
- **Participants**: Execute transaction logic
- **Coordinator Log**: Records transaction progress

**Phases**:
1. **Prepare**: Participants prepare and log "ready"
2. **Commit**: Coordinator sends "commit" command
3. **Execute**: Participants execute transaction
4. **Acknowledge**: Participants confirm commit

**Example**:
```python
# Simplified 2PC
def transfer_funds(from_user, to_user, amount):
    # Phase 1: Prepare
    db1.prepare("debit", from_user, amount)
    db2.prepare("credit", to_user, amount)
    
    # Phase 2: Commit
    if db1.ready and db2.ready:
        db1.commit()
        db2.commit()
    else:
        db1.rollback()
        db2.rollback()
```

**Failure Scenario**: Coordinator crashes after commit, participants unsure → Inconsistent state (participants must query coordinator logs)

### Three-Phase Commit (3PC)

**Problem**: 2PC has blocking issue - coordinator is single point of failure.

**Participants**:
- **Coordinator**
- **Cohort**: Participants
- **Cohort Log**

**Phases**:
1. **CanCommit?**: Coordinator asks participants "can you commit?"
2. **PreCommit**: Participants log "yes" or abort
3. **DoCommit**: Participants commit or abort

**Advantages**:
- Non-blocking
- Can proceed even if coordinator fails (participants decide based on logs)

### Saga Pattern

**Problem**: 2PC blocking in microservices with long transactions.

**Solution**: Break large transaction into sequence of smaller local transactions (sagas) with compensating actions.

**Example**: Order Processing Saga

```sql
-- Saga: Order Processing (5 local transactions)
-- Transaction 1: Create Order
BEGIN;
INSERT INTO orders (order_id, status) VALUES (123, 'pending');
COMMIT;

-- Transaction 2: Reserve Inventory
BEGIN;
UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 456;
COMMIT;

-- Transaction 3: Charge Payment
BEGIN;
INSERT INTO payments (order_id, amount) VALUES (123, 99.99);
COMMIT;

-- Transaction 4: Ship Order
BEGIN;
UPDATE orders SET status = 'shipped' WHERE order_id = 123;
COMMIT;

-- Transaction 5: Mark Complete
BEGIN;
UPDATE orders SET status = 'completed' WHERE order_id = 123;
COMMIT;

-- Compensating Action: If any transaction fails
-- Transaction 3 failed → Insert refund
INSERT INTO refunds (order_id, amount) VALUES (123, 99.99);

-- Transaction 4 failed → Return inventory
UPDATE inventory SET quantity = quantity + 1 WHERE product_id = 456;
```

---

## Database Normalization

### Normal Forms

| Normal Form | Rule | Eliminates | Example |
|--------------|------|---------------|----------|
| **1NF** | Atomic values | Repeating groups | Users: {name: "Alice", name: "Bob"} |
| **2NF** | No partial dependencies | Data redundancy | Products: {prod: "Widget", price: 10, category: "A"} |
| **3NF** | No transitive dependencies | Update anomalies | Departments: {dept: "Sales", manager: "Alice"} |
| **BCNF** | No non-trivial dependencies | Complex dependencies | Course prerequisites |

### Normalization Example

```sql
-- Unnormalized (1NF violation)
CREATE TABLE orders (
    order_id INT,
    customer_name VARCHAR(100),
    customer_address VARCHAR(200),
    customer_phone VARCHAR(20),
    product_name VARCHAR(100),
    product_category VARCHAR(50),
    product_price DECIMAL(10,2),
    quantity INT,
    order_date TIMESTAMP
);

-- 1NF: Atomic values
-- 2NF: Separate customers and products
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    address VARCHAR(200),
    phone VARCHAR(20)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    category VARCHAR(50),
    price DECIMAL(10,2)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id),
    product_id INT REFERENCES products(product_id),
    quantity INT,
    order_date TIMESTAMP
);

-- 3NF: Remove transitive dependencies
-- If customer has multiple addresses, normalize addresses
CREATE TABLE customer_addresses (
    customer_id INT REFERENCES customers(customer_id),
    address VARCHAR(200),
    is_primary BOOLEAN
);

-- Customer now has one record, multiple addresses linked
```

---

## Concurrency Control

### Optimistic Locking

**Approach**: Assume no conflicts, check for them at commit time.

```python
# Optimistic Locking: Check version before update
def update_account_balance(account_id, delta, current_version):
    # Step 1: Get current state
    account = db.query("SELECT version, balance FROM accounts WHERE id = ?", account_id)
    
    # Step 2: Verify version hasn't changed
    if account['version'] != current_version:
        raise Exception("Concurrent modification detected")
    
    # Step 3: Update and increment version
    db.execute("""
        UPDATE accounts
        SET balance = balance + ?, version = version + 1
        WHERE id = ?
    """, (delta, account_id))
```

**Pros**:
- No locks (non-blocking)
- Good performance
- Works well in low-contention environments

**Cons**:
- Failed transactions require retry logic
- More complex than pessimistic locking

### Pessimistic Locking

**Approach**: Lock resources before modifying them.

```sql
-- Pessimistic Locking: Lock row before update
BEGIN;
-- Lock the account
SELECT balance FROM accounts WHERE id = 101 FOR UPDATE;
-- Now safe to update
UPDATE accounts SET balance = balance - 100 WHERE id = 101;
COMMIT;
```

**Pros**:
- Simple to implement
- Prevents conflicts
- Good for high-contention environments

**Cons**:
- Locks block other transactions
- Potential for deadlocks
- Reduced concurrency

---

## Replication Strategies

### Statement-Based Replication

**How It Works**: SQL statements executed on master are sent to slaves.

**Example**:
```sql
-- On master
CREATE TABLE orders (id INT, amount DECIMAL);
INSERT INTO orders VALUES (1, 100.00);

-- On slaves (replay)
CREATE TABLE orders (id INT, amount DECIMAL);
INSERT INTO orders VALUES (1, 100.00);
```

**Pros**:
- Simple to implement
- Low latency on slaves

**Cons**:
- Slaves can serve stale data during replication lag
- No failover (slaves are read-only)

### Row-Based Replication

**How It Works**: Binary log of row changes sent to slaves.

**Pros**:
- Lower network traffic
- More efficient than statement-based
- Supports point-in-time recovery

**Cons**:
- Requires binlog
- More complex

---

## Database Sharding

### Horizontal Scaling

**Sharding**: Split data across multiple database instances.

### Sharding Key Strategies

| Strategy | Description | Example |
|-----------|-------------|----------|
| **Hash-based** | Consistent routing | shard = hash(user_id) % num_shards |
| **Range-based** | Efficient range queries | shard = (user_id >= 0 AND user_id < 1000) |
| **Directory-based** | Geographically localized | shard = region (us-east-1, us-west-2) |

### Example: Hash Sharding

```sql
-- Create users table with shard key
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    shard_key INT,  -- user_id % 4
    name VARCHAR(100),
    email VARCHAR(255)
);

-- Query specific shard
SELECT * FROM users WHERE shard_key = 1;
```

### Sharding Challenges

- **Cross-shard queries**: Queries spanning multiple shards
- **Rebalancing**: Moving data between shards
- **Join complexity**: Joins across shards are expensive
- **Transactions**: Cross-shard transactions require distributed protocols

---

## Connection Pooling

**Why Needed**: Creating database connections is expensive. Reuse connections.

### Python Example

```python
import psycopg2.pool
from contextlib import contextmanager

# Create connection pool
pool = psycopg2.pool.SimpleConnectionPool(
    minconn=1,
    maxconn=10,
    host='localhost',
    database='mydb',
    user='user',
    password='password'
)

@contextmanager
def get_db_connection():
    conn = pool.getconn()
    try:
        yield conn
    finally:
        pool.putconn(conn)

# Usage
with get_db_connection() as conn:
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users")
    results = cursor.fetchall()
```

---

## Interview Questions

**Q1: What is ACID?**

**A1**: ACID stands for Atomicity, Consistency, Isolation, Durability. It's a set of properties that guarantee database transactions are processed reliably:
- Atomicity: All-or-nothing
- Consistency: Valid state transitions
- Isolation: Concurrent transactions don't interfere
- Durability: Committed transactions survive failures

**Q2: What is the CAP theorem?**

**A2**: CAP states that in a distributed system, you can only simultaneously have two of Consistency, Availability, and Partition Tolerance. All distributed systems must choose which two properties to optimize for their use case.

**Q3: What's the difference between optimistic and pessimistic locking?**

**A3**:
- Optimistic Locking: Assume no conflicts, check for them at commit time. No locks, better performance in low-contention. Fails require retry.
- Pessimistic Locking: Lock resources before modifying. Prevents conflicts, but locks block other transactions. Better for high-contention.

**Q4: When would you use BASE instead of ACID?**

**A4**: Use BASE (Basically Available, Soft State, Eventual Consistency, Yes) instead of ACID when availability is more important than strong consistency. Examples: caching layers, social media feeds, e-commerce product catalogs where some stale data is acceptable.

**Q5: What is the Saga pattern?**

**A5**: The Saga pattern is a way to manage distributed transactions by breaking a large transaction into a sequence of smaller local transactions (sagas) with compensating actions. If any sub-transaction fails, compensating actions undo the effects of completed transactions. Used in microservices where 2PC blocking is undesirable.

**Q6: What's 2PC and what problem does it solve?**

**A6**: Two-Phase Commit (2PC) ensures atomicity across multiple distributed resources (different databases or services). It solves the problem of maintaining consistency in a distributed system by having a coordinator that manages the commit process across all participants. It guarantees that either all participants commit or all rollback, ensuring atomicity.

---

## Related Notes

- [[Data Warehousing]] - Database transactions in warehouse context
- [[System Design]] - Distributed systems, CAP theorem
- [[SQL Interview Questions]] - Transaction queries and optimization

---

## Resources

- [CAP vs ACID: What's the difference? - Budibase](https://budibase.com/blog/cap-vs-acid-whats-the-difference/)
- [ACID CAP Theorem in Data Engineering - The Hidden Trade-offs - Medium](https://medium.com/@montypoddar08/cap-theorem-in-data-engineering-the-hidden-trade-offs-d0bbef35fe6)
- [The CAP Theorem in DBMS - GeeksforGeeks](https://www.geeksforgeeks.org/dbms/the-cap-theorem-in-dbms/)
- [Database Transaction Properties - Microsoft Docs](https://docs.microsoft.com/en-us/previous-versions/sql/sql-server-2014/database-engine-and-features-transactions/database-properties)

---

**Progress**: 🟢 Learning (concepts understood, need implementation practice)

**Next Steps**:
- [ ] Practice isolation levels with SQL transactions
- [ ] Implement optimistic locking in Python
- [ ] Design sharding strategy for large dataset
- [ ] Learn distributed transaction patterns (2PC, Saga)
- [ ] Practice concurrency control in application code

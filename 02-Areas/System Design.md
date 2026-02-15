---
tags: [system-design, scalability, cap, caching, load-balancing, distributed, microservices]
date: 2026-02-15
status: learning
---

# System Design: Scalability & Distributed Systems

## Overview

System design is the process of defining the architecture, components, modules, interfaces, and data for a system to satisfy specified requirements.

**Key Goal**: Build systems that scale to handle growth in users, data, and traffic.

---

## Scalability Strategies

### Vertical Scaling (Scale Up)

**Definition**: Add more power (CPU, RAM) to a single instance.

| Metric | Description | Pros | Cons |
|---------|-------------|------|-------|
| **CPU** | Add cores | Simple | Limited by architecture |
| **RAM** | Add memory | Easy | Maximum memory limit |
| **Storage** | Add disk space | Straightforward | Single point of failure |
| **Network** | Upgrade bandwidth | Fast | Expensive hardware |

**Use Case**:
- Monolithic application
- Single-node database
- Read-heavy workload
- Scale: 100x increase needs 100x more powerful machine

**Example**:
```
Initial: 1 server, 16GB RAM, 8 CPU
Load: 10,000 users/s
Final: 1 server, 128GB RAM, 32 CPU, 10x upgrade cost
```

### Horizontal Scaling (Scale Out)

**Definition**: Add more instances/servers.

| Metric | Description | Pros | Cons |
|---------|-------------|------|-------|
| **Nodes** | Add servers | Unlimited scaling | Complex coordination |
| **Connections** | Increase connections | Higher throughput | Network overhead |
| **Storage** | Distributed storage | Elastic | Data consistency |
| **Requests** | Load balancer distributes | High availability | Requires state management |

**Use Case**:
- Stateless applications (web servers)
- Microservices
- Distributed cache
- Message queues
- Scale: 10x increase needs 10x more servers

**Example**:
```
Initial: 1 server, 1,000 requests/s
Load: 10,000 users/s
Architecture:
┌─────────────────────────────────┐
│  Load Balancer              │
└──────────────┬──────────────┘
               │
    ┌──────────┴──────────┐
    │  Server 1 Server 2 Server 3 │
    │  (1/3 traffic) (1/3 traffic) (1/3 traffic) │
    └─────────────────────────────┘
Final: 10 servers, 10,000 requests/s capacity
```

---

## Load Balancing

### Load Balancer Types

| Type | Algorithm | Use Case | Example |
|-------|-----------|----------|----------|
| **Round Robin** | Rotate requests evenly | Same-size servers |
| **Least Connections** | Route to server with fewest active connections | Long-running connections |
| **IP Hash** | Route by client IP (sticky) | Session persistence |
| **Random** | Distribute randomly | Stateless services |
| **Weighted** | Route by server capacity | Heterogeneous servers |
| **Consistent Hash** | Same client always hits same server | Session + cache coherency |

### Python Example: Weighted Round Robin

```python
import random
import time
from typing import List, Dict

class WeightedServer:
    def __init__(self, host: str, port: int, weight: int):
        self.host = host
        self.port = port
        self.weight = weight
        self.active_connections = 0

class WeightedRoundRobin:
    def __init__(self, servers: List[WeightedServer]):
        self.servers = servers
        self.current_index = 0
        self.total_weight = sum(s.weight for s in servers)

    def get_server(self) -> WeightedServer:
        # Select server based on weight (more weight = more requests)
        total = self.total_weight
        index = self.current_index
        target = self.servers[index].weight

        # Find next index using weighted round robin
        while target < random.random() * total:
            index = (index + 1) % len(self.servers)
            target += self.servers[index].weight

        self.current_index = index
        return self.servers[index]

# Usage
servers = [
    WeightedServer("server1", 8080, 3),  # Weight = CPU cores
    WeightedServer("server2", 8080, 1),
    WeightedServer("server3", 8080, 2)
]

lb = WeightedRoundRobin(servers)

# Simulate 100 requests
for i in range(100):
    server = lb.get_server()
    server.active_connections += 1
    print(f"Request {i+1}: {server.host}:{server.port} (weight: {server.weight})")
```

### SQL Example: Load Balancer Decision

```sql
-- Simulate load balancer routing decisions
WITH servers AS (
    SELECT 'server1' AS host, 8080 AS port, 3 AS weight
    UNION ALL SELECT 'server2', 8080, 3, 1
    UNION ALL SELECT 'server3', 8080, 2
),
requests AS (
    SELECT generate_series(1, 100) AS request_id
)

-- Least Connections: Route to server with fewest active connections
SELECT
    r.request_id,
    s.host,
    s.port,
    'least_connections' AS strategy
FROM requests r
CROSS JOIN LATERAL (
    SELECT s.host, s.port, COUNT(*) OVER () AS total,
           ROW_NUMBER() OVER (ORDER BY COUNT(*) ASC) AS rank
    FROM server_log sl
    WHERE sl.event_time > NOW() - INTERVAL '1 minute'
) s
WHERE s.rank = 1
ORDER BY r.request_id
LIMIT 10;

-- Weighted Round Robin: Route based on server weight (CPU cores)
SELECT
    r.request_id,
    s.host,
    s.port,
    'weighted_round_robin' AS strategy,
    s.weight AS server_weight
FROM requests r
CROSS JOIN servers s
WHERE s.port = 8080
ORDER BY r.request_id
LIMIT 10;
```

---

## Caching Strategies

### Cache Types

| Type | Location | TTL | Eviction Policy | Use Case |
|------|----------|-----|----------------|----------|
| **Browser Cache** | Client browser | Session | LRU | Static assets |
| **CDN Cache** | Edge servers | Hours | LRU | Global content delivery |
| **Application Cache** | Application server | Minutes | LRU/TTL | Database query results |
| **Distributed Cache** | Separate cluster | Seconds/Minutes | LRU/Random | Shared sessions |
| **Database Cache** | Database | Variable | Write-through | Query acceleration |

### Python: Redis Cache Example

```python
import redis
import json
from datetime import timedelta

class CacheService:
    def __init__(self, redis_host='localhost', redis_port=6379):
        self.redis = redis.StrictRedis(
            host=redis_host,
            port=redis_port,
            decode_responses=True
        )

    def get_user_profile(self, user_id: int) -> dict:
        # Try cache first
        cache_key = f"user_profile:{user_id}"
        cached = self.redis.get(cache_key)
        if cached:
            return json.loads(cached)

        # Cache miss - fetch from database
        # Simulate database call
        profile = self._fetch_from_db(user_id)

        # Write to cache with 1 hour TTL
        self.redis.setex(
            cache_key,
            json.dumps(profile),
            timedelta(hours=1)
        )

        return profile

    def invalidate_user_profile(self, user_id: int) -> None:
        """Remove from cache (e.g., after update)"""
        cache_key = f"user_profile:{user_id}"
        self.redis.delete(cache_key)

    def _fetch_from_db(self, user_id: int) -> dict:
        # Simulate database query
        return {
            'user_id': user_id,
            'name': f'User {user_id}',
            'email': f'user{user_id}@example.com',
            'preferences': {'theme': 'dark', 'language': 'en'}
        }

# Usage
cache = CacheService()

# First request (cache miss)
profile1 = cache.get_user_profile(101)
print(f"Request 1: {'cache' if profile1 else 'db'}")

# Second request (cache hit)
profile2 = cache.get_user_profile(101)
print(f"Request 2: {'cache' if profile2 else 'db'}")

# Update user (invalidate cache)
cache.invalidate_user_profile(101)

# Third request (cache miss again)
profile3 = cache.get_user_profile(101)
print(f"Request 3: {'cache' if profile3 else 'db'}")
```

### SQL: Cache-Aside Pattern

```sql
-- Cache-Aside: Check cache, then database
-- Step 1: Check cache (simulated as CTE)
WITH cache_check AS (
    SELECT
        cache_key,
        cached_data,
        EXTRACT(EPOCH FROM cached_at) AS age_seconds
    FROM cache_store
    WHERE cache_key = 'user_profile:101'
        AND cached_at > NOW() - INTERVAL '1 hour'
),
-- Step 2: Fetch from DB if cache miss
fresh_data AS (
    SELECT
        user_id,
        name,
        email,
        preferences
    FROM users
    WHERE user_id = 101
)
-- Step 3: Return cached or fresh data
SELECT
    COALESCE(cache_check.cached_data, fresh_data.*) AS profile_data,
    'cache' AS source
FROM cache_check
FULL OUTER JOIN fresh_data ON TRUE
LIMIT 1;

-- Update cache after fresh fetch
INSERT INTO cache_store (cache_key, cached_data, cached_at)
SELECT 'user_profile:101',
       json_build_object('user_id', profile_data.user_id, 'name', profile_data.name),
       NOW()
WHERE NOT EXISTS (
    SELECT 1 FROM cache_check WHERE cached_data IS NOT NULL
);
```

---

## CAP Theorem (Detailed)

### CAP in Real-World Systems

| System | Choice | Trade-off |
|---------|---------|-----------|
| **Traditional RDBMS** | CA (Strong Consistency) | Single master, no partition tolerance |
| **Distributed SQL** | CP (Partition Tolerance) | Synchronous replication, eventual consistency |
| **NoSQL (Mongo)** | AP (High Availability) | Async replication, eventual consistency |
| **Cassandra** | AP (High Availability) | Tunable consistency (quorum) |
| **DynamoDB** | AP (High Availability) | Configurable consistency level |

### System Design Example: E-Commerce System

```
                    ┌─────────────────────────────────┐
                    │  Load Balancer      │
                    └──────────────┬──────────────┘
                                   │
                    ┌──────────────┴───────────┐
                    │                            │
            ┌─────────────┐       ┌─────────────┐      ┌─────────────┐
            │  Web Cluster│       │  App Cluster │      │  DB Cluster │
            │  CA (Strong │       │  CP (Partition│      │  AP (High   │
            │   Consistency)│       │   Tolerance)  │      │  Availability)│
            └─────────────┘       └─────────────┘      └─────────────┘
                    │                            │
                    └──────────────┬──────────────┘
                                   │
                    ┌──────────────────────────────────┐
                    │  Distributed Cache (Redis)       │
                    │  CP (Partition Tolerance)        │
                    └──────────────────────────────────┘
```

**CAP Choices**:
- **Web Cluster**: AP (Availability over consistency - cached sessions OK)
- **App Cluster**: CP (Partition Tolerance - eventual consistency acceptable)
- **DB Cluster**: CA (Strong Consistency - transactions critical)

---

## Microservices vs Monolith

### Comparison

| Aspect | Monolith | Microservices |
|---------|-----------|---------------|
| **Deployment** | One unit | Many independent services |
| **Scalability** | Vertical (scale up) | Horizontal (scale out) |
| **Complexity** | Simpler codebase | Complex distributed system |
| **Communication** | In-memory function calls | Network calls (slower) |
| **Data Consistency** | Single ACID database | Eventually consistent |
| **Failure Impact** | System down = all down | Service down = partial degradation |

### Python: Microservices Communication (REST)

```python
import requests
from typing import Dict, Any
from datetime import datetime

class UserClient:
    def __init__(self, base_url: str = "http://localhost:8080"):
        self.base_url = base_url

    def get_user(self, user_id: int) -> Dict[str, Any]:
        # Call User Service
        response = requests.get(f"{self.base_url}/users/{user_id}")
        return response.json()

    def create_order(self, user_id: int, items: list) -> Dict[str, Any]:
        # Call Order Service
        response = requests.post(
            f"{self.base_url}/orders",
            json={"user_id": user_id, "items": items}
        )
        return response.json()

class OrderClient:
    def __init__(self, base_url: str = "http://localhost:8081"):
        self.base_url = base_url

    def get_order(self, order_id: int) -> Dict[str, Any]:
        # Call Order Service
        response = requests.get(f"{self.base_url}/orders/{order_id}")
        return response.json()

# Microservices Orchestration (Simplified Saga)
class OrderSaga:
    def __init__(self, user_client: UserClient, order_client: OrderClient):
        self.user_client = user_client
        self.order_client = order_client

    def create_order_flow(self, user_id: int, items: list):
        """Saga pattern: Execute order flow with compensating actions"""
        try:
            # Step 1: Create order (Order Service)
            order_response = self.order_client.create_order(user_id, items)
            order_id = order_response['order_id']
            print(f"✅ Order created: {order_id}")

            # Step 2: Reserve inventory (Inventory Service)
            # Simulate inventory call
            inventory_response = self._reserve_inventory(items)
            print(f"✅ Inventory reserved: {len(items)} items")

            # Step 3: Process payment (Payment Service)
            payment_response = self._process_payment(user_id, items)
            print(f"✅ Payment processed: ${payment_response['amount']}")

            # Step 4: Ship order (Order Service)
            ship_response = self.order_client.ship_order(order_id)
            print(f"✅ Order shipped: {ship_response['tracking_number']}")

            return {"status": "completed", "order_id": order_id}

        except Exception as e:
            print(f"❌ Saga failed: {str(e)}")
            # Compensating actions (rollback)
            self._compensate_failure(user_id, items, e)
            return {"status": "failed", "error": str(e)}

    def _reserve_inventory(self, items: list) -> Dict[str, Any]:
        # Simulate inventory service call
        return {"status": "reserved", "items": items}

    def _process_payment(self, user_id: int, items: list) -> Dict[str, Any]:
        # Simulate payment service call
        total = sum(item['price'] for item in items)
        return {"status": "success", "amount": total}

    def _compensate_failure(self, user_id: int, items: list, error: Exception):
        # Compensating actions for saga pattern
        print(f"⚠️  Compensating: Cancel order, release inventory")
        # In real implementation:
        # - Cancel order (Order Service)
        # - Release inventory (Inventory Service)
        # - Refund payment (Payment Service)

# Usage
saga = OrderSaga()

# Execute order flow
result = saga.create_order_flow(
    user_id=101,
    items=[
        {"product_id": 1, "price": 99.99, "quantity": 1},
        {"product_id": 2, "price": 49.99, "quantity": 2}
    ]
)

print(f"Result: {result}")
```

---

## Database Sharding (Detailed)

### Python: Hash Sharding Client

```python
import hashlib

class ShardManager:
    def __init__(self, num_shards: int = 4):
        self.num_shards = num_shards
        self.shard_map = {
            'shard_0': 'db-0.example.com',
            'shard_1': 'db-1.example.com',
            'shard_2': 'db-2.example.com',
            'shard_3': 'db-3.example.com'
        }

    def get_shard(self, key: str) -> str:
        """Consistent hash sharding - same key always hits same shard"""
        # Hash the key
        hash_value = int(hashlib.md5(key.encode()).hexdigest(), 16)
        shard_index = hash_value % self.num_shards
        return f"shard_{shard_index}"

    def route_query(self, query: str, params: dict) -> dict:
        """Route query to correct shard"""
        shard_key = self.get_shard(query.get('user_id', ''))
        db_host = self.shard_map[shard_key]
        # In real implementation, execute query on db_host
        return {
            'shard': shard_key,
            'db_host': db_host,
            'query': query,
            'params': params
        }

# Usage
shard_manager = ShardManager(num_shards=4)

# Route different user queries
query1 = shard_manager.route_query(
    "SELECT * FROM users WHERE user_id = 101",
    {'limit': 10}
)
print(f"User 101 query routed to: {query1['shard']} ({query1['db_host']})")

query2 = shard_manager.route_query(
    "SELECT * FROM users WHERE user_id = 102",
    {'limit': 10}
)
print(f"User 102 query routed to: {query2['shard']} ({query2['db_host']})")

query3 = shard_manager.route_query(
    "SELECT * FROM users WHERE user_id = 101",  # Same user - same shard
    {'limit': 10}
)
print(f"User 101 query routed to: {query3['shard']} ({query3['db_host']})")
```

### SQL: Distributed Query (Cross-Shard Join)

```sql
-- Query data across multiple shards using UNION ALL
-- Each shard is a separate database/table
SELECT * FROM shard_0.users WHERE user_id = 101
UNION ALL
SELECT * FROM shard_1.users WHERE user_id = 101
UNION ALL
SELECT * FROM shard_2.users WHERE user_id = 101
UNION ALL
SELECT * FROM shard_3.users WHERE user_id = 101;

-- Alternative: Use application-side sharding (query specific shard)
-- Better performance, requires application-level routing
```

---

## Performance Optimization

### Database Indexing Strategy

```sql
-- Single-column index (fast for equality queries)
CREATE INDEX idx_users_email ON users(email);

-- Composite index (fast for queries with email + status)
CREATE INDEX idx_users_email_status ON users(email, status);

-- Covering index (supports queries not using index)
CREATE INDEX idx_users_status_created ON users(status, created_at) WHERE status = 'active';

-- Partial index (index prefix only - smaller, faster)
CREATE INDEX idx_users_email_partial ON users(email(20));
```

### Query Optimization Patterns

```sql
-- IN list optimization (UNION ALL vs IN)
SELECT * FROM users WHERE user_id IN (101, 102, 103, 104);

-- EXISTS vs JOIN for existence check
SELECT u.name FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.user_id);

-- Pagination for large result sets (keyset pagination)
-- Avoid OFFSET (slow)
SELECT * FROM users
WHERE user_id > 101 AND user_id <= 200
ORDER BY user_id
LIMIT 100;
```

---

## Interview Questions

**Q1: How would you design a system that handles 1M users?**

**A1**: Use horizontal scaling with load balancers.
- Web servers: Stateless, easy to scale out
- Application servers: Microservices architecture
- Database: Sharded (horizontal partitioning)
- Cache: Distributed Redis cluster for session data
- Trade-off: CP system (partition tolerance) with eventual consistency

**Q2: What's the difference between vertical and horizontal scaling?**

**A2**:
- **Vertical**: Add resources to single machine. Limited by hardware max.
- **Horizontal**: Add more machines. Unlimited scaling potential.
- **Hybrid**: Start with vertical for simplicity, move to horizontal for scale.

**Q3: How does CAP theorem affect system design?**

**A3**: You can only pick 2 of 3 properties (Consistency, Availability, Partition Tolerance).
- For distributed systems (microservices, NoSQL), AP is often chosen (High Availability + Partition Tolerance) accepting eventual consistency.
- For traditional databases, CA (Strong Consistency) with limited partition tolerance.
- System design must match use case requirements (financial systems need strong consistency, social media can tolerate eventual consistency).

**Q4: How would you handle database sharding?**

**A4**:
- **Consistent Hash Sharding**: Same key always goes to same shard (good for range queries, bad for hot keys)
- **Range Sharding**: Keys distributed in ranges. Good for sequential queries, can cause hotspots
- **Directory Sharding**: Shards map to specific servers. Good for rebalancing, complex routing
- **Geographic Sharding**: Users routed to nearest data center (low latency)
- I would choose consistent hash or directory sharding for most use cases.

**Q5: What's read-your-writes?**

**A5**: Consistency model where writes are immediately visible to reads. Strong consistency (CA system). Trade-off: Higher write latency due to synchronous replication.

---

## Related Notes

- [[Database Fundamentals]] - ACID, transactions, isolation levels
- [[Data Warehousing]] - Distributed database systems
- [[Slowly Changing Dimensions SCDs]] - Handling updates in warehouses
- [[Apache Flink - Real-Time Analytics]] - Stateful stream processing
- [[Apache Kafka - Event Streaming]] - Distributed messaging

---

## Resources

- [System Design Primer](https://github.com/donnemartin/system-design-primer)
- [Designing Data-Intensive Applications](https://www.cs.illinois.edu/~dingm/notes.pdf)
- [CAP Theorem Explained](https://www.ibm.com/topics/cap-theorem-consistency-availability-partition-tolerance/)
- [Scalability Rules](https://highscalability.com/)

---

**Progress**: 🟢 Learning (concepts understood, need practice)

**Next Steps**:
- [ ] Practice load balancer implementation (Python/Go)
- [ ] Implement distributed caching (Redis/Memcached)
- [ ] Design sharding strategy for large dataset
- [ ] Practice CAP trade-offs in real systems
- [ ] Build microservices with Saga pattern
- [ ] Optimize database queries with proper indexing

# Python Interview Questions

> Core Python concepts and data engineering-specific Python questions

## Python Language Questions

### Q1: What is a descriptor in Python?

**Answer**: A descriptor is a particular Python object that allows you to define how an attribute is accessed or modified in a class. Descriptors are commonly used to define properties, enabling you to control how attributes are accessed and modified.

---

### Q2: How do you handle circular imports in Python?

**Answer**: Circular imports occur when two or more modules import each other. To handle circular imports in Python, you can use several techniques:

- Import the module at the function level
- Use a separate module for the shared code
- Use a lazy import

---

### Q3: What are the differences between a shallow copy and a deep copy in Python?

**Answer**:

- **Shallow copy**: Creates a new object that points to the original object's memory location. Valid when creating a new object that refers to the same data as the original object.
- **Deep copy**: Constructs a new object with its memory that contains a copy of the original object's data. Applicable when creating a completely separate copy of the original data.

---

### Q4: What is the difference between a list comprehension and a generator expression in Python?

**Answer**:

- **List comprehension**: A shorthand syntax for creating a list from an iterable. Creates a list in memory.
- **Generator expression**: A shorthand syntax for creating a generator from an iterable. Creates a generator object that generates values on the fly.

---

### Q5: What is the difference between bound and unbound methods in Python?

**Answer**:

- **Bound method**: A method that is bound to an instance of a class. Can access the instance's attributes and methods.
- **Unbound method**: A method that is not bound to an instance of a class. Cannot access instance attributes or methods.

---

## Data Engineering Python Questions

### Q1: What is lambda architecture, and how does it work?

**Answer**: A lambda architecture is a data processing architecture that combines batch processing and stream processing to provide both real-time and historical views of data. It processes incoming data in parallel using batch processing and stream processing systems and merges the results to give a complete data view.

---

### Q2: What are the differences between a data lake and a data warehouse?

**Answer**:

- **Data lake**: A large, centralized repository for storing raw data from multiple sources. Typically stores large volumes of unstructured or semi-structured data.
- **Data warehouse**: A repository for storing structured and processed data that has been transformed and cleaned for analysis. Optimized for analytics.

---

### Q3: How do you optimize a database for write-heavy workloads?

**Answer**: Optimizing a database for write-heavy workloads typically involves:

- **Sharding**: Distributing data across multiple nodes
- **Partitioning**: Splitting data into manageable chunks
- **Clustering**: Grouping related data together
- **Indexing**: Using indexes strategically
- **Caching**: Reducing write operations
- **Batching**: Grouping write operations together

---

### Q4: What is the difference between a batch processing system and a stream processing system?

**Answer**:

- **Batch processing system**: Processes data in discrete batches. Typically used for analyzing historical data.
- **Stream processing system**: Processes data in real time as it arrives. Used for processing real-time data.

---

### Q5: What are the challenges of working with distributed systems, and how can they be addressed?

**Answer**: Challenges include:

- **Network latency**: Can be addressed through data replication and caching
- **Data consistency**: Can be managed through consensus protocols
- **Fault tolerance**: Can be improved through redundancy and partitioning
- **Load balancing**: Distributes workload across multiple nodes

---

## General Technical Concepts

### Q1: What is the difference between a mutex and a semaphore?

**Answer**:

- **Mutex**: A locking mechanism that ensures only one thread can access a shared resource at a time. Typically used to protect critical sections of code.
- **Semaphore**: A signaling mechanism that allows multiple threads to access a shared resource concurrently up to a specified limit. Used to control access to shared resources.

---

### Q2: What is the CAP theorem, and how does it relate to distributed systems?

**Answer**: According to the CAP theorem, a distributed system cannot concurrently offer the three guarantees of:

- **Consistency**: All nodes concurrently see the same data
- **Availability**: The system keeps working even if some nodes fail
- **Partition tolerance**: The system can still function in the event of a network partition

You can only achieve two of these three properties simultaneously.

---

### Q3: What is the difference between monolithic and microservices architecture?

**Answer**:

- **Monolithic architecture**: All components of an application are tightly coupled and deployed as a single unit.
- **Microservices architecture**: An application is broken down into a collection of small, loosely coupled services that can be developed and deployed independently.

Microservices offer greater flexibility and scalability but require more complex infrastructure.

---

### Q4: What is the difference between ACID and BASE consistency models?

**Answer**:

- **ACID** (Atomicity, Consistency, Isolation, Durability): Provides strong guarantees around data consistency and integrity.
- **BASE** (Basically Available, Soft state, Eventually consistent): Sacrifices strong consistency in favor of availability and partition tolerance. Used in distributed systems where strong consistency is difficult to achieve.

---

### Q5: What are the different types of joins in SQL, and how do they work?

**Answer**:

- **INNER JOIN**: Returns rows when there is a match in both tables
- **LEFT OUTER JOIN**: Returns all rows from the left table and matching rows from the right table
- **RIGHT OUTER JOIN**: Returns all rows from the right table and matching rows from the left table
- **FULL OUTER JOIN**: Returns all rows from both tables with null values where no match is found

---

## Related Topics

- [[02-Areas/Python/Advanced Python]]
- [[05-Interview Prep/SQL Interview Questions]]
- [[05-Interview Prep/Data Modeling]]

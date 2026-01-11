# Database Fundamentals Interview Questions

> Essential database concepts for data engineers

## Database Types

### Q1: What is a primary key, and why is it important in a relational database?

**Answer**: A primary key is the unique identifier for each row in a table. It:

- Cannot be null or non-unique, which enforces data integrity
- Ensures no two rows are the same
- Helps establish referential integrity so that relationships can be established between tables
- Provides a unique way to reference each record

---

### Q2: What is a foreign key?

**Answer**: A foreign key is a column that references the primary key of another table. This:

- Establishes relationships between tables
- Ensures referential integrity
- Can contain null values (unlike primary keys)
- Links related data across tables

---

### Q3: What is the difference between a relational and a non-relational database?

**Answer**:

| Aspect | Relational Database | Non-Relational Database |
|--------|---------------------|-------------------------|
| **Schema** | Based on a relational model or schema; data organized into tables (rows and columns) | No schema; data can be stored without a defined relationship |
| **Data Type** | Best for structured data | Best for semi-structured and unstructured data |
| **Examples** | PostgreSQL, MySQL, Oracle | MongoDB, Cassandra, Redis |
| **ACID** | ACID compliant | Typically not ACID compliant |
| **Querying** | SQL | Various query languages or APIs |

---

### Q4: What is database normalization, and why is it important?

**Answer**: Normalization is the process of minimizing redundancy and dependencies by enforcing five normal forms. It:

- Helps establish relationships between tables
- Maintains data integrity
- Ensures data is stored consistently
- Eliminates data duplication
- Reduces anomalies during insert, update, and delete operations

**The Normal Forms**:

1. **First Normal Form (1NF)**: Eliminate repeating groups
   - Each table has a primary key
   - All columns contain atomic or indivisible values

2. **Second Normal Form (2NF)**: Eliminate redundant data
   - All non-primary or foreign key columns are completely dependent on the primary key

3. **Third Normal Form (3NF)**: Eliminate columns not dependent on a key
   - All columns are dependent only on the primary key

4. **Boyce-Codd Normal Form (BCNF)**: A stricter version of 3NF
   - For every dependency, the left side must be a superkey

5. **Fourth Normal Form (4NF)**: Eliminate multi-valued dependencies
   - No multi-valued dependencies exist

---

## ACID Properties

### Q5: What are the ACID properties of a database transaction?

**Answer**:

- **Atomicity**: A transaction must either be completed in its entirety or not executed at all. Ensures that partial transactions are not committed.
- **Consistency**: A transaction will follow all defined business rules and database constraints.
- **Isolation**: Transactions are completely independent from one another.
- **Durability**: All data is permanent after a transaction is committed, even in the case of system failures.

---

## CAP Theorem

### Q6: What is the CAP theorem?

**Answer**: According to the CAP theorem, a distributed system cannot concurrently offer all three guarantees:

1. **Consistency**: All nodes concurrently see the same data
2. **Availability**: The system keeps working even if some nodes fail
3. **Partition tolerance**: The system can still function in the event of a network partition

You can only achieve two of these three properties simultaneously. Understanding the CAP theorem is essential for data engineers making architectural decisions.

**CAP Trade-offs**:
- **CA**: Consistency and Availability (without partition tolerance)
- **CP**: Consistency and Partition tolerance (sacrifices availability)
- **AP**: Availability and Partition tolerance (sacrifices consistency)

---

## OLTP vs OLAP

### Q7: What is the difference between OLTP and OLAP databases?

**Answer**:

| Aspect | OLTP (Online Transaction Processing) | OLAP (Online Analytical Processing) |
|--------|--------------------------------------|-------------------------------------|
| **Purpose** | Support day-to-day business transactions | Support complex data analysis |
| **Focus** | High-speed data entry, updates, and retrieval | Complex queries and calculations |
| **Data** | Real-time transactional data | Historical data |
| **Optimization** | Optimized for simple queries | Optimized for complex queries |
| **ACID** | ACID compliant | May not be ACID compliant |
| **Use Cases** | Updating customer records, inventory management, processing orders | Creating forecasts, reports, BI activities |

---

## Database Triggers

### Q8: What are database triggers?

**Answer**: Triggers are sets of code that get executed when an action occurs. They are normally run after data is manipulated or modified in the database.

**Three types of SQL triggers**:

1. **DML Triggers**: Executed when data is inserted, updated, or deleted from a table
2. **DDL Triggers**: Executed when the structure of a table is modified (addition or removal of a column)
3. **Statement Triggers**: Executed when certain SQL statements are run in a query

**Uses**:
- Enforcing predefined business rules
- Updating data
- Running data quality checks
- Automating tasks
- Executing stored procedures

**Note**: Because triggers can impact database performance, they should be used sparingly and not for complex actions.

---

## Indexing

### Q9: What is database indexing, and why is it important?

**Answer**: Indexing is a data structure technique used to quickly locate and access the data in a database table.

**Benefits**:
- Speeds up data retrieval
- Improves query performance
- Reduces disk I/O operations

**Trade-offs**:
- Increases storage requirements
- Slows down INSERT, UPDATE, and DELETE operations
- Requires maintenance

**Index Types**:
- **Primary Index**: On the primary key
- **Unique Index**: Ensures column values are unique
- **Composite Index**: On multiple columns
- **Clustered Index**: Determines physical order of data

---

## Constraints

### Q10: What are database constraints?

**Answer**: Constraints are rules enforced on data columns to ensure data accuracy and reliability.

**Types of Constraints**:

1. **NOT NULL**: Ensures column cannot have NULL values
2. **UNIQUE**: Ensures all values in a column are different
3. **PRIMARY KEY**: Uniquely identifies each record
4. **FOREIGN KEY**: Ensures referential integrity between tables
5. **CHECK**: Ensures all values in a column satisfy a specific condition
6. **DEFAULT**: Sets a default value for a column when no value is specified
7. **INDEX**: Used to create and retrieve data from the database quickly

---

## Additional Key Concepts

### NoSQL Databases

**Advantages**:
- **Flexibility**: Can store data in multiple formats without a predefined schema
- **Performance**: Faster performance with large amounts of data
- **Scalability**: Horizontal scaling capabilities

**Disadvantages**:
- Not ACID compliant (unsuitable for transaction data)
- Not suitable for advanced querying
- Limited support for complex joins

**Use Cases**:
- Real-time data processing
- Content management systems (CMS)
- Search applications
- IoT data storage

---

## Related Topics

- [[02-Areas/Data Modeling]]
- [[05-Interview Prep/SQL Interview Questions]]
- [[05-Interview Prep/System Design Framework]]

==============================
FILE: Databases (DBMS)
==============================

### HIGH PRIORITY

---

Q1. What is normalization? Explain 1NF, 2NF, 3NF with examples.

A1.
Normalization reduces data redundancy and prevents anomalies (insert, update, delete anomalies) by organizing data into well-structured tables.

**1NF**: Every column holds atomic values — no lists, no repeating groups. If a student has multiple phone numbers, don't store them as "123, 456" in one cell. Create a separate row or a related table.

**2NF**: Must be in 1NF, and every non-key column must depend on the entire primary key, not just part of it. If your composite key is (StudentID, CourseID) and `StudentName` depends only on `StudentID`, move `StudentName` to a separate Students table.

**3NF**: Must be in 2NF, and no non-key column should depend on another non-key column (no transitive dependencies). If you have (EmployeeID, DepartmentID, DepartmentName), the department name depends on DepartmentID, not EmployeeID. Move department info to its own table.

In practice, most production schemas are in 3NF. Beyond that (BCNF, 4NF) you're usually over-engineering unless you have very specific data integrity requirements.

---

Q2. What is the difference between SQL and NoSQL databases? When would you choose one over the other?

A2.
**SQL (relational)**: Structured schema, tables with rows and columns, ACID transactions, SQL query language. PostgreSQL, MySQL. Best when data is structured, relationships are important, and you need strong consistency — financial systems, e-commerce, anything transactional.

**NoSQL**: Flexible/dynamic schemas, various data models — document (MongoDB), key-value (Redis), column-family (Cassandra), graph (Neo4j). Optimized for specific access patterns.

Choose NoSQL when: your data is semi-structured or schema evolves rapidly, you need horizontal scalability at massive scale, your access pattern is simple key-based lookups, or your data is naturally hierarchical (documents nested within documents).

Choose SQL when: you need complex joins, ACID transactions across multiple tables, or strong referential integrity. The truth is — most applications work fine with PostgreSQL. People over-rotate to NoSQL. Start relational, move to NoSQL when you have a specific reason like extreme write throughput or key-value access patterns.

---

Q3. What are ACID properties? Why do they matter in transactional databases?

A3.
**Atomicity**: A transaction is all-or-nothing. If a bank transfer debits account A but fails to credit account B, the entire transaction rolls back. No partial states.

**Consistency**: A transaction moves the database from one valid state to another. Constraints (foreign keys, unique, check) are enforced. You can't end up with orphaned child records.

**Isolation**: Concurrent transactions don't interfere with each other. If two users buy the last item simultaneously, isolation ensures only one succeeds. The level of isolation is configurable (read committed, serializable, etc.).

**Durability**: Once committed, data survives crashes. It's written to disk (WAL — write-ahead log) before the commit is acknowledged. Even if the server loses power, committed data is safe.

They matter because without ACID, you get corrupted data, lost transactions, and inconsistent state — which is catastrophic for financial, healthcare, and inventory systems.

---

Q4. What is an index? How do B-tree and hash indexes differ?

A4.
An index is a data structure that speeds up data retrieval at the cost of additional storage and slower writes — like the index at the back of a textbook.

**B-tree index** (the default in most databases): A balanced tree where data is sorted. Supports equality lookups, range queries (`WHERE age > 25`), sorting, and prefix matching (`LIKE 'abc%'`). Nodes are designed to match disk page sizes for efficient I/O. PostgreSQL and MySQL use B+ trees, where all data pointers are at the leaf level.

**Hash index**: Uses a hash function to map keys to buckets. O(1) for exact-match lookups — extremely fast for `WHERE id = 42`. But it cannot handle range queries, sorting, or partial matches at all. Limited use case.

Rule of thumb: use B-tree for almost everything. Use hash indexes only when you exclusively do exact-match lookups and your database supports them well (some engines don't guarantee crash-safe hash indexes).

---

Q5. What is a transaction isolation level? Explain the common levels and the tradeoffs.

A5.
Isolation levels control how much one transaction can see of another's uncommitted work. Higher isolation = more correctness, lower concurrency.

**Read Uncommitted**: Can see other transactions' uncommitted changes (dirty reads). Almost never used — basically no isolation.

**Read Committed** (PostgreSQL default): Only sees committed data. Prevents dirty reads. But if you read the same row twice in one transaction, it might have changed (non-repeatable read).

**Repeatable Read** (MySQL InnoDB default): Once you read a row, it stays the same for the entire transaction. Prevents non-repeatable reads. But phantom reads can still happen — new rows matching your WHERE clause might appear.

**Serializable**: Full isolation — transactions execute as if they were serial. Prevents everything including phantom reads. But it's the slowest — transactions may block or fail due to serialization conflicts.

Most applications use Read Committed. Serializable is for financial reconciliation or inventory where absolute correctness matters and you accept the performance cost.

---

Q6. What are the different types of joins in SQL, and when would you use each?

A6.
**INNER JOIN**: Returns only matching rows from both tables. Use it when you need records that have a relationship in both tables — e.g., orders and their customers.

**LEFT JOIN**: Returns all rows from the left table, with NULLs for non-matching right-side columns. Use when you want all items from one side even if there's no match — e.g., all customers including those with zero orders.

**RIGHT JOIN**: Same as LEFT, but from the right table's perspective. Rarely used — just swap your table order and use LEFT JOIN.

**FULL OUTER JOIN**: Returns all rows from both sides, with NULLs where there's no match. Use for reconciliation — comparing two datasets to find what's in one but not the other.

**CROSS JOIN**: Cartesian product — every row from table A paired with every row from table B. Use for generating combinations, like all product-color pairs. Be careful — it can explode in size.

**SELF JOIN**: Joining a table to itself. Used for hierarchical data — finding employees and their managers from the same `employees` table.

---

Q7. What is denormalization, and why is it used in production systems?

A7.
Denormalization intentionally adds redundancy back into a normalized schema to improve read performance. You trade write complexity for read speed.

Example: instead of joining `orders`, `users`, and `products` on every query, you store `customer_name` and `product_name` directly in the `orders` table. Reads are faster because you eliminate joins, but writes are more complex — you need to update `orders` when a customer changes their name.

When to use it:
- Read-heavy workloads where joins are the bottleneck
- Analytics/reporting tables where you run aggregation queries
- Caching layers — materialized views are a form of controlled denormalization
- NoSQL databases where joins are impossible or expensive

The rule: normalize for correctness first, then denormalize strategically for performance based on measured bottlenecks. Don't denormalize prematurely.

---

Q8. What is a deadlock in a database? How do databases detect and resolve them?

A8.
A deadlock occurs when two transactions each hold a lock the other needs, and neither can proceed.

Transaction A locks row 1, tries to lock row 2. Transaction B locks row 2, tries to lock row 1. Both wait forever.

**Detection**: Most databases run a deadlock detector that checks for cycles in the wait-for graph. If A waits for B, and B waits for A, there's a cycle.

**Resolution**: The database chooses a victim transaction (usually the one with less work done or the younger one) and rolls it back, releasing its locks. The other transaction proceeds.

**Prevention**: Acquire locks in a consistent order (always lock row 1 before row 2). Keep transactions short. Use lower isolation levels when possible. Some applications use "SELECT ... FOR UPDATE NOWAIT" to fail immediately instead of waiting.

---

Q9. What is a primary key vs. a foreign key? Explain their roles in referential integrity.

A9.
**Primary key**: Uniquely identifies every row in a table. It's non-null, unique, and immutable (ideally). It can be a single column (surrogate key like auto-increment ID) or composite (combination of columns). Every table should have one.

**Foreign key**: A column in one table that references the primary key of another table. It enforces referential integrity — you can't insert an order with a `customer_id` that doesn't exist in the `customers` table.

**Referential integrity** means relationships between tables are always valid. Foreign keys prevent:
- Inserting orphaned records (order for a non-existent customer)
- Deleting a parent record that has children (delete customer who has orders)
- Updating a key to a value that doesn't exist

Cascade options (ON DELETE CASCADE, SET NULL, RESTRICT) control what happens when the parent record is modified. Use CASCADE carefully — it can silently delete more data than intended.

---

Q10. What is the CAP theorem, and how does it apply to distributed databases?

A10.
CAP says a distributed system can guarantee at most two of three properties:
- **Consistency**: Every read returns the most recent write.
- **Availability**: Every request gets a response (even if it's stale).
- **Partition Tolerance**: The system continues despite network failures between nodes.

Since network partitions are unavoidable in distributed systems, the real choice is between CP and AP during a partition:

**CP** (Consistency + Partition Tolerance): During a partition, some requests may be rejected to maintain consistency. HBase, MongoDB (in strong consistency mode). Used when stale data is unacceptable — banking, inventory.

**AP** (Availability + Partition Tolerance): During a partition, all requests get a response, but data may be stale. Cassandra, DynamoDB. Used when availability matters more — social media feeds, caching.

When there's no partition, you can have all three. CAP is about what you sacrifice when things go wrong.

---

### MEDIUM PRIORITY

---

Q11. What is a stored procedure? What are the pros and cons?

A11.
A stored procedure is pre-compiled SQL code stored in the database that you can call by name. Instead of sending a multi-statement query from your application, you call `EXEC process_monthly_billing()`.

**Pros**: Reduced network traffic (one call vs. multiple queries), pre-compiled execution plan (faster), centralized business logic (enforced regardless of which application connects), security (grant EXECUTE without exposing table access).

**Cons**: Hard to version control and deploy (they live in the database, not your codebase), difficult to debug and test, language-specific to the database vendor (PL/pgSQL vs T-SQL), can become a maintenance nightmare when business logic splits between app code and stored procedures.

My take: use them for data-intensive operations where round-trip latency matters (batch processing, complex reports). Keep business logic in your application layer where it's testable and version-controlled.

---

Q12. What is a view? How does a materialized view differ from a regular view?

A12.
A **regular view** is a saved SQL query. It's a virtual table — every time you query the view, it re-executes the underlying query. No data is stored. `CREATE VIEW active_users AS SELECT * FROM users WHERE status = 'active'`. It simplifies complex queries and acts as an abstraction layer.

A **materialized view** actually stores the query results on disk. It's a cached snapshot. Reads are fast because you're reading precomputed data. But it can become stale — you need to refresh it periodically (`REFRESH MATERIALIZED VIEW`).

Use regular views for simplicity and real-time data. Use materialized views for expensive computations that don't need to be up-to-the-second — dashboards, reporting queries, denormalized aggregations. PostgreSQL supports both. MySQL doesn't natively support materialized views (you simulate them with tables and triggers).

---

Q13. What is a composite index, and how does column order affect query performance?

A13.
A composite index is a single B-tree index on multiple columns. `CREATE INDEX idx ON orders(customer_id, order_date)`.

Column order is critical because the index sorts data left-to-right. This index efficiently supports:
- `WHERE customer_id = 42` — uses the first column ✓
- `WHERE customer_id = 42 AND order_date > '2024-01-01'` — uses both columns ✓
- `WHERE customer_id = 42 ORDER BY order_date` — index-only scan ✓

But NOT:
- `WHERE order_date > '2024-01-01'` without customer_id — can't skip the first column ✗

Think of it like a phone book sorted by last name, then first name. You can look up "Smith," but you can't efficiently look up everyone named "John" regardless of last name.

Rule: put the most selective column (highest cardinality) or the most-filtered column first. Put range conditions last because they stop the index from being used for subsequent columns.

---

Q14. What is the difference between optimistic and pessimistic locking?

A14.
**Pessimistic locking**: Lock the row when you read it, prevent anyone else from modifying it until you're done. `SELECT ... FOR UPDATE`. If someone else tries, they wait (block) or fail.

Use it when: conflicts are frequent and you can't afford retries — inventory countdown, financial transactions.

**Optimistic locking**: Don't lock on read. On write, check if the data changed since you read it (using a version number or timestamp). If it changed, abort and retry.

```sql
UPDATE products SET stock = 9, version = 6 
WHERE id = 1 AND version = 5
```

If zero rows affected, someone else updated it first.

Use it when: conflicts are rare and you want maximum concurrency — most web applications, content management systems.

Trade-off: pessimistic = less concurrency but guaranteed success; optimistic = more concurrency but occasional retry overhead.

---

Q15. What is database sharding? What are the trade-offs?

A15.
Sharding splits a large database horizontally across multiple servers. Each shard holds a subset of the data — shard 1 has users A-M, shard 2 has N-Z (range-based), or hash(user_id) % N determines the shard (hash-based).

**Benefits**: Horizontal scalability beyond a single machine's limits. Each shard handles a portion of the load. Enables geographic distribution.

**Trade-offs**:
- **Cross-shard queries are expensive**: Joining data across shards requires scatter-gather, which is slow.
- **Rebalancing is painful**: Adding a shard means migrating data. Consistent hashing helps minimize this.
- **No cross-shard transactions**: ACID across shards requires distributed transactions (2PC), which are slow and complex.
- **Operational complexity**: Backup, schema migration, monitoring — everything multiplied by N shards.

Before sharding: exhaust vertical scaling, read replicas, caching, and query optimization. Sharding is a last resort, not a first solution.

---

Q16. What is a write-ahead log (WAL)? Why is it important for crash recovery?

A16.
The WAL records every change to a log file on disk before applying it to the actual data files. If the database crashes mid-operation, it replays the WAL on restart to recover committed transactions and roll back uncommitted ones.

The flow: transaction writes changes to the WAL → WAL fsyncs to disk → database acknowledges the commit → later, a background process writes changes to the actual data files (checkpoint).

Why not just write to data files directly? Random writes to data files are slow (scattered across disk). WAL writes are sequential, which is much faster. The actual data file updates happen in batches during checkpoints.

WAL also enables replication — replicas read and replay the leader's WAL to stay in sync. PostgreSQL's streaming replication and MySQL's binary log are WAL-based.

---

Q17. Explain the concept of MVCC (Multi-Version Concurrency Control).

A17.
MVCC allows readers and writers to work concurrently without blocking each other. Instead of locking rows during reads, the database keeps multiple versions of each row.

When a transaction starts, it gets a snapshot — a consistent view of the database at that point in time. If another transaction modifies a row, the original transaction still sees the old version. No waiting, no blocking.

**PostgreSQL implementation**: Every row has a `xmin` (creating transaction) and `xmax` (deleting transaction) field. A transaction can only see rows where `xmin` is committed before its snapshot and `xmax` is not yet committed.

**Benefit**: Readers don't block writers, writers don't block readers. Massive concurrency improvement over lock-based approaches.

**Cost**: Old row versions accumulate — PostgreSQL needs VACUUM to clean them up. This is the "bloat" problem that requires tuning autovacuum settings.

---

Q18. What is a cursor in a database? When is it useful?

A18.
A cursor lets you process query results row by row instead of loading the entire result set into memory. It's essentially an iterator over a query result.

Use cases:
- Processing millions of rows where loading all into memory would crash your application
- When each row requires complex processing that can't be expressed in SQL
- Stored procedures that need to iterate over a result set

Example: migrating 10 million user records, applying a transformation to each. Without a cursor, you'd need to load all rows or batch them manually. With a cursor, you fetch and process one at a time (or in small batches).

But cursors are slow compared to set-based SQL operations. If you can express the logic in a single UPDATE or INSERT...SELECT, always prefer that over cursors. Cursors should be a last resort for procedural operations that truly can't be done in set-based SQL.

---

Q19. What are database triggers? When are they appropriate?

A19.
Triggers are procedures that automatically execute in response to specific database events: INSERT, UPDATE, DELETE on a table. They run before or after the event.

**Appropriate uses**: Audit logging (record every change to a sensitive table), maintaining denormalized counters (update `comment_count` in `posts` when a comment is inserted), enforcing complex business rules that can't be expressed with constraints.

**Problems**: They're invisible side effects — an INSERT on one table can trigger cascading operations across other tables. Debugging is a nightmare because the trigger logic isn't in your application code. Performance can degrade with complex trigger chains. They don't appear in version control unless you have a schema migration strategy.

Prefer application-level logic for business rules, and use triggers sparingly for audit trails or referential integrity that the database constraint system can't handle.

---

### LOW PRIORITY

---

Q20. What is a database connection pool, and why is it essential for application performance?

A20.
Creating a database connection is expensive — TCP handshake, authentication, memory allocation on the server. If every web request creates a new connection and closes it after, you're wasting time and resources.

A connection pool maintains a set of open, reusable connections. When your app needs a connection, it borrows one from the pool. When done, it returns it. The connection stays open for the next request.

Configuration matters: too few connections = requests queue up waiting. Too many = database gets overwhelmed, context switching increases. A common formula: pool size = (core_count * 2) + spindle_count. For most apps, 10-30 connections per application instance is reasonable.

Tools: HikariCP (Java standard), pgBouncer (PostgreSQL connection pooler — sits between your app and database), SQLAlchemy pool (Python).

---

Q21. Explain the differences between clustered and non-clustered indexes.

A21.
**Clustered index**: Determines the physical storage order of rows on disk. The table IS the index — leaf nodes contain the actual data rows. You can only have one per table because data can only be physically sorted one way. In InnoDB, the primary key is always the clustered index.

**Non-clustered index**: A separate structure with pointers to the actual rows. Leaf nodes contain the indexed columns plus a pointer (row ID or primary key) to the full row. You can have many per table.

Performance implication: if your query is covered entirely by a non-clustered index (all selected columns are in the index), it's an "index-only scan" — the database never touches the table. Otherwise, it does a "bookmark lookup" to fetch the remaining columns from the table, which can be expensive for large result sets.

---

Q22. What are the key differences between OLTP and OLAP databases?

A22.
**OLTP** (Online Transaction Processing): Handles day-to-day operations — user registrations, orders, payments. Characteristics: short transactions, many concurrent users, reads and writes, normalized schema, row-oriented storage. PostgreSQL, MySQL.

**OLAP** (Online Analytical Processing): Handles analytics — monthly revenue reports, user behavior analysis. Characteristics: complex aggregation queries, fewer concurrent users, mostly reads, denormalized/star schema, column-oriented storage. Snowflake, BigQuery, ClickHouse.

Column-oriented storage is key for OLAP: when you `SUM(revenue)` across a billion rows, column storage reads only the revenue column. Row storage would read every column of every row just to get revenue.

Most companies run both: OLTP for the application, ETL/ELT into an OLAP system for analytics. Don't run heavy analytics on your OLTP database — it'll kill your application's performance.

---

Q23. What is eventual consistency, and how does it differ from strong consistency?

A23.
**Strong consistency**: After a write, every subsequent read (from any node) sees the updated value. It's what you expect from a single-server database.

**Eventual consistency**: After a write, replicas will converge to the same value eventually, but reads may temporarily return stale data. There's a window where different nodes return different values.

DynamoDB, Cassandra, and DNS use eventual consistency. When you update a DNS record, it doesn't propagate instantly — some users see the old IP for minutes or hours.

It's acceptable when: slightly stale data is tolerable (social media likes count, product view count, user feed). It's unacceptable when: showing stale data causes real harm (bank balance, inventory count, authentication status).

Many systems offer tunable consistency — DynamoDB lets you choose strong or eventual per query. Use strong consistency for critical reads, eventual for everything else.

---

Q24. What is a database migration, and how should it be managed in production?

A24.
A database migration is a versioned, incremental change to your database schema — adding a column, creating a table, modifying an index. Tools like Flyway, Alembic, Knex, or Django migrations track which migrations have been applied.

**Best practices**:
- Migrations should be idempotent and reversible (include an up and down).
- Never modify a migration that's already been applied to production. Create a new one.
- Test migrations on a copy of production data — column type changes on large tables can lock the table for hours.
- Separate schema changes from data migrations. Deploy schema first, then data migrations.
- Use non-locking DDL when possible: `ALTER TABLE ... ADD COLUMN` is fine in PostgreSQL; `ALTER TABLE ... ADD INDEX` should be done with `CONCURRENTLY`.

Version control your migrations alongside your application code. They're part of the deployment pipeline.

---

==============================
ADDITIONAL MISSING Q&A (GAP FILL)
==============================

### CRITICAL

---

Q25. What is database replication? Compare master-slave, master-master, and quorum-based replication.

A25.
Replication copies data across multiple database servers for availability, durability, and read scalability.

**Master-slave (primary-replica)**: One master handles all writes. Replicas receive write logs asynchronously and serve reads. If master fails, promote a replica to master (failover). **Pros**: Simple, scales reads horizontally. **Cons**: Single point of write failure, replication lag causes stale reads from replicas, failover isn't instant.

**Master-master (multi-primary)**: Multiple nodes accept writes. Changes propagate between nodes. **Pros**: Write availability — if one master goes down, others continue. **Cons**: Write conflicts — two masters modify the same row simultaneously. Conflict resolution is complex (last-writer-wins loses data, application-level merging is hard). Used sparingly.

**Quorum-based (Dynamo-style)**: Writes go to W of N nodes, reads from R of N nodes. If W + R > N, at least one read node has the latest data (quorum overlap). W=2, R=2, N=3 gives strong consistency. W=1, R=1 gives eventual consistency with high availability. **Pros**: Tunable consistency and availability. **Cons**: More complex, need conflict resolution for concurrent writes.

**In practice**: Most production systems use primary-replica for simplicity. DynamoDB/Cassandra use quorum-based. Multi-primary is used for geo-distributed writes (CockroachDB, Google Spanner — but with consensus, not simple multi-primary).

---

Q26. How do you design a database schema for a given set of requirements? Walk through your approach.

A26.
**Step 1: Identify entities**: Read the requirements and list the nouns — Users, Orders, Products, Reviews. Each becomes a table.

**Step 2: Define attributes**: For each entity, list its properties. User: id, name, email, created_at. Order: id, user_id, total, status, order_date.

**Step 3: Identify relationships**: User places Orders (1-to-many → foreign key). Order contains Products (many-to-many → junction table `order_items`). Product has Reviews (1-to-many).

**Step 4: Normalize**: Start in 3NF — eliminate redundancy. Each fact is stored once. If you find yourself duplicating data, create a separate table and reference it with a foreign key.

**Step 5: Choose primary keys**: Prefer auto-incrementing integers or UUIDs. Natural keys (email, SSN) can change and cause cascading updates.

**Step 6: Add indexes**: Index columns used in WHERE, JOIN, and ORDER BY clauses. Index foreign keys. Don't over-index — each index slows writes.

**Step 7: Consider denormalization**: For read-heavy patterns, denormalize strategically. Store `order_total` in the orders table instead of computing it from order_items every time. Accept the maintenance burden.

**Step 8: Plan for scale**: Will any table grow to billions of rows? Consider partitioning. Will any query join across huge tables? Consider materialized views.

---

Q27. What is a query optimizer, and how does it decide the execution plan?

A27.
The query optimizer is the database's "brain" — it takes your SQL query and determines the most efficient way to execute it. The same query can be executed in many ways; the optimizer picks the cheapest.

**What it considers**: Available indexes, table statistics (row count, data distribution, cardinality), join order, join algorithms (nested loop, hash join, merge join), whether to do a sequential scan or index scan, sort strategies.

**Cost-based optimization**: The optimizer estimates the cost (disk I/O, CPU, memory) of each possible plan and picks the lowest-cost one. Costs are estimated using table statistics — `ANALYZE` in PostgreSQL updates these statistics.

**Common decisions**:
- Index scan vs sequential scan: For selecting 1 row from 1M → index scan. For selecting 900K from 1M → sequential scan (index lookup per row is more expensive than reading the whole table).
- Join order: Joining small table first, then large table, reduces intermediate result size.
- Join algorithm: Nested loop for small tables, hash join for equi-joins on large tables, merge join for pre-sorted data.

**When the optimizer is wrong**: Stale statistics lead to bad plans. Run `ANALYZE` after bulk loads. Use `EXPLAIN ANALYZE` to see actual vs estimated row counts — large discrepancies indicate stale stats.

---

### IMPORTANT

---

Q28. What are database partitioning strategies? How do they differ from sharding?

A28.
**Partitioning**: Splitting a single table into smaller pieces within the same database server. The database manages it transparently — queries hit the right partition automatically.

- **Range partitioning**: Split by value range. Orders partitioned by date (one partition per month). Queries with date filters only scan relevant partitions.
- **Hash partitioning**: Hash the partition key, distribute rows across partitions. Even distribution but can't efficiently do range queries.
- **List partitioning**: Split by explicit value lists. Customers partitioned by region (APAC, EMEA, Americas).

**Sharding**: Distributing data across multiple database servers. Each shard is a separate database instance on different hardware.

**Key difference**: Partitioning = one server, multiple logical divisions. Sharding = multiple servers, distributed data. Partitioning improves query performance on a single machine. Sharding provides horizontal scalability beyond one machine's capacity.

**Sharding is harder**: Cross-shard queries are expensive (joining data across servers). Rebalancing shards when adding nodes is complex. Application routing logic must know which shard holds which data.

---

Q29. What are common anti-patterns in database design?

A29.
**Entity-Attribute-Value (EAV)**: One table with columns (entity_id, attribute_name, attribute_value). Seems flexible — store any attribute without schema changes. But: no type safety (everything is a string), no constraints, queries are horribly complex (`WHERE attribute_name = 'color' AND attribute_value = 'red'` instead of `WHERE color = 'red'`), no joins. Use JSON columns or proper schema design instead.

**God table**: One massive table with 200 columns trying to represent everything. Half the columns are NULL for any given row. Split into focused tables with clear relationships.

**Polymorphic associations**: A comment table with `commentable_type` (post, photo, video) and `commentable_id`. Can't enforce foreign key constraints (which table does the ID reference?). Use separate junction tables or table inheritance.

**Soft deletes pitfall**: `is_deleted = true` instead of actually deleting. Every query needs `WHERE is_deleted = false`. Forgotten filters leak deleted data. Indexes still include deleted rows. Use audit tables or event sourcing for delete history instead.

**N+1 query pattern**: Fetch 100 orders, then for each order fetch items — 101 queries. Use JOINs or batch fetching instead.

---

Q30. What is a covering index, and how does it eliminate table lookups?

A30.
A covering index contains all the columns needed by a query — the database engine can satisfy the query entirely from the index without accessing the table data (no "bookmark lookup").

**Example**: Query: `SELECT name, email FROM users WHERE status = 'active'`. Index on (status) → database finds matching rows in the index, then looks up each row in the table to get name and email. Index on (status, name, email) → the index itself contains everything. No table access needed. Much faster.

**How to spot it**: In PostgreSQL, `EXPLAIN` shows "Index Only Scan" (covering) vs "Index Scan" + "Heap Fetch" (not covering). In MySQL, look for "Using index" in the Extra column.

**Trade-off**: Wider indexes use more disk space and slow down writes (more data to update on inserts/updates). Don't make every index cover every query — focus on the most frequent and performance-critical queries.

**INCLUDE clause (PostgreSQL)**: `CREATE INDEX ON users (status) INCLUDE (name, email)`. The `INCLUDE` columns are stored in the index but not part of the search key. Best of both worlds — narrow search key, wide coverage.

---

### GOOD-TO-HAVE

---

Q31. What is the N+1 query problem? How do you detect and fix it?

A31.
**The problem**: Fetch a list of 100 orders (1 query). For each order, fetch its items (100 queries). Total: 101 queries. With 10,000 orders → 10,001 queries. Each query has network roundtrip + parsing + execution overhead.

**Detection**: Enable query logging and look for repeated identical queries with different IDs. ORMs often hide this — you don't see the SQL unless you log it. Django's `django-debug-toolbar`, Rails' `bullet` gem, SQLAlchemy's echo mode.

**Fixes**:
- **Eager loading (JOIN)**: `SELECT * FROM orders JOIN items ON items.order_id = orders.id`. One query. But large result set if orders have many items.
- **Batch loading (IN clause)**: `SELECT * FROM items WHERE order_id IN (1, 2, 3, ..., 100)`. Two queries total. ORM equivalents: Django's `prefetch_related()`, SQLAlchemy's `joinedload()`.
- **DataLoader pattern (GraphQL)**: Batch individual requests within a single execution cycle. Collect all IDs, make one batched query.

---

Q32. What is a time-series database? When would you choose one over a relational DB?

A32.
A time-series database (TSDB) is optimized for data indexed by time — metrics, logs, IoT sensor data, stock prices. Every data point has a timestamp and one or more values.

**Why not a regular relational DB?** Write volume is extremely high (millions of data points per second). Queries are almost always time-range based (`WHERE time > '2024-01-01' AND time < '2024-02-01'`). Data naturally ages — old data can be downsampled or deleted. Column-oriented storage compresses time-series data extremely well (sequential timestamps, repeating tags).

**Features**: Automatic data retention policies, continuous aggregation (pre-compute hourly/daily rollups), specialized functions (rate-of-change, moving average, percentiles over time windows), high write throughput.

**Examples**: InfluxDB, TimescaleDB (PostgreSQL extension — best of both worlds), Prometheus (metrics), ClickHouse (analytics + time-series).

**When to use**: Monitoring/observability, IoT data, financial market data, application metrics. When NOT to use: transactional data, relational data with complex joins.

---

Q33. What are column-family stores (Cassandra, HBase)? How do they differ from row-oriented databases?

A33.
**Row-oriented (PostgreSQL, MySQL)**: Stores data row by row on disk. Reading a full row is fast. But reading one column across millions of rows requires reading every row — wasteful.

**Column-family stores**: Group columns into families, store each family together. Within a family, data is stored by column. Reading specific columns across many rows is very fast. Writing a full row involves multiple column families — potentially slower.

**Cassandra**: Distributed, highly available, tunable consistency. Data model: keyspace → table → partition (identified by partition key) → rows → columns. Optimized for write-heavy workloads. Scales horizontally with linear performance increase. No single point of failure.

**Use cases**: Time-series data, messaging systems, IoT data, user activity logs — high write volume, queries on specific columns, eventual consistency is acceptable.

**NOT suitable for**: Complex joins (Cassandra has no JOINs), transactions across partitions, ad-hoc queries on arbitrary columns. If you need these, use a relational database or add a secondary index (but Cassandra secondary indexes are limited).
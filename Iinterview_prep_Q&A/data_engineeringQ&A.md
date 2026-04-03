==============================
FILE: Data Engineering
==============================

### HIGH PRIORITY

---

Q1. What is the difference between ETL and ELT? When do you prefer one architecture over the other?

A1.
**ETL** (Extract, Transform, Load): Data is extracted from sources, transformed in a staging area (middleware), then loaded into the destination warehouse. The transformation engine sits between source and destination. Traditional approach — Informatica, Talend.

**ELT** (Extract, Load, Transform): Data is extracted and loaded raw into the destination, then transformed inside the warehouse using its compute power. Modern approach — dbt + Snowflake, BigQuery, Databricks.

**Why ELT is winning**: Modern cloud warehouses have massive compute. It's cheaper and simpler to dump raw data into the warehouse and transform it there using SQL (via dbt) than to maintain a separate transformation layer. Raw data is preserved (you can re-transform), and transformations are version-controlled SQL.

**When ETL still makes sense**: When you need to filter sensitive data before it reaches the warehouse (PII masking), when the destination has limited compute (on-prem databases), or when transformations are complex and non-SQL (ML feature engineering in Python).

---

Q2. What is Apache Spark, and how does it differ from MapReduce?

A2.
Both are distributed data processing frameworks, but Spark is essentially MapReduce's successor.

**MapReduce**: Writes intermediate results to disk between stages (map → disk → shuffle → disk → reduce). Reliable but slow due to I/O. Programming model is limited — everything is a map or reduce function.

**Spark**: Keeps data in memory across stages (RDDs / DataFrames). 10-100x faster for iterative operations (ML training, multi-step ETL). Rich API — map, filter, join, groupBy, window functions — in Python, Scala, Java, R. Supports batch AND streaming (Structured Streaming).

**Spark's key concepts**: Lazy evaluation (transformations are planned, not executed, until an action like `collect()` or `write()`), DAG execution (optimizes the execution plan), partitioning (data split across nodes for parallel processing).

When to use: large-scale data processing (TB+), complex transformations, ML pipelines. For smaller data (< 100GB), you might not need Spark — Pandas, DuckDB, or your warehouse's SQL engine could be sufficient and much simpler.

---

Q3. What is a data warehouse vs. a data lake? When do you use each?

A3.
**Data Warehouse**: Structured, schema-on-write. Data is cleaned, transformed, and organized into a star/snowflake schema before loading. Optimized for analytical queries (OLAP). Snowflake, BigQuery, Redshift. Reliable for dashboards and business intelligence.

**Data Lake**: Semi-structured and unstructured, schema-on-read. Raw data dumped in any format — JSON, CSV, Parquet, images, logs. S3, Azure Data Lake, HDFS. Cheap storage. But can become a "data swamp" without governance.

**Data Lakehouse** (modern approach): Combines both. Store raw data in the lake (Parquet/Delta format), but add warehouse-like features — ACID transactions, schema enforcement, time travel. Delta Lake, Apache Iceberg, Apache Hudi. Query with SQL engines (Databricks SQL, Trino, Athena).

**Decision**: Start with a warehouse for structured analytics. Add a lake when you need to store raw/unstructured data cheaply. Consider a lakehouse if you want one system for both.

---

Q4. What is Apache Kafka, and when would you use it in a data pipeline?

A4.
Kafka is a distributed event streaming platform — a durable, high-throughput, fault-tolerant message bus. Producers publish messages to topics, consumers read from topics.

**Key properties**: Messages are persisted to disk (not just queued in memory), ordered within a partition, and retained for a configurable duration (hours, days, or forever). Consumers track their own offset — they can replay from any point.

**Use cases in data pipelines**:
- **Real-time ingestion**: Application events → Kafka → Spark Streaming / Flink → warehouse
- **CDC (Change Data Capture)**: Database changes → Debezium → Kafka → downstream consumers
- **Decoupling services**: Service A publishes events, services B/C/D consume independently
- **Event sourcing**: Kafka as the system of record for all state changes

**When not to use**: Simple batch ETL that runs once/day — cron + Python script is simpler. Small-scale systems where a message queue (RabbitMQ, SQS) would suffice. Kafka has significant operational complexity.

---

Q5. What is data partitioning, and why is it important for query performance?

A5.
Partitioning divides a large table into smaller, physically separate segments based on a column's value. When you query with a filter on the partition column, the engine only scans relevant partitions — this is called partition pruning.

**Range partitioning**: By date — one partition per month. `SELECT * FROM events WHERE event_date = '2024-01-15'` only scans the January 2024 partition.

**Hash partitioning**: Hash(user_id) % N — distributes rows evenly. Good for avoiding hotspots but doesn't support range queries.

**List partitioning**: By discrete values — one partition per country or category.

**Impact**: On a 10TB table partitioned by month, a query filtering on one month scans ~833GB instead of 10TB. In Spark/Hive, partitioning also determines the directory structure on disk — each partition is a folder.

**Best practice**: Partition on columns you frequently filter on. Don't over-partition — millions of tiny partitions (one per user ID) create metadata overhead and slow query planning.

---

Q6. What is schema-on-read vs. schema-on-write? What are the trade-offs?

A6.
**Schema-on-write** (warehouses): You define the schema before loading data. Data is validated, cleaned, and structured at write time. Queries are fast because data is organized. But: rigid — changing the schema requires migrations, and you lose data that doesn't fit the schema.

**Schema-on-read** (data lakes): Data is stored raw. Schema is applied when you read/query it. Flexible — store everything, figure out the structure later. But: queries are slower (parsing at read time), data quality issues hide until query time, and different consumers may interpret the same data differently.

**Practical trade-off**: Schema-on-write is better for known, stable use cases (daily sales reports). Schema-on-read is better for exploratory/unknown use cases (data science exploration, log analysis where the format changes).

Modern lakehouses try to give you both: store data in a semi-structured format (Parquet with Delta/Iceberg metadata for schema evolution) and enforce schema at write time while allowing schema evolution.

---

Q7. What are the common file formats in data engineering (Parquet, Avro, ORC)?

A7.
**Parquet** (columnar): Stores data by column. Excellent for analytical queries that read specific columns (`SELECT avg(salary) FROM employees` reads only the salary column). Supports nested types, compression per column. The standard format for data lakes. Works with Spark, Hive, Presto, pandas.

**ORC** (Optimized Row Columnar): Similar to Parquet but optimized for Hive. Slightly better compression and read performance in Hive ecosystem. Less universal outside Hive.

**Avro** (row-based): Stores data by row. Better for write-heavy workloads and full-record reads. Schema is embedded in the file (self-describing). Good for Kafka message serialization and data that needs schema evolution (you can add fields without breaking consumers).

**CSV/JSON**: Human-readable, universally supported, but inefficient — no compression, no schema enforcement, slow to parse at scale.

**Rule of thumb**: Parquet for analytics and storage, Avro for streaming and inter-service data exchange, CSV/JSON for small datasets and interoperability.

---

Q8. What is a data pipeline, and how do you make it idempotent?

A8.
A data pipeline is a series of automated steps that move and transform data from sources to destinations. Extract → Transform → Load, scheduled to run periodically.

**Idempotency** means running the pipeline multiple times with the same input produces the same result — no duplicates, no missing data. This is critical because pipelines fail and get retried.

**How to achieve it**:
- **Overwrite, don't append**: Instead of `INSERT INTO target`, use `INSERT OVERWRITE PARTITION (date='2024-01-15')`. Re-running overwrites the same partition cleanly.
- **Upsert/Merge**: `MERGE INTO target USING source ON key WHEN MATCHED UPDATE WHEN NOT MATCHED INSERT`.
- **Deduplication**: Assign unique IDs to each record, deduplicate before writing.
- **Date-bounded processing**: Process only data for a specific date range. Re-processing the same date produces the same output.

A non-idempotent pipeline that appends on every run is a ticking time bomb — one retry doubles the data.

---

Q9. What is a DAG (Directed Acyclic Graph) in the context of workflow orchestration?

A9.
A DAG defines the execution order and dependencies of tasks in a data pipeline. Each node is a task, each edge is a dependency. "Acyclic" means no circular dependencies — task A can't depend on task B while B depends on A.

In **Apache Airflow** (the most popular orchestrator), a DAG is defined in Python:

```python
extract >> transform >> load  # extract runs first, then transform, then load
[task_a, task_b] >> task_c    # A and B run in parallel, C waits for both
```

**Why it matters**: Complex pipelines have dozens of tasks with intricate dependencies. A DAG engine handles scheduling, retries, alerting, backfills, and dependency resolution automatically. Without it, you're managing cron jobs and manually handling failures.

Other orchestrators: Prefect, Dagster, Luigi, dbt (for SQL transformation DAGs). Dagster adds asset-based thinking — each node is a data asset, not just a task.

---

Q10. What is Change Data Capture (CDC)? How is it implemented?

A10.
CDC detects and captures changes (inserts, updates, deletes) in a source database and streams them to downstream systems in near real-time.

**Implementation approaches**:
- **Log-based CDC** (best): Read the database's transaction log (WAL in Postgres, binlog in MySQL). Non-intrusive — no queries on the source database. Captures every change including deletes. Debezium is the standard open-source tool — it reads the log and publishes changes to Kafka.
- **Trigger-based**: Database triggers write changes to a changelog table. Works anywhere but adds overhead to every write and is hard to maintain.
- **Timestamp-based**: Query for records with `updated_at > last_run`. Misses deletes (no row to query) and hard deletes. Simple but incomplete.
- **Snapshot diffing**: Compare full snapshots between runs. Works for any source but expensive and slow.

**Use cases**: Keeping a search index in sync with a database, replicating data to a warehouse in real-time, event sourcing, cache invalidation.

---

Q11. Explain the difference between batch processing and stream processing.

A11.
**Batch processing**: Collect data over a period (hourly, daily), then process it all at once. MapReduce, Spark batch jobs, dbt runs. Think: "process all of yesterday's orders at 2 AM."

**Stream processing**: Process events as they arrive, one at a time or in micro-batches. Kafka Streams, Flink, Spark Structured Streaming. Think: "detect fraud on each transaction within seconds."

**Trade-offs**:
- Batch: simpler to build, debug, and reason about. Higher latency (minutes to hours). Easier to achieve exactly-once semantics.
- Stream: lower latency (seconds to milliseconds). More complex — handling late data, out-of-order events, state management. Harder to debug.

**Lambda architecture**: Run both — batch for accuracy (reprocess everything nightly), stream for speed (approximate real-time). Complex to maintain.

**Kappa architecture**: Stream-only — replay the event stream for corrections instead of maintaining a separate batch layer. Simpler if your streaming layer (Kafka + Flink) is mature enough.

Most teams use batch processing for 90% of workloads and add streaming only where low latency is a business requirement.

---

Q12. What is a star schema vs. a snowflake schema?

A12.
Both are dimensional modeling approaches for data warehouses.

**Star schema**: A central fact table (e.g., `sales_facts` with revenue, quantity, timestamps) surrounded by dimension tables (customer, product, date, store). Dimensions are denormalized — `customer_dim` has city, state, country all in one table. Simple queries, fast joins.

**Snowflake schema**: Dimensions are normalized — `customer_dim` links to `city_dim`, which links to `state_dim`, which links to `country_dim`. Saves storage but requires more joins.

**Which to use**: Star schema in most cases. The storage savings from snowflaking are negligible in modern column-store warehouses (they compress well). The extra joins in snowflake schemas slow down queries and make them more complex.

**Fact table types**: Transaction facts (one row per event — each sale), snapshot facts (periodic state — daily inventory levels), accumulating facts (lifecycle tracking — order placed → shipped → delivered).

---

### MEDIUM PRIORITY

---

Q13. What is data lineage, and why is it important?

A13.
Data lineage tracks where data comes from, how it moves, and what transformations are applied — from source to final dashboard.

**Why it matters**:
- **Debugging**: A metric looks wrong. Lineage tells you which tables, transformations, and sources contributed to it. You trace backwards to find the bug.
- **Impact analysis**: Before changing a table's schema, lineage shows you every downstream dashboard, model, and report that would break.
- **Compliance**: Regulators (GDPR, HIPAA) require knowing where personal data is stored and how it flows.
- **Trust**: Business users trust data more when they can see its provenance.

**Tools**: dbt provides model-level lineage automatically. Apache Atlas, OpenLineage, DataHub, and Atlan provide cross-system lineage. Modern data platforms like Monte Carlo add lineage with anomaly detection.

---

Q14. What is a slowly changing dimension (SCD)? Explain Type 1 and Type 2.

A14.
Dimensions change over time — a customer moves cities, a product's price changes. SCDs define how to handle these changes.

**Type 1 — Overwrite**: Just update the record. Old value is lost. Simple, but you can't analyze historical data ("what city was this customer in when they made that purchase?").

**Type 2 — Add a new row**: Keep the old record and add a new one with version tracking. Add `effective_date`, `end_date`, and `is_current` columns. The old row shows `is_current = false`, the new row shows `is_current = true`.

Customer_dim:
| id | name | city | effective_date | end_date | is_current |
| 1 | Aman | Delhi | 2020-01-01 | 2024-06-01 | false |
| 1 | Aman | Mumbai | 2024-06-01 | NULL | true |

**Type 2 is the standard** for any dimension where historical tracking matters — which is most of them.

**Type 3**: Add a column for the previous value (e.g., `current_city`, `previous_city`). Only tracks one level of history. Rarely used.

---

Q15. How do you handle late-arriving data in a data pipeline?

A15.
Late data arrives after its time window has been processed — an event from 11:50 PM arrives after the midnight batch has already run.

**Strategies**:
- **Reprocessing**: Re-run the batch for the affected partition. If your pipeline is idempotent (overwrite partition), this is clean. Most reliable but requires backfill infrastructure.
- **Watermarks** (streaming): Define a watermark — "I'll wait 10 minutes past the event time window before closing it." Events arriving after the watermark are either dropped or sent to a late data side output. Apache Flink and Spark Structured Streaming support this.
- **Append + reconcile**: Append late data to the current partition, then periodically reconcile — deduplicate and correct aggregations.
- **Lambda architecture**: Batch layer processes complete historical data nightly and corrects any gaps from the streaming layer.

**Best practice**: Design pipelines to expect late data. Use event time (when the event happened) for partitioning, not processing time (when you received it). Airflow's `execution_date` helps distinguish "which period is this run for" from "when did this run execute."

---

Q16. What is data quality, and how do you validate it in pipelines?

A16.
Data quality means data is accurate, complete, consistent, timely, and follows expected patterns. Bad data → bad decisions → financial losses.

**Validation approaches**:
- **Schema validation**: Columns exist, types match, no unexpected nulls. Great Expectations, dbt tests.
- **Row count checks**: Did the pipeline produce roughly the expected number of rows? A 50% drop suggests a source issue.
- **Uniqueness**: Primary keys should be unique. Duplicate detection.
- **Range checks**: Prices should be positive. Ages between 0-150. Dates not in the future.
- **Referential integrity**: Every `order.customer_id` should exist in `customers`.
- **Freshness**: Data should arrive within expected SLAs — if yesterday's data isn't here by 8 AM, alert.

**dbt tests** are the simplest entry point:
```yaml
- name: orders
  columns:
    - name: id
      tests: [unique, not_null]
    - name: amount
      tests: [not_null, positive_values]
```

**Monitoring tools**: Monte Carlo, Elementary, Soda — detect anomalies automatically (distribution shifts, volume changes).

---

Q17. What is Apache Airflow, and how does it work?

A17.
Airflow is a workflow orchestrator for data pipelines. You define DAGs in Python, and Airflow handles scheduling, execution, retries, alerting, and dependency management.

**Core concepts**:
- **DAG**: The pipeline definition — a Python file defining tasks and dependencies.
- **Operator**: A task type — `BashOperator`, `PythonOperator`, `SnowflakeOperator`, `SparkSubmitOperator`.
- **Scheduler**: Parses DAGs, creates task instances based on the schedule, and queues them.
- **Executor**: Runs the tasks. LocalExecutor (single machine), CeleryExecutor (distributed workers), KubernetesExecutor (one pod per task).
- **XComs**: Inter-task communication — one task passes small data to another.

**Best practices**: Keep DAGs idempotent. Use `execution_date` for date-bounded processing. Don't put heavy logic in the DAG file — call external scripts or functions. Use sensors sparingly (they hold worker slots). Prefer KubernetesExecutor or task-level Kubernetes pods for isolation.

**Alternatives**: Dagster (asset-oriented, better developer experience), Prefect (simpler API, better error handling), Mage (newer, UI-focused).

---

Q18. What is a medallion architecture (Bronze/Silver/Gold)?

A18.
A data architecture pattern popularized by Databricks that organizes data in layers of increasing quality.

**Bronze (Raw)**: Data as-is from source systems — no transformations. Preserves the original format. Source of truth for debugging and reprocessing. Append-only.

**Silver (Cleaned)**: Deduplicated, validated, schema-enforced. Bad records are filtered or quarantined. Joins are applied. Consistent data types. This is where most data engineering work happens.

**Gold (Business-ready)**: Aggregated, business-logic-applied. Ready for dashboards and ML. `daily_revenue_by_region`, `customer_lifetime_value`. Modeled as star schema dimensions and facts.

**Benefits**: Clear data quality expectations at each layer. Debugging traces back from Gold → Silver → Bronze. Reprocessing replays from Bronze without re-ingesting from sources.

This maps well to dbt's staging → intermediate → marts pattern. Bronze/staging = raw source models. Silver/intermediate = cleaned joins. Gold/marts = business metrics.

---

### LOW PRIORITY

---

Q19. What is data skew, and how does it affect distributed processing?

A19.
Data skew means data is unevenly distributed across partitions. If you partition by `customer_id` and one customer has 10M records while others have 1K, the partition with the large customer becomes a bottleneck — all other tasks finish quickly while one task runs for hours.

**Impact**: The job's runtime is determined by the slowest task. Resources on other nodes sit idle.

**Solutions**:
- **Salting**: Add a random suffix to the skewed key (`customer_id + random(0,9)`), process in parallel, then aggregate. `key_1`, `key_2`, ..., `key_9`.
- **Broadcast join**: If the small table fits in memory, broadcast it to all nodes instead of shuffling the large table by the skewed key.
- **Adaptive Query Execution** (Spark 3.0+): Automatically detects skew and splits oversized partitions at runtime.
- **Custom partitioning**: Repartition by a more evenly distributed column or use a composite key.

---

Q20. What is the difference between a data engineer, a data analyst, and a data scientist?

A20.
**Data Engineer**: Builds and maintains the infrastructure — pipelines, warehouses, streaming systems. Skills: Python, SQL, Spark, Airflow, Kafka, cloud platforms. Makes data available and reliable.

**Data Analyst**: Explores data to answer business questions — dashboards, reports, ad-hoc queries. Skills: SQL, Excel, Tableau/Looker, basic Python/R. Translates data into business insights.

**Data Scientist**: Builds statistical models and ML algorithms — predictions, recommendations, NLP. Skills: Python, statistics, ML frameworks (scikit-learn, PyTorch), feature engineering. Requires clean data from engineers.

**Overlap**: In smaller teams, one person does all three. In larger orgs, there's also the Analytics Engineer (dbt, SQL transformations, data modeling — bridges engineering and analysis) and ML Engineer (productionizes models — bridges science and engineering).

---

Q21. What is a materialized view in data engineering, and how does it differ from a regular table?

A21.
A materialized view is a precomputed query result stored physically on disk. It combines the convenience of a view (defined by a query) with the performance of a table (data is stored).

**vs. regular view**: A view re-runs the query every time. A materialized view caches the result. Reading is fast; the trade-off is staleness, since it needs to be refreshed.

**vs. regular table**: A table is independent — you manage its data explicitly. A materialized view is derived — it's tied to its source query. Some databases support incremental refresh (only recompute changed data).

**Use cases**: Dashboard queries that aggregate millions of rows but only need to update every hour. Cross-table joins that are too expensive to run on every request. In PostgreSQL, `REFRESH MATERIALIZED VIEW CONCURRENTLY` updates the view without locking reads.

---

Q22. What are the key considerations when choosing a message queue vs. an event stream?

A22.
**Message Queue** (RabbitMQ, SQS): Messages are consumed and deleted. Each message is processed by exactly one consumer (work queue pattern). Good for task distribution — background jobs, email sending. Simple, reliable, point-to-point.

**Event Stream** (Kafka, Kinesis): Events are persisted and can be consumed by multiple consumers independently. Each consumer tracks its own offset. Events are retained for days/weeks. Good for event-driven architecture — multiple services react to the same event.

**Choose message queue when**: You want tasks distributed among workers, messages are one-time-use, you don't need replay, and you want simple setup.

**Choose event stream when**: Multiple consumers need the same data, you need replay capability (reprocess from yesterday), events should be durable, you're building event-driven or microservices architecture.

**Overlap**: Kafka can act as a queue (consumer groups with one consumer per partition). SQS can fan out with SNS. The lines blur, but the mental models differ.

---

Q23. What are dbt models, and how does dbt fit into a modern data stack?

A23.
dbt (data build tool) transforms data inside your warehouse using SQL. Each model is a SQL SELECT statement that creates a table or view.

```sql
-- models/marts/daily_revenue.sql
SELECT date, SUM(amount) as revenue
FROM {{ ref('stg_orders') }}
WHERE status = 'completed'
GROUP BY date
```

`{{ ref() }}` creates a dependency — dbt builds a DAG and executes models in the right order.

**How it fits**: Sources (Fivetran/Airbyte) load raw data → dbt transforms it → BI tools (Looker/Tableau) visualize it. dbt replaces the "T" in ELT.

**Key features**: Version-controlled SQL transformations, built-in testing (`not_null`, `unique`, `relationships`), documentation generation, incremental models (process only new data), Jinja templating for DRY SQL.

dbt has become the standard for analytics engineering. It brings software engineering practices (version control, testing, CI/CD, modularity) to SQL transformations.

---

Q24. What is data governance, and why does it matter?

A24.
Data governance is the set of policies, processes, and standards for managing data — who can access it, how it's defined, where it lives, and what quality standards it meets.

**Key components**:
- **Access control**: Who can see which data. PII should be restricted. Role-based access.
- **Data catalog**: A searchable inventory of all datasets — what they contain, where they are, who owns them. Tools: DataHub, Atlan, Alation.
- **Data definitions**: Agreed-upon business definitions. What exactly does "active user" mean? Is it daily login? Monthly? Document it.
- **Privacy compliance**: GDPR right-to-deletion, data retention policies, PII masking/anonymization.
- **Ownership**: Every dataset has an assigned owner responsible for its quality and documentation.

**Why it matters**: Without governance, teams use different definitions of the same metric, sensitive data leaks, nobody knows which data to trust, and regulatory fines follow. It's not exciting, but it's what separates mature data organizations from chaotic ones.

==============================
ADDITIONAL MISSING Q&A (GAP FILL)
==============================

### CRITICAL

---

Q25. What is a data lakehouse, and how does it combine data lake and data warehouse concepts?

A25.
A data lakehouse adds warehouse-like structure and performance to a data lake. The idea: keep raw data in cheap object storage (S3, ADLS) in open formats (Parquet), but add a metadata/transaction layer on top that gives you ACID transactions, schema enforcement, and SQL query performance.

**Data Lake**: Store anything (structured, semi-structured, unstructured) cheaply. But no transactions, no schema enforcement, no time travel. "Data swamp" risk — data goes in but nobody trusts it.

**Data Warehouse**: Structured, schema-enforced, optimized for SQL queries (Redshift, BigQuery, Snowflake). But expensive, and you have to ETL data in — separate from the lake.

**Lakehouse**: The middle ground. Technologies like **Delta Lake** (Databricks), **Apache Iceberg** (Netflix origin), and **Apache Hudi** enable ACID transactions, schema evolution, time travel (query data as of a past timestamp), and efficient upserts — all on top of Parquet files in object storage.

Architecture: Raw data lands in the lake → Bronze layer (raw, append-only) → Silver layer (cleaned, deduplicated) → Gold layer (aggregated, business-ready). This is the "medallion architecture." One storage system, multiple quality tiers.

Why it matters: You stop maintaining separate lake + warehouse + ETL between them. Databricks, Snowflake, and most modern data platforms are converging on this pattern.

---

Q26. How does Apache Flink compare to Spark Structured Streaming?

A26.
Both process streaming data, but their architectures differ fundamentally:

**Apache Flink**: True stream processing engine. Processes events one-at-a-time (or in small batches for efficiency). Sub-second latency. Natively streaming — batch is a special case of streaming.

**Spark Structured Streaming**: Micro-batch architecture. Collects events into small batches (100ms-seconds), processes each as a mini DataFrame. Latency is higher (seconds), but it inherits Spark's mature ecosystem.

Key differences:
- **Latency**: Flink wins — true event-at-a-time. Spark has micro-batch overhead (though Continuous Processing mode reduces this)
- **State management**: Flink has first-class state management with RocksDB-backed state, incremental checkpointing. Spark's state is managed through checkpoints but is less flexible
- **Event time**: Both support event time and watermarks, but Flink's windowing is more expressive
- **Exactly-once**: Both achieve it, but Flink's checkpointing (Chandy-Lamport algorithm) is more elegant
- **Ecosystem**: Spark has broader adoption, better ML integration (MLlib), and a larger community. Flink is stronger for complex event processing (CEP)

Choose Flink for sub-second latency requirements and complex streaming logic. Choose Spark Streaming when your team already uses Spark and latency of a few seconds is acceptable.

---

Q27. What is a data catalog and why is it important?

A27.
A data catalog is a searchable inventory of all data assets in your organization — tables, dashboards, pipelines, ML models. It answers: "What data do we have, where is it, what does it mean, and can I trust it?"

**Core features**:
- **Discovery**: Search for datasets by name, description, tags. "Where is the revenue data?"
- **Schema & lineage**: See column types, upstream/downstream dependencies. "What feeds into this dashboard?"
- **Ownership**: Every dataset has an owner. "Who do I ask about this table?"
- **Documentation**: Business definitions, data dictionaries, usage examples
- **Quality indicators**: Freshness, completeness scores, anomaly flags
- **Access management**: Request access, understand who can see what

**Tools**: DataHub (LinkedIn, open source), Atlan, Alation, Apache Atlas, Amundsen (Lyft), Google Data Catalog.

**Lineage** is the killer feature — trace data from source to final dashboard. If an upstream pipeline breaks, lineage tells you exactly which downstream tables and dashboards are affected.

Without a catalog, data engineers spend ~30% of their time just finding and understanding data. It's the difference between a team of 5 serving themselves and a team of 5 drowning in Slack questions.

---

### IMPORTANT

---

Q28. How do you achieve exactly-once semantics in stream processing?

A28.
Exactly-once means each event is processed exactly once, even if failures occur. It's the hardest guarantee, and the truth is nuanced:

**The problem**: A consumer reads event A, processes it, and needs to commit the offset. If it crashes after processing but before committing, event A will be replayed → at-least-once (duplicates). If it commits before processing, event A may be lost → at-most-once.

**Kafka's approach** (idempotent producer + transactions):
- **Idempotent producer**: Each message has a sequence number; the broker deduplicates retries
- **Transactions**: Producer can atomically write to multiple partitions and commit offsets in the same transaction. Either everything succeeds or nothing does.
- **Consumer**: Read committed — only see messages from committed transactions

**Flink's checkpointing**: Flink uses the Chandy-Lamport distributed snapshot algorithm. It injects checkpoint barriers into the stream. Each operator saves its state when it receives a barrier. On failure, Flink restores from the last successful checkpoint and replays from that point.

**Practical reality**: True exactly-once within the system is achievable. But if your pipeline writes to an external system (database, API), you need **idempotent sinks** — the external write should be safe to retry. Use upserts with natural keys, or deduplication on the consumer side.

---

Q29. What is data mesh, and how does it differ from centralized data architectures?

A29.
Data mesh is a decentralized approach to data architecture proposed by Zhamak Dehghani. Instead of one central data team owning all data, each domain team owns and publishes their own data as a product.

**Four principles**:
1. **Domain ownership**: The team that generates the data (orders, payments, users) owns their data pipeline and quality. No central data team bottleneck.
2. **Data as a product**: Each domain publishes data with SLAs — documentation, quality guarantees, discoverability. Treat downstream consumers like customers.
3. **Self-serve platform**: A central platform team provides tools (compute, storage, cataloging, access control) so domain teams don't each build infrastructure from scratch.
4. **Federated governance**: Centrally agreed standards (naming, formats, privacy) but decentralized execution.

**vs Centralized**: In a centralized model, one data engineering team builds all pipelines — they become a bottleneck. Data mesh distributes this responsibility. But it requires mature engineering culture — every team needs data engineering skills.

**Criticism**: It can lead to inconsistency, duplicated effort, and complexity if the self-serve platform isn't strong. It works best for large organizations (100+ data engineers, 10+ domains). For smaller teams, a centralized approach is usually more practical.

---

Q30. What is backfilling in data pipelines, and how do you handle it?

A30.
Backfilling means reprocessing historical data — either because a bug was found, a new column was added, business logic changed, or a new pipeline needs to process existing data.

**Challenges**:
- **Scale**: You might need to reprocess months or years of data. A pipeline designed for daily incremental loads may not handle the volume.
- **Idempotency**: Re-running the pipeline should produce the same results without duplicates. Design for `INSERT OVERWRITE` or upserts, not `INSERT APPEND`.
- **Dependencies**: Downstream consumers might be affected during the backfill. Notify or pause them.
- **Resource contention**: Backfills can overwhelm compute resources, affecting production workloads.

**Best practices**:
- Design pipelines with partition-based processing from day one. A date-partitioned pipeline can reprocess any date range
- Use parameterized execution: `my_pipeline --start-date 2024-01-01 --end-date 2024-03-31`
- Airflow's `backfill` command: `airflow dags backfill --start-date 2024-01-01 --end-date 2024-03-31 my_dag`
- Process in batches (weekly chunks) rather than all at once
- Use separate compute clusters/queues for backfills to avoid impacting production

---

Q31. What are Apache Iceberg, Delta Lake, and Apache Hudi? How do they compare?

A31.
All three are **open table formats** that add database-like features to data lakes:

**Delta Lake** (Databricks): Uses a transaction log (`_delta_log/`) to track changes. ACID transactions, time travel, schema evolution, Z-ORDER for data skipping. Tightly integrated with Databricks and Spark.

**Apache Iceberg** (Netflix origin): Metadata-first design with snapshot isolation. Hidden partitioning (partition without exposing it in file paths), partition evolution (change partitioning without rewriting data), schema evolution. Growing fast — supported by Snowflake, AWS, Trino, Spark, Flink.

**Apache Hudi** (Uber origin): Optimized for incremental processing and upserts. Copy-on-Write (COW) tables for read-heavy, Merge-on-Read (MOR) tables for write-heavy. Built-in CDC (Change Data Capture) support.

**Key comparisons**:
- **Upserts**: Hudi was built for this. Iceberg and Delta added it later.
- **Partition evolution**: Iceberg is strongest — no data rewrite needed.
- **Ecosystem**: Delta has Databricks ecosystem. Iceberg has broadest multi-engine support. Hudi has strong CDC.
- **Time travel**: All three support it, but implementation differs.

The market is converging — Delta and Iceberg are interoperable through UniForm (Delta) and support is broadening. For new projects, Iceberg is gaining the most momentum due to vendor neutrality.

---

### GOOD-TO-HAVE

---

Q32. What are data contracts and why are they important?

A32.
A data contract is a formal agreement between a data producer and its consumers specifying the schema, semantics, quality guarantees, and SLAs for a dataset.

**What a contract includes**:
- **Schema**: Column names, types, nullability — versioned and enforced
- **Semantics**: What each field means. "revenue" = gross or net?
- **SLAs**: Freshness (data available within 1 hour), completeness (>99.5%), availability
- **Breaking change policy**: How schema changes are communicated and versioned
- **Ownership**: Who to contact when something breaks

**Why they matter**: Without contracts, a producer team renames a column or changes its meaning, and downstream dashboards silently break. Contracts make these dependencies explicit and enforce them.

**Implementation**: Tools like Soda, Great Expectations, or custom validators enforce contracts at pipeline boundaries. Schema registries (Confluent Schema Registry for Kafka) enforce schema compatibility. dbt tests can validate contract clauses.

**Contract-driven development**: The producer publishes a contract → consumers code against it → the contract is tested in CI/CD → breaking changes require contract version bumps and consumer migration. It's API design principles applied to data.

---

Q33. What's the difference between push-based and pull-based data ingestion?

A33.
**Push-based**: The source system sends data to the data platform. Examples: Kafka producers publish events, webhooks POST data, applications write to a streaming endpoint. The data platform is passive — it receives whatever is pushed.

Pros: Lower latency (data arrives as it's generated), source controls the timing. Cons: The data platform must handle variable load, backpressure management needed, source must be instrumented.

**Pull-based**: The data platform reaches out and extracts data from the source on a schedule. Examples: Batch ETL queries a database nightly, Airbyte/Fivetran connectors poll APIs, CDC tools read database logs.

Pros: Data platform controls the load, simpler for the source (no instrumentation needed), can retry easily. Cons: Higher latency (depends on polling frequency), wasted work if nothing changed, can overload the source during pulls.

**Hybrid**: Many modern architectures use both. Operational events stream in via Kafka (push). Reference data and legacy systems are pulled on schedule. CDC is somewhere in between — you pull the change log, but the changes were pushed to the log by the database.

Choose push for real-time event data. Choose pull for batch, legacy systems, and external APIs you don't control.
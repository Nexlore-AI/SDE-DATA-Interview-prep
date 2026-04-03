# Spark & Airflow — Interview Q&A

---

## Apache Spark

### 1. What is Apache Spark and why is it faster than MapReduce?

Apache Spark is a **distributed computing engine** for large-scale data processing.

**Why faster than MapReduce:**
- **In-memory processing:** Keeps intermediate data in RAM (MapReduce writes to disk between stages)
- **DAG execution:** Optimizes the entire computation graph (MapReduce is limited to map → shuffle → reduce)
- **Lazy evaluation:** Only computes when an action is triggered → enables optimization
- **Result:** 10-100x faster than MapReduce for iterative workloads (ML, graph processing)

### 2. Explain the Spark architecture.

```
Driver Program (SparkContext)
    ↓
Cluster Manager (YARN / Mesos / K8s / Standalone)
    ↓
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Executor │  │ Executor │  │ Executor │
│  Task    │  │  Task    │  │  Task    │
│  Task    │  │  Task    │  │  Task    │
│  Cache   │  │  Cache   │  │  Cache   │
└──────────┘  └──────────┘  └──────────┘
```

- **Driver:** Main process. Creates SparkContext, builds DAG, schedules tasks.
- **Cluster Manager:** Allocates resources (YARN most common in production).
- **Executor:** JVM process on worker nodes. Runs tasks and caches data.
- **Task:** Smallest unit of work (operates on one partition).

### 3. What is an RDD? How does it differ from DataFrame and Dataset?

| Feature | RDD | DataFrame | Dataset |
|---------|-----|-----------|---------|
| Type safety | Yes (compile-time) | No (runtime) | Yes (compile-time) |
| Schema | No | Yes (structured) | Yes (structured) |
| Optimization | None (no Catalyst) | Catalyst optimizer | Catalyst optimizer |
| API | Functional (map, filter) | SQL-like (select, where) | Both |
| Language | Scala, Java, Python | All | Scala, Java only |
| Performance | Slowest | Fast (optimized) | Fast (optimized) |

**Modern Spark:** Use **DataFrames** (PySpark) or **Datasets** (Scala). RDDs are legacy.

### 4. What is lazy evaluation in Spark?

Spark doesn't execute **transformations** immediately. It builds a **DAG (Directed Acyclic Graph)** of operations and only executes when an **action** is called.

**Transformations (lazy):** `map`, `filter`, `select`, `join`, `groupBy` → returns a new DataFrame
**Actions (eager):** `count`, `collect`, `show`, `write`, `save` → triggers execution

**Benefits:**
- Optimizer can rearrange/combine operations
- Avoids unnecessary computation
- Can do predicate pushdown (filter early)

### 5. What is the difference between narrow and wide transformations?

| Narrow | Wide |
|--------|------|
| Data stays in same partition | Data moves across partitions (shuffle) |
| `map`, `filter`, `union` | `groupBy`, `join`, `repartition`, `distinct` |
| Fast (no network I/O) | Expensive (network transfer + disk I/O) |
| Pipeline-able | Creates stage boundary |

**Key insight:** Shuffles are the most expensive operations in Spark. Minimize them.

### 6. Explain Spark's Catalyst Optimizer.

Catalyst is Spark SQL's **query optimizer** that optimizes DataFrames/SQL queries:

1. **Analysis:** Resolves column names, types, relations
2. **Logical Optimization:** Pushes filters down, prunes columns, combines projections
3. **Physical Planning:** Chooses join strategies (broadcast vs sort-merge), generates code
4. **Code Generation (Tungsten):** Generates optimized JVM bytecode

**Example optimization:** If you write `df.select("*").filter(col("age") > 25)`, Catalyst will push the filter before the select and only read the `age` column (predicate pushdown + column pruning).

### 7. What are the different join strategies in Spark?

| Strategy | When Used | Performance |
|----------|-----------|-------------|
| **Broadcast Hash Join** | Small table joins large table (< 10 MB default) | Fastest — small table sent to all executors |
| **Sort-Merge Join** | Both tables are large | Default for large-large joins. Sorts + merges. |
| **Shuffle Hash Join** | Medium tables | Shuffles both, builds hash table |
| **Cartesian Join** | Cross join | Extremely expensive — avoid |

**Hint:** Force broadcast with `broadcast(df_small)`:
```python
result = df_large.join(broadcast(df_small), "key")
```

### 8. What is data skew and how do you handle it?

**Data skew:** Uneven distribution of data across partitions. One partition has 90% of data → one task takes forever.

**Solutions:**
- **Salting:** Add random prefix to skewed key, join on salted key, then aggregate
- **Broadcast join:** If one side is small enough, broadcast it
- **Repartition:** Redistribute data more evenly
- **AQE (Adaptive Query Execution):** Spark 3.0+ auto-handles skew by splitting large partitions
- **Two-phase aggregation:** Pre-aggregate within partitions, then final aggregate

### 9. What is partitioning and bucketing in Spark?

**Partitioning:** Splits data into directories by column value.
```python
df.write.partitionBy("year", "month").parquet("/data/events")
# Creates: /data/events/year=2024/month=01/
```
- Great for queries that filter on partition column
- Avoid high cardinality columns (too many small files)

**Bucketing:** Hashes data into fixed number of files by column.
```python
df.write.bucketBy(32, "user_id").sortBy("user_id").saveAsTable("user_events")
```
- Eliminates shuffle for joins on bucketed column
- Fixed number of files regardless of data volume

### 10. How do you tune Spark performance?

1. **Executor sizing:** `--executor-memory 8g --executor-cores 5` (5 cores per executor is a sweet spot)
2. **Parallelism:** `spark.sql.shuffle.partitions = 200` (default). Set to 2-3x number of cores.
3. **Broadcast threshold:** `spark.sql.autoBroadcastJoinThreshold = 100MB` (increase for medium tables)
4. **Caching:** `df.cache()` / `df.persist(StorageLevel.MEMORY_AND_DISK)` for reused DataFrames
5. **File format:** Use **Parquet** (columnar, compressed, schema-aware). Avoid CSV/JSON.
6. **Avoid shuffles:** Use broadcast joins, pre-partition data, use coalesce instead of repartition when reducing partitions
7. **AQE:** Enable `spark.sql.adaptive.enabled = true` (default in Spark 3.2+)

---

## Apache Airflow

### 11. What is Apache Airflow?

Airflow is a **workflow orchestration platform** that lets you programmatically author, schedule, and monitor data pipelines.

**Key concepts:**
- **DAG (Directed Acyclic Graph):** A pipeline definition — a set of tasks with dependencies
- **Operator:** A single task (Python, Bash, SQL, Spark, etc.)
- **Task Instance:** A specific run of a task
- **Scheduler:** Triggers DAGs based on schedule/dependencies
- **Executor:** Runs tasks (Local, Celery, Kubernetes)

### 12. What is a DAG in Airflow?

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

with DAG(
    dag_id="etl_pipeline",
    start_date=datetime(2024, 1, 1),
    schedule="@daily",
    catchup=False,
) as dag:
    
    extract = PythonOperator(task_id="extract", python_callable=extract_data)
    transform = PythonOperator(task_id="transform", python_callable=transform_data)
    load = PythonOperator(task_id="load", python_callable=load_data)

    extract >> transform >> load  # Dependencies
```

**Properties:**
- Defined in Python (full programmatic control)
- Scheduled (cron-like: `@daily`, `0 6 * * *`)
- `catchup=False` prevents backfilling all missed runs

### 13. What are the different Airflow operators?

| Operator | Purpose |
|----------|---------|
| `PythonOperator` | Run a Python function |
| `BashOperator` | Run a bash command |
| `PostgresOperator` | Execute SQL on Postgres |
| `S3ToRedshiftOperator` | Copy S3 data to Redshift |
| `SparkSubmitOperator` | Submit a Spark job |
| `KubernetesPodOperator` | Run a task in a K8s pod |
| `EmailOperator` | Send email notifications |
| `BranchPythonOperator` | Conditional branching logic |
| `DummyOperator` | No-op (used for DAG structure) |
| `Sensor` | Wait for a condition (file exists, partition ready) |

### 14. What is the difference between `@daily` schedule and `timedelta(days=1)`?

- `@daily` / `schedule="0 0 * * *"` — Runs at **midnight UTC** every day. Aligned to calendar.
- `timedelta(days=1)` — Runs **24 hours after the last run**. Relative timing.

**Important:** Airflow runs a DAG for the **previous** interval. A DAG with `start_date=Jan 1` and `@daily` first runs at **Jan 2 00:00**, processing Jan 1's data. The `execution_date` will be `Jan 1`.

### 15. What are XComs in Airflow?

XComs (Cross-Communication) allow tasks to **exchange small data** between each other.

```python
def push_data(**context):
    context['ti'].xcom_push(key='row_count', value=1000)

def pull_data(**context):
    count = context['ti'].xcom_pull(task_ids='extract', key='row_count')
    print(f"Extracted {count} rows")
```

**Limitations:**
- Stored in Airflow's metadata DB → keep data small (< few KB)
- Not for passing datasets — use S3/GCS paths instead
- Can cause DB bloat if overused

### 16. What are Sensors in Airflow?

Sensors are operators that **wait for a condition** to be met:

```python
from airflow.sensors.s3_key_sensor import S3KeySensor

wait_for_file = S3KeySensor(
    task_id="wait_for_data",
    bucket_name="my-bucket",
    bucket_key="data/{{ ds }}/events.parquet",
    poke_interval=300,  # Check every 5 minutes
    timeout=3600,       # Give up after 1 hour
    mode="reschedule",  # Free up worker slot between checks
)
```

**Modes:**
- `poke` (default): Holds the worker slot while waiting (wastes resources)
- `reschedule`: Frees the slot between checks (preferred for long waits)

### 17. How do you handle failures and retries in Airflow?

```python
default_args = {
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
    'retry_exponential_backoff': True,
    'email_on_failure': True,
    'email': ['team@company.com'],
    'on_failure_callback': slack_alert,
}
```

**Strategies:**
- **Retries:** Automatic retry with backoff
- **SLA:** Set `sla=timedelta(hours=2)` to alert if task takes too long
- **Callbacks:** `on_failure_callback`, `on_success_callback`
- **Idempotency:** Tasks should be safely re-runnable (use `INSERT OVERWRITE`, not `INSERT INTO`)

### 18. Explain Airflow executors.

| Executor | How It Works | Use Case |
|----------|-------------|----------|
| **SequentialExecutor** | One task at a time | Development/testing only |
| **LocalExecutor** | Multiple tasks on one machine (multiprocessing) | Small-medium workloads |
| **CeleryExecutor** | Distributes to Celery workers (Redis/RabbitMQ) | Large production workloads |
| **KubernetesExecutor** | Each task runs in its own K8s pod | Dynamic resource allocation, isolation |

**Production recommendation:** KubernetesExecutor (best isolation, autoscaling) or CeleryExecutor (most mature).

### 19. What is the difference between Airflow and Prefect/Dagster?

| Feature | Airflow | Prefect | Dagster |
|---------|---------|---------|---------|
| Scheduling | Cron-based | Event-driven + cron | Cron + event |
| DAG definition | Python files in dag_folder | Python decorators | `@asset`, `@op` decorators |
| Data awareness | No (task-centric) | Limited | Built-in (asset-centric) |
| Testing | Hard (global state) | Easy (functional) | Easy (type-checked) |
| UI | Good | Good | Excellent |
| Maturity | Most mature, largest community | Growing | Growing |

### 20. Best practices for Airflow DAGs?

1. **Idempotent tasks:** Re-running produces same result (use date partitions, OVERWRITE)
2. **Atomic tasks:** Each task does one thing. Don't combine extract + transform.
3. **No heavy processing in DAG file:** DAG files are parsed frequently — keep them lightweight
4. **Use Jinja templates:** `{{ ds }}` for execution date, `{{ params.table }}` for params
5. **TaskGroups:** Group related tasks for cleaner UI
6. **Dynamic DAGs:** Use loops carefully, avoid creating thousands of DAGs
7. **Monitor:** Set SLAs, failure alerts, and track DAG run durations
